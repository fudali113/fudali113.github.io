title: 初识Service Mesh
date: 2017-12-28
tags: ["Service Mesh"]
categories:
  微服务
  Service Mesh
---
## 简介
[Service Mesh的诞生](http://www.servicemesh.cn/?/article/21)

软件应用从来都是遵循高内聚低耦合的原则；从互联网刚诞生时的七层网络协议栈的定义，到今天service mesh的诞生；对于微服务，我们更加希望我们的服务实体能够更加的专注于业务逻辑的实现，其他的事我们应该交给更专业的工具去做；
 ## 格局
 [Service Mesh 时代的选边与站队](http://www.servicemesh.cn/?/article/25)
 
 目前成熟的已经在生产环境进过验证的有Bouyant(最早提出service mesh概念，由twitter两位工程师创建)公司的[linkerd](https://github.com/linkerd/linkerd)与lyft的[envoy](https://github.com/envoyproxy/envoy),但是目前该两个相对于与kubernets的集成方面需要依赖于 [istio](https://github.com/istio/istio)(由google，IBM，lyft成立相关研发组研发) 项目(istio由数据平面和控制平面组成，前面提到的两位在istio中都相当于充当着数据平面的位置，由于istio提供灵活的建构，[数据平面与控制平面](http://www.servicemesh.cn/?/article/24)是通过特定api进行交流的，顾数据平面对istio项目是可以替换的)；由于istio的推出，且Bouyant公司就是为service mesh而生，自己必须要在这个领域占有发言权，顾推出了[conduit](https://github.com/runconduit/conduit)与istio竞争争取在该领域获取发言权。
 * istio 目前0.4版本，还不能在生成环境使用
 * conduit 17年12月发布0.1，也还不能在生成环境使用
 
## 对于我们
### 概念 
由于我们使用了spring cloud技术栈来构建我们的微服务，对于我们的对比其实就是spring cloud 提供的sdk集成方式 与 service mesh 提供的基础设施与具体业务分离的对比。

对于之前我们的分析，其实可以发现，互联网的发展，最开始的时候都是我们在应用之中做很多的事情让我们的引用支持各种功能来实现我们的业务；最后随着技术的成熟，基础的设施与技术与实际业务独立开来，制定相关的协议和应用相关的工具来实现一样的功能。

由此我们可以看出，service mesh才应该是微服务的最终形态。

### 实施
* 因为service mesh的前提是我们最好能够在kubernets里面运行我们的应用，所以需要有kubernets平台
* 对于我们应用集成的spring cloud 相关的组件，我们只需要删除相关的依赖，有service mesh去管理我们服务间的调用，我们连我们之前的feign api都是可以复用；所以，我们在平台完整的情况下，迁移的成本非常低

### 优点
* 底层基础设施与业务代码解耦，降低我们开发应用的复杂性，我们可以使用更适合的语言更适合的框架来构建我们应用而不用去管理那一坨复杂的spring cloud依赖
* service mesh还可以给我们带来与应用解耦的流量控制，访问控制，转发策略(结合kunernets在做发布策略的时候应该非常爽)。。。
