---
title: Spring Web MVC 源码核心源码解读
tags:
    - 源码
    - 原理
categories:
    - java
    - Spring MVC
---

先上一张图：

![SpringMVC核心架构图](http://wx4.sinaimg.cn/large/929194b4gy1fkd5w65prkj20vd0h7q5q.jpg)

`doDispatch`源码(去除了一些代码)：
```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;

    try {
        ModelAndView mv = null;
        Exception dispatchException = null;

        try {
            processedRequest = checkMultipart(request);

            // 通过 request 确定 处理器。  -> 从 handlerMappings 中去找，只要handlerMapping中的handler ，UrlPathHelper 通过 request
            mappedHandler = getHandler(processedRequest);
            if (mappedHandler == null || mappedHandler.getHandler() == null) {
                noHandlerFound(processedRequest, response);
                return;
            }

            // 通过 处理器类 确定 处理适配器
            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

            // 执行handlerExecutionChain中拦截器的前置处理方法，正序遍历 interceptors, i++
            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                return;
            }

            // 执行实际的 handle 方法
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

            // 执行handlerExecutionChan中拦截器的后置处理方法，倒序遍历 interceptors, i--
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        }
        catch (Exception ex) {
            dispatchException = ex;
        }
        // 处理结果： modelAndView解析 或 异常处理
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
}
```

#### 相关对象
* DispatcherServlet: 拦截Web请求到 Spring Web MVC，作为整个流程的调度员

* HandlerMapping: web请求映射到处理器

* HandlerAdapter：支持多种类型的处理器，最终一个请求只会转发到一个请求，接口以下方法
	- supports()：判断传入的handler:Object 是否是当前HandlerAdapter支持的
	- handle(): 将handler强转为当前HandlerAdapter所支持的类型，比如SimpleControllerHandlerAdapter将handler转为Controller，然后再将request传递给controller执行
	- getLastModified: 获取最后修改时间，用于lastModified缓存


* ViewResolver：将逻辑视图名称解析为具体视图

* Controller: 处理器，进行功能处理

* Interceptor: 拦截器，可以做一些指定的权限拦截

#### 用到的设计模式
1. 适配器模式
`HandlerAdapter`**处理适配器**接口：有`SimpleControllerHandlerAdapter`、`SimpleServletHandlerAdapter`等适配器实现，使用handler instanceof 分别适配到Controller、Serverlet来处理。

2. 模板方法模式
`View`接口定义有`render()`方法，在`AbstractView`中实现的`render()`就是模板方法，定义了整个执行步骤(有几个相同的步骤)，部分方法则由具体的子类去实现
```java
public void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
    Map<String, Object> mergedModel = createMergedOutputModel(model, request, response);
    prepareResponse(request, response); // 本类实现
    renderMergedOutputModel(mergedModel, getRequestToExpose(request), response); // 需要子类实现的抽象方法
}
```
