---
title: PHP匿名函数以及call_user_func的思考
tags: [laravel, 匿名函数, 闭包]
categories: [PHP]
slug: thinking-about-php-anonymous-function-and-calluserfunc
date: 2016-09-04 21:27:00
updated: 2016-09-04 21:27:00
toc: false
---

![图片alt](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/2019/03/06/0664506a46a7649adc31545165d08f41.jpg)


配图依然与主题无关，仅仅是我个人喜欢而已，今天要说的是关于 `call_user_func` 的一些理解。

以前对于闭包和匿名函数的理解不是太好。所以，今天在 `phphub` 提问的时候犯了一个比较二的错误，趁着晚上回家赶紧总结一下，增强自己的记忆。

话说以前对于闭包或者匿名函数都是自己写自己调用，所以没有出现过什么问题，但是前几天在弄 `laravel` 分页的时候，发现里面有一个 `$presenter` 自己知道这个可能会让我的代码更简洁，但是我却不知道如何调用，在 `phphub` 提问的时候有人告诉了我怎么使用，但是我却对参数列表比较迷惑，是如何获取参数列表的。今天终于有人告诉了我，我才发现对于 `php` 的一些函数理解的并不是很到位。于是乎赶紧翻阅 `php` 文档，把 `call_user_func` 再次理解一下。参见： [http://php.net/manual/zh/function.call-user-func.php](http://php.net/manual/zh/function.call-user-func.php) 。

文档说的比较明白，第一个参数是方法名称，余下的参数就是调用方法的参数列表了。

```php
    public function render(Presenter $presenter = null)
    {
        if (is_null($presenter) && static::$presenterResolver) {
            $presenter = call_user_func(static::$presenterResolver, $this);
        }

        $presenter = $presenter ?: new BootstrapThreePresenter($this);

        return $presenter->render();
    }
```

所以，这个 `render` 方法里面就比较容易理解了，当我们传入的 `$presenter` 为空的时候，并且 `static::$presenterResolver` ($static是延迟绑定，这个也可以在文档里面找到)已经赋值的时候，就会执行 `if` 代码段里面的内容了。在我的代码中主要应用的是 `LengthAwarePaginator` 类里面的相关方法（其实还有一个 `Paginator` 类，这个类是用在那个 `simple` 的简单分页中的），在这里面我并没有找到为 `$presenter` 赋值的方法，所以我们去他们的父类 `AbstractPaginator` 中寻找，果然里面有为 `$presenter` 赋值的方法。

```php
    /**
     * Set the default Presenter resolver.
     *
     * @param  \Closure  $resolver
     * @return void
     */
    public static function presenter(Closure $resolver)
    {
        static::$presenterResolver = $resolver;
    }
```

可以看到这里面接受的参数是一个闭包或者说匿名函数（这两个东西我还得理解一下）。

至此，一些就很通顺了。我们传入的匿名函数的参数是 `AbstractPaginator` 的具体实现类，里面实现了分页相关渲染的方法。

所以，我们最终在provider里面待用相关的设置方法

```php
        // 设定分页主题
        LengthAwarePaginator::presenter(function ($paginator) {
            return new AmazeuiThreePresenter($paginator);
        });
```

这样我们在模板里面直接调用 `render` 即可，也不用时刻记得传入参数了。