# Typecho 源码分析（3）


## 前情提要

通过之前我们的分析，我们已经把安装流程搞定了，本篇开始，我们分析前台相关的流程。

## 正文开始

我们又一次回到了 `index.php` 在第一篇说完前几行判断是否安装之后，我们就暂时离开了这个文件，当我们安装完以后，进入首页就又回到了这里，我们跳过判断安装那一块，直接看后面。

```php
/** 初始化组件 */
Typecho_Widget::widget('Widget_Init');

/** 注册一个初始化插件 */
Typecho_Plugin::factory('index.php')->begin();

/** 开始路由分发 */
Typecho_Router::dispatch();

/** 注册一个结束插件 */
Typecho_Plugin::factory('index.php')->end();
```

别看只有4行，却是大量的精髓都在这边，我们一行一行看。
第一行，初始化组件，我们进入代码

```php
/**
    * 工厂方法,将类静态化放置到列表中
    *
    * @access public
    * @param string $alias 组件别名
    * @param mixed $params 传递的参数
    * @param mixed $request 前端参数
    * @param boolean $enableResponse 是否允许http回执
    * @return Typecho_Widget
    * @throws Typecho_Exception
    */
public static function widget($alias, $params = NULL, $request = NULL, $enableResponse = true)
{
    $parts = explode('@', $alias);
    $className = $parts[0];
    $alias = empty($parts[1]) ? $className : $parts[1];

    if (isset(self::$_widgetAlias[$className])) {
        $className = self::$_widgetAlias[$className];
    }

    if (!isset(self::$_widgetPool[$alias])) {
        /** 如果类不存在 */
        if (!class_exists($className)) {
            throw new Typecho_Widget_Exception($className);
        }

        /** 初始化request */
        if (!empty($request)) {
            $requestObject = new Typecho_Request();
            $requestObject->setParams($request);
        } else {
            $requestObject = Typecho_Request::getInstance();
        }

        /** 初始化response */
        $responseObject = $enableResponse ? Typecho_Response::getInstance()
        : Typecho_Widget_Helper_Empty::getInstance();

        /** 初始化组件 */
        $widget = new $className($requestObject, $responseObject, $params);

        $widget->execute();
        self::$_widgetPool[$alias] = $widget;
    }

    return self::$_widgetPool[$alias];
}
```

先看第一个参数 `$alias` 第一步把传入的变量利用 `@` 拆分成两个变量，一个是要初始化的类名，另一个是个别名，如果没有别名的设置，雷德明就是别名。 然后在判断别名池里面有没有这个别名，如果包含了这个别名，类名就变成别名池设置的类名。这个别名池，在这个类下的 `alias` 方法中可以设置，当前这个方法貌似没有使用。

继续下去，如果组件池里面没有这个实例，就要初始化这个类，先初始化了，`request`，初始化的时候，会把 `get` 和 `pos`t 参数，设置到 `request` 的 `$_widgetAlias` 里面。然后调用了 `request` 的 `setParams` 把 `widget` 方法传入的 `params` 参数传入，这个方法，会把 `params` 设置到 `request` 的 `_params` 里面。当然，如果传入的 `request` 参数不是空，则会获取已经存在的 `request`，不过 `request` 的 `getInstance` 方法中不存在 `request` 实例的话，也会重新初始化一下。

紧接着会根据 `$enableResponse` 是否为 `true` 决定，是创建 `resposne` 或者 `Typecho_Widget_Helper_Empty`。`Typecho_Widget_Helper_Empty` 这个类，我们后面再说。

最后，初始化我们传入的 `class` ，并且执行 `execute` 方法。最后把 `class` 实例，放入到组件池当中，并且把组件返回。

好了，现在我们说说初始化的 `Widget_Init` 类。这个类继承了 `Typecho_Widget` 类，这个类是个组件根类，这里面封装了不少数据结构，以及 `request` 、`response` 之类的东西，这个类，我们在后面用到的时候分别来说再说。初始化 `Typecho_Widget` 的时候，会传入 `request` 、`resposne` 、`$params`。`request` 、`resposne` 会赋值给实例的 `request` 、`resposne`。紧接着初始化了 `Typecho_Config`, 这个类就当做数组看就行了。再把 `$params` 调用给 `Typecho_Config` 的 `setDefault` 方法，设置初始值。

## 下期预告

紧接着我们看看 `Widget_Init` 类的 `execute` 方法。这里面方法也很复杂，我们下期再说。


---

> 作者:   
> URL: http://localhost:1313/posts/typecho-source-code-analysis-3/  

