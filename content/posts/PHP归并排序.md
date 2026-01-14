---
title: PHP归并排序
tags: [排序, 归并排序]
categories: [PHP, 算法]
slug: php-merge-sort
date: 2016-09-25 22:23:00
updated: 2016-09-25 22:23:00
toc: false
---


```php
<?php

function sortArr($arr) {
    if (count($arr) < 2) {
        return $arr;
    }
    $mid = count($arr) / 2;
    $arr1 = array_slice($arr, 0, $mid);
    $arr2 = array_slice($arr, $mid, count($arr));
    $arr1 = sortArr($arr1);
    $arr2 = sortArr($arr2);

    return mergeArr($arr1, $arr2);
}

function mergeArr($arr1, $arr2) {

    if (!is_array($arr1)) {
        $arr1[] = $arr1;
    }
    if (!is_array($arr2)) {
        $arr2[] = $arr2;
    }

    $i =0;
    $j = 0;
    $arr1Length = count($arr1);
    $arr2Length = count($arr2);
    $returnArr = [];
    while($i < $arr1Length && $j < $arr2Length) {
        if($arr1[$i] > $arr2[$j]) {
            $returnArr[] = $arr2[$j];
            $j++;
        } else {
            $returnArr[] = $arr1[$i];
            $i++;
        }
    }
    for($tmp = $i; $tmp < $arr1Length; $tmp++) {
        $returnArr[] = $arr1[$tmp];
    }
    for($tmp = $j; $tmp < $arr2Length; $tmp++) {
        $returnArr[] = $arr2[$tmp];
    }
    return $returnArr;
}

$arr = [1,3,2,5,7,9,3,1];


$sortableArr = sortArr($arr);

foreach ($sortableArr as $a) {
    print_r($a . "\n");
}


```

说实话，算法这东西，挺好玩的，以前还搞过快排，但是忘却了，今天这个归并排序应该很难忘记了，毕竟理解起来要比快排好得多，但是快排回来后面补充