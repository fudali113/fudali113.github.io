title: spring cloud 微服务异常记录与报警
date: 2017-11-02
tags: ["spring cloud"]
categories:
  微服务
  监控
---
## 前言 ##
当我们的应用在线上正常运转起来了，在正常情况下我们不需要再担心任何的事情，但是bug总是不可避免的会出现；此时我们就需要一种相关的机制能够发现我们系统中的异常并通知到相关人员，不然等到用户进行反馈时才能知道发生了bug是很影响用户体验的也是不可控的，这两者都是不可接受的。

## 介绍 ##
我所在的团队目前正在使用spring cloud相关套件进行微服务的开发，所以我的介绍与实践也是在该技术栈下进行，同时可能会使用到elasticsearch。
我们使用spring mvc来进行业务开发，feign来做restful接口远程调用框架，zuul作为服务网关来对外开放接口。这个技术栈在使用spring cloud进行开发的团队是非常常见的。

## 思路 ##
因为最开始我们就有在网关记录统一的访问日志，并使用filebeat将其同步到elasticsearch以方便后期数据的查询与分析，但是只是这样子是不够的；我们想要直接能够从日志中能够判断是否发生了毕竟明确的异常，我们需要有一个点或者相关的阈值去确定什么情况是异常，可能会有bug，需要进行相关的操作去进行报警。所以我们需要在日志记录上面做一些文章，让我们记录的日志能够有足够有力和准确的信息让我们去判断是不是异常，然后去触发一系列的操作(报警等等...)。
经过一定的分析之后我认为异常是一个很好的判断是否有bug的点，因为没有异常不一定没有bug，但是有没有被捕获异常的请求一定是有bug的；所以以此作为切入点，深入思考。

## 实现 ##
### 开发中的定制 ###
首先我们的基础架构指定了统一的错误码来对外进行提示，同时在业务层以抛出异常来对外进行提示。
我们将它定义为：
```
public class DomainServerException extends Exception {
    // 平台定义错误码
    private int code;
    // http status
    private int status;
    // 具体错误信息，面向开发者的提示， Exception的message用于面向用户的提示
    private int error;
    // 相关异常的堆栈信息
    private int stack;

    public DomainServiceException(int code, String message, Throwable throwable) {
        super(message, throwable);
        this.code = code;
        if (isServerError())
            this.stack = ExceptionUtils.getStackTrace(throwable);
    }
}
```
其中的stack信息就是为了我们进行异常追踪而添加的字段，当`http status为5**`或者`平台定义错误码为服务器异常`的时候会加载相关异常堆栈信息并设值到stack字段。具体是在`DomainServiceException`构造函数进行或者在`spring ErrorController`中进行相关设值操作(因为我们使用异常来抛出错误码，所以我们对`spring MVC`默认的`ErrorController`进行了定制)。
我们`ErrorController`的返回类型定义为
```
public class HttpErrorResponse implements Serializable {
    private Date timestamp;
    private Integer status;
    private String error;
    private String message;         
    private String stack;           // 异常堆栈  方便记录同时在前后端调试的时候信息也更加丰富
    private String exception;       // 异常类型
    private String path;            // 错误请求路径
    private Integer errorCode;      // 平台定义错误码
}
```
对于错误的情况我们抛出`DomainServerException`或者其他未捕获异常，`DomainServerException`默认为我们的业务错误，同时也可作为异常错误，但是我们在进行异常错误处理为`DomainServerException`是会将上层异常堆栈传入构造函数生成`DomainServerException`异常对象(`注意:此模式下一定不要去处理你不知道该怎么处理的异常，如果你处理不了就一直往外抛，ErroController能够正确的处理并记录他然后供报警使用`)。
此时我们抛出的`HttpErrorResponse`可能会被两个地方用到:
1. 服务之间的调用
2. zuul转发来的请求
* 对于第一种情况，因为我们使用`feign`来进行服务间远程调用，我们重写了`ErrorDecoder`来进行`HttpErrorResponse`与`DomainServerException`或者`DomainServerException的子类`(通过exception字段来进行类型判断)并向外抛出。级联调用一次类推，最终都会到网关一层进行处理。所以第一种情况最终也会变为第二种情况。

* 对于第二种情况我们，因为我们有错误码的定义并且在正常情况下我们也会返回错误，但是正常的结果却是没有错误码的，所以我们在`zuul`实现了一个`type为“post”的filter来对返回值进行格式化，同时也对老的平台与新的平台进行输出格式化`。在这里面我们判断服务返回的内容是否有异常并进行相关的记录(存入相关信息到`RequestContext`)，最后在统一日志记录Filter(包含正常filter和zuul 异常filter)进行统一记录相关信息。同时因为我们在zuul也写了一些胶水接口，所以我们在Zuul继承了普通服务的ErroController实现了ZuulErrorController同时也会记录异常信息。

### 日志的储存和报警 ###
记录怎么样的日志已经确定了，我们使用filebeat来讲日志数据传输到elasticsearch中。现在我们elasticsearch中就有错误码和stack的信息了，很明显，stack信息是很明显的错误信息，紫瑶该字段一出现就表示我们的代码又问题，我们可以根据这个很好的去报警。对于错误码信息，可能会比较复杂，我们需要判断他在某些情况下的一个阈值，当我们在某种情况下相关错误码超过了该阈值就报警(目前该块的应用还需要多思考)
对于elasticsearch查询报警的工具有[elastalert](https://github.com/fudali113/esalert),但是我对于该工具不是很感冒，同时我也疲于应对python部署那些复杂的依赖，我正在使用golang开发一款功能更简洁，学习成本更低的工具。如果在内部试用的还行应该会进行开源。

`未完待续`