---
title: "Laravel Livewire 使用笔记（2）"
date: 2022-10-28T10:33:19+08:00
categories: [php]
slug: laravel-livewire-note-2
tags: [laravel, livewire]
---
![](https://www.cimple.ink/static/images/laravel-livewire.png)

## 前言
上一篇我们完成了项目的创建，`livewire` 的引入，以及第一个 `component` 的创建。今天我们继续，来给 `component` 增加点操作以及从数据库获取内容。

## 触发一个操作
修改 `Counter.php`
```php
<?php

namespace App\Http\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public $count = 0;

    public function increment() {
        $this->count++;
    }

    public function render()
    {
        return view('livewire.counter');
    }
}

```

修改 `counter.blade.php`
```php
<div>
    <h1>{{ $count }}</h1>
    <button wire:click="increment">+</button>
</div>
```

可以看到我们在 `Counter.php` ，增加了一个 `publish` 属性 `$counter` 和一个 `publish` 方法 `increment`。在前端页面可以直接使用 `blade` 语法显示 `$count` 和 通过 `wire` 方法调用 `increment` 方法。这块是不是很爽，跟 `vue` 很像。

打开页面，就可以看到数字显示和按钮了，点击按钮就会发现上方的数字变化了
![](https://www.cimple.ink/static/images/20221028105034.png)
那么这后面发生了什么呢？让我们打开开发者面板的网络标签，然后点击一下按钮看看发生了什么吧。
![](https://www.cimple.ink/static/images/20221028105206.png)
可以看到，触发了一个请求，不过这个请求并没有在我们的定义中出现，那么为什么会调用成功呢？（这块留个悬念，在最后来说明这块。）
可以自行查看 `载荷` 和 `响应` 标签来查看提交和返回了什么？这块是不是也跟 `vue` 很像？不过提交和返回的内容会比 `vue` 多一些。但是对比要学习 `vue` 或 `react` 的时间，用 `livewire` 几乎没有成本，多一点的提交和返回内容又有什么呢？但是还得说一嘴，其实简单了说就是个 `ajax` 请求，然后在对应的 `id` 区域刷新内容罢了。我司早在 n 年前就用 `jquery` 实现了类似的功能。所以说，不管用什么新技术，要多想想多看看，为什么会产生新的东西？是真的技术发展，还是对既有东西的高级封装，再或者说效率的提升。只要不是简单的高级封装，我们都是可以深入了解一下的，为什么会带来提升。

## 更进一步
上面那些，加上上一篇等于是把官方的 `QuickStart` 和 `Installation` 部分搞定了。接下来我们继续再来创建一个组件，这个组件从数据库获取一些数据。

首先创建组件 `./vendor/bin/sail artisan make:livewire Post/Show`。可以看到这个跟第一篇的创建方式几乎一致，不过有一个微小差别就是最后那部分。我们用了个分割线，这个有什么好处呢，就是按照文件夹来划分了，毕竟一个模块一个文件夹里面有该模块下的各种组件，方便我们的项目管理。

紧接着创建路由。
```php
Route::prefix('/post')->group(function () {
    Route::get('/show/{post}', App\Http\Livewire\Post\Show::class);
});
```
这里用到了路由器前缀以及分组，因为后续大部分内容都是在这个前缀下的。

再来我们创建模型，创建模型以及填充测试数据，我们都是用 laravel 提供的命令。
创建模型 
```php
./vendor/bin/sail artisan make:model Post --seed --migration
```
![](https://www.cimple.ink/static/images/202210281156537.png)
通过这个命令一次创建出来了模型、填充器 和 数据库迁移文件。各个文件的位置如图。
首先编辑迁移文件
```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->string('title')->comment('标题')->unique();
    $table->foreignId('user_id')->comment('作者')->constrained();
    $table->text('content')->comment('内容');
    $table->timestamps();
});
```
设置我们需要的一些字段。紧接着执行迁移
```php
./vendor/bin/sail artisan migrate
```
这样就在数据库上把所有的表都创建好了。数据填充部分可以看看文档，或者我在最后贴出的livewire-demo 项目的github
数据填充好后我们就可以在视图里面尝试显示数据了。
编辑 `show.blade.php` 
```php
<div>
    {{-- Care about people's approval and you will be their prisoner. --}}
    <div>{{$post->id}}</div>
    <div>{{$post->title}}</div>
    <div>{{$post->user->name}}</div>
    <div>{{$post->content}}</div>
</div>
```
在浏览器打开对应路由的 `url` 应该就可以看到文章的 `id` 和 `title` 了。

## 要不再弄个列表？
单篇文章已经可以展示了，要不我们在弄个文章列表吧，创建组件
```php
./vendor/bin/sail artisan make:livewire Post/pagination 
```
这个名称是为了后期分页考虑的，第一步我们就只在这边显示全部文章就好
编辑 Pagination.php
```php
class Pagination extends Component
{
    public $posts;

    public function mount()
    {
        $this->posts = Post::all();
    }

    public function render()
    {
        return view('livewire.post.pagination');
    }
}
```
直接返回所有文章
编辑视图
```php
<div>
    {{-- In work, do what you enjoy. --}}
    @foreach($posts as $post)
        @include('livewire.post.show', ['post' => $post])
    @endforeach
</div>
```
可以看到用了个循环把单个文章内容传入到了上面 `show` 里面去了，通过这个也可以学会一些参数的传递方式。后续我们会利用这个构造更复杂的东西。
在路由 `post` 组里面新增一个
```php
Route::get('/show', App\Http\Livewire\Post\Pagination::class);
```
然后访问 `http://localhost/post/show` 就会看到所有的内容了。
![](https://www.cimple.ink/static/images/202210281553195.png)

## 优化样式
功能方面今天就弄到这边先，接下来我们用 `tailwindcss` 优化一下样式，这里的内容就在代码库里面自己看吧。

## 总结
原本还想写点功能的，但是想想还是算了，让我在思考后再写，明天会把这个坑填上。不过今天写了个列表，也算一换一了。

今天获得的储备知识，
1. 引用其他组件。组件中的 `public` 属性可以在模板上直接使用。
2. 这个 `mount` 方法作用是什么呢？可以看下 `livewire` 的声明周期。

上面这些都可以自行查看官方文档来了解一下。

demo 项目地址：[livewire-demo](https://github.com/crazyhl/livewire-demo)