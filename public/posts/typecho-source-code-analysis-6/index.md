# Typecho 源码分析（6）



## 前情提要

上一篇说了一下插件相关的东西，可是发现插件需要很多东西去说，于是就没说全，因为还是要抓紧把全部流程跑通，所以就省略下来了，这篇我们说一下路由相关的。等这个说完，等于就把全部流程跑通了。后面我们就可以展开来说各种模块了。

## 正文开始

```php
/** 开始路由分发 */
Typecho_Router::dispatch();
```

index 文件的最后一行了。开始吧，让我们进入方法内部，

```php
 /**
    * 路由分发函数
    *
    * @return void
    * @throws Exception
    */
public static function dispatch()
{
    /** 获取PATHINFO */
    $pathInfo = self::getPathInfo();

    foreach (self::$_routingTable as $key => $route) {
        if (preg_match($route['regx'], $pathInfo, $matches)) {
            self::$current = $key;

            try {
                /** 载入参数 */
                $params = NULL;

                if (!empty($route['params'])) {
                    unset($matches[0]);
                    $params = array_combine($route['params'], $matches);
                }

                $widget = Typecho_Widget::widget($route['widget'], NULL, $params);

                if (isset($route['action'])) {
                    $widget->{$route['action']}();
                }

                Typecho_Response::callback();
                return;

            } catch (Exception $e) {
                if (404 == $e->getCode()) {
                    Typecho_Widget::destory($route['widget']);
                    continue;
                }

                throw $e;
            }
        }
    }

    /** 载入路由异常支持 */
    throw new Typecho_Router_Exception("Path '{$pathInfo}' not found", 404);
}

```

第一行，获取 `pathInfo` 我们再次进入方法。

```php
/**
     * 获取全路径
     *
     * @access public
     * @return string
     */
    public static function getPathInfo()
    {
        if (NULL === self::$_pathInfo) {
            self::setPathInfo();
        }

        return self::$_pathInfo;
    }

```

`pathInfo` 为 `null` 的时候执行 `set` 方法，默认值是 `/` 。我们看下返回值是什么。
`string(1) "/"` 返回了一个 `/` 是默认值。
这个值是从哪里过来的呢，是在 `init` 里面赋值的，我们看一下方法。

```php
 /**
     * 获取当前pathinfo
     *
     * @access public
     * @param string $inputEncoding 输入编码
     * @param string $outputEncoding 输出编码
     * @return string
     */
    public function getPathInfo($inputEncoding = NULL, $outputEncoding = NULL)
    {
        /** 缓存信息 */
        if (NULL !== $this->_pathInfo) {
            return $this->_pathInfo;
        }

        //参考Zend Framework对pahtinfo的处理, 更好的兼容性
        $pathInfo = NULL;

        //处理requestUri
        $requestUri = $this->getRequestUri();
        $finalBaseUrl = $this->getBaseUrl();

        // Remove the query string from REQUEST_URI
        if ($pos = strpos($requestUri, '?')) {
            $requestUri = substr($requestUri, 0, $pos);
        }

        if ((NULL !== $finalBaseUrl)
            && (false === ($pathInfo = substr($requestUri, strlen($finalBaseUrl)))))
        {
            // If substr() returns false then PATH_INFO is set to an empty string
            $pathInfo = '/';
        } elseif (NULL === $finalBaseUrl) {
            $pathInfo = $requestUri;
        }

        if (!empty($pathInfo)) {
            //针对iis的utf8编码做强制转换
            //参考http://docs.moodle.org/ja/%E5%A4%9A%E8%A8%80%E8%AA%9E%E5%AF%BE%E5%BF%9C%EF%BC%9A%E3%82%B5%E3%83%BC%E3%83%90%E3%81%AE%E8%A8%AD%E5%AE%9A
            if (!empty($inputEncoding) && !empty($outputEncoding) &&
            (stripos($_SERVER['SERVER_SOFTWARE'], 'Microsoft-IIS') !== false
            || stripos($_SERVER['SERVER_SOFTWARE'], 'ExpressionDevServer') !== false)) {
                if (function_exists('mb_convert_encoding')) {
                    $pathInfo = mb_convert_encoding($pathInfo, $outputEncoding, $inputEncoding);
                } else if (function_exists('iconv')) {
                    $pathInfo = iconv($inputEncoding, $outputEncoding, $pathInfo);
                }
            }
        } else {
            $pathInfo = '/';
        }

        // fix issue 456
        return ($this->_pathInfo = '/' . ltrim(urldecode($pathInfo), '/'));
    }
```

