# Typecho 源码分析（9）-- 部分Security&User 组件分析


## 前情提要

前面我们分析了插件列表，但是 `html` 部分我们没有分析，今天我们就来分析一下 `url` 生成部分。

## 正文开始

后台的 `common.php` 会加载 `Security` 组件。
先看 `Security` 的 `execute` 方法，

```php
/**
    * 初始化函数
    *
    */
public function execute()
{
    $this->_options = $this->widget('Widget_Options');
    $user = $this->widget('Widget_User');

    $this->_token = $this->_options->secret;
    if ($user->hasLogin()) {
        $this->_token .= '&' . $user->authCode . '&' . $user->uid;
    }
}
```

首先会加载 `Options` ，然后会加载 `User` 组件。
我们再去看 `User` 组件，这个组件的构造方法会加载 `Option`，再去看 `User` 的 `execute` 方法，

```php
public function execute()
{
    if ($this->hasLogin()) {
        $rows = $this->db->fetchAll($this->db->select()
        ->from('table.options')->where('user = ?', $this->_user['uid']));

        $this->push($this->_user);

        foreach ($rows as $row) {
            $this->options->__set($row['name'], $row['value']);
        }

        //更新最后活动时间
        $this->db->query($this->db
        ->update('table.users')
        ->rows(array('activated' => $this->options->time))
        ->where('uid = ?', $this->_user['uid']));
    }
}
```

这里会判断是否登陆，我们先去看 `hasLogin` 方法，

```php
public function hasLogin()
{
    if (NULL !== $this->_hasLogin) {
        return $this->_hasLogin;
    } else {
        $cookieUid = Typecho_Cookie::get('__typecho_uid');
        if (NULL !== $cookieUid) {
            /** 验证登陆 */
            $user = $this->db->fetchRow($this->db->select()->from('table.users')
            ->where('uid = ?', intval($cookieUid))
            ->limit(1));

            $cookieAuthCode = Typecho_Cookie::get('__typecho_authCode');
            if ($user && Typecho_Common::hashValidate($user['authCode'], $cookieAuthCode)) {
                $this->_user = $user;
                return ($this->_hasLogin = true);
            }

            $this->logout();
        }

        return ($this->_hasLogin = false);
    }
}
```

如果 `_hasLogin` 不是 `NULL` 就返回当前状态，如果不是，就去 `cookie` 里面获取 `uid`，如果 `uid` 不是 `NULL`，就去数据区获取这个用户，然后在去 `cookie` 获取 `__typecho_authCode` 值，如果用户存在，并且 `__typecho_authCode` 通过了验证，就把查到的 `user` 放到 `User` 的 `_user` 值中，并且把 `_hasLogin` 设置为真返回，如果没有通过验证，就退出用户，并且把 `_hasLogin` 设置为假返回。

现在我们去看看，`Typecho_Common::hashValidate` 方法，

```php
public static function hashValidate($from, $to)
{
    if ('$T$' == substr($to, 0, 3)) {
        $salt = substr($to, 3, 9);
        return self::hash($from, $salt) === $to;
    } else {
        return md5($from) === $to;
    }
}
```

这里就是个验证算法，大家看下就好，我们就不多说了，包括里面的 `hash` 方法。`logout` 方法，我们在登录部分再说。

回到 `User` 的 `execute` 方法，如果登录成功了，就去 `Options` 里面获取喝这个登录用户的单独配置，然后把登录的用户 `_user` 放到组件的 `stack` 中，接着遍历用户的配置，放到 `options` 变量里面，最后刷新这个用户的活跃时间，我们看下 `push` 这个方法

```php
public function push(array $value)
{
    //将行数据按顺序置位
    $this->row = $value;
    $this->length ++;

    $this->stack[] = $value;
    return $value;
}
```

把放入的 `value` 放到组件的 `row` 中，把组件的 `length` 加一，再把值放入到 `stack` 中，返回 `value`，`row` 和 `stack` 中，我们后面再说。

再次回到 `Security` 的 `execute` 方法，
从选项中获取 `secret` 放入到组件的 `token` 中，如果用户登录了，在拼接登录用户的 `authCode` 和 `uid` 到组件的 `token` 中。

我们回到 `plugins.php` 文件，看未启用插件列表代码部分的这行代码

```php
<a href="<?php $security->index('/action/plugins-edit?activate=' . $deactivatedPlugins->name); ?>"><?php _e('启用'); ?></a>
```

主要看 `$security->index` 这个方法，我们进入方法内部去看。

```php
public function index($path)
    {
        echo $this->getIndex($path);
    }
```

这个方法调用了内部的 `getIndex` 方法并输出出来，看 `getIndex` 方法。

```php
public function getIndex($path)
    {
        return Typecho_Common::url($this->getTokenUrl($path), $this->_options->index);
    }
```

先看 `$this->_options->index` 的值是多少，这个值在 `Option` 的 `execute` 方法并没有进行赋值，我们可以看到，`Option` 里面有个 `___index` 方法，但是我们获取 `index` 的值的时候为什么会调用这个方法呢，我们可以看 `Option` 的基类里面的魔术方法

