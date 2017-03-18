title: doob入门
date: 2016-01-13
tags: ["golang","doob","web"]
categories:
  doob文档
---
## 快速开始 ##

### 下载依赖 ###
```
go get github.com/fudali113/doob
```

### 创建main函数文件demo.go ###
```
package main

import "github.com/fudali113/doob"

func main() {
	doob.Start(8888)
}

func init(){
  router := doob.DefaultRouter()
  router.Get("/test", func (ctx *doob.Context) interface{} {
    return "test"
  })
}
```

### 编译运行 ###
```
go run demo.go
```

浏览效果
打开浏览器并访问 [http://localhost:8888/test](http://localhost:8888/test)
