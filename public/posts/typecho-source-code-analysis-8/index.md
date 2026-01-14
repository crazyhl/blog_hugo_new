# Typecho 源码分析（8）-- 后台插件列表


## 前情提要

前面 7 篇文章基本上已经分析完成 `Typecho` 的运行流程了，从本篇开始就开始分析各种模块了，原本是想分析路由的，但是我更对插件感兴趣，所以就从插件开始了。

## 正文开始

我们先不分析插件的加载流程，因为我们还没有启用任何插件，所以我们从后台的插件列表开始。我们打开 `admin\plugins.php` 文件。前面的几行加载我们稍后再说，直奔重点插件列表而去。

```php
<?php Typecho_Widget::widget('Widget_Plugins_List@activated', 'activated=1')->to($activatedPlugins); ?>
```
这行代码会获取启用的插件列表，我们来到 `Widget\Plugins\List.php` 文件，为什么是这个文件，请看前三篇文章和 `var\Typecho\Common.php` 的 `__autoLoad` 方法就了解了。

 调用这个文件的 `execute` 方法，
 
 首先获取插件目录，设置默认参数，获取已经启用的插件，把已经启用的插件放到

```php
$this->activatedPlugins = $plugins['activated'];
```

里面，紧接着如果插件目录存在，就开始遍历插件目录。
获取插件，根据目录和文件的形态，返回插件信息

```php
return array($pluginName, $pluginFileName);
```

接下来获取插件信息，
判断插件版本，判断插件是否在数据库中启用，如果启用了，就放到插件list 的栈中。

上面的是获取激活的插件部分，第二部分是获取禁用的插件，我们用禁用的部分测试一下上面的对不对，
插件目录 

```php
array(1) { [0]=> string(40) "/var/www/typecho//usr/plugins/HelloWorld" }
```

设置默认参数，

```php
$this->parameter->setDefault(array('activated' => NULL));
```

这边传入的的是null，返回的却是0。

```php
object(Typecho_Config)#27 (1) {
  ["_currentConfig":"Typecho_Config":private]=>
  array(1) {
    ["activated"]=>
    string(1) "0"
  }
}
```

是因为在初始化 `List` 组件的时候，我们传入了 `params` `activated=0`,这边相关的可以去看 组件 初始化的部分，
以及 `$this->parameter->setDefault($params)`  这个方法的具体实现。

接下来导出数据库里面激活的插件信息，现在还是空的，

```php
 $this->activatedPlugins = $plugins['activated'];
```

把 `activated` 放到 `$this->activatedPlugins` 中。

如果插件目录不为空，就开始遍历插件目录里面的插件，剩下的就可以参照上面的说明了。

```php
/** 默认即插即用 */
$info['activated'] = true;

if ($info['activate'] || $info['deactivate'] || $info['config'] || $info['personalConfig']) {
    $info['activated'] = isset($this->activatedPlugins[$pluginName]);

    if (isset($this->activatedPlugins[$pluginName])) {
        unset($this->activatedPlugins[$pluginName]);
    }
}

if ($info['activated']  == $this->parameter->activated) {
    $this->push($info);
}
```

说说这块吧，默认激活的，然后进入if判断，判断已激活的插件是否在数据存储中，如果存在，就在 `activatedPlugins` 卸载掉，这一步就是为了防止数据重复。

`$info['activated']  == $this->parameter->activated` 这行判断，就是看跟传入的参数是否相同，如果相同就放入到组件的 stack 中。

页面html部分就自己看看好了，不分析了。

## 下期预告

这篇开始就不会大篇幅的贴源码了，只贴一些重要的东西，大家要自己去看源码了，自己追踪一下总会更好，下篇会是一个短篇，分析部分的 `$security` 这个组件


---

> 作者: [M1racleHao](https://github.com/crazyhl)  
> URL: http://localhost:1313/posts/typecho-source-code-analysis-8/  

