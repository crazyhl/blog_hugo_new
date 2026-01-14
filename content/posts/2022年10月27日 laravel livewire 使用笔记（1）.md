---
title: "Laravel Livewire 使用笔记（1）"
date: 2022-10-27T19:28:58+08:00
categories: [php]
slug: laravel-livewire-note-1
tags: [laravel, livewire]
---
![](https://gh-image.crazyhl.workers.dev/M1racle-Hao/blog-image/master/laravel-livewire.png)

## 前言（唠叨）
最近重拾 `php`，开始写自己的东西（最近2年都在用 `go`）。至于为何换回 `php` 我会在后面东西弄到 `30%` 以后在详细来说。

哥们给我推荐了一个库，这个库用到了 `livewire` 我看了一下感觉很有意思，想用这个来组织构建我的项目，于是就有了下面的笔记（当然如果没有没用这个也算是一个学习过程了）。当然这一系列的笔记，也可以当做部分的官方文档的翻译。

> 本次所有的代码都是在 `laravel-sail` 下操作的，命令相关请自行做替换。
> 
> 对应所有文档都是在当前发布日期对应的 `laravel` 和 `livewire` 等相关依赖的最新版本下进行的，后续对应库如有更新，请自行翻阅官方文档，thx。

## 创建项目

```bash
curl -s "https://laravel.build/livewire-demo?with=mysql&devcontainer" | bash
```

我们的测试项目预计只需要 mysql 就够了。所以只引进了 mysql 依赖。livewire-demo 则是我们项目的名字。

```bash
cd livewire-demo && ./vendor/bin/sail up -d
```

用后台方式启动项目，当然想看日志把 `-d` 去掉。打开 `http://localhost` 应该就可以看到项目首页了。
![](https://gh-image.crazyhl.workers.dev/M1racle-Hao/blog-image/master/20221027204713.png)

## 安装 livewire

```bash
./vendor/bin/sail composer require livewire/livewire
```

## 创建第一个组件

### 创建 layouts/app.blade.php
在 `resources/views/` 下创建 `layouts` 文件夹，并创建 `app.blade.php` 文件
```php
<html>
<head>
    <title>Laravel Livewire Demo</title>
    @livewireStyles
</head>
<body>
    {{ $slot }}
    @livewireScripts
</body>
</html>
```
此处使用的是组件式的方式实现的基础 layout 模板。具体可看 `laravel` 官方文档 [visit Laravel's documentation](https://laravel.com/docs/blade#components)。

### 创建组件

```bash
./vendor/bin/sail artisan make:livewire counter
```
这个命令会在 `app/Http/Livewire/` 文件夹下创建 `Counter.php` 文件 和 `resources/views/livewire` 文件夹下创建 `counter.blade.php` 文件。 `Counter.php` 实现逻辑, `counter.blade.php` 实现前端。

编辑 `counter.blace.php` 
```php
<div>
    <h1>Hello World!</h1>
</div>
```

编辑路由 `routes/web.php` 新增下面的代码。
```php
Route::get('/counter', Counter::class);
```

现在访问 `http://localhost/counter` 就可以看到下面的样子了。
![](https://gh-image.crazyhl.workers.dev/M1racle-Hao/blog-image/master/20221027211211.png)

## 总结
好了今天我们就到这了，至此 `livewire` 的第一步已经成功迈出了。

我们说下上面没有具体说的几个点。

1. 为什么创建 `layouts/app.blade.php`？ 这个是 `livewire` 的默认配置，想看各种配置可以自己导出配置文件 `./vendor/bin/sail artisan livewire:publish --config` 来查看。
2. 路由是怎么回事？这块我们回来第二篇文章来说，以及 `livewire` 是如何交互的。

再见，今天就到这，明天继续