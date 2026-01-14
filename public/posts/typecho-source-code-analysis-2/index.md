# Typecho 源码分析（2）



## 前情提要

上一篇我们分析到了 `install.php` 文件的 `set_include_path`。今天我们继续。

## 进入安装流程

来到了引入 `Typecho/Common.php` 这样，也许你会很奇怪，找不到这个目录呢？不要忘了我们上面设置了好几个引入路径，所以我们要一个个的查找，最后我们会在 `var` 目录下，找到 `Typecho/Common.php`。你看用这种方法引入，找文件都不好弄。所以命名空间什么的才会愈发的重要，`set_include_path` 这种引入的方案也会逐渐减少使用。

紧接着执行 `Typecho_Common::init();` 这行代码的方法就在刚才我们引入的文件中，我们进入这个方法。

```php
/**
     * 自动载入类
     *
     * @param $className
     */
    public static function __autoLoad($className)
    {
        @include_once str_replace(array('\\', '_'), '/', $className) . '.php';
    }

    /**
     * 程序初始化方法
     *
     * @access public
     * @return void
     */
    public static function init()
    {
        /** 设置自动载入函数 */
        spl_autoload_register(array('Typecho_Common', '__autoLoad'));

        /** 兼容php6 */
        if (function_exists('get_magic_quotes_gpc') && get_magic_quotes_gpc()) {
            $_GET = self::stripslashesDeep($_GET);
            $_POST = self::stripslashesDeep($_POST);
            $_COOKIE = self::stripslashesDeep($_COOKIE);

            reset($_GET);
            reset($_POST);
            reset($_COOKIE);
        }

        /** 设置异常截获函数 */
        set_exception_handler(array('Typecho_Common', 'exceptionHandle'));
    }
```

先看 `if (function_exists('get_magic_quotes_gpc') && get_magic_quotes_gpc()) {` 这段，这块在php5.4 以后永远都返回 false 了，所以这段在5.4 以后不会执行。如果是老版本呢，这块的处理就是把一些转义带有反斜线的字符给恢复过来，变成原始的内容。
然后剩下的部分就是注册自动加载，和异常处理函数了。自动加载可以看我上一篇文章，然后自己理解一下。

异常处理部分，就是根据是否是 `debug` 两种输出模式。

```php
/**
     * 异常截获函数
     *
     * @access public
     * @param $exception 截获的异常
     * @return void
     */
    public static function exceptionHandle($exception)
    {
        if (defined('__TYPECHO_DEBUG__')) {
            echo '<pre><code>';
            echo '<h1>' . htmlspecialchars($exception->getMessage()) . '</h1>';
            echo htmlspecialchars($exception->__toString());
            echo '</code></pre>';
        } else {
            @ob_end_clean();
            if (404 == $exception->getCode() && !empty(self::
                $exceptionHandle)) {
                $handleClass = self::$exceptionHandle;
                new $handleClass($exception);
            } else {
                self::error($exception);
            }
        }

        exit;
    }
```

`self::$exceptionHandle` 这个会在初始化的时候我们再说。
如果没有 exceptionHandle 的时候会调用 `error` 方法，进行错误输出，并且记录 `error_log` 

```php
/**
     * 输出错误页面
     *
     * @access public
     * @param mixed $exception 错误信息
     * @return void
     */
    public static function error($exception)
    {
        $isException = is_object($exception);
        $message = '';

        if ($isException) {
            $code = $exception->getCode();
            $message = $exception->getMessage();
        } else {
            $code = $exception;
        }

        $charset = self::$charset;

        if ($isException && $exception instanceof Typecho_Db_Exception) {
            $code = 500;
            @error_log($message);

            //覆盖原始错误信息
            $message = 'Database Server Error';

            if ($exception instanceof Typecho_Db_Adapter_Exception) {
                $code = 503;
                $message = 'Error establishing a database connection';
            } else if ($exception instanceof Typecho_Db_Query_Exception) {
                $message = 'Database Query Error';
            }
        } else {
            switch ($code) {
                case 500:
                    $message = 'Server Error';
                    break;

                case 404:
                    $message = 'Page Not Found';
                    break;

                default:
                    $code = 'Error';
                    break;
            }
        }


        /** 设置http code */
        if (is_numeric($code) && $code > 200) {
            Typecho_Response::setStatus($code);
        }

        $message = nl2br($message);

        if (defined('__TYPECHO_EXCEPTION_FILE__')) {
            require_once __TYPECHO_EXCEPTION_FILE__;
        } else {
            echo
<<<EOF
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="{$charset}">
        <title>{$code}</title>
        <style>
            html {
                padding: 50px 10px;
                font-size: 16px;
                line-height: 1.4;
                color: #666;
                background: #F6F6F3;
                -webkit-text-size-adjust: 100%;
                -ms-text-size-adjust: 100%;
            }

            html,
            input { font-family: "Helvetica Neue", Helvetica, Arial, sans-serif; }
            body {
                max-width: 500px;
                _width: 500px;
                padding: 30px 20px;
                margin: 0 auto;
                background: #FFF;
            }
            ul {
                padding: 0 0 0 40px;
            }
            .container {
                max-width: 380px;
                _width: 380px;
                margin: 0 auto;
            }
        </style>
    </head>
    <body>
        <div class="container">
            {$message}
        </div>
    </body>
</html>
EOF;
        }

        exit;
    }
```

