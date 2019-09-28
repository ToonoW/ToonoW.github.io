---
date: 2016-11-14
layout: post
title: NSURLSession阅读文档笔记
category: iOS
---

# NSURLSession

`NSURLSession`和与它相关的类提供了下载内容的API。这些API提供了丰富的委托方法，这些方法帮助验证和给你的app在挂起后有后台下载的能力。



## 概览

`NSURLSession`类原生支持`data, file, http, https`这些URL格式。你使用它的同时可以使用代理服务器和SOCKS网关。也可以设置应用内使用的网络协议和URL schemes。

使用`NSURLSession`的API的时候，你很可能需要多个会话。例如做一个浏览器，每一个标签页就需要一个会话。

一个作业里面有一个设置`NSURLSession`一些属性的对象，它能够决定这个连接的行为，例如去设置最大同时进行的连接数而实现一个单个的host（不理解），是否允许通过蜂窝数据去连接等等。这些行为都是在你创建它的配置对象的时候同时被定义的：

* 单例里面的网络通信会话可以给简单的请求使用。它是不可以自定义的，但是如果你的需求很简单的话它是很好用的。你能获取这个会话单例通过调用`sharedSession`类方法。
* 默认的会话的行为配置很像单例的会话（除非你自定义它的功能），但是允许你通过委托方法渐渐地获取数据。你也可以通过` NSURLSessionConfiguration`类的`defaultSessionConfiguration`方法得到默认的会话配置对象。
* 临时会话和默认会话很像，但是不写缓存和cookies或者是写硬盘的资格。你能通过`NSURLSessionConfiguration`类的`ephemeralSessionConfiguration`方法创建一个临时会话配置对象。
* 后台会话允许你的app挂起后在后台仍然执行上传和下载内容的操作。通过`NSURLSessionConfiguration `类的`backgroundSessionConfiguration: `方法你可以创建一个后台会话配置对象。


会话配置对象也包括引用指向URL缓存和cookie
