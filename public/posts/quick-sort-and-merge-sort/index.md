# 快排和归并排序


## 代码
```golang
func quickSort(intArr []int) []int {
	if len(intArr) <= 1 {
		return intArr
	}

	val := intArr[0]
	var left []int
	var middle []int
	var right []int
	middle = append(middle, val)
	for i := 1; i < len(intArr); i++ {
		if intArr[i] > val {
			right = append(right, intArr[i])
		} else if intArr[i] < val {
			left = append(left, intArr[i])
		} else {
			middle = append(middle, intArr[i])
		}
	}
	intArr = append(left, append(middle, right...)...)

	//return intArr
	left = quickSort(left)
	right = quickSort(right)

	return append(left, append(middle, right...)...)
}

func mergeSort(intArr []int) []int {
	if len(intArr) < 2 {
		return intArr
	}

	left := mergeSort(intArr[0 : len(intArr) / 2])
	right := mergeSort(intArr[len(intArr) / 2:])
	var retult []int
	i := 0
	j := 0

	for i < len(left) && j < len(right) {
		if left[i] < right[j] {
			retult = append(retult, left[i])
			i++
		} else if left[i] > right[j] {
			retult = append(retult, right[j])
			j++
		} else {
			retult = append(retult, left[i])
			retult = append(retult, right[j])
			i++
			j++
		}
	}

	for ; i < len(left); i++ {
		retult = append(retult, left[i])
	}
	for ; j < len(right); j++ {
		retult = append(retult, right[j])
	}

	return retult
}
```

## 快排
先说说快排吧，中心思想就是找出一个元素当作中心，把小于中心的放到左边，把大于中心的放到右边，然后就是不断的重复这个操作，终止条件就是当只有一个元素的时候就停下来。

## 归并
这个跟快排的区别就是不着中心，而是直接变成两半，终止条件依然是只有一个元素就停止。紧着这遍历左右两个数组，用两个索引一起跑，小的插入结果集，然后移动索引。当两个数组跑完后，把剩余的结果插入数组即可。

## 总结
写出来是写出来了，但是呢，跟最优还是又差别的，空间会浪费，快排没有做到原地。自己在想想怎么优化吧。

---

> 作者: [M1racleHao](https://github.com/crazyhl)  
> URL: http://localhost:1313/posts/quick-sort-and-merge-sort/  

