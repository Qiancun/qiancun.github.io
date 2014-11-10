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

