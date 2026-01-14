---
title: Typecho 源码分析（5）
tags: [Typecho 源码分析]
categories: [PHP]
slug: typecho-source-code-analysis-5
date: 2019-11-18 10:50:52
---

## 前情提要

上一篇我们已经分析完了组件的初始化相关的东西，今天我们继续。插件部分。

## 正文开始

```php
/** 注册一个初始化插件 */
Typecho_Plugin::factory('index.php')->begin();
```

进入方法内部

```php

    /**
     * 获取实例化插件对象
     *
     * @access public
     * @param string $handle 插件
     * @return Typecho_Plugin
     */
    public static function factory($handle)
    {
        return isset(self::$_instances[$handle]) ? self::$_instances[$handle] :
        (self::$_instances[$handle] = new Typecho_Plugin($handle));
    }
```

就是看插件池有没有这个插件，没有就初始化，如果有就返回已存在的。
到这里我们还是没有这个插件的，执行初始化。

```php
/**
     * 插件初始化
     *
     * @access public
     * @param string $handle 插件
     */
    public function __construct($handle)
    {
        /** 初始化变量 */
        $this->_handle = $handle;
    }
```

这里初始化的时候就是给插件赋值一下。

```php
/** 注册一个结束插件 */
Typecho_Plugin::factory('index.php')->end();
```

这两个 `begin` 和 `end` 方法，我们在插件部分再说。

插件到这边就先告一段落。

## 下期预告

这篇比较短，因为用到的方法，很少或者没有用到。我们下期，说一下 路由部分，这块估计会说比较多的东西。我们下篇再见
