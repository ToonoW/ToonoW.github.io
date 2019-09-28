---
date: 2016-08-10
title: (译)为什么SDWebImage比其他的图片下载组件更好
layout: post
category: iOS
---

## 为什么SDWebImage比其他的图片下载组件更好？

原文出自Github项目说明文档[How is SDWebImage better than X?](https://github.com/rs/SDWebImage/wiki/How-is-SDWebImage-better-than-X%3F)

### 自从 iOS 5.0 开始，NSURLCache 增加负责处理磁盘存储的工作，相对于简单直接的 NSURLRequest 这是 SDWebImage 的一大优势。

iOS NSURLCache 做内存和磁盘存储 HTTP response 的数据（从 iOS 5开始）。每一次命中内存或磁盘中的数据，你的程序必须将 HTTP 数据转换成 UIImage 对象。这个过程需要大量的操作，例如数据解码（HTTP 数据是已编码的），内存中的数据复制等等。

另一方面，SDWebImage 在内存中缓存已装载图片的 UIImage 对象在内存中，并且将已压缩的原始图像（已解码）存储在磁盘里。UIImage 作为 NSCache 被存储在内存中，所以不再需要被来回复制，而且你的程序或者系统需要它之后内存马上可以被释放。

另外，图片解码在后台线程被 SDWebImageDecoder 触发，而且图片解码一般发生在主线程，当你第一次在 UIImageView 里面用 UIImage 的时候。

最后，SDWebImage 会完全地绕过复杂而且经常配置错误的 HTTP 缓存控制会话。这极大地提高了缓存的查询效率。

### 在 AFNetworking 为 UIImageView 提供了相似的功能之后，SDWebImage 是否还有用武之地？

可以说没有。AFNetworking 充分利用 URL 加载系统框架，而缓存用`NSURLCache`，以及为`UIImage`和`UIButton`内建的可配置的内存缓冲，内存缓冲默认使用`NSCache`。缓存机制能被进一步配置，可以使用相应的`NSURLRequest`。AFNetworking 也提供其他 SDWebImage 功能，例如后台解压图片数据。

如果你已经使用了 AFNetworking ，而且你只是想要一个轻便的异步图片加载`category`的话，那么内建的 UIKit 拓展很可能就能满足你的需求。