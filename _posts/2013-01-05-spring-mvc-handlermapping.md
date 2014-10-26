---
layout: post
title:  "springmvc-handlermapping解析"
date:   2013-01-05 22:37:00
categories: Note
---

## HanlerMaping
`HandlerMapping`是一个接口，接口只有一个方法：

```
HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;
```

它的作用是将request映射到springmvc中的HandlerExecutionChain。HandlerExecutionChain是一个具体的类，里面包含一个Handler和一系列Interceptors。接口的好处在于扩展性上，开发者可以定义新的映射规则。

再来看一下HanlderMapping的类层次结构图:
![icon](http://qiancun.github.io/pic/handlermapping_arc.jpg)

* __AbstractHandlerMapping:__ 为HandlerMapping提供了基础实现，提供了默认的order、默认的handler、默认的interceptors，还有两个后面在url匹配时都要用到的类：UrlPathHelper，AntPathMatcher。
* __AbstractUrlHandlerMapping:__ 提供了基于url-map规则的基础实现，同时支持了direct-match和Ant-Style match两种匹配方案。匹配规则是最长的是best match。
* __AbstractDetectingUrlHandlerMapping:__ 该类扩展了url配置地，可以配置controller上(ControllerBeanNameHandlerMapping，ControllerClassNameHandlerMapping)，也可以配置在xml——ean的name属性（BeanNameUrlHandlerMapping），还可以在annotion上（DefaultAnnotationHandlerMapping）
* __SimpleUrlHandlerMapping:__ url-map规则的最简化实现类。
* __AbstractHandlerMethodMapping:__ 3.1之后新增annotation-map规则的实现，替代了2.5的DefaultAnnotationHandlerMapping。

其实总结一下就是，handlermapping实现类负责将Httprequest对象匹配到springmvc中的handler，为保持最大的通用性，handler在springmvc其实只是个Object对象，具体的handler对象根据url匹配之后，可以是HttpRequestHandler、Controller、HandlerMethod等，也可以是其他任何实现类。

## HandlerExecutionChain

该类封装了一个Handler对象，一组intercetpors拦截器对象。DispatcherServlet是调用具体handler处理前后，都会将控制交给HandlerExecutionChain对象，HandlerExecutionChain内部又会将流程转给内部的HandlerInterceptor接口的具体实现类，以进行前置（applyPreHandle）、后置处理（applyPostHandle）。



