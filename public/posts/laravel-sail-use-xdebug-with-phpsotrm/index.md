# Laravel Sail Phpstorm 开启 Xdebug


以前弄 java 的时候常年跟 dubug 作伴。换到 `php` 后就没用过 `debug`。因为 `var_dump` 是真爽啊。但是如今又用回了 `laravel` 不得不再次把 `debug` 提上日程了。因为不论 `laravel` 还是 `symfony` 封装的太到位了，如果没有 `debug` 的调用栈。真的没法去追代码了。

闲话少叙，开整。

创建项目后，编辑 `.env` 文件 添加一行 `SAIL_XDEBUG_MODE=develop,debug,coverage`。这三个 `mode` 的解释，可以看这里[https://xdebug.org/docs/all_settings](https://xdebug.org/docs/all_settings)。

> 注意，编辑 `.env` 文件后需要重启 sail, 以使上面的配置生效。

浏览器安装 [xdebug-helper](https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc) 扩展，其他浏览器可以自行查找。扩展安装好后可以在扩展按钮选择 `debug` 。浏览器到这就差不多了。

接下来弄 `phpstorm` 。
![](https://www.cimple.ink/static/images/202211031353198.png)

选择红框的选项，就可以开启监听了。开启监听后，会提示配置没有，我们就可以自己配置一下了。
![](https://www.cimple.ink/static/images/202211031355851.png)
按照图里的配置就可以了。
最后还需要配置 php 的 deubg 看图即可
![](https://www.cimple.ink/static/images/202211031358157.png)

不出意外的话，就可以跑起来了。
好了，今天就到这，这几天我就用 debug。追踪一些代码了。后续，等我准备好了就来。

---

> 作者:   
> URL: http://localhost:1313/posts/laravel-sail-use-xdebug-with-phpsotrm/  

