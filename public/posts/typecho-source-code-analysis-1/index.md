# Typecho 源码分析（1）


## 先说点前置的东西

从今天开始我要开始写源码分析的文章了，以前用 csdn 博客写过一些 android 和 java 的东西，后来脑袋抽筋被我删除了。所以这次等于是全新的开始，准备输出一些东西了。做了 5 年的开发，发现自己缺乏很多东西，所以这次从源码分析开始，输出东西以及补充自己的知识。

## 为什么选用 typecho ？
这个博客系统很经典，说是 cms 也不为过，毕竟他的对手 wordpress 也可以说是 cms 了，现在估计更多的企业都是用它来做展示站了。而不仅仅是博客。

也许会有朋友说，这么老的代码分析有何用？但是我想说虽然它没有用 composer，没有各种新奇的东西，而且还有很多 `php` 和 `html` 混排的代码。但是他的代码我认为我可以分析的比较清楚，没有更多外部的引用，让我们可以全身心的投入到它本身的代码中去。不用考虑各种引用的代码，让我们在深入进去，比如如果分析 laravel，就要分析很多引入库了。另外，这个代码的注释写的太好了。当然了，后续我也会分析这种代码的。

## 我分析的流程
分析代码有很多种
1. 一个文件分析完，在分析另一个
2. 遇到一个方法或者流程的改变就进入到相应方法或者流程

我在这边选用第二种，这样更符合我的风格

## 正文开始
#### 进入 `index.php`

```php
/** 载入配置支持 */
if (!defined('__TYPECHO_ROOT_DIR__') && !@include_once 'config.inc.php') {
    file_exists('./install.php') ? header('Location: install.php') : print('Missing Config File');
    exit;
}
```

如果没有定义 `__TYPECHO_ROOT_DIR__` 并且 引入 `config.inc.php` 失败就进入判断。
这时候检测  `./install.php`  如果存在就转到 `./install.php` 文件，否则就报错 `Missing Config File`。

第一次进入这个文件的时候，我们肯定是没有安装的，所以我们就进入 `install.php`

#### 来到 `install.php`

```php
<?php if (!file_exists(dirname(__FILE__) . '/config.inc.php')): ?>
```
检测 `install.php` 同目录下是否存在 `config.inc.php` 文件 如果存在就到了第 35 行开始执行下面这段代码

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
引入 `config.inc.php` 配置文件，紧接着查询 `table.options` 表，是否包含 `'name = installed` 的行，如果没有没有检测到就报错 404，如果存在就继续执行。当然现在的我们是没有安装的，我们执行上面的代码，如果已经安装的流程，我们会在后面分析。现在让我们回到上面的部分看这段代码 

```php
<?php
/**
 * Typecho Blog Platform
 *
 * @copyright  Copyright (c) 2008 Typecho team (http://www.typecho.org)
 * @license    GNU General Public License 2.0
 * @version    $Id$
 */

/** 定义根目录 */
define('__TYPECHO_ROOT_DIR__', dirname(__FILE__));

/** 定义插件目录(相对路径) */
define('__TYPECHO_PLUGIN_DIR__', '/usr/plugins');

/** 定义模板目录(相对路径) */
define('__TYPECHO_THEME_DIR__', '/usr/themes');

/** 后台路径(相对路径) */
define('__TYPECHO_ADMIN_DIR__', '/admin/');

/** 设置包含路径 */
@set_include_path(get_include_path() . PATH_SEPARATOR .
__TYPECHO_ROOT_DIR__ . '/var' . PATH_SEPARATOR .
__TYPECHO_ROOT_DIR__ . __TYPECHO_PLUGIN_DIR__);

/** 载入API支持 */
require_once 'Typecho/Common.php';

/** 程序初始化 */
Typecho_Common::init();
```
前面几个定义路径我们就不说了，注释写的很完美，我们说说 `set_include_path` 和 `get_include_path` 这两个方法可以看 [https://www.jianshu.com/p/303feaaeded1](https://www.jianshu.com/p/303feaaeded1) 这篇文章，这块还得说下 `PATH_SEPARATOR` 这个东西，不仅是 `PATH_SEPARATOR` 还有 `DIRECTORY_SEPARATOR ` 这两个分隔符，我们写代码的时候容易写死，但是实际上用这种定义好的常量，他会根据系统来决定用什么分隔符，这就很容易避免 windows 和 linux 系统不一致的区别，导致代码报错。`set_include_path` 和 `get_include_path` 配置好了系统引入路径和 typecho 自定义的引入路径，包括了 `var` 、插件目录，这两个字目录。以后再引入文件的时候，系统就会根据设置的这些目录，引入相关文件了。这块代码，再现在我们大量使用 namespace，以及 comopser 使用的概率很低了。关于 composer 的一些东西，我会在另一篇去说明。

## 先暂停一下
今天就先到这吧，我决定明天写完 composer 用的 `spl_autoload_register` 和 `set_include_path` 的相关对比后，在继续

## PS
好久没写了，感觉状态不好，估计以后会慢慢提升的。

---

> 作者:   
> URL: http://localhost:1313/posts/typecho-source-code-analysis-1/  

