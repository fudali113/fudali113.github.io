title: 排序-归并排序
date: 2017-03-17
tags: ["golang","排序","学习"]
categories:
  算法
---

## 前言 ## 
归并排序体现了递归分治的策略
思想是将数组拆分为两段，在对两段进行想通操作，直到数组元素为1，在对数组进行排序合并

## 代码 ##
### go ###
```
package merge

// MergeSort 归并排序
func MergeSort(array []int) []int {
	middle := len(array) / 2
	if middle < 1 {
		return array
	}
	return merge(MergeSort(array[:middle]), MergeSort(array[middle:]))
}

// merge 合并两个排好序的数组唯一组合两个元素并排好序的数组
func merge(a1 []int, a2 []int) []int {
	a1Index := 0
	a2Index := 0
	res := make([]int, 0, len(a1)+len(a2))
	for {
		if a1[a1Index] <= a2[a2Index] {
			res = append(res, a1[a1Index])
			if a1Index == len(a1)-1 {
				res = append(res, a2[a2Index:]...)
				break
			}
			a1Index++
		} else {
			res = append(res, a2[a2Index])
			if a2Index == len(a1)-1 {
				res = append(res, a1[a1Index:]...)
				break
			}
			a2Index++
		}
	}
	return res
}

```
[golang code on github](https://github.com/fudali113/learn-basic/blob/master/sort/merge/merge.go)

## 总结 ##

虽然归并排序的运行时间是O(NlogN)，但是他合并两个已排序数组到一个附加数组会更占用内存。

与其他O(NlogN)的流行算法相比，归并排序是比较元素最少的，且运行时间严重依赖于比较元素和在数组(已经临时数组)中移动元素的开销(理论上使用更少内存的算法是可能的，但所得到的算法是复杂的和不实际的)。

这些开销是于语言相关的(java中泛型数组中的元素是引用类型的，顾移动元素开销不大，所以归并排序是java泛型数组的默认排序方式。

快速排序作为基础类型的排序方式，java基础类型是值传递的，因为比较与数据移动的开销是类似的，快排使用少得多的数据移动足以补偿那些附加的比较而且还有盈余)。

在go中也存在相同的问题，所以在不同的输入数据选择不同的排序方式再能得到最好的性能。

平均运行时间：O(NlogN)
最坏运行时间：O(NlogN)
是否稳定：稳定

