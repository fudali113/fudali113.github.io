title: 排序-堆排序
date: 2017-03-25
tags: ["golang","排序","学习"]
categories:
  算法
---
## 简介 ##
堆排序是利用堆的特性实现的排序方式;
因为最小堆始终保证堆顶元素为最小值，且每此获取操作为logN;
顾堆排序为NlogN的时间复杂度


## 代码 ##
### go ###
```
package heap

import "github.com/fudali113/learn-basic/adt"

// HeapSort 堆排序
func HeapSort(array []int) {
	arrayLen := len(array)
	minHeap := adt.GetHeap(false)
	minHeap.Insert(array...)
	for i := 0; i < arrayLen; i++ {
		array[i], _ = minHeap.Delete()
	}
}
```

## 总结 ##
时间复杂度: NlogN
是否稳定: 不稳定
