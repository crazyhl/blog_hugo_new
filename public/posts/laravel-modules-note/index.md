# 2024 年 12 月 1 日 Laravel Modules 使用小记


最近准备用 laravel 写点实验性的东西，由于要准备精准划分模块，所以就发现了 [laravel-modules](https://github.com/nWidart/laravel-modules) 这个完美符合我需求的库。

安装的时候严格根据实际的版本参照文档安装即可 [https://laravelmodules.com/docs/v11/installation-and-setup](https://laravelmodules.com/docs/v11/installation-and-setup)。不过这里有个注意点，在 `Autoload` 部分需要配置
```php
"extra": {
    "laravel": {
        "dont-discover": []
    },
    "merge-plugin": {
        "include": [
            "Modules/*/composer.json"
        ]
    }
},
```
我在安装的时候，没有注意这点，导致模块没有自动加载，花费了很久的时间，这里就记录下来了，其实就是看文档不仔细。

创建模块的时候参照这里即可 [https://laravelmodules.com/docs/v11/creating-a-module](https://laravelmodules.com/docs/v11/creating-a-module) 。一定要把目录结构部分了解清楚，这就是一个很精简的 `laravel` 结构。

模块开发完后，准备发布了，就参照这篇文档 [https://laravelmodules.com/docs/v11/publishing-modules](https://laravelmodules.com/docs/v11/publishing-modules)。
注意点：
* 模块 `compose.json` 文件中的 `type` 为 `laravel-module`。这个在默认生成的 `composer.json` 文件中是没有的，记得添加上。
* 安装 `https://github.com/joshbrw/laravel-module-installer` 这个包会将安装的模块移动到 `Modules` 目录中，这个操作是可选的，毕竟封装好的模块，理论上是可以直接用的，如果有修改的需求，在安装这个包就可以了。
* 其他项目在需要应用模块的时候也需要安装 `laravel-modules` 这个别忘了。
* 模块需要发布的时候，要将模块移动到一个单独的文件，并且模块名称全部小写并且以`-module`结尾。虽然文档是这么说的，但是我觉得没必要，因为我们大部分都是将模块发布到 github 或者自建的 gitea 中，此时我们在创建项目的时候将模块名称全部小写，并且以`-module` 结尾就可以了。 
  >例如，名为Quotes的模块将被重命名为quotes-module 。
* 安装模块后执行 `php artisan module:enable moduleName` 命令来启用模块，其实这个我觉得也可以省略，毕竟我觉得一个完备的 `laravel-modules` 基座项目，需要在可视化的情况下来激活这些。`php artisan module:migrate moduleName` `php artisan module:seed moduleName` 用这两个命令运行模块的迁移文件，以及数据填充文件。

今天就先记录这么多，模块更多的使用相关，以及权限相关。我会在下一篇文章中继续来记录。

---

> 作者:   
> URL: http://localhost:1313/posts/laravel-modules-note/  

