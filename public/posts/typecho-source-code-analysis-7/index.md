# Typecho 源码分析（7）


## 题外话

两周之前搬了家，以前搬家从来没觉得东西这么多，收拾起来这么麻烦，基本上整理整理就可以过日子了。这次换了个整租，才发现屋子大了也不好，东西找不到，现在基本上算是步入正轨了，不过nas还没有就位，等我再整理整理在看看吧nas放到哪里。机械硬盘实在是太吵了，这次要放到一个安静的地方。

## 前情提要

第五篇简单分析了插件，其实什么都没说。上一篇分析了路由部分，也说的比较混乱，本周五和周六我用了一些时间，仔细的读了源码，把自己混乱的部分也都弄清了，所以这篇我就慢慢的再说一次路由，用两个请求，首页和文章页 来分析，将来在模块分析的时候也会把路由需要的部分，在分析。

## 正文开始

我们再次来到路由的 `dispatch` 方法

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

首先获取 `pathInfo` ，这个 `pathInfo` 是从那里获取的呢，是从 `Init` 里面初始化的，我们看下初始化部分

```php
$pathInfo = $this->request->getPathInfo();
```

进入这个方法

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
        var_dump($requestUri);
        $finalBaseUrl = $this->getBaseUrl();
        var_dump($requestUri);

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

这个方法了里面第一步，如果有 `pathInfo` 就返回，如果没有就进入后续的流程，我们这里面肯定是没有的，所以继续后续执行

先获取了 `$requestUri` ，

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

进入方法内部，第一步还是判断是否存在，不存在就从 `$_SERVER` 中获取相关参数，因为我们是在nginx中，所以在下面这个判断中获取参数

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

获得 `REQUEST_URI`，紧接着解析 用 `parse_url` 解析 获取到的 `uri` 得到 `parts`，紧接着判断 如果

```php
    if (isset($_SERVER['HTTP_HOST']) && strstr($requestUri, $_SERVER['HTTP_HOST'])) {
                if (false !== $parts) {
                    $requestUri  = (empty($parts['path']) ? '' : $parts['path'])
                                 . ((empty($parts['query'])) ? '' : '?' . $parts['query']);
                }
            } elseif (!empty($_SERVER['QUERY_STRING']) && empty($parts['query'])) {
                // fix query missing
                $requestUri .= '?' . $_SERVER['QUERY_STRING'];
            }
```

`server` 的 `host` 存在 并且 `uri` 在 `host` 里面，就判断解析的 `parts` 是否为 `false`， 然后拼接 `uri`，这里`if (isset($_SERVER['HTTP_HOST']) && strstr($requestUri, $_SERVER['HTTP_HOST']))`是 `false` 所以走下面的判断逻辑,

```php
    elseif (!empty(_SERVER['QUERY_STRING']) && empty(parts['query']))
```

当前这个url 

```php
http://typecho.test/index.php/archives/1/
```

也是false，所以请求的 uri 就是 

```php
/index.php/archives/1/
```

紧接着获取 `getBaseUrl`

```php
/**
     * getBaseUrl  
     * 
     * @access public
     * @return string
     */
    public function getBaseUrl()
    {
        if (NULL !== $this->_baseUrl) {
            return $this->_baseUrl;
        }

        //处理baseUrl
        $filename = (isset($_SERVER['SCRIPT_FILENAME'])) ? basename($_SERVER['SCRIPT_FILENAME']) : '';

        if (isset($_SERVER['SCRIPT_NAME']) && basename($_SERVER['SCRIPT_NAME']) === $filename) {
            $baseUrl = $_SERVER['SCRIPT_NAME'];
        } elseif (isset($_SERVER['PHP_SELF']) && basename($_SERVER['PHP_SELF']) === $filename) {
            $baseUrl = $_SERVER['PHP_SELF'];
        } elseif (isset($_SERVER['ORIG_SCRIPT_NAME']) && basename($_SERVER['ORIG_SCRIPT_NAME']) === $filename) {
            $baseUrl = $_SERVER['ORIG_SCRIPT_NAME']; // 1and1 shared hosting compatibility
        } else {
            // Backtrack up the script_filename to find the portion matching
            // php_self
            $path    = isset($_SERVER['PHP_SELF']) ? $_SERVER['PHP_SELF'] : '';
            $file    = isset($_SERVER['SCRIPT_FILENAME']) ? $_SERVER['SCRIPT_FILENAME'] : '';
            $segs    = explode('/', trim($file, '/'));
            $segs    = array_reverse($segs);
            $index   = 0;
            $last    = count($segs);
            $baseUrl = '';
            do {
                $seg     = $segs[$index];
                $baseUrl = '/' . $seg . $baseUrl;
                ++$index;
            } while (($last > $index) && (false !== ($pos = strpos($path, $baseUrl))) && (0 != $pos));
        }

        // Does the baseUrl have anything in common with the request_uri?
        $finalBaseUrl = NULL;
        $requestUri = $this->getRequestUri();

        if (0 === strpos($requestUri, $baseUrl)) {
            // full $baseUrl matches
            $finalBaseUrl = $baseUrl;
        } else if (0 === strpos($requestUri, dirname($baseUrl))) {
            // directory portion of $baseUrl matches
            $finalBaseUrl = rtrim(dirname($baseUrl), '/');
        } else if (!strpos($requestUri, basename($baseUrl))) {
            // no match whatsoever; set it blank
            $finalBaseUrl = '';
        } else if ((strlen($requestUri) >= strlen($baseUrl))
            && ((false !== ($pos = strpos($requestUri, $baseUrl))) && ($pos !== 0)))
        {
            // If using mod_rewrite or ISAPI_Rewrite strip the script filename
            // out of baseUrl. $pos !== 0 makes sure it is not matching a value
            // from PATH_INFO or QUERY_STRING
            $baseUrl = substr($requestUri, 0, $pos + strlen($baseUrl));
        }

        return ($this->_baseUrl = (NULL === $finalBaseUrl) ? rtrim($baseUrl, '/') : $finalBaseUrl);
    }
```