```php
protected function ___index()
{
    return ($this->rewrite || (defined('__TYPECHO_REWRITE__') && __TYPECHO_REWRITE__)) 
        ? $this->rootUrl : Typecho_Common::url('index.php', $this->rootUrl);
}
```

```php
public function __get($name)
{
    if (array_key_exists($name, $this->row)) {
        return $this->row[$name];
    } else {
        $method = '___' . $name;

        if (method_exists($this, $method)) {
            return $this->$method();
        } else {
            $return = $this->pluginHandle()->trigger($plugged)->{$method}($this);
            if ($plugged) {
                return $return;
            }
        }
    }

    return NULL;
}
```

可以看到这里面，会查询 `$method = '___' . $name` 这样的方法名是否存在，如果存在就会调用这个方法。
我们再看下 `___index` 方法运行了什么，在开启了 `rewrite` 以后，就会返回 `rootUrl`，如果没有开启就会生成 url，我们看下 url 的生成方法。

```php
Typecho_Common::url('index.php', $this->rootUrl)
```

这个方法传入了两个参数，第一个是 `index.php` 和 `rootUrl`，那么 `rootUrl` 是怎么来的呢。这个值在 `Option` 的 `execute` 方法生成，我们看下

```php
$this->rootUrl = defined('__TYPECHO_ROOT_URL__') ? __TYPECHO_ROOT_URL__ : $this->request->getRequestRoot();
if (defined('__TYPECHO_ADMIN__')) {
            /** 识别在admin目录中的情况 */
            $adminDir = '/' . trim(defined('__TYPECHO_ADMIN_DIR__') ? __TYPECHO_ADMIN_DIR__ : '/admin/', '/');
            $this->rootUrl = substr($this->rootUrl, 0, - strlen($adminDir));
        }
```

如果设置了 `__TYPECHO_ROOT_URL__` 就返回 `__TYPECHO_ROOT_URL__` 没有设置就调用 `Request` 的 `getRequestRoot` 方法。然后在判断，是不是 `/admin` 结尾，如果是以这个结尾就说明是

```php
 public function getRequestRoot()
{
    if (NULL === $this->_requestRoot) {
        $root = rtrim(self::getUrlPrefix() . $this->getBaseUrl(), '/') . '/';
        
        $pos = strrpos($root, '.php/');
        if ($pos) {
            $root = dirname(substr($root, 0, $pos));
        }

        $this->_requestRoot = rtrim($root, '/');
    }

    return $this->_requestRoot;
} 
```

先获取 `getUrlPrefix` ，

```php
 public static function getUrlPrefix()
{
    if (empty(self::$_urlPrefix)) {
        if (defined('__TYPECHO_URL_PREFIX__')) {
            self::$_urlPrefix == __TYPECHO_URL_PREFIX__;
        } else if (!defined('__TYPECHO_CLI__')) {
            self::$_urlPrefix = (self::isSecure() ? 'https' : 'http') . '://' 
                . (isset($_SERVER['HTTP_HOST']) ? $_SERVER['HTTP_HOST'] : $_SERVER['SERVER_NAME']);
        }
    }

    return self::$_urlPrefix;
}
```

这个方法就是拼接了 `server` 的参数，获取了完整的请求url 域名部分。

紧接着调用了 `getBaseUrl`，这个就是获取了请求的文件，这个方法的解析请看前面的文章。

生成了 `root` 之后，会判断 `'.php/'` 的位置，如果查到了这个字符串，就会获取一下 `root` 的 `dirname`，这个就可以理解为，把最后的文件过滤掉，保留前面的部分。

好了，我们看下 `url` 方法

```php
public static function url($path, $prefix)
    {
        $path = (0 === strpos($path, './')) ? substr($path, 2) : $path;
        return rtrim($prefix, '/') . '/' . str_replace('//', '/', ltrim($path, '/'));
    }
```

会先判断path是否已 `./` 开头，如果是，就截取一下，然后在把 `prefix`  放到前面，拼接 `path` 。

再看 `getTokenUrl` 方法

```php
public function getTokenUrl($path)
    {
        $parts = parse_url($path);
        $params = array();

        if (!empty($parts['query'])) {
            parse_str($parts['query'], $params);
        }

        $params['_'] = $this->getToken($this->request->getRequestUrl());
        $parts['query'] = http_build_query($params);

        return Typecho_Common::buildUrl($parts);
    }
```

解析 `path`，如果解析的 `url` 包含 `query` ,就再次生成`params`，在调用 `getToken` 生成加密的串，这个生成就是 `md5` 一下。最后在生成url。

可以看到生成的 `url` 就是 `http://typecho.test/index.php/action/plugins-edit?activate=HelloWorld&_=4d799a66e315807b50ca3773ede882f3`。

## 下期预告

这篇文章说的比较乱，大家需要自己好好的多读一读，这样才能更好的理解后面的东西。下篇我们还在 `plugins.php` 里面徘徊。我们看更多的东西，下期再见。

---

> 作者: [M1racleHao](https://github.com/crazyhl)  
> URL: http://localhost:1313/posts/typecho-source-code-analysis-9/  

