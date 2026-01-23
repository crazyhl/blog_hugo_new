---
title: Filament 使用 RelationManager  相关标签废弃后应该如何处理
subtitle:
date: 2026-01-23T21:49:05+08:00
slug: How-should-Filament-handle-deprecated-RelationManager-related-tags
tags:
  - filament
categories:
  - php

# See details front matter: https://fixit.lruihao.cn/documentation/content-management/introduction/#front-matter
---

<!--more-->
以前在使用 `filament` 的时候，总是想着能用就行。但是这次准备实战以后，就会注意很多东西。一些被标记废弃的标签，就会想办法解决掉，万一以后升级版本移除了，改动就会很大。比如当我在使用 `RelationManger` 的时候就发现这些被弃用了。
![](https://raw.githubusercontent.com/M1racle-Hao/blog-image/master/20260123215219773.png)

根据提示需要重写 `table()` 的相关方法，既然如此，我们就去 `table` 里面寻找。最终在 `HasRecord` 的 traits 里面找到了需要调用的方法。

> Deprecated:  the `table()` method to configure the table.

我用一个举例 `modelLabel`标签就重写 `modelLabel()` 方法就可以了，示例如下

```php
public function table(Table $table): Table
{
    return $table
        ->heading('会员卡管理')
        ->modelLabel('会员卡')
        ->headerActions([
            CreateAction::make(),
        ]);
}
```
找方法很麻烦，也许是我很懒，敲几个字母没找到同名的方法就懵了。最后一顿查才找到相关方法，以后还需要更细心一些。