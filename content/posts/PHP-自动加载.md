---
title: PHP 自动加载
tags: [自动加载, spl_autoload_register]
categories: [PHP]
slug: php-autoloading
date: 2016-09-18 22:52:00
updated: 2016-09-18 22:52:00
toc: false
---

做编程3年了，几乎每年一个转型 （java -> android（web app） -> php），这个节奏太不舒服了。想深入了解什么的时候，总是无奈的转型。现在自己也算是彻底的稳定下来了。可以安心的研究 php 了。php 也做了一年多了，一直在用框架，也就是最近几个月吧，认识到了自己的不足之处，开始逐步研究一些细节方面的东西。

这个博客使用 laravel 弄的，里面很多的细节都是在以前的 java 的时候接触过，但是以前弄 java 的时候就没有太细致的研究。现在也就顺道一起弥补回来。

今天要写的是自动加载， php 中如果想要加载一个类的话，一般采用 require 或者 include 但是呢，自动引入了命名空间以后，类名加上命名空间超级长，然后每个文件开头一排 require 外加一排 use 这个文件无疑太长，而且如果某一次忘记了引入文件，如果项目过大，那么结果就是两个字爆炸 boom shakalaka。排查起来超级麻烦。

所以，就有了自动加载，以前用自动加载都是使用 `__autoload` 但是这个方法有一个bug，就是只能使用一次。这样造成的问题就是如果我们有多种状态的引用的话，就会产生很长的代码，这是我们所不想见到的。于是就有了 `spl_autoload_register` 其实，从文档上来看他就是 autoload 的一种实现，但是区别是，这个方法可以执行多次。但是以我这一段看代码来看，其实也就是使用了一次。但是安全第一，而且为了标准还是使用 `spl_autoload_register` 。

上面说了一大堆没用，其实有用的就是一个方法 `spl_autoload_register` ，具体看文档： [http://php.net/manual/zh/function.autoload.php](http://php.net/manual/zh/function.autoload.php)。

下面进入正题，开始我们的自动加载吧。我直接上代码了：

```php
<?php

use Test32\Test;

spl_autoload_register(function ($class) {
    $path = str_replace('\\', DIRECTORY_SEPARATOR, $class);
    $path .= '.php';
    if (file_exists($path)) {
        require_once ($path);
    } else {
        throw new Exception('Class not exist!');
    }
}, true);

$test = new Test();

$test->sayHello();
```

现在来解释一下：这个方法会接受三个参数，第一个是一个回掉函数，后面两个是bool类型的参数。后面两个很少使用，所以我们就着重说一下第一个回掉函数，这个函数有一个参数是一个包含命名空间的类名，也就是我们use的那一个部分的名称，我们在这里是使用层次文件夹的方式，构造的目录和文件结构的。所以，在我们收到名称之后，先转换为路径在加上 `.php` 的文件名后缀，判断一下文件是否存在。如果存在就 `require` 进去。如果不存在就抛出一个异常，告知class不存在。这样要简单的自动加载器就完成了。

过一段会写composer的各种加载方式。今天就到这里了。后续还会继续写反射相关的东西。然后我自己的大轮子也会以这个自动加载作为一切的基础。逐步完善轮子