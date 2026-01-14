---
title: Let's Build A Forum with Laravel and TDD 笔记 001
tags: [Let's Build A Forum with Laravel and TDD]
categories: [PHP]
slug: lets-build-a-forum-with-laravel-and-tdd-notes-001
date: 2018-05-30 14:50:00
updated: 2018-05-30 14:50:00
toc: false
---

![图片alt](https://www.cimple.ink/images/2018/05/30/89a4759f89f6e8680d9c06d0661d13db.png)

这个教程是 laracasts 上面的一个系列教程，对于我来说有很多比较多的东西没有接触过，所以起码看完第一集来说对我还是很有帮助的，在其他论坛也是有相关笔记的，我仔细想了一下如果也是想他们那样记录每一步的操作来说意义并不大，毕竟我可以直接看他们的文章。所以这里面还记录一些我个人的感悟

1. 初步了解了 factory 的用法，比如在 thinker 虚拟数据，在测试的时候模拟数据，但是更多的东西还得看看文档才可以
2. 以后再使用 `php artisan` 命令的时候多多看看 `--help` 会有很多简便的方式让我们能少执行几个命令 比如 `php artisan make:model xxxModel -mr` 这个命令就会帮助我们生成 resource 形式的 controller 以及迁移文件。
3. 在 laravel 5.6 中 factory 是分文件的，个人理解可能会比较清晰，不用全部都写在一个文件里面了。方便管理



最后，再说一句，弄这个东西，给我的docker完善了不少
