-------
layout: post
title:  "Suro: Message Payload"
date:   2014-11-05 22:37:00
description: suro: message payload
categories: suro
-------

Suro client使用Apache Thrift和Suro Server通信。Suro的有效负载消息是一个普通的字节数组。开发者可以扩展自己的压缩和解压缩方法来处理这些数组。

消息的头部包含以下内容：
* Application Name：发送消息的应用程序名词。
* Routing Key：这个Key值标识了相同的消息类型，用来将相同类型的消息路由（route）到不同的存储池（sinks）中。route key通常用存储池的名词表示。
* Compression：0表示不压缩，1表示LZF压缩（默认）。
* CRC：表示消息的CRC32算法运算结果。这个域用来检查可能的数据冲突。

消息的主体是字节数组（byte array）。消息体的内容由开发者自己定义。开发者可以扩展自己的序列化|解序列化方法，或其他压缩方案来处理消息体。

Suro Client发送消息集到Suro Server端。一个消息集是一组消息的集合。一个消息集除了routing key外，包含相同的消息头部，消息集的消息体部分则是经过压缩后的消息体的顺序排列。

你可以通过MessageSetBuilder来创建Message或者MessageSet。


