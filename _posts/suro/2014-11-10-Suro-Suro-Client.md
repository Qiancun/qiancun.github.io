-------
layout: post
title:  "Suro: Suro Client"
date:   2014-11-05 22:37:00
description: Suro: Suro Client
categories: suro
-------

有两种Suro client实现，即同步、异步。
* 同步Suro客户端一个发送一个message对象，或一个messageSet对象，以确保该消息被Suro服务器端正确接收。它会阻塞，直到接收到服务器端的ACK确认报文。
* 异步Suro客户端首先将message发送到一个队列之中，然后异步的消费这个队列当中的message，将其发送到服务器端去。Message分组处理，每组的大小是可配置的。开发人员可以设置messages的最长等待时间。如果超过了最长等待时间，即使某个组未达配置的发送长度，异步线程也会发送盖组消息。

# 连接管理
suro client采用了连接池技术管理和server的连接。它至多保持一个同suro server的连接。连接池是单例实现。
如果发送端的数量多于接收端，那么客户端会建立额外的连接，由于thrift是线程不安全的，因此这些额外的连接是在连接池之外建立的。建立额外的连接所需要

# 异步客户端
开发者可以使用内存队列或者基于文件系统的队列实现。内存队列的吞吐量比文件队列高25%。然而，当suro client无法处理越来越多的message时，内存队列会丢弃message。相比这点，由于磁盘的特性，文件队列拥有一个更长的队列长度。文件队列的实现参考了bigqueue实现。

# Suro 客户端配置
值|描述|缺省值
:---------------|:---------------|:---------------
SuroClient.clientType|同步或者异步|异步
SuroClient.connectionTimeout|连接服务器端的连接超时时间（毫秒）|5000
SuroClient.retryCount|该数值表示消息没有被成功消费后的重试次数|5

# 连接服务的负载均衡策略
值|描述|缺省值
:---------------|:---------------|:---------------
SuroClient.loadBalancerType|static or eureka|
SuroClient.loadBalancerServer|如果是静态策略，其值应该为一列hostname:port的值；若其值是eureka，那么值为Eureka服务器的地址|

# 连接服务的超时配置
值|描述|缺省值
:---------------|:---------------|:---------------
SuroClient.minimum.reconnect.timeInterval||90000
SuroClient.reconnect.interval|客户端每发送240条message后会更新connection链接|240
SuroClient.reconnect.timeInterval||30000

# 异步客户端配置
值|描述|缺省值
:---------------|:---------------|:---------------
SuroClient.asyncSenderThreads|发送线程的数目|3
SuroClient.asyncBatchSize|当队列中的message到达200条时，客户端会发送这批messageet|200
SuroClient.asyncTimeout||5000
SuroClient.asyncQueueType||memory
SuroClient.asyncMessageQueueCapacity||10000
SuroClient.asyncJobQueueCapacity||async message queue capacity divided by async batch size
SuroClient.asyncFilQueuePath|文件队列的地址|/logs/suroClient
