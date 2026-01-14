# Dijkstra算法 PHP 实现


20171119 利用周末优化了一下代码。这次完全默写了出来。感觉好多了

这个是根据算法图解自己写的，理解的不是很好，注释都在代码里面，过几天再默写一下，好好理解一下

```php
<?php

// 构造图
$graphs = [];
$graphs['start'] = [];
$graphs['start']['a'] = 5;
$graphs['start']['b'] = 2;

$graphs['a'] = [];
$graphs['a']['c'] = 4;
$graphs['a']['d'] = 2;

$graphs['b'] = [];
$graphs['b']['a'] = 8;
$graphs['b']['d'] = 7;

$graphs['c'] = [];
$graphs['c']['d'] = 6;
$graphs['c']['fin'] = 3;

$graphs['d'] = [];
$graphs['d']['fin'] = 1;

$graphs['fin'] = [];

// 处理过的节点
$processed = [];
// 所有节点的消费
$costs = [];
// 所有节点的父节点
$parents = [];

function getNodeCost($graphs, $nodeName = 'start', $isStartNode = false)
{
    if (empty($graphs)) {
        return [];
    }
    $costs = [];
    foreach ($graphs[$nodeName] as $name => $length) {
        if ($isStartNode) {
            $costs[$name] = $length;
        } else {
            $costs[$name] = INF;
        }
        $costs += getNodeCost($graphs, $name);
    }

    return $costs;
}

$costs = getNodeCost($graphs, 'start', true);

function getNodeParents($graphs, $nodeName = 'start', $isStartNode = false)
{
    if (empty($graphs)) {
        return [];
    }
    $parents = [];
    foreach ($graphs[$nodeName] as $name => $length) {
        if ($isStartNode) {
            $parents[$name] = $nodeName;
        } else {
            $parents[$name] = null;
        }
        $parents += getNodeParents($graphs, $name);
    }

    return $parents;
}

$parents = getNodeParents($graphs, 'start', true);

function findLowestCostNode($costs, $processed)
{
    $lowestNodeLength = INF;
    $lowestNOdeName = '';
    foreach ($costs as $name => $length) {
        if ($length < $lowestNodeLength && !in_array($name, $processed)) {
            $lowestNodeLength = $length;
            $lowestNOdeName = $name;
        }
    }

    return $lowestNOdeName;
}

$nodeName = findLowestCostNode($costs, $processed);
while ($nodeName) {
    $currentNodeCost = $costs[$nodeName];
    foreach ($graphs[$nodeName] as $name => $length) {
        if ($currentNodeCost + $length < $costs[$name]) {
            $costs[$name] = $currentNodeCost + $length;
            $parents[$name] = $nodeName;
        }
    }
    $processed[] = $nodeName;
    $nodeName = findLowestCostNode($costs, $processed);
}

var_dump($costs);
var_dump($parents);
```


---

> 作者:   
> URL: http://localhost:1313/posts/dijkstra-algorithm-php-implementation/  

