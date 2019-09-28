---
date: 2016-11-29
title: Core Bluetooth 编程指南(一)  基本介绍
layout: post
category: iOS
---

# 关于 Core Bluetooth

Core Bluetooth 框架提供了一系列的功能类帮助你的 iOS 和 macOS 设备与低功耗的蓝牙设备通信。例如你的 app 可以发现并且与低功耗外围设备通讯交互，就像穿戴心率仪和电子室温计。OS X 10.9 和 iOS 6 以上的系统都可以充当低功耗外围设备了，为其他 iOS 和 macOS 的设备提供数据。

![层级关系图](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBTechnologyFramework_2x.png)

## 技术概览

蓝牙低功耗无线技术是基于 Bluetooth 4.0 技术标准，标准制定了一些列协议去协调低功耗设备之间的通信。Core Bluetooth 框架抽象了蓝牙协议的复杂方法。也就是说，它隐藏了底层的技术标准细节，开发者更加容易地开发 app 和蓝牙低功耗设备通信交互。

### Centrals 和 Peripherals 是 Core Bluetooth 框架中的关键角色

在蓝牙低功耗设备的通信中，central (中心)和 peripheral (外围)是两个关键的角色。peripheral 拥有数据，并且被其他设备请求数据。而 central 则是利用从 peripheral 获取到的数据去完成一些任务。例如室内温度调节器可能会装备有蓝牙低功耗技术，提供室内温度的数值给 iOS app 显示给用户看和选择调节温度，创造良好的用户体验。

每个角色执行自己的职责的时候都做了一系列不同的任务。peripheral 通过广播让自己被其他设备发现，而 central 通过扫描去发现有它们感兴趣的数据的 peripheral。当 central 发现到一个对应的 peripheral ，那么 central 会请求连接，连接成功后开始与 peripheral 数据交互。peripheral 以合适的方法给 central 回应。

