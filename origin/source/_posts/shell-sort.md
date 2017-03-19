title: 排序-希尔排序
date: 2017-03-15
tags: ["golang","排序","学习"]
categories:
  算法
---

## 前言 ##

个人认为希尔排序是对插入排序的一种优化，
他利用一定的算法来决定每趟遍历比较两个元素之前的距离，最后的比较距离为1(此时等同于插入排序)。
所以希尔排序也称为缩减增量排序
他相比插入排序所在的优势是，他可以一次交换两个较远距离的元素，而插入排序交换两个相聚n的元素位子需要n次交换。

因为他根据一个算法来决定每趟比较的距离，所以该算法的好坏也会在一定程度上决定希尔排序的性能


## 代码 ##
### go ###
```
package main

// 利用每次除以2的等比增量数列来决定每趟比较距离
func sort(array []int) {
	n := 0
	arrayLen := len(array)
	for gap := arrayLen / 2; gap > 0; gap /= 2 {
		for i := gap; i < arrayLen; i++ {
			iv := array[i]
			for j := i; j-gap > 0 && array[j-gap] > iv; j -= gap {
				n++
				array[j-gap], array[j] = iv, array[j-gap]
			}
		}
	}
	println("---------------", n)
}
func main() {
	array := []int{10, 14, 73, 25, 23, 13, 27, 94, 33, 39, 25, 59, 94, 65, 82, 45}
	sort(array)
	for _, v := range array {
		println(v)
	}
}
```
[golang code in github](https://github.com/fudali113/learn-basic/blob/master/sort/shell/shell.go)


## 总结 ##
希尔排序的性能在实践中的性能是完全可以接受的；
使用希尔增量是最坏情形运行时间复杂度为O(N^2)
对于好的增量序列，最坏时间复杂度还可以优化

编程的简单特点，使它成为对适度的大量输入数据进行排序的常用算法

最优运行时间：O(N)
最坏运行时间：O(N^2)
是否稳定：不稳定

