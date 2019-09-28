---
layout: post
title: 在Ubuntu/Debian 上装Pillow
category: Python
---

在使用Django的图片存储的时候用到了`Pillow`这个库，在OS X中是很容易通过`pip install Pillow`安装上的，但是在Ubuntu/Debian 就遇到状况了，很多奇怪的错误导致不能直接装上pillow，原因好像是说缺少编译的图形库。找了很多社区还是没办法解决，最后在[pillow的文档](http://pillow-cn.readthedocs.org/zh_CN/latest/installation.html)上找到了解决方案。

> 注解 Fedora, Debian/Ubuntu, and ArchLinux 已经包含了 Pillow。

我们先装上以下工具：

```shell
$ sudo apt-get install python3-dev python3-setuptools
```

然后就可以通过`easy_install`安装

```shell
$ sudo easy_install Pillow
```