---
date: 2016-12-27
title: (译)PyQt Threading 线程基础教程
category: Python
layout: post
---

我在之前写了一片[PyQt新手入门](http://nikolak.com/pyqt-qt-designer-getting-started/)，这篇文章介绍了Qt Designer的简单使用，还有PyQt入门知识，如果你对PyQt还不熟悉，建议你去看看再回来。那篇文章有个关于PyQt很重要的知识还没有讲述，就是在PyQt中编写多线程程序。

如果你是一只老鸟，只是希望看有注释的[代码例子](https://gist.github.com/Nikola-K/8b5b510a5c85c3e207fb)，那就点进去吧。

**注意：**这个教程假定你熟悉了我之前写的入门教程，而且你明白什么是网络编程和多线程编程。

本文会编写一个简单的程序，程序可以从网上拉取文章(reddit.com的订阅)，而且API限定每个请求之间至少相隔2s。文章会讲述你单线程编程的情况下会遇到的情况。而UI的设计会很简单，因为这不是文章的重点，