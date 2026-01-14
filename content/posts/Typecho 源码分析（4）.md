---
title: Typecho 源码分析（4）
tags: [Typecho 源码分析]
categories: [PHP]
slug: typecho-source-code-analysis-4
date: 2019-11-14 10:50:52
---


## 上篇文章忘记说的

我们还是在入口文件徘徊，而且还是在初始化 `widget` 的第一行。我们来到了初始化 `Widget_Init` 的地方。在 `Init->execute` 方法中，我们到了初始化 `option` 的地方。

初始化的构造地方，还初始化了 `db` 数据库。

数据库连接的地方在，`index` 文件的引入 `config` 有构造连接，大家不要忘记哦。

## 正文开始

我们来看 `Option->execute()` 方法。

```php
/**
     * 执行函数
     *
     * @access public
     * @return void
     */
    public function execute()
    {
        $this->db->fetchAll($this->db->select()->from('table.options')
        ->where('user = 0'), array($this, 'push'));
        
        /** 支持皮肤变量重载 */
        if (!empty($this->row['theme:' . $this->row['theme']])) {
            $themeOptions = NULL;
        
            /** 解析变量 */
            if ($themeOptions = unserialize($this->row['theme:' . $this->row['theme']])) {
                /** 覆盖变量 */
                $this->row = array_merge($this->row, $themeOptions);
            }
        }
        
        $this->stack[] = &$this->row;

        /** 动态获取根目录 */
        $this->rootUrl = defined('__TYPECHO_ROOT_URL__') ? __TYPECHO_ROOT_URL__ : $this->request->getRequestRoot();
        if (defined('__TYPECHO_ADMIN__')) {
            /** 识别在admin目录中的情况 */
            $adminDir = '/' . trim(defined('__TYPECHO_ADMIN_DIR__') ? __TYPECHO_ADMIN_DIR__ : '/admin/', '/');
            $this->rootUrl = substr($this->rootUrl, 0, - strlen($adminDir));
        }

        /** 初始化站点信息 */
        if (defined('__TYPECHO_SITE_URL__')) {
            $this->siteUrl = __TYPECHO_SITE_URL__;
        } else if (defined('__TYPECHO_DYNAMIC_SITE_URL__') && __TYPECHO_DYNAMIC_SITE_URL__) {
            $this->siteUrl = $this->rootUrl;
        }

        $this->originalSiteUrl = $this->siteUrl;
        $this->siteUrl = Typecho_Common::url(NULL, $this->siteUrl);
        $this->plugins = unserialize($this->plugins);

        /** 动态判断皮肤目录 */
        $this->theme = is_dir($this->themeFile($this->theme)) ? $this->theme : 'default';

        /** 增加对SSL连接的支持 */
        if ($this->request->isSecure() && 0 === strpos($this->siteUrl, 'http://')) {
            $this->siteUrl = substr_replace($this->siteUrl, 'https', 0, 4);
        }

        /** 自动初始化路由表 */
        $this->routingTable = unserialize($this->routingTable);
        if (!isset($this->routingTable[0])) {
            /** 解析路由并缓存 */
            $parser = new Typecho_Router_Parser($this->routingTable);
            $parsedRoutingTable = $parser->parse();
            $this->routingTable = array_merge(array($parsedRoutingTable), $this->routingTable);
            $this->db->query($this->db->update('table.options')->rows(array('value' => serialize($this->routingTable)))
            ->where('name = ?', 'routingTable'));
        }
    }
```

我们来看第一行的 `fetchAll` 方法

```php
/**
     * 一次取出所有行
     *
     * @param mixed $query 查询对象
     * @param array $filter 行过滤器函数,将查询的每一行作为第一个参数传入指定的过滤器中
     * @return array
     */
    public function fetchAll($query, array $filter = NULL)
    {
        //执行查询
        $resource = $this->query($query, self::READ);
        $result = array();

        /** 取出过滤器 */
        if (!empty($filter)) {
            list($object, $method) = $filter;
        }

        //取出每一行
        while ($rows = $this->_adapter->fetch($resource)) {
            //判断是否有过滤器
            $result[] = $filter ? call_user_func(array(&$object, $method), $rows) : $rows;
        }

        return $result;
    }
```

这里面有两个步骤，第一个是去除所有行，如果没有 `filter` 参数，就把结果放到 `$result` 里面，如果有 `$filter` 参数, 就会调用 `$filter` 的方法，把并且把 `$fiter` 方法的返回值放入到 `$result` 里面。这里面传入了 `option->push` 方法，等于把结果都传入到了 `$option->row` 里面，所以可以看到 `execute` 方法没有接受返回值，因为目的已经达到了。

紧接着初始化了各种参数，比较值得看的就是初始化路由表。

路由表这块就是根据 `url` 执行相应的 `action` 。这块我们在需要的地方在具体进入再说。

紧接着就回到了 `Init` 里面，这里面就初始化了 `cookie` `router` 等组件，这些组件我们在到了请求那边再说。

最后我们说一下 `User` 那行，这块判断了，是否登录，决定是否开启 `session`。
初始化 `User` 的时候 获取了 `db` 和 `option`，这两个字在前边都初始化了，所以直接就可以拿到数据。
在 `execute` 这边判断是否登陆以后，如果登陆了，会把 登录的用户信息，放到 `$option` 里面，然后刷新上次活动时间。

在看看 `hasLogin` 方法，如果 `hasLogin` 有值，就利用这个值，如果没有，就通过 `cookie` 校验，通过后，更新 `hasLogin` 的值，并且把值返回。

至此，`Init` 就都跑完了。

## 下期预告

我们用了两篇跑完了，一个初始化，不要担心，随着我们的初始化的东西越来越多，以后，就会快起来了。

下期，我们来看

```php
/** 注册一个初始化插件 */
Typecho_Plugin::factory('index.php')->begin();
```
