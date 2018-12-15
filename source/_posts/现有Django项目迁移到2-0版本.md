---
title: 译 - 现有 Django 项目迁移到新版本
date: 2017-12-04 10:43:51
tags:
- Python
- Django
categories:
- Python
- Web
---

>  2.0 版本正式发布，正考虑将现有的项目迁移到新版，顺手翻译官方的[迁移文档](https://docs.djangoproject.com/en/2.0/howto/upgrade-version/)。

虽然版本的迁移是一个复杂的过程，但是迁移到最新版的 Django 你会有以下的毗益：

* 新功能和改进。
* 漏洞修复。
* 老版本 Django 不会再有安全更新。（查看[详情](https://docs.djangoproject.com/en/2.0/internals/release-process/#backwards-compatibility-policy)）
* 跟随新版本 Django 的更新去更新你的代码库，从而使未来的更新更加轻松。

以下的一些考虑尽可能让你的升级更加顺滑流畅。

## 必读

如果这是你第一次进行 Django 的版本升级工作，这篇 [Django 发布流程指南](https://docs.djangoproject.com/en/2.0/internals/release-process/)。

之后，你应该熟悉一下新版本的变动。

* 阅读每个你使用的版本之后到你计划升级的版本的'final'的[发行说明](https://docs.djangoproject.com/en/2.0/releases/)，以便知道需要了解的变动信息。
* 阅读与升级工作有关的版本的[弃用时间表](https://docs.djangoproject.com/en/2.0/internals/deprecation/)。

要特别关注不向后兼容的变动，这样才能清楚成功升级需要做一些什么工作。

如果你的升级需要跨越多个功能版本（例如 A.B 到 A.B+2），通常通过增量更新的方法会更容易实现（例如 A.B 到 A.B+1 到 A.B+2）比起一下子跳跃过多个版本。建议对于每个功能版本，使用最新的发行补丁。

从 LTS 版本升级到下一个版本的时候也是推荐使用增量更新。

## 依赖

在通常情况下，你的 Django 依赖项也需要升级到最新的版本。如果 Django 的版本是最近发行，或者你的一些依赖项支持还不是很好，一部分的依赖还没支持最新版本的 Django。在这些情况下，你需要等到你的依赖项发行新版本。

## 解决弃用警告

升级之前，我们建议你去解决一些在当前版本 Django 项目中出现的弃用警告。在升级之前修复这些警告，确保你清楚需要变动的区域。

在 Python 中，弃用警告默认是静默的。你必须打开它的提醒，可以通过 `- Wall` Python 命令行参数，或者 `[PYTHONNWARNINGS](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONWARNINGS)` 环境变量。例如，在运行测试的时候启用提醒：

```shell
$ python -Wall manage.py test
```

如果你没有使用 Django 测试器，你可能需要清楚，任何没有被捕获的输出会隐藏弃用警告。例如，如果你使用 py.test：

```shell
$ PYTHONWARNINGS=all py.test tests --capture=no
```

解决当前使用 Django 版本所有的弃用警告之后才能够继续升级工作。

第三方程序可能使用弃用的 API 去支持多个版本的 Django，所以在第三方包中的弃用警告我们可以忽略。如果包不支持最新版本的 Django，请考虑给它发起一个 issue 或者给它一个更新的合并请求。

## 安装

当你做好了前置工作，是时候去[安装新版 Django](https://docs.djangoproject.com/en/2.0/topics/install/)。如果你使用 [virtualenv](https://virtualenv.pypa.io/) 而且准备进行的是大版本更新，你可能需要重新建立一个虚拟环境并安装上所有依赖去更新。

你需要什么什么步骤取决于你的安装过程。最简便的方法是使用 pip 配合 `--upgrade` 或者 `-U`参数：

```shell
$ pip install -U Django
```

pip 会自动卸载旧版本的 Django。

如果你使用其他的安装过程，你可能需要[卸载旧版本的 Django](https://docs.djangoproject.com/en/2.0/topics/install/#removing-old-versions-of-django) 和看看完整的安装教程。

## 测试

当新环境配置后了之后，运行测试命令。同样的，打开弃用警告，那么警告就会出现在测试的输出中（你也可以将这个参数用在程序的运行上，通过 `manage.py runserver`）：

```shell
$ python -Wall manage.py test
```

运行测试之后，修复错误。趁着发行说明还在脑海里，赶紧重构你的代码去排除警告，享受新功能带来的好处吧。

## 部署

当你的程序成功地运行在新版本 Django 之后，你可以[部署](https://docs.djangoproject.com/en/2.0/howto/deployment/)更新过的程序。

如果你使用 Django自带的缓存，你应该在升级之后清除缓存。否则你的程序可能报错，例如，你缓存的是 `pickled` 对象，但是不能保证新的 Django 版本是兼容 `pickled` 的。一个过去的例子，缓存 `pickled` `HttpResponse`对象，直接或者间接地使用 [`cache_page()` ](https://docs.djangoproject.com/en/2.0/topics/cache/#django.views.decorators.cache.cache_page)装饰器。