这个方法首先获取了 `requestUri`

```php
/**
     * 获取请求地址
     * 
     * @access public
     * @return string
     */
    public function getRequestUri()
    {
        if (!empty($this->_requestUri)) {
            return $this->_requestUri;
        }

        //处理requestUri
        $requestUri = '/';

        if (isset($_SERVER['HTTP_X_REWRITE_URL'])) { // check this first so IIS will catch
            $requestUri = $_SERVER['HTTP_X_REWRITE_URL'];
        } elseif (
            // IIS7 with URL Rewrite: make sure we get the unencoded url (double slash problem)
            isset($_SERVER['IIS_WasUrlRewritten'])
            && $_SERVER['IIS_WasUrlRewritten'] == '1'
            && isset($_SERVER['UNENCODED_URL'])
            && $_SERVER['UNENCODED_URL'] != ''
            ) {
            $requestUri = $_SERVER['UNENCODED_URL'];
        } elseif (isset($_SERVER['REQUEST_URI'])) {
            $requestUri = $_SERVER['REQUEST_URI'];
            $parts       = @parse_url($requestUri);
           
            if (isset($_SERVER['HTTP_HOST']) && strstr($requestUri, $_SERVER['HTTP_HOST'])) {
                if (false !== $parts) {
                    $requestUri  = (empty($parts['path']) ? '' : $parts['path'])
                                 . ((empty($parts['query'])) ? '' : '?' . $parts['query']);
                }
            } elseif (!empty($_SERVER['QUERY_STRING']) && empty($parts['query'])) {
                // fix query missing
                $requestUri .= '?' . $_SERVER['QUERY_STRING'];
            }
        } elseif (isset($_SERVER['ORIG_PATH_INFO'])) { // IIS 5.0, PHP as CGI
            $requestUri = $_SERVER['ORIG_PATH_INFO'];
            if (!empty($_SERVER['QUERY_STRING'])) {
                $requestUri .= '?' . $_SERVER['QUERY_STRING'];
            }
        }

        return $this->_requestUri = $requestUri;
    }

```

我们主要看 

```php
} elseif (isset($_SERVER['REQUEST_URI'])) {
    $requestUri = $_SERVER['REQUEST_URI'];
    $parts       = @parse_url($requestUri);

    if (isset($_SERVER['HTTP_HOST']) && strstr($requestUri, $_SERVER['HTTP_HOST'])) {
        if (false !== $parts) {
            $requestUri  = (empty($parts['path']) ? '' : $parts['path'])
                            . ((empty($parts['query'])) ? '' : '?' . $parts['query']);
        }
    } elseif (!empty($_SERVER['QUERY_STRING']) && empty($parts['query'])) {
        // fix query missing
        $requestUri .= '?' . $_SERVER['QUERY_STRING'];
    }
}
 ```

 这个判断里面的，因为我们主要是 `nginx` 不是用的`iis` 。
 可以看到最后拿到的 `requestUri` 就是请求 `/` 后面的所有加上参数。
 紧接着就是获取 `$finalBaseUrl` 这个就是请求前缀，用于选择目录。
 最后在通过 `$finalBaseUrl` 获取最后的 `$pathInfo`。
 然后再去初始化时候的 `route` 里面匹配，就找到了 `controller` 和 `method`。

## 下期预告

我们找到了需要调用的方法，我们在下篇在继续


---

> 作者: [M1racleHao](https://github.com/crazyhl)  
> URL: http://localhost:1313/posts/typecho-source-code-analysis-6/  

