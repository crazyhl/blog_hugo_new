# 189.旋转数组





## 题目描述

![](https://www.cimple.ink/static/images/6efdb97d0abca1ac56563637bfdd3b65.png)

## 分析

其实就是移动数组，无非就是怎么移动更快速的问题，说明中提到想出更多的解决方案。于是我就想到了下面几种方法。而且要求使用空间复杂度的O(1)的原地算法。空间复杂度这边我平时接触的不多，所以简单来想就是用更少的变量就好了。于是我想到了下面的几种算法。

1. 找出位置裁剪数组，再次组合
2. 两两交换
3. 依然是两两交换，优化点是批量移动那块一次处理

## 解法

### 解法1

``` golang
func rotate(nums []int, k int)  {
    position := len(nums) - k % len(nums)
    if position <= 0 {
        return
    }
		leftNums := nums[position:]
		rightNums := nums[:position]
    temps := append(leftNums, rightNums...)
    for i:=0; i < len(nums); i++ {
        nums[i] = temps[i]
    }
    return
}
```

这块有几个坑，一个是测试用例，出现了数组移动步数大于数组的长度情况，这时候，其实就等于是转了一圈在多余几步。那么我们在裁剪的时候只要裁剪数组长度的余数就可以了。还有一个是如果位置是小于等于 0 就不用动，因为是右转所以小雨 0 也没有考虑。另一个坑是最后那个赋值，还得覆盖到源数组，一开始可能是我没读清楚题目，所以失败了两次。

所以就带来了问题，那么用个临时数组，那空间占用不就大了么？所以想到了解法二

![](https://www.cimple.ink/static/images/6dbefc58f8690aae152cddb5b7efcccb.png)

### 解法2

```golang
func rotate(nums []int, k int) {
	middlePosition := len(nums) - k%len(nums)
	if middlePosition <= 0 {
		return
	}
	flag := 0
	for i := middlePosition; i < len(nums); i++ {
		// 拿到起始位
		tmp := nums[i]
		// 向后移动一批
		for j := i - 1; j >= i - middlePosition; j-- {
			nums[j+1] = nums[j]
		}
		nums[flag] = tmp
		flag++
	}
	return
}
```

这里我们没有采用外部数组了，只用了两个变量来进行交换，主要比较麻烦的是批量的向后移动。这块就是把数字移动到前面，然后再把前面的数字，集中往后移动一次。

![](https://www.cimple.ink/static/images/23b8c988d9468cad0d85ced48b303d47.png)

可以看到这边内存减少使用了，但是时间翻了二十倍，耗时在哪呢，就在我们批量移动那一块。所以是不是可以考虑不要每次移动，可以在最后一次移动呢？

### 解法3

```golang
func rotate(nums []int, k int) {
   middlePosition := len(nums) - k%len(nums)
   if middlePosition <= 0 {
      return
   }
   flag := 0
   for i := middlePosition; i < len(nums); i++ {
      tmp := nums[flag]
      nums[flag] = nums[i]
      nums[i] = tmp
      flag++
   }
   fmt.Println(nums)

   // 换后部分
   // 可以根据规律进行移动
   moveCount := 0
   k = k % len(nums)
   moveCount = middlePosition - k
   for moveCount < 0 {
      moveCount = middlePosition + moveCount
   }

   fmt.Println(middlePosition)
   fmt.Println(moveCount)
   for i := moveCount; i > 0; i-- {
      for j := (k + i - 1) % len(nums); j < len(nums)-(moveCount-i)-1; j++ {
         tmp := nums[j]
         nums[j] = nums[j+1]
         nums[j+1] = tmp
      }
   }

   return
}
```

![](https://www.cimple.ink/static/images/5ac4e4a12fabd9b6c5fc1cb8a21421aa.png)

可以看到内存减少了0.2 时间增大了6倍，这个结果我个人是可以接受的。这边优化到的时间就是移动次数大大减少了。但是还可能会有更优的方法。不过在这边我们就不深入了。到这我觉得差不多够了。



## 总结

如果要更少内存的话，我唯一想到的就是依次循环了，不过时间会大大增长，循环的次数更多。

综合看来我更喜欢第一种解法，简单明了，就是占用大一点。



好久没写东西了，忽然发现没事解题也是个不错的选择，以后没有东西写就解题，毕竟简单的题那么多，可以做很久，哈哈





---

> 作者: [M1racleHao](https://github.com/crazyhl)  
> URL: http://localhost:1313/posts/leetcode-189/  

