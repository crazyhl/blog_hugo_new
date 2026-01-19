---
title: 给 Filament 的表格日期时间字段全局设置默认格式
subtitle:
date: 2026-01-19T10:39:44+08:00
slug: set-default-format-globally-for-datetime-fields-in-filament-table.
tags:
  - filament
categories:
  - php


# See details front matter: https://fixit.lruihao.cn/documentation/content-management/introduction/#front-matter
---

filament 的表单构建虽然快速，但是在日期时间字段，每个地方都要指明格式。很是不爽，翻了一下 filament 的文档，也没有找到相关设置。这时候就体现 ai 的好了。他会很深入的阅读文档以及 api 文档，总会在边边角角的地方，帮我找到合适的解决方案。

在 `AppServiceProvider.php` 的`boot` 方法添加，如下的代码，就可以给所有的表格设置默认的时间格式了。

```php
use Filament\Tables\Table;

public function boot(): void
    {
        Table::configureUsing(function (Table $table) {
            $table
                ->defaultDateTimeDisplayFormat('Y-m-d H:i:s');
        });
    }
```

在调用的其他地方直接写 `->datetime()`  就可以了。非常省心，也不用担心忘记写格式的问题了。

争取每天都能学会一个小技巧，并且分享出来