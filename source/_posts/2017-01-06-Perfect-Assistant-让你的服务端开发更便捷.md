---
layout: post
title: Perfect Assistant 让你的服务器开发更便捷
category: swift
---

![Perfect Server Side Swift](http://ww3.sinaimg.cn/large/006tKfTcgw1fbgvgqj0ozj31880obq5t.jpg)

Apple 公司推出的开源语言 Swift 已经在各种场景可以使用了，而且也有越来越多的开发者探索它在服务端的大功用。为了提高服务端开发者的效率，[perfect.org](http://perfect.org/)放出了新的开发管理app [Perfect Assistant](https://www.perfect.org/en/assistant/)。

这个程序适用于macOS平台，Perfect Assistant 的理念是通过一个工具能解决创建和管理服务端 Swift 项目。适用于新手和经验丰富的开发者，一个 app 帮助解决从项目创建到部署到 AWS。

如果你是 Swift 服务端开发的新手，Perfect Assistant 内置的几个模版项目会帮助你快速配置好项目。如果你已经有自己的模版了，Perfect Assistant 也会帮助你导入自己的模版。Perfect Assistant 和 Xcode 可以兼容，所以你不必担心代码在服务器和 IDE 之间的传输。你可以选择一次编译同时支持 macOS 和 Linux。

项目管理器是 Perfect Assistant 的亮点。里面集成了包管理器，让你方地添加各种第三方库依赖。Perfect Assistant 可以在目录和终端中打开项目，并且提供编译部署选项，让你部署在本地、Linux、Amazon EC2，还有‘Clean’功能去清除依赖库。

现在的版本只支持 AWS 的部署，接下来会继续增加 Microsoft Azure。

![AWS](http://ww3.sinaimg.cn/large/006y8lVagw1fbgx7rg4m6j30w10mnjwq.jpg)

我们可以从 [Kitura](https://github.com/IBM-Swift/Kitura)(一个 IBM 的 开源web框架) 看出 IBM 对 Swift 的看好。正如预期的那样，IBM的 Swift工 作面向企业用户，公司已经接触到开发人员社区，甚至参加了 WWDC 2016上的服务器端 Swift 演讲。

没有其他的产品和 Xcode 关系如此紧密，或者和 Perfect Assistant 功能相似。“Perfect Assistant 是企业 Swift 服务端开发重要程序中的一大进步”Sean Stephens, PerfectlySoft 的 CEO 如是说。“过去一年中，我们团队和 Swift 社区紧密合作，来满足整个产品线的开发需求。Perfect Assistant 就是我们努力的体现，我们希望在我们的测试版中能收集到更多的改进意见。”

Perfect Assistant 是[独立研究](https://medium.com/@rymcol/linux-ubuntu-benchmarks-for-server-side-swift-vs-node-js-db52b9f8270b#.egx8zr176)的后续结果，研究是关于在 Linux 上比较 Node.js 和 Perfect 2.0的性能。结果让人很惊喜，无论是 Perfect 2.0 还是 Vapor, Zewo 和 IBM 的 Kitura 在网络请求和JSON数据的处理方面都比 Node.js 要优秀。

这里没有“银弹”，但是 Perfect Assistant 是 Swift 服务端开发最好的选择。如果你感兴趣想看看的 Perfect Assistant， [这里](https://www.perfect.org/en/assistant/)是 Perfect Assistan 的下载地址。