到这里 `init` 方法已经执行完了，我们回到 `install.php` 文件继续往下看。

```php
// 挡掉可能的跨站请求
if (!empty($_GET) || !empty($_POST)) {
if (empty($_SERVER['HTTP_REFERER'])) {
    exit;
}

$parts = parse_url($_SERVER['HTTP_REFERER']);
if (!empty($parts['port'])) {
    $parts['host'] = "{$parts['host']}:{$parts['port']}";
}

if (empty($parts['host']) || $_SERVER['HTTP_HOST'] != $parts['host']) {
    exit;
}
}
```

这块就是挡掉跨域攻击，比如 iframe 的嵌套页面，为了安全，我们会判断 `referer` 如果跟请求的 `host` 不一致就阻挡掉

```php
$options = new stdClass();
$options->generator = 'Typecho ' . Typecho_Common::VERSION;
list($soft, $currentVersion) = explode(' ', $options->generator);

$options->software = $soft;
$options->version = $currentVersion;

list($prefixVersion, $suffixVersion) = explode('/', $currentVersion);

/** 获取语言 */
$lang = _r('lang', Typecho_Cookie::get('__typecho_lang'));
$langs = Widget_Options_General::getLangs();

if (empty($lang) || (!empty($langs) && !isset($langs[$lang]))) {
    $lang = 'zh_CN';
}

if ('zh_CN' != $lang) {
    $dir = defined('__TYPECHO_LANG_DIR__') ? __TYPECHO_LANG_DIR__ : __TYPECHO_ROOT_DIR__ . '/usr/langs';
    Typecho_I18n::setLang($dir . '/' . $lang . '.mo');
}

Typecho_Cookie::set('__typecho_lang', $lang);
```

设置版本，设置语言，顺便把语言写入到 `cookie` 中。

剩下的部分就都是安装流程了，我们慢慢拆分来看。
先说 安装文件 最后的部分

```php
<?php
include 'admin/copyright.php';
include 'admin/footer.php';
?>
```

因为这两个文件都因为开头

```php
<?php if(!defined('__TYPECHO_ADMIN__')) exit; ?>
```

由于没有定义那个常量而退出了，所以这两个部分都在我们用到的时候再说。

```php
<li<?php if (!isset($_GET['finish']) && !isset($_GET['config'])) : ?> class="current"<?php endif; ?>><span>1</span><?php _e('欢迎使用'); ?></li>
<li<?php if (isset($_GET['config'])) : ?> class="current"<?php endif; ?>><span>2</span><?php _e('初始化配置'); ?></li>
<li<?php if (isset($_GET['start'])) : ?> class="current"<?php endif; ?>><span>3</span><?php _e('开始安装'); ?></li>
<li<?php if (isset($_GET['finish'])) : ?> class="current"<?php endif; ?>><span>4</span><?php _e('安装成功'); ?></li>
```

这块就是根据 `url` 的参数状态决定显示的问题 `_e` 和 `_t` 都是 一个是翻译并 `echo` 另一个是翻译。
注意哦，`start` 这个状态，在正常状态时看不到的哦，只有失败才会看得到。

### 安装第一步

显示一些说明文件，如果语言配置有多个，那么就出现语言选择列表框，不过默认就只有一个简体中文。然后点击下一步以后会跳转到当前 `url` ，增加`config` 参数。

### 进入配置

点击下一步以后我们就进入到了配置的步骤，当我们输入完相关数据参数，以及管理员信息以后点击下一步，会 `post` 方法跳转到当前 `config` 网址。这里有个主要注意的地方是，当我们改变数据库的适配器以后，会跳转到切换相应的数据库适配器配置页面。而且会在页面加载的时候判定支持什么数据。这两段在下面的代码中