> 相关章节 [Core Bluetooth Overview](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothOverview/CoreBluetoothOverview.html#//apple_ref/doc/uid/TP40013257-CH2-SW1)

### Core Bluetooth 简化常见的 Bluetooth 任务

Core Bluetooth 框架抽象出了 Bluetooth 4.0 标准的底层细节。所以，许多常见的蓝牙处理任务你可以在你的 app 中轻易实现。如果你正在开发一个 app ，这个 app 充当 central 角色，Core Bluetooth 框架会让你的 app 简单轻松地发现和连接 peripheral 外围设备，而且还可以和它进行数据交互。另外，Core Bluetooth 也让你的本地设备（iOS ， macOS）担任 peripheral 角色简单可行。

> 相关章节 [Performing Common Central Role Tasks](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonCentralRoleTasks/PerformingCommonCentralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH3-SW1), [Performing Common Peripheral Role Tasks](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonPeripheralRoleTasks/PerformingCommonPeripheralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH4-SW1)

### iOS App 状态影响 Bluetooth 的行为

当你的 iOS app 在后台运行或者处于挂起状态的时候，它的蓝牙连接也会受影响。默认情况下，你的 app 是无法在后台或者挂起状态下执行蓝牙任务。也就是说，如果你的 app 需要在后台或者挂起状态下执行蓝牙任务，你能声明它支持一个或者两个 Core Bluetooth 后台执行模式(一个是 central 的后台模式，一个是 peripheral 的后台模式)。只有当你声明了一个或者两个后台执行模式，某些蓝牙任务在你的程序处于后台时会有不同的操作。你在设计 app 的时候就应该考虑这些差异了。

甚至会这样，你的 app 虽然支持后台运行，但是有可能会为了释放内存给前台的程序而被系统中止。在 iOS 7 ，Core Bluetooth 支持保存 central 和 peripheral 的 manager 对象的状态信息，而且在 app 重新打开的时候可以恢复已经保存的的状态信息。你可以了利用这些特性去支持需要长期蓝牙连接的设备。

> 相关章节 [Core Bluetooth Background Processing for iOS Apps](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothBackgroundProcessingForIOSApps/PerformingTasksWhileYourAppIsInTheBackground.html#//apple_ref/doc/uid/TP40013257-CH7-SW1)

### 遵循最佳实践去提高用户体验

Core Bluetooth 框架帮助你的 app 与许多蓝牙低功耗设备进行通信交流。遵循最佳实践，以负责任的方式去使用这一层的控制蓝牙的方法，并提高用户体验。

例如，许多蓝牙任务你实现 central 或者 peripheral 通过设备板载无线电传输信号。因为你的设备无线电被很多形式的无线通讯共享，又因为无线电的使用对设备的电池寿命有影响，所以你的设计你的 app 的时候最大限度地减少使用无线电。

> 相关章节 [Best Practices for Interacting with a Remote Peripheral Device](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/BestPracticesForInteractingWithARemotePeripheralDevice/BestPracticesForInteractingWithARemotePeripheralDevice.html#//apple_ref/doc/uid/TP40013257-CH6-SW1), [Best Practices for Setting Up Your Local Device as a Peripheral](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/BestPracticesForSettingUpYourIOSDeviceAsAPeripheral/BestPracticesForSettingUpYourIOSDeviceAsAPeripheral.html#//apple_ref/doc/uid/TP40013257-CH5-SW1)

## 如何阅读这份文档

如果你未曾使用过 Core Bluetooth 框架，或者对基本的蓝牙低功耗设备概念不熟悉，请完整地阅读这份文档。在 [Core Bluetooth 概览](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothOverview/CoreBluetoothOverview.html#//apple_ref/doc/uid/TP40013257-CH2-SW1)你将会学到关键术语和基本概念，他们将帮助你理解接下来的内容。

在你理解了关键术语之后，阅读[做常见的 central 角色任务](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonCentralRoleTasks/PerformingCommonCentralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH3-SW1)将会告诉你如何开发你的 app 去本地设备实现 central 角色的功能。相似地，阅读[做常见的 peripheral 角色任务](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonPeripheralRoleTasks/PerformingCommonPeripheralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH4-SW1)会向你介绍如何在你的本地设备实现 peripheral 角色的功能。

为了确保你的 app 运行健壮并且遵从最佳实践，阅读最后的章节[ Core Bluetooth 后台进程 iOS 篇](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothBackgroundProcessingForIOSApps/PerformingTasksWhileYourAppIsInTheBackground.html#//apple_ref/doc/uid/TP40013257-CH7-SW1), [和远程 peripheral 设备交互的最佳实践](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/BestPracticesForInteractingWithARemotePeripheralDevice/BestPracticesForInteractingWithARemotePeripheralDevice.html#//apple_ref/doc/uid/TP40013257-CH6-SW1), 还有[设置你的本地设备为 peripheral 的最佳实践](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/BestPracticesForSettingUpYourIOSDeviceAsAPeripheral/BestPracticesForSettingUpYourIOSDeviceAsAPeripheral.html#//apple_ref/doc/uid/TP40013257-CH5-SW1)。

## 看了又看

官方的[蓝牙标准国际组织(SIG)网站](http://www.bluetooth.org/)提供最权威的蓝牙低功耗无线设备技术的信息。你可以找到这篇文章 [Bluetooth 4.0 标准](https://www.bluetooth.org/en-us/specification/adopted-specifications)。

如果你正在设计硬件产品，而且它是使用蓝牙技术和 Apple 产品交互的，包括 Mac, iPhone, iPad, iPod touch，请阅读[ Apple 产品蓝牙附件设计指南](https://developer.apple.com/hardwaredrivers/BluetoothDesignGuidelines.pdf)。如果你的蓝牙硬件产品(通过蓝牙连接 iOS 设备)需要通过 iOS 设备产生通知消息，请阅读[ *Apple 通知中心服务(ANCS)标准*](https://developer.apple.com/library/content/documentation/CoreBluetooth/Reference/AppleNotificationCenterServiceSpecification/Introduction/Introduction.html#//apple_ref/doc/uid/TP40013460)。