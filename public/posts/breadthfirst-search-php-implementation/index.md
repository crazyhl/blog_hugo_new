# 广度优先搜索 PHP 实现


直接上代码了，注释都在代码里面了。

```php
<?php
/**
 * 广度搜索
 *
 * 你的朋友关系，以及朋友的朋友的关系，查看你的朋友或者朋友的朋友是不是包含 m 结尾的名字
 */


// 需要检索的数组
$graph = [];
$graph['you'] = ['alice', 'bob', 'claire'];
$graph['bob'] = ['anuj', 'peggy'];
$graph['alice'] = ['peggy'];
$graph['claire'] = ['thom', 'jonny'];
$graph['anuj'] = [];
$graph['peggy'] = [];
$graph['thom'] = [];
$graph['jonny'] = [];

// 搜索过的数组
$searchedItem = [];
// 待检索的数组
$waitSearchArray = [];
// 将第一层的关系加入到等待搜索的数组
$waitSearchArray = array_merge($waitSearchArray, $graph['you']);

// 将结果元素赋值为 false
$resultName = false;

//需要查找的字符
$findChar = 'z';

// 如果等待搜索的数组不为空就循环查找
while ($waitSearchArray) {
    // 从队列头部弹出一个元素
    $name = array_shift($waitSearchArray);
    // 如果待检查的元素在已经搜索过的数组中，就跳过，这个是用来防止循环检查的
    if (in_array($name, $searchedItem)) {
        continue;
    }
    // 获取最后一个字符
    $lastChar = substr($name, strlen($name) - 1, 1);
    // 如果最后一个字符是 m，就说明找到了，把结果赋值，然后跳出循环
    if ($lastChar == $findChar) {
        $resultName = $name;
        break;
    }

    // 到这里了说明没有找到，那么把这个人的名字，放入到已经搜索过的数组中
    $searchedItem[] = $name;
    // 然后再把这个名字的朋友关系加入到待搜索的数组中
    $waitSearchArray = array_merge($waitSearchArray, $graph[$name]);
}


var_dump($resultName);
```


---

> 作者:   
> URL: http://localhost:1313/posts/breadthfirst-search-php-implementation/  

