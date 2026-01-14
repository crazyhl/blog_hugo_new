---
title: Php依赖注入容器Pimple的笔记
tags: [IOC, DI, Container, 容器]
categories: [PHP]
slug: php-dependency-injection-note-from-the-container-pimple
date: 2017-03-27 22:44:00
updated: 2017-03-27 22:44:00
toc: false
---

![图片alt](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/2019/03/06/0664506a46a7649adc31545165d08f41.jpg)

话说许久没有写技术类的笔记了，也该写点东西了，距离上次说 container 已经过去4个月了，期间也一直在学习，但是不知道写点什么好，说实话还是很怀念 14 年下半年，那半年的进步真的很大，那时候自己愿意看东西，也愿意写东西，也许写东西能让自己进步更快吧，所以17年了也应该继续进步了，否则问题大大的啊，下班的时候跟小伙伴聊天，做技术的就应该一直学习，否则就会被拉下很远很远。好了，说了这么多废话，说点正文吧，其实，想写 laravel 的 container 的笔记的，可是，自己看了许久，也没有很好的理解，理解到的部分也是很片面的东西，不过 laravel 的 container 真的很强大，要比今天写的这个强大的多，不过强大的带来的问题，就是可能会慢，毕竟用到了反射，今天这个 Pimple 则相对简单的多了，没有那么多复杂的东西，用起来也很顺手而且比较容易理解，所以今天就用这个做笔记了。

原本想分段做这个的笔记的，后来觉得应该吧知识点拆分开说明，然后再用整体代码在里面做注释的方式来解释这段更好一些，于是乎，准备开整，btw，过几天准备更新一下编辑器，现在这个自己搞得编辑器还是有些简陋的。好吧，进入正文了。


```php
class Container implements \ArrayAccess
```

