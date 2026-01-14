# 说说 Php 文件的加载

## 从 `get_include_path` 开始

```php
<?php
$path = get_include_path();
var_dump($path);
```
输出
```shell
root@c6c2fe3c9a93:/var/www/php_test# php test_get_include_path.php
string(20) ".:/usr/local/lib/php"
```
这个会输出我们配置的 `include` 目录，我当前的设置下，会有 php 的 lib 目录，和当前目录，让我们来测试一下。

首先测试，当前目录下的文件引入，下面这个是我们在项目目录创建的文件
```php
<?php

function testCurrent() {
    var_dump('This is current include Output');
}
```
然后我们在另一个文件引入
```php
<?php
require 'test_include_current.php';

testCurrent();
```
会得到输出
```shell
root@c6c2fe3c9a93:/var/www/php_test# php test_get_include_path.php
string(30) "This is current include Output"
```
ok，当前文件夹没问题，我们在看看 `lib` 目录，我们在 `lib` 目录下，在我的实验环境中就是 `/usr/local/lib/php` 目录，放入了一个文件，然后在代码中引入，调用方案跟上面差不多，也就不贴代码了，都是一样的，依然可以得到输出。

这时候我们可以看到，在这两个目录里面的文件，都可以直接引入，引入文件的方法都可以正常使用。我们在随便找个目录写入个文件，看看能否引入呢?

我们随便找个目录写入了文件，然后 `include` 进来 ，`include` 是有返回值的，我们打印了一下，可以看到是 `false`，说明这个文件在 `include_path` 是没有找到的。所以引入失败了。

最后我们在测试个目录，我们就在当前文件夹下测试就好了，别的目录也是一致的。
结构是这样的，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191107145954478.png)
代码是
```php
<?php
$res = include 'include_folder/test_include_current.php';
var_dump($res);
testCurrent();
```
依然可以调用，完美。
最后，关于 `include` 要看php的文档，上面说的那么多都是文档里面写的。[https://www.php.net/manual/zh/function.include.php](https://www.php.net/manual/zh/function.include.php)

## 走进 `set_include_path`
这个方法是昨天在分析 `typecho` 的时候看到的，所以就引出了这篇文章。这方法是设置 `include_path` 用的，平时我们在写代码的时候不可能都在一个文件夹下写代码，那很难管理了，所以就会分成各种文件夹，但是这样，在 `include` 的时候又要写文件夹岂不是很麻烦，所以就可以调用这个方法，设置一下引入目录，以后引入的时候就可以不用写冗长的目录路径了。
但是这种会有一个什么问题呢，不同的文件夹下依然不能存在同名文件夹，否则可能会引入错误的文件。导致出现问题。

## 在进一步 `__autoload`
随着不断的进步，项目越来越大，文件起名是个问题，各种考虑名字不要冲突，还要时刻记得引入文件，这会让人很头大，大家都想要有没有什么办法，可以自动引入文件，这时候 `面向对象`和 `__autoload` 还有 `namespace` 让大家重新拾起了希望。

关于面向对象啥的大家还是自己看文档，这边我就直接从 `__autoload` 开始了。文档在这[https://www.php.net/manual/en/function.autoload.php](https://www.php.net/manual/en/function.autoload.php)。当尝试加载一个未定义的类的时候会触发这个方法，我们的希望来了，先看实例代码
```php
<?php

function __autoload($class)
{
    var_dump($class);
}

$cls = new TestClass();
```
现在最上面写好 `__autoload` 方法，我们就输出一下 $class 的名称，当我们运行后，可以看到如下输出：
```shell
root@c6c2fe3c9a93:/var/www/php_test# php test_autoload.php
string(9) "TestClass"
```

可以看到我们这边成功输出了我们的类名，如果在这个文件里面有这个类会怎样呢，结果就是，不会触发这方法。那么利用这个特性，我们就可以搞事情了，我们就可以吧命名空间跟文件目录对应上，这样我们就可以用这个方法去引入文件了。来，看个例子，

```php
// test_autoload.php
<?php
function __autoload($class)
{
    var_dump($class);

    require_once $class . '.php';
}

$cls = new TestClass();
```

```php
// TestClass.php
<?php


class TestClass
{
    public function __construct()
    {
        var_dump('TestClass is construct');
    }
}
```
可以看到结果
```shell
root@c6c2fe3c9a93:/var/www/php_test# php test_autoload.php
string(9) "TestClass"
string(22) "TestClass is construct"
```
我们的两个输出都看到了，说明什么，文件成功引入，那么如何支持 `namespace` 呢，我直接上解决方案，大家可以自己试试看

```shell
function __autoload($class)
{
    var_dump($class);
    $cls = str_replace('\\', DIRECTORY_SEPARATOR, $class);
    var_dump($cls);
    require_once $cls . '.php';
}
```

这边只是进行了简单的调整，更多的可以看下面的东西。

## 升级 `spl_autoload_register`

这东西其实就是 `__autoload` 只不过 autoload 有的一些限制，被这个方法升级了。而且这个方法还有一个对应方法是 `spl_autoload_unregister` 加载完成后，直接卸载加载方法，进一步减少冲突的可能。

## 大步走向 `composer`

不得不说这个东西整的真好，定义了一系列加载规范，再配合起来 `psr` 标准，现在 php 的生态也逐步好起来了。
`composer` 这个东西，就是把你的配置文件，做了一份映射，然后用 `spl_autoload_register` 来加载文件的，我们直奔我们需要的东西去吧。进入 `autoload_real.php` 文件，如果你找不到这个文件，先去了解一下 `composer` 在继续哦，或者干脆不看下面的， 这个只不过就是花式用了 `spl_autoload_register` 而已。这个文件里面的 `getLoader` 方法其实就是加载的方法，不过这个方法里面最终调用的都是  `ClassLoader` 这个类里面的相关方法，把各种格式的文件帮助我们引入进来。其实就是这么简单，感觉他高级的原因就是我们只需要配置一下各种格式就好了，剩下的都帮我们搞定了。更多的东西大家可以自己看 classLoader 里面的代码哟。

## 结语
好了，这次的分析比上次满意多了，我们继续回归到 `typecho` 的分析中去。

---

> 作者:   
> URL: http://localhost:1313/posts/talk-about-the-loading-of-php-files/  

