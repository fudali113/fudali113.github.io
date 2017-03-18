title: 排序-插入排序
date: 2017-03-13
tags: ["golang","排序","学习"]
categories:
  算法
---

## 前言 ##
  -
  今天感觉受到了一些打击，感觉自己基础知识过于缺乏，顾决定从今日起，有空就写一篇博客，加强自己在算法与数据结构、网络、设计模式方面的知识
  -
  今天就想来理解一下排序算法中最简单的插入排序：
  插入排序思路为顺序遍历数组，并在遍历中从顺序遍历到的元素倒序遍历回
  倒序遍历是判断相邻两个元素是否符合排序规则
  如果符合则停止倒序遍历
  如果不符合则交换相邻两个元素位置并继续倒序遍历
## 代码实现 ##
   ### go ####
```
ppackage main

func sort(array []int) {
	n := 0
	// 从1开始遍历,因为0前面没有数值
	for i := 1; i < len(array); i++ {
		// 在此处放置与在 代码1 处放置效果一样
		// 因为若 jv > iv 的话 ， j 与 j-1 调换位置， 下一次循环时 array[j] 依然是当初的 iv
		iv := array[i]
		for j := i; j >= 0 && array[j-1] >= iv; j-- {
			// 代码1  iv := array[j]

			// 如果 jv > iv , 调换两个元素之间的位置
			n++
			array[j], array[j-1] = array[j-1], iv

		}
	}
	println("----------------", n)
}

func main() {
	array := []int{10, 14, 73, 25, 23, 13, 27, 94, 33, 39, 25, 59, 94, 65, 82, 45}
	sort(array)
	for _, v := range array {
		println(v)
	}
}

```
[golang code in github](https://github.com/fudali113/learn-basic/blob/master/sort/insertion/insertion.go)

  ### java ###
  ```
public class Insertion {

    public static void main(String[] args) {
        Integer[] oo = new Integer[]{34, 8, 64, 51, 32, 21};
	    sort(oo);
        for (Integer var : oo) {
            System.out.println(var);
        }
    }

    public static <T> void sort(T[] array) {
        for (int i = 1; i< array.length; i++) {
            T it = array[i];
            for (int j = i; j > 0; j--) {
                T jt = array[j - 1];
                if (compare(jt, it) < 0) {
                    break;
                }else {
                    array[j] = jt;
                    array[j - 1] = it;
                }
            }
        }
    }

    public static <T> int compare(T o1, T o2) {
        return (Integer)o1 - (Integer)o2;
    }

}
  ```
[java code in github](https://github.com/fudali113/learn-basic/blob/master/sort/insertion/Insertion.java)



## 总结 ##
从原理和显示方式可以看出，当数组本就按排序规则排好序时，时间复杂度最小为 O(N)；当数组为倒序时未最坏情况，测试时间复杂度为O(N^2)
当数组中根据排序规则倒序的元素对越多，时间复杂度越大，当数组根据排序规则完全倒序时，时间复杂度最大为 2+3+4+...+N = O(N)
