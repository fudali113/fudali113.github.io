title: 排序-归并排序
date: 2017-03-17
tags: ["golang","排序","学习"]
categories:
  算法
---

## 代码 ##
```
package merge

func mergeSort(array []int) []int {
	middle := len(array) / 2
	if middle == 1 {
		return array
	}
	return merge(mergeSort(array[:middle]), mergeSort(array[middle:]))
}

func merge(a1 []int, a2 []int) []int {
	a1Index := 0
	a2Index := 0
	resIndex := 0
	resLen := len(a1) + len(a2)
	res := make([]int, resLen)
	for i := 0; i < resLen; i++ {

		if a1Index == len(a1)-1 {
			for ; a2Index < len(a2); a2Index++ {
				res[resIndex] = a2[a2Index]
				resIndex++
			}
			break
		}

		if a2Index == len(a1)-1 {
			for ; a1Index < len(a1); a1Index++ {
				res[resIndex] = a1[a1Index]
				resIndex++
			}
			break
		}

		if a1[a1Index] < a2[a2Index] {
			res[resIndex] = a1[a1Index]
			a1Index++
		} else {
			res[resIndex] = a2[a2Index]
			a2Index++
		}
		resIndex++
	}
	return res
}
```
[golang code in github](https://github.com/fudali113/learn-basic/blob/master/sort/merge/merge.go)
待改善
