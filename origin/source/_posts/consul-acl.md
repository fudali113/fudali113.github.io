title: consul权限控制
date: 2017-03-24
tags: ["consul"]
categories:
  微服务
---
## 前言 ##
最近公司准备实践微服务，在服务发现与注册方面选择了consul；
主要是基于易用与功能全面的考虑；

因为之前只是有点点初步的理解，并没有去深入的了解与实践，实践的时候发现consul的权限控制有一点不好理解；
所以只有跟我们老大(大波哥)一起去踩坑；

## 内容 ##

* 
    consul的acl控制默认是关闭的，需要在任意配置文件中添加`acl_master_token`配置项
    此时默认的控制策略是可写的(`acl_default_policy为write`)
    因为consul默认有匿名token(即`anonymous`),此token当用户acl开启且用户token为空时默认使用该token
    因为此时默认策略为全部可写，此时`anonymous`也拥有全部的权限包括acl的读写权限
    顾想要设置权限控制应该使`acl_default_policy为deny`，此时默认权限控制为全部否定。

* 
    当配置之后可能会发现打开ui依然可见自己的consul服务，这一点会让人很奇怪，这是consul在检测服务时默认不检测consul服务的权限：

    `consul/consul/acl.go#347` version：v0.7.5
    ```
    // allowService is used to determine if a service is accessible for an ACL.
    func (f *aclFilter) allowService(service string) bool {
        if service == "" || service == ConsulServiceID {
            return true
        }
        return f.acl.ServiceRead(service)
    }
    ```
    [github issue # 2816](https://github.com/hashicorp/consul/issues/2816)

    这个貌似可以使使用`enforceVersion8`进行控制，未实现


* 
    对于ui中多次输错token出现403页面，可以使用清除浏览器缓存或者换个浏览器进入ui界面，这应该做的一定的安全举措




