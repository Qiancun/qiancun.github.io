---
layout: post
title:  "springmvc-handlerAdapter解析"
date:   2013-01-14 22:37:00
description: HandlerAdapter源码分析
categories: spring
---

##HandlerAdapter作用

handlerAdapter是springmvc另一个重要的接口，它负责封装了handler的调用逻辑，对外有三个方法API：

```
//判断这个adapter是否可以处理这类handler
boolean supports(Object handler);
//封装handler处理request的逻辑
ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
long getLastModified(HttpServletRequest request, Object handler);
```

## HandlerAdapter层次结构

HandlerAdapter的类层次结构如下：<br>
![image](http://qiancun.github.io/images/HandlerAdapter.png)

* __SimpleControllerHandlerAdapter :__ 将handler转型Controller，并将流程控制转到handler的handleRequest方法中。
* __HttpRequestHandlerAdapter :__ 将handler转型HttpRequestHandler，并将流程控制交给HttpRequestHandler的handleRequest方法
* __SimpleServletHandlerAdapter :__ 同理，cast转型为Servlet，流程交给service方法
* __AnnotationMethodHandlerAdapter :__ 2.5新增了@Requestmapping，该类就是支持为此而生。
* __AbstractHandlerMethodAdapter :__ 3.1新增，内部只是提供了默认的顺序参数值order，该参数在DispatcherServlet的流程控制中，在挑选合适的HandlerAdapter时起作用。因为有些Handler可能不止一个的adapter类，所以order值越低的adapter会优先使用。
* __RequestMappingHandlerAdapter :__ 3.1新增，重写了@RequestMapping的实现，替代了AnnotatonMethodHandlerAdapter。对请求方法的解析，方法签名的确定，参数的确定都是在这里进行处理的。


以下分析requestMappingHandlerAdapter的部分代码逻辑：
####supportsInternal(扩展实现了supports方法)

```
//表示所有HandlerMethod类型的Handler都可以被该Adapter处理
@Override
	protected boolean supportsInternal(HandlerMethod handlerMethod) {
		return true;
	}
```

####handleInternal(扩展实现了handler方法)

```
@Override
protected final ModelAndView handleInternal(HttpServletRequest request,
        HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

    //检查该handlerMethod是否使用了@SessionAttributes
    if (getSessionAttributesHandler(handlerMethod).hasSessionAttributes()) {
        // Always prevent caching in case of session attribute management.
        checkAndPrepare(request, response, this.cacheSecondsForSessionAttributeHandlers, true);
    }
    else {
        // Uses configured default cacheSeconds setting.
        checkAndPrepare(request, response, true);
    }

    // Execute invokeHandlerMethod in synchronized block if required.
    if (this.synchronizeOnSession) {
        HttpSession session = request.getSession(false);
        if (session != null) {
            Object mutex = WebUtils.getSessionMutex(session);
            synchronized (mutex) {
                return invokeHandleMethod(request, response, handlerMethod);
            }
        }
    }

    return invokeHandleMethod(request, response, handlerMethod);
}
```

```
private ModelAndView invokeHandleMethod(HttpServletRequest request,
        HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

    ServletWebRequest webRequest = new ServletWebRequest(request, response);

    WebDataBinderFactory binderFactory = getDataBinderFactory(handlerMethod);
    ModelFactory modelFactory = getModelFactory(handlerMethod, binderFactory);
    ServletInvocableHandlerMethod requestMappingMethod = createRequestMappingMethod(handlerMethod, binderFactory);

    ModelAndViewContainer mavContainer = new ModelAndViewContainer();
    mavContainer.addAllAttributes(RequestContextUtils.getInputFlashMap(request));
    modelFactory.initModel(webRequest, mavContainer, requestMappingMethod);
    mavContainer.setIgnoreDefaultModelOnRedirect(this.ignoreDefaultModelOnRedirect);

    AsyncWebRequest asyncWebRequest = WebAsyncUtils.createAsyncWebRequest(request, response);
    asyncWebRequest.setTimeout(this.asyncRequestTimeout);

    final WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    asyncManager.setTaskExecutor(this.taskExecutor);
    asyncManager.setAsyncWebRequest(asyncWebRequest);
    asyncManager.registerCallableInterceptors(this.callableInterceptors);
    asyncManager.registerDeferredResultInterceptors(this.deferredResultInterceptors);

    if (asyncManager.hasConcurrentResult()) {
        Object result = asyncManager.getConcurrentResult();
        mavContainer = (ModelAndViewContainer) asyncManager.getConcurrentResultContext()[0];
        asyncManager.clearConcurrentResult();

        if (logger.isDebugEnabled()) {
            logger.debug("Found concurrent result value [" + result + "]");
        }
        requestMappingMethod = requestMappingMethod.wrapConcurrentResult(result);
    }

    requestMappingMethod.invokeAndHandle(webRequest, mavContainer);

    if (asyncManager.isConcurrentHandlingStarted()) {
        return null;
    }

    return getModelAndView(mavContainer, modelFactory, webRequest);
}
```


