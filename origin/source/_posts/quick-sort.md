title: 排序-快速排序
date: 2017-03-18
tags: ["golang","排序","学习"]
categories:
  算法
---
## 前言 ##
快速排序是实践中一种快速的排序方式；他的平均运行时间是O(NlogN)，最坏运行时间是O(N^2)，但稍加努力可使最坏情形极难出现。
并且可通过与堆排序的结果，可以使几乎所有的输入都能达到快速排序的快速运行时间。

## 简介 ##
快速排序可划分为简单的四个步骤(设输入数组为array)：
 * 如果array中元素是0或者1，直接返回
 * 取array中任一元素pivot，称之为枢纽元(pivot)
 * 将array中不包含pivot按与pivot的比较大小划分成连个不相交的集合
 * 递归对划分的两个合集进行该四步走并获得返回值与pivot合并成返回数组

## 代码 ##
### go ###
```
package quick

// QuickSort 快速排序
// 是否稳定?
// 不稳定 可以试着分析下[5,1,2,2,3,4,5]
func QuickSort(array []int) {
	quickSort(array, 0, len(array))
}

// quickSort 快速排序私有递归方法
func quickSort(array []int, left, right int) {
	if right-left < 2 {
		return
	}
	medianIndex := median3(array, left, right)
	nowMedianIndex := swap(array, left, right, medianIndex)
	quickSort(array, left, nowMedianIndex)
	quickSort(array, nowMedianIndex+1, right)
}

// swap 按最开始的中位置下标将数组分为比中位置大和比中位值小的两个数组
// left right 遵循右开左闭
// return 划分后的中位值下标
func swap(array []int, left, right, medianIndex int) (nowMedianIndex int) {
	median := array[medianIndex]
	array[medianIndex], array[right-1] = array[right-1], median
	leftPoint := left
	rightPoint := right - 2
	leftStop := false
	rightStop := false
	for {
		// 如果左滑动指针停止，不做任何操作，等待右滑动指针也停止交换
		if !leftStop {
			if array[leftPoint] <= median {
				leftPoint++
			} else {
				leftStop = true
			}
		}

		if !rightStop {
			if array[rightPoint] >= median {
				rightPoint--
			} else {
				rightStop = true
			}
		}

		//当左滑动指针与右滑动指针相交后，左滑动指针位置一定处于大于等于中位值的位置
		if leftPoint > rightPoint {
			array[leftPoint], array[right-1] = array[right-1], array[leftPoint]
			nowMedianIndex = leftPoint
			return
		}

		// 如果都停止，交换位置并重启滑动
		if leftStop && rightStop {
			array[leftPoint], array[rightPoint] = array[rightPoint], array[leftPoint]
			leftStop, rightStop = false, false
		}

	}
}

// median3 三数中值分割法
// 比较数组开头，结尾和中间位置的值，放回处于中间的值得数组下标
// return 中位值数组下标
func median3(array []int, left, right int) int {
	index := (right + left - 1) / 2
	start := array[left]
	median := array[index]
	end := array[right-1]
	if median > start {
		if median <= end {
			return index
		} else if end >= start {
			return right - 1
		} else {
			return left
		}
	} else {
		if median >= end {
			return index
		} else if start >= end {
			return right - 1
		} else {
			return left
		}
	}
}


```
[golang code on github](https://github.com/fudali113/learn-basic/blob/master/sort/quick/quick.go)
## 总结 ##

平均运行时间：O(NlogN)
最坏运行时间：O(N^2)
是否稳定：不稳定，试着分析[5,1,2,2,3,4,5]即可找到
```
选取枢纽元为2，是枢纽元到数组末尾
5   1   2   5   3   4   2
i                   j

进行第一次交换的地方
5   1   2   5   3   4   2
i   j

第一次交换过后
1   5   2   5   3   4   2
j   i

此时 i > j 将枢纽元从末尾与i位置元素调换
1   2   2   5   3   4   5

此时作为枢纽元的2从最开始的index=3变成了index=1
跑到了与之相等的index为2的元素的前面，值为5的元素也发生了相似的状况
顾快速排序为不稳定的排序
```