```php
<?php
    $adapters = array('Mysql', 'Mysqli', 'Pdo_Mysql', 'SQLite', 'Pdo_SQLite', 'Pgsql', 'Pdo_Pgsql');
    foreach ($adapters as $firstAdapter) {
        if (_p($firstAdapter)) {
            break;
        }
    }
    $adapter = _r('dbAdapter', $firstAdapter);
    $parts = explode('_', $adapter);

    $type = $adapter == 'Mysqli' ? 'Mysql' : array_pop($parts);
?>
<?php require_once './install/' . $type . '.php'; ?>
```

```js
<script>
var _select = document.config.dbAdapter;
_select.onchange = function() {
    setTimeout("window.location.href = 'install.php?config&dbAdapter=" + this.value + "'; ",0);
}
</script>
```

不同的适配器会加载不同的数据库配置页面，我们这边用的是 `mysql` ，我们进入 `install/mysql.php` 页面，里面有很多环境的判断 sae、gae、bae 什么的判定。这些我们都跳过，直接看下面。

```php
<?php  else: ?>
    <li>
        <label class="typecho-label" for="dbHost"><?php _e('数据库地址'); ?></label>
        <input type="text" class="text" name="dbHost" id="dbHost" value="<?php _v('dbHost', 'localhost'); ?>"/>
        <p class="description"><?php _e('您可能会使用 "%s"', 'localhost'); ?></p>
    </li>
    <li>
        <label class="typecho-label" for="dbPort"><?php _e('数据库端口'); ?></label>
        <input type="text" class="text" name="dbPort" id="dbPort" value="<?php _v('dbPort', '3306'); ?>"/>
        <p class="description"><?php _e('如果您不知道此选项的意义, 请保留默认设置'); ?></p>
    </li>
    <li>
        <label class="typecho-label" for="dbUser"><?php _e('数据库用户名'); ?></label>
        <input type="text" class="text" name="dbUser" id="dbUser" value="<?php _v('dbUser', 'root'); ?>" />
        <p class="description"><?php _e('您可能会使用 "%s"', 'root'); ?></p>
    </li>
    <li>
        <label class="typecho-label" for="dbPassword"><?php _e('数据库密码'); ?></label>
        <input type="password" class="text" name="dbPassword" id="dbPassword" value="<?php _v('dbPassword'); ?>" />
    </li>
    <li>
        <label class="typecho-label" for="dbDatabase"><?php _e('数据库名'); ?></label>
        <input type="text" class="text" name="dbDatabase" id="dbDatabase" value="<?php _v('dbDatabase', 'typecho'); ?>" />
        <p class="description"><?php _e('请您指定数据库名称'); ?></p>
    </li>

<?php  endif; ?>
<input type="hidden" name="dbCharset" value="<?php _e('utf8'); ?>" />

<li>
    <label class="typecho-label" for="dbCharset"><?php _e('数据库编码'); ?></label>
    <select name="dbCharset" id="dbCharset">
        <option value="utf8"<?php if (_r('dbCharset') == 'utf8'): ?> selected<?php endif; ?>>utf8</option>
        <option value="utf8mb4"<?php if (_r('dbCharset') == 'utf8mb4'): ?> selected<?php endif; ?>>utf8mb4</option>
    </select>
    <p class="description"><?php _e('选择 utf8mb4 编码至少需要 MySQL 5.5.3 版本'); ?></p>
</li>

<li>
    <label class="typecho-label" for="dbEngine"><?php _e('数据库引擎'); ?></label>
    <select name="dbEngine" id="dbEngine">
        <option value="MyISAM"<?php if (_r('dbEngine') == 'MyISAM'): ?> selected<?php endif; ?>>MyISAM</option>
        <option value="InnoDB"<?php if (_r('dbEngine') == 'InnoDB'): ?> selected<?php endif; ?>>InnoDB</option>
    </select>
</li>
```

这块就是我们显示配置的位置哟。
我们填写完配置信息以后，`post` 提交当前页面，进入各种判定的部分。