这个类实现了 ArrayAccess 接口，ArrayAccess 是个什么东西了，大家首先看一下 PHP 官网的手册 [http://php.net/manual/zh/class.arrayaccess.php](http://php.net/manual/zh/class.arrayaccess.php) 好吧，我就简单说一句，就是让对象能够像数组一样操作了。具体需要实现的几个接口，大家看一下手册吧，我觉得我的说明肯定没有手册写得好，我更多理解的东西，会在代码注释中写一下。

```php
public function __construct(array $values = array())
    {
        $this->factories = new \SplObjectStorage();
        $this->protected = new \SplObjectStorage();

        foreach ($values as $key => $value) {
            $this->offsetSet($key, $value);
        }
    }
```

在构造方法中又出现了 SplObjectStorage 类，那么这个 SplObjectStorage 是个什么东西呢，我们再去看一下手册 [http://php.net/manual/zh/class.splobjectstorage.php](http://php.net/manual/zh/class.splobjectstorage.php)  这个在手册中可惜翻译的不完全（根本没有翻译），所以我就简单的说说，这个类其实是实现了 Countable , Iterator , Serializable , ArrayAccess 这4个接口，这4个接口大家也可以看一下手册，看完了就知道这个类是干什么的了，我再来拆分说下 Countable 就是让一个类可以用一个计数器， Iterator 就是可以用 foreach 这些去循环，Serializable 就是可以序列化，ArrayAccess 上面说过了就不多说了。其实刚开始很难理解这个类，不过多看看手册，别人的示例代码，自己在多写一些可能就了解了。这个如何理解我也说不太好。大家见谅。

好吧，我觉得是额外知识点的就上面那些，等多的笔记我将在下面的代码注释中说明白，大家可以看看，顺便帮忙指正。

```php
<?php

/*
 * This file is part of Pimple.
 *
 * Copyright (c) 2009 Fabien Potencier
 *
 * Permission is hereby granted, free of charge, to any person obtaining a copy
 * of this software and associated documentation files (the "Software"), to deal
 * in the Software without restriction, including without limitation the rights
 * to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
 * copies of the Software, and to permit persons to whom the Software is furnished
 * to do so, subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be included in all
 * copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
 * FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
 * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
 * LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
 * OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
 * THE SOFTWARE.
 */

namespace Pimple;

/**
 * Container main class.
 *
 * @author  Fabien Potencier
 */
class Container implements \ArrayAccess
{
    private $values = array(); // 存储 value 的数组
    private $factories; // 存储工厂方法的对象
    private $protected; // 存储保护方法的对象
    private $frozen = array(); // 存储冻结的数组，也就是在这个数组里面的 key 的 value 是不可更改的了
    private $raw = array(); // 存储
    private $keys = array(); // 存储 key 的数组

    /**
     * Instantiate the container.
     *
     * Objects and parameters can be passed as argument to the constructor.
     *
     * @param array $values The parameters or objects.
     */
    public function __construct(array $values = array())
    {
        // 构造工厂对象以及保护方法对象
        $this->factories = new \SplObjectStorage();
        $this->protected = new \SplObjectStorage();
        // 把初始化的值存放到现有的里面
        foreach ($values as $key => $value) {
            $this->offsetSet($key, $value);
        }
    }

    /**
     * Sets a parameter or an object.
     *
     * Objects must be defined as Closures.
     *
     * Allowing any PHP callable leads to difficult to debug problems
     * as function names (strings) are callable (creating a function with
     * the same name as an existing parameter would break your container).
     *
     * @param string $id    The unique identifier for the parameter or object
     * @param mixed  $value The value of the parameter or a closure to define an object
     *
     * @throws \RuntimeException Prevent override of a frozen service
     * 设置相关值以及对象
     */
    public function offsetSet($id, $value)
    {
        // 如果这个值被 frozen 了，就不允许更改了
        // 应该是为了保持高效性，会把 get 过的值存储起来，所以就不在调用了也就不允许更改了
        // 其实也可以更改，在下面的方法中说明
        if (isset($this->frozen[$id])) {
            throw new \RuntimeException(sprintf('Cannot override frozen service "%s".', $id));
        }
        // 存储值方法
        $this->values[$id] = $value;
        // 存储 key 的值
        $this->keys[$id] = true;
    }

    /**
     * Gets a parameter or an object.
     *
     * @param string $id The unique identifier for the parameter or object
     *
     * @return mixed The value of the parameter or an object
     *
     * @throws \InvalidArgumentException if the identifier is not defined
     * 获取值得方法
     */
    public function offsetGet($id)
    {
        // 如果没有设置 key 则抛出异常
        if (!isset($this->keys[$id])) {
            throw new \InvalidArgumentException(sprintf('Identifier "%s" is not defined.', $id));
        }

        // 如果 raw 里面已经有了， 或者 values 里面存储对应 key 的值不是个 obj，或者 protected 里面也有对应的值 或者值里面的方法存在，则直接返回，values 数组里面的结果
        if (
            isset($this->raw[$id])
            || !is_object($this->values[$id])
            || isset($this->protected[$this->values[$id]])
            || !method_exists($this->values[$id], '__invoke')
        ) {
            return $this->values[$id];
        }

        // 如果工厂方法里面设置了相关方法则要直接返回
        if (isset($this->factories[$this->values[$id]])) {
            return $this->values[$id]($this);
        }

        // 获取值里面的方法
        $raw = $this->values[$id];
        // 执行上面获取到的方法获取返回值 并且覆盖 values
        $val = $this->values[$id] = $raw($this);
        // 把原始方法存储到 raw 数组里面，用来给 raw 方法调用
        $this->raw[$id] = $raw;
        // 把这个值设置为冻结，不允许将来的更改
        $this->frozen[$id] = true;
        // 返回结果值
        return $val;
    }

    /**
     * Checks if a parameter or an object is set.
     *
     * @param string $id The unique identifier for the parameter or object
     *
     * @return bool
     * 获取 key 是否存在
     */
    public function offsetExists($id)
    {
        return isset($this->keys[$id]);
    }

    /**
     * Unsets a parameter or an object.
     *
     * @param string $id The unique identifier for the parameter or object
     * 删除掉 key
     */
    public function offsetUnset($id)
    {
        // 如果存在则删除相关的值
        if (isset($this->keys[$id])) {
            // 如果存储的是个对象，则删除相关的值
            if (is_object($this->values[$id])) {
                unset($this->factories[$this->values[$id]], $this->protected[$this->values[$id]]);
            }
            // 删除普通数组里面的值
            unset($this->values[$id], $this->frozen[$id], $this->raw[$id], $this->keys[$id]);
        }
    }

    /**
     * Marks a callable as being a factory service.
     *
     * @param callable $callable A service definition to be used as a factory
     *
     * @return callable The passed callable
     *
     * @throws \InvalidArgumentException Service definition has to be a closure of an invokable object
     * 设置工厂方法
     */
    public function factory($callable)
    {
        if (!method_exists($callable, '__invoke')) {
            throw new \InvalidArgumentException('Service definition is not a Closure or invokable object.');
        }

        $this->factories->attach($callable);

        return $callable;
    }

    /**
     * Protects a callable from being interpreted as a service.
     *
     * This is useful when you want to store a callable as a parameter.
     *
     * @param callable $callable A callable to protect from being evaluated
     *
     * @return callable The passed callable
     *
     * @throws \InvalidArgumentException Service definition has to be a closure of an invokable object
     */
    public function protect($callable)
    {
        if (!method_exists($callable, '__invoke')) {
            throw new \InvalidArgumentException('Callable is not a Closure or invokable object.');
        }

        $this->protected->attach($callable);

        return $callable;
    }

    /**
     * Gets a parameter or the closure defining an object.
     *
     * @param string $id The unique identifier for the parameter or object
     *
     * @return mixed The value of the parameter or the closure defining an object
     *
     * @throws \InvalidArgumentException if the identifier is not defined
     * 其实这个就是获取设置的对象或者方法
     */
    public function raw($id)
    {
        if (!isset($this->keys[$id])) {
            throw new \InvalidArgumentException(sprintf('Identifier "%s" is not defined.', $id));
        }
        if (isset($this->raw[$id])) {
            return $this->raw[$id];
        }

        return $this->values[$id];
    }

    /**
     * Extends an object definition.
     *
     * Useful when you want to extend an existing object definition,
     * without necessarily loading that object.
     *
     * @param string   $id       The unique identifier for the object
     * @param callable $callable A service definition to extend the original
     *
     * @return callable The wrapped callable
     *
     * @throws \InvalidArgumentException if the identifier is not defined or not a service definition
     * 修改已经存在的值
     */
    public function extend($id, $callable)
    {
        if (!isset($this->keys[$id])) {
            throw new \InvalidArgumentException(sprintf('Identifier "%s" is not defined.', $id));
        }

        if (!is_object($this->values[$id]) || !method_exists($this->values[$id], '__invoke')) {
            throw new \InvalidArgumentException(sprintf('Identifier "%s" does not contain an object definition.', $id));
        }

        if (!is_object($callable) || !method_exists($callable, '__invoke')) {
            throw new \InvalidArgumentException('Extension service definition is not a Closure or invokable object.');
        }

        $factory = $this->values[$id];

        $extended = function ($c) use ($callable, $factory) {
            return $callable($factory($c), $c);
        };

        if (isset($this->factories[$factory])) {
            $this->factories->detach($factory);
            $this->factories->attach($extended);
        }

        return $this[$id] = $extended;
    }

    /**
     * Returns all defined value names.
     *
     * @return array An array of value names
     * 获取所有的 key
     */
    public function keys()
    {
        return array_keys($this->values);
    }

    /**
     * Registers a service provider.
     *
     * @param ServiceProviderInterface $provider A ServiceProviderInterface instance
     * @param array                    $values   An array of values that customizes the provider
     *
     * @return static
     * 注册自己的服务用的，后续文章体现
     */
    public function register(ServiceProviderInterface $provider, array $values = array())
    {
        $provider->register($this);

        foreach ($values as $key => $value) {
            $this[$key] = $value;
        }

        return $this;
    }
}

```


好吧，上面注释的都已经写出来了，个人觉得很简单，但是有几个方法我觉得可以扩展开来说，可能在过几天的文章里面展开来说说，基本上就是 register、extend和 raw 这几个方法的具体说明了。


哦对了，开始我觉得 factory 和 protect 方法差不多，其实还是有区别的，就是 factory 会有一个 $container 的参数，protect 方法就是存储的普通方法，区别就这么简单了。不过更多的方法，也会在后续文章说明的。
