---
layout: post
title:  "cat-client: CatFilter Overview"
date:   2013-01-05 22:37:00
description: cat-client: CatFilter
categories: cat
---

CatFilter是Cat Client的入口，通过在应用模块的web.xml中配置这个filter，拦截相应的url请求。CatFilter需要配置一个pattern的Key-Value值，以便告诉Cat Client需要监控哪些URL。

Context上下文的几个成员变量：Mode[Header:X-CAT-SOURCE|Container|2, X-CAT-ID|非空|1, 0], Type[URL], TraceMode[X-CAT-TRACE-MODE|true|false](这个context是ThreadLocal上存储),

# CatFilter初始化
入口：CatFilter.init方法
## catFilter的初始化过程：
Step 1:容器初始化懒加载Cat.initialize --> Init PlexusContainer --> init ModuleContext,Module,ModuleInitializer --> CatClientModule.execute
第一个请求经过该模块的时候，Cat Client会懒加载一个PlexusContainer实例，这边用到org.unidal.initialization包下的几个module初始化类帮助将流程转到具体Module的CatClientModule.execute, CatClientModule.execute方法会被实际执行。Cat类上的主要实例都初始化完成了。
ps：最底层的容器是PlexusContainer，org.unidal包下的ModuleContext也是一个容器，client中的Cat也时封装了底层的这个PlexusContainer。
Step 2:ClientConfigManager.initialize方法 --> 再将contrainer设置到Cat单例中 --> 开启StatusUpdateTask线程 --> 开启ClientConfigReloader线程
StatusUpdateTask线程： 发送Heartbeat事件
ClientConfigReloader线程：
//容器初始化完成



# Filter的拦截过程：MessageTree构建及消息数据的传递
然后顺序执行CatFilter上的四个Handler的Handle方法。
## catFilter的4个CatHandler的Hanlder方法：
* ENVIRONMENT[mode, type, ThreadLocal上下文(_g cookie), TreaceMode]、
* ID_SETUP[根据Mode的值分别设置三个ID: RootId, ParentId, Id]：2的话需要设置三个Id，0的话需要设置Id值：并将三个ID值设置到线程local的MessageTree中；[设置chain]
* LOG_CLIENT_PAYLOAD[logRequestClientInfo, logRequestPayload]
* LOG_SPAN[利用Transaction API计算这次URL的用时]

