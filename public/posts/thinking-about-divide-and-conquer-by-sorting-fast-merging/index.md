# 通过快排和归并排序思考分治法


话说以前看过很多算法相关的书，大部分讲的都很模糊，然后直接上代码，当时看似理解了，可是时间一长就忘光光了，以前看书学习的都是如何去做，而自己也没有深入的思考过为何这么做，单纯的就是背下来那种。当学习到一定程度之后，发现基础知识还是非常重要的。

今天要说的就是分治法，什么是分治法，就是分而治之，先把把大的问题分解为小一些的问题。逐步把小问题解决了，在把这些解决的小问题合并了就把大问题解决了，这就是我所理解的分治法。

我们在看算法排序相关的时候，都会有并归排序和快速排序，这两个方法都是用分治法解决的，只不过解决的方案不一样。下面是我用 php 实现的代码，大家可以先看下，然后我会在代码后面对两种方式做说明。


```php
/**
 * 归并排序
 * @param $arr
 * @return array
 */
function mergeSort($arr)
{
    $arrCount = count($arr);
    if ($arrCount < 2) {
        return $arr;
    }
    $mid = ceil($arrCount / 2); // 向上取整一下
    $leftArr = array_slice($arr, 0, $mid);
    $rightArr = array_slice($arr, $mid, $arrCount - $mid);
    return mergeArr(mergeSort($leftArr), mergeSort($rightArr));
}

/**
 * 最终合并数组的方法
 * @param $leftArr
 * @param $rightArr
 * @return array
 */
function mergeArr($leftArr, $rightArr)
{
    $i = 0;
    $j = 0;
    $returnArr = [];
    while ($i < count($leftArr) && $j < count($rightArr)) {
        if ($leftArr[$i] < $rightArr[$j]) {
            $returnArr[] = $leftArr[$i];
            $i++;
        } else if ($leftArr[$i] > $rightArr[$j]) {
            $returnArr[] = $rightArr[$j];
            $j++;
        } else {
            $returnArr[] = $leftArr[$i];
            $returnArr[] = $rightArr[$j];
            $i++;
            $j++;
        }
    }
    for ($temp = $i; $temp < count($leftArr); $temp++) {
        $returnArr[] = $leftArr[$temp];
    }
    for ($temp = $j; $temp < count($rightArr); $temp++) {
        $returnArr[] = $rightArr[$temp];
    }
    return $returnArr;
}

/**
 * 快速排序
 * @param $arr
 * @return array
 */
function quickSort($arr)
{
    if (count($arr) < 2) {
        return $arr;
    }
    $flag = $arr[0];
    $lessArr = [];
    $largeArr = [];
    $flagArr = [];
    $flagArr[] = $flag;

    for ($i = 1; $i < count($arr); $i++) {
        if ($arr[$i] < $flag) {
            $lessArr[] = $arr[$i];
        } else if ($arr[$i] > $flag) {
            $largeArr[] = $arr[$i];
        } else {
            $flagArr[] = $arr[$i];
        }
    }

    return array_merge(quickSort($lessArr), $flagArr, quickSort($largeArr));
}
```


先来说一下快排，因为快排的代码比较简短。快排的解决方案是，找出一个数字，一般都是采用数组的第一个数字作为标识数字，然后把比这个数字小的放到一个数组里面，在把比这个数字大的放到一个数组里面。这样，我们就已经确定好中间的数字了，然后我们在采用递归的方式分别处理比标志位数字小的数组和比标志位大的数组。 但是在处理递归的时候我们要注意一个点，就是何时终止递归，让其返回，这个点就是当数组的元素个数小于2的时候，因为这个时候已经不需要进行比较了。

好吧，简单吧，其实我们可以用笔在纸上画一画模拟一下流程，就会更加的清晰。

再来说说归并，归并理解起来要比快排麻烦一些，因为并没有那么直观，不直观的原因就是最后一步了，我们从头开始吧。归并排序思想就是把数组折半拆分，然后拆成一个个的小单元，那么多小是小呢，就是上面快排说的终止点，这个终止点就是当数组元素小于2的时候，因为这个时候已经没有办法继续拆分了。拆分后就是合并，那么如何合并呢，这个合并就是我个人认为复杂地方，合并的是偶我们比较每个数组的值，具体的方法看看代码吧，就是利用游标。找出小的来，然后放到结果数组中，这样一次次的合并之后每个小数组就都是有序的，然后在一次次的合并回去，这个归并我觉得看看就好了，实际上就用快排好啦。起码我个人没用过归并在实际项目中。


好吧上面简单的说了一下两个排序的方式，其实我最想说的就是分治法，其实在我刚开始学习算法的时候就是知道了哦，这个排序是利用分治法，那么如何分而治之我却没有深入的了解，在最近看一个老外的算法书的时候忽然灵光一闪，想把这两个方式比较一下，也就有了今天的文章。其实啊，现在再让我说什么事分治法，我也会说把大问题分解成小问题，小问题解决后，合并到大问题上面也就解决了。说了半天还是废话把，再想想上面两个排序拆分的方式，差别就是拆分方式不同，这其实才是我想说的，观看问题的角度不同，解决方案也不同，多看看别人的代码，多思考自己的方案，多学习。才能更灵活，我个人觉得学习算法不是让我们死记硬背应付面试的，而是拓宽了我们的思路，在合适的时候选用合适的方法罢了。


加油吧，新的算法书看了44%了，这周争取搞定，虽然之后200页。


---

> 作者:   
> URL: http://localhost:1313/posts/thinking-about-divide-and-conquer-by-sorting-fast-merging/  