首先获取从 `server` 的  `SCRIPT_FILENAME` 获取 `$filename` ，如果`SCRIPT_FILENAME` 存在，则用 `basename` 方法获取 `$filename` ,当前的 `filename` 是 `index.php` 。`basenme` 方法的作用就是返回路径中的文件名，当前 如果 `SCRIPT_FILENAME` 值是 `/var/www/typecho/index.php` ，所以文件名就是 

```php 
index.php
```

紧接着判断 `server` 中的 `SCRIPT_NAME` 或 `PHP_SELF` 的内容经过 `basename` 处理后的文件名是否跟 `filename` 相同。我们的请求在 `SCRIPT_NAME` 这里的判断就符合了条件，所以 `baseurl` 就是 `server` 中的 `SCRIPT_NAME` 的值。

```php
/index.php
```

接下来 判断 `baseurl` 或 `baseurl` 的 `dirname` 在 `requesturi` 中是否开头，我们这里的场景是 

```php
else if (0 === strpos($requestUri, dirname($baseUrl))) 
```

这里的判断中达成的，所以 

```php
$finalBaseUrl = rtrim(dirname($baseUrl), '/');
```

就是吧 `dirname` 后的 `$baseUrl` 去掉右侧的/后的值。最后 `baseurl` 就是  

```php
(NULL === $finalBaseUrl) ? rtrim($baseUrl, '/') : $finalBaseUrl
```

判断 `finalBaseUrl` 是否为 `null` ，如果为 `null` 就是把 `baseurl` 去掉右侧的 `/` ，否则就是 `finalBaseUrl` 。我们这里 `finalBaseUrl` 不是 `null`，所以 baseurl就是 finalBaseUrl。为 `/index.php`。

接下来判断 `requesturi` 中是否包含 `?` ,如果包含，就截取 `？` 前面的不部分，我们这边不包含，所以 `requesturi` 依然是 

```php
/index.php/archives/1/
```

然后判断 `$finalBaseUrl` 是否为 `null`, 如果是 `null` ，`$pathInfo = $requestUri;` ，如果不是并且

```php
false===(pathInfo =substr(requestUri, strlen(finalBaseUrl))) 
```

`substr` 后的 `pathinfo` 是 `false` ，就是没有提取到子串的时候 `pathinfo` 是 `/`。 我们的场景下，成功提取到了，所以 `pathinfo` 就是

```php
/archives/1/
```

接下来，`pathinfo` 不为空的时候对 `iis` 请求的编码，这里不存在，就忽略了，如果pathinfo是空，就赋值 /。最后完整的 pathinfo就是

```php
'/' . ltrim(urldecode($pathInfo), '/')
```

去掉左侧的 `/` 在拼接一个 `/` 这个目的就是防止做的没有 `/`。 最后 pathinfo就是

```php
/archives/1/
```

最后把，`Typecho_Router::setPathInfo($pathInfo);`  设置到路由里面。

接下来就回到了路由的 `dispatch` 方法。首先获取一下 `pathinfo` 。
然后用配置里面的 `routeTable` 进行匹配，这个 `routeTable` 就是在数据库里面的配置，可以看 `option` 表里面的数据。

遍历 `routeTable` ，用路由里面的 `regex` 来匹配 `pathInfo` ，如果没有匹配到，就抛出 路由 没有匹配到的 404。

如果匹配到了，把 路由的 `key` 设置到 `current` 。

如果设置了路由的 `params` ，就把匹配到的参数跟 `params` 组合成数组。例如

```php
if (!empty($route['params'])) {    
    unset($matches[0]);    $params = array_combine($route['params'], $matches)
;}



http://typecho.test/index.php/archives/1/

array(6) { ["url"]=> string(24) "/archives/[cid:digital]/" ["widget"]=> string(14) "Widget_Archive" ["action"]=> string(6) "render" ["regx"]=> string(26) "|^/archives/([0-9]+)[/]?$|" ["format"]=> string(13) "/archives/%s/" ["params"]=> array(1) { [0]=> string(3) "cid" } } array(2) { [0]=> string(12) "/archives/1/" [1]=> string(1) "1" }
```

上面这种路径的话，就会把匹配到的 `1` 跟 `params` 组合，合成参数数组

```php
array(1) { ["cid"]=> string(1) "1" }
```

给后续的方法使用。

紧接着初始化，路由对应的组件，上面这个文章详情的例子就是 `Widget_Archive` ,
然后，判断是否设置了 路由的 `action` ， 如果设置了就执行这个方法

```php
if (isset($route['action'])) {    
    $widget->{$route['action']}();
}
```

最后调用，

```php
Typecho_Response::callback();
```

最后就返回了，如果执行相关方法出错了，就执行异常部分。

```php
if (404 == $e->getCode()) {
    Typecho_Widget::destory($route['widget']);    
    continue;
}
throw $e;
```

到这，整个路由就跑完了，大家可以多多的测试各种页面看看各种结果。

## 下期预告

下次我们就来具体的分析插件，这个好玩的东西，刚开始学 `php` 的时候，就觉得很高级，后来看过 `thinkphp3.2` 的源码的时候也在其他地方看到了类似的东西，这个做法真的很好玩。敬请期待。


---

> 作者:   
> URL: http://localhost:1313/posts/typecho-source-code-analysis-7/  

