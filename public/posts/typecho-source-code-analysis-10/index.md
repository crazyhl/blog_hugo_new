# Typecho 源码分析（10）-- DoAction 分析


## 前情提要

前面我们分析了插件列表，看到 html 部分，我们看到了请求的url 包含 action 我们这篇就分析这里。

## 正文开始

在做到插件启用相关部分的时候。发现了一个链接，

```php
http://typecho.test/index.php/action/plugins-edit?activate=HelloWorld&_=a05deb76f571cfb798d3904cc6cecf77
```

这块我就比较好奇了，为什么跟首页部分的不太一样吗，难道是又包装了一层？我们从路由分发来看一下。

```php

array(6) { ["url"]=> string(22) "/action/[action:alpha]" ["widget"]=> string(9) "Widget_Do" ["action"]=> string(6) "action" ["regx"]=> string(32) "|^/action/([_0-9a-zA-Z-]+)[/]?$|" ["format"]=> string(10) "/action/%s" ["params"]=> array(1) { [0]=> string(6) "action" } }
```

可以看到，这个路由，匹配了 `action` 开头的 `url`，执行的组件是 `Do`，参数是一个`action`。具体路由的分析请看前几篇文章。

我们看下 `Do` 的执行部分

```php
public function execute()
{
    /** 验证路由地址 **/
    $action = $this->request->action;

    //兼容老版本
    if (empty($action)) {
        $widget = trim($this->request->widget, '/');
        $objectName = 'Widget_' . str_replace('/', '_', $widget);

        if (preg_match("/^[_a-z0-9]$/i", $objectName) && Typecho_Common::isAvailableClass($objectName)) {
            $widgetName = $objectName;
        }
    } else {
        /** 判断是否为plugin */
        $actionTable = array_merge($this->_map, unserialize($this->widget('Widget_Options')->actionTable));

        if (isset($actionTable[$action])) {
            $widgetName = $actionTable[$action];
        }
    }

    if (isset($widgetName) && class_exists($widgetName)) {
        $reflectionWidget =  new ReflectionClass($widgetName);
        if ($reflectionWidget->implementsInterface('Widget_Interface_Do')) {
            $this->widget($widgetName)->action();
            return;
        }
    }

    throw new Typecho_Widget_Exception(_t('请求的地址不存在'), 404);
}
```

先获取 请求的 `action` ，这个值是从构造组件的时候传递过来的，

```php
$widget = Typecho_Widget::widget($route['widget'], NULL, $params);
```

`$params` 这个值就会传入到 `request`  的构造中，看下面

```php
$requestObject = new Typecho_Request();$requestObject->setParams($request);
```

具体大家可以追一下代码就了解了。

通过判断 `action` 是否为空来决定 `widgetname`，判断的部分大家可以自己看一下。

紧接着，会去用反射相关的方法去调用具体的逻辑。

反射我们会单独找一篇文章去说明。对了，还有一个判断，是会这个组件是否实现了 `Widget_Interface_Do` 接口。

## 下期预告
这篇比较短，但是不乱，所以可以说是很简洁了，下篇我们继续在插件查遍看看还有什么要说的。

---

> 作者:   
> URL: http://localhost:1313/posts/typecho-source-code-analysis-10/  

