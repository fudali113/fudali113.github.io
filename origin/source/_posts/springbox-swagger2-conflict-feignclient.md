title: springbox-swagger2使用
date: 2017-04-20
tags: ["swagger","spring cloud"]
categories:
  微服务
---
## 与FeignClient冲突 ##
- Q:
    使用springbox-swagger2 2.2.2版本进行注释api生成文档，但是当basePackage路径下存在feignClient注释的类时，将会报NullPointerException.出现这种情况是因为2.2.2版本没有对feignClient初始化时调用refreshContext作兼容。
- A: 
    升级版本到最新版，在2.5.0中改问题被彻底解决;[springbox-swagger2 github issue 1074](https://github.com/springfox/springfox/issues/1074);