```php
if (_r('created') && !file_exists('./config.inc.php')) {
    echo '<p class="message error">' . _t('没有检测到您手动创建的配置文件, 请检查后再次创建') . '</p>';
    $success = false;
} else {
    if (NULL == _r('userUrl')) {
        $success = false;
        echo '<p class="message error">' . _t('请填写您的网站地址') . '</p>';
    } else if (NULL == _r('userName')) {
        $success = false;
        echo '<p class="message error">' . _t('请填写您的用户名') . '</p>';
    } else if (NULL == _r('userMail')) {
        $success = false;
        echo '<p class="message error">' . _t('请填写您的邮箱地址') . '</p>';
    } else if (32 < strlen(_r('userName'))) {
        $success = false;
        echo '<p class="message error">' . _t('用户名长度超过限制, 请不要超过 32 个字符') . '</p>';
    } else if (200 < strlen(_r('userMail'))) {
        $success = false;
        echo '<p class="message error">' . _t('邮箱长度超过限制, 请不要超过 200 个字符') . '</p>';
    }
}
```

这部分会进行一些判定相关的东西。不符合规范的会进行报错 `if (_r('created') && !file_exists('./config.inc.php')) {` 注意这块，我们后面再说。

```php
$_dbConfig = _rFrom('dbHost', 'dbUser', 'dbPassword', 'dbCharset', 'dbPort', 'dbDatabase', 'dbFile', 'dbDsn', 'dbEngine');

$_dbConfig = array_filter($_dbConfig);
$dbConfig = array();
foreach ($_dbConfig as $key => $val) {
    $dbConfig[strtolower(substr($key, 2))] = $val;
}

// 在特殊服务器上的特殊安装过程处理
if (_r('config')) {
    $replace = array_keys($dbConfig);
    foreach ($replace as &$key) {
        $key = '{' . $key . '}';
    }

    if (!empty($_dbConfig['dbDsn'])) {
        $dbConfig['dsn'] = str_replace($replace, array_values($dbConfig), $dbConfig['dsn']);
    }
    $config = str_replace($replace, array_values($dbConfig), _r('config'));
}

if (!isset($config) && $success && !_r('created')) {
    $installDb = new Typecho_Db($adapter, _r('dbPrefix'));
    $installDb->addServer($dbConfig, Typecho_Db::READ | Typecho_Db::WRITE);


    /** 检测数据库配置 */
    try {
        $installDb->query('SELECT 1=1');
    } catch (Typecho_Db_Adapter_Exception $e) {
        $success = false;
        echo '<p class="message error">'
        . _t('对不起, 无法连接数据库, 请先检查数据库配置再继续进行安装') . '</p>';
    } catch (Typecho_Db_Exception $e) {
        $success = false;
        echo '<p class="message error">'
        . _t('安装程序捕捉到以下错误: " %s ". 程序被终止, 请检查您的配置信息.',$e->getMessage()) . '</p>';
    }
}
```

这块是获取数据库连接配置，然后对数据库进行连接，数据库相关的代码是 `typecho` 自己的封装的，大家可以自己看一下，很厉害。如果连接失败，会进行报错。
如果成功了，就重置数据库相关信息，这块应该是应对重复安装的。然后 `cookie` 写入数据库配置信息

```php
// 重置原有数据库状态
if (isset($installDb)) {
    try {
        $installDb->query($installDb->update('table.options')
            ->rows(array('value' => 0))->where('name = ?', 'installed'));
    } catch (Exception $e) {
        // do nothing
    }
}

Typecho_Cookie::set('__typecho_config', base64_encode(serialize(array_merge(array(
    'prefix'    =>  _r('dbPrefix'),
    'userName'  =>  _r('userName'),
    'userPassword'  =>  _r('userPassword'),
    'userMail'  =>  _r('userMail'),
    'adapter'   =>  $adapter,
    'siteUrl'   =>  _r('userUrl')
), $dbConfig))));
```

注意下面这段

```php
if (_r('created')) {
    header('Location: ./install.php?start');
    exit;
}
```

这段什么意思？我们后面再说

```php
/** 初始化配置文件 */
$lines = array_slice(file(__FILE__), 1, 31);
$lines[] = "
/** 定义数据库参数 */
\$db = new Typecho_Db('{$adapter}', '" . _r('dbPrefix') . "');
\$db->addServer(" . (empty($config) ? var_export($dbConfig, true) : $config) . ", Typecho_Db::READ | Typecho_Db::WRITE);
Typecho_Db::set(\$db);
";
$contents = implode('', $lines);
if (!Typecho_Common::isAppEngine()) {
    @file_put_contents('./config.inc.php', $contents);
}
```

这段就写入配置文件了。

