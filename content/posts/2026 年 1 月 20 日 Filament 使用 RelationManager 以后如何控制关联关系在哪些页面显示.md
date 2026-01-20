---
title: Filament 使用 RelationManager 以后如何控制关联关系在哪些页面显示
subtitle:
date: 2026-01-20T09:38:17+08:00
slug: How-to-control-which-pages-display-the-association-relationship-after-using-RelationManager-in-Filament
tags:
  - filament
categories:
  - php
---
在使用 `Filament` 的时候经常需要引入关联关系。此时就需要使用 `RelationManager` 了。

但是这个关联关系，默认情况下会在 `View` 和 `Edit` 的页面都显示。

如果想控制在指定页面显示就需要参照 [https://filamentphp.com/docs/5.x/resources/managing-relationships#conditionally-showing-relation-managers](https://filamentphp.com/docs/5.x/resources/managing-relationships#conditionally-showing-relation-managers)这篇文档来进行控制。

例子：
```php
public static function canViewForRecord(Model $ownerRecord, string $pageClass): bool
{
    $canViewForRecord = parent::canViewForRecord($ownerRecord, $pageClass);

    return $canViewForRecord && $pageClass === ViewShop::class;
}
```

我这个就是控制在`View` 的时候显示的，调用父方法，是继承回来权限控制部分。文档里面并没有调用父方法，那么就会失去权限控制了。