```php
if (!file_exists('./config.inc.php')) {
                                    ?>
<div class="message notice"><p><?php _e('安装程序无法自动创建 <strong>config.inc.php</strong> 文件'); ?><br />
<?php _e('您可以在网站根目录下手动创建 <strong>config.inc.php</strong> 文件, 并复制如下代码至其中'); ?></p>
<p><textarea rows="5" onmouseover="this.select();" class="w-100 mono" readonly><?php echo htmlspecialchars($contents); ?></textarea></p>
<p><button name="created" value="1" type="submit" class="btn primary">创建完毕, 继续安装 &raquo;</button></p></div>
<?php
} else {
    header('Location: ./install.php?start');
    exit;
}
```

如果写入文件失败了，就会跳转到当前页面了并且携带 `created` 参数，就应对上一步的判定了。如果写入成功了，就跳转到 `start`。

```php
// 安装不成功删除配置文件
if(!$success && file_exists(__TYPECHO_ROOT_DIR__ . '/config.inc.php')) {
    @unlink(__TYPECHO_ROOT_DIR__ . '/config.inc.php');
}

安装失败了，就删除文件。

### 来到 `start`
这一步就到了比较关键的一步了。
```php
require_once dirname(__FILE__) . '/config.inc.php';

//判断是否已经安装
$db = Typecho_Db::get();
try {
    $installed = $db->fetchRow($db->select()->from('table.options')->where('name = ?', 'installed'));
    if (empty($installed) || $installed['value'] == 1) {
        Typecho_Response::setStatus(404);
        exit;
    }
} catch (Exception $e) {
    // do nothing
}
```

这块就应对上前面说得了，如果包含了配置文件，就会链接数据库，并且查询是否已安装了，如果已安装了就报错 `404`。

```php
<?php if (!isset($db)) : ?>
                <h1 class="typecho-install-title"><?php _e('安装失败!'); ?></h1>
                <div class="typecho-install-body">
                    <form method="post" action="?config" name="config">
                    <p class="message error"><?php _e('您没有上传 config.inc.php 文件, 请您重新安装!'); ?> <button class="btn primary" type="submit"><?php _e('重新安装 &raquo;'); ?></button></p>
                    </form>
                </div>
                <?php else : ?>
```

如果没有连接 `db` 就报错。
如果一切顺利就执行创建数据库，初始化配置文件。如果失败了就报错安装失败，如果成功了，就跳转到，安装成功。
上面说的看不到 安装过程 页面就是因为太快了，所以就一闪而过了，上面表述的不清楚，这里在说明一下。

```php
 if(('Mysql' == $type && (1050 == $code || '42S01' == $code)) ||
('SQLite' == $type && ('HY000' == $code || 1 == $code)) ||
('Pgsql' == $type && '42P07' == $code)) {
    if(_r('delete')) {
        //删除原有数据
        $dbPrefix = $config['prefix'];
        $tableArray = array($dbPrefix . 'comments', $dbPrefix . 'contents', $dbPrefix . 'fields', $dbPrefix . 'metas', $dbPrefix . 'options', $dbPrefix . 'relationships', $dbPrefix . 'users',);
        foreach($tableArray as $table) {
            if($type == 'Mysql') {
                $installDb->query("DROP TABLE IF EXISTS `{$table}`");
            } elseif($type == 'Pgsql') {
                $installDb->query("DROP TABLE {$table}");
            } elseif($type == 'SQLite') {
                $installDb->query("DROP TABLE {$table}");
            }
        }
        echo '<p class="message success">' . _t('已经删除完原有数据') . '<br /><br /><button class="btn primary" type="submit" class="primary">'
            . _t('继续安装 &raquo;') . '</button></p>';
    } elseif (_r('goahead')) {
        //使用原有数据
        //但是要更新用户网站
        $installDb->query($installDb->update('table.options')->rows(array('value' => $config['siteUrl']))->where('name = ?', 'siteUrl'));
        unset($_SESSION['typecho']);
        header('Location: ./install.php?finish&use_old');
        exit;
    } else {
            echo '<p class="message error">' . _t('安装程序检查到原有数据表已经存在.')
            . '<br /><br />' . '<button type="submit" name="delete" value="1" class="btn btn-warn">' . _t('删除原有数据') . '</button> '
            . _t('或者') . ' <button type="submit" name="goahead" value="1" class="btn primary">' . _t('使用原有数据') . '</button></p>';
    }
```

这块就是在异常的时候如果数据库存在，的判断过程。删库或者使用原有数据库，然后等我们决策后，在决定安装流程。

## 结语

至此，安装全部搞定了，我们分析完了一个安装模块，接下来说些什么呢，先说前台，部分，然后在说明上面没说明的 `db` 部分，最后说后台。我们下次再见


---

> 作者:   
> URL: http://localhost:1313/posts/typecho-source-code-analysis-2/  

