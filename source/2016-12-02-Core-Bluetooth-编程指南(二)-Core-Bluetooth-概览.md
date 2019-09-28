---
title: Core Bluetooth 编程指南(二)  Core Bluetooth 概览
layout: post
category: iOS
---

Core Bluetooth 框架让你的 iOS 和 Mac app 和蓝牙设备通信。例如，你的 app 可以发现和与其他蓝牙 peripheral 设备交互，例如心率监测器，数字温度调节器，甚至其他 iOS 设备。

这个框架封装了 Bluetooth 4.0 标准。也就是说，它隐藏了许多底层细节，让开发者更加容易地操控蓝牙设备。本章节介绍一些概念和相关术语，它会帮助你更好地去利用 Core Bluetooth 框架。

> 注意：一个 iOS app 被连接或者 iOS 10.0 以后的系统，必须包含文件 `Info.plist`去添加描述键，否则程序会崩溃。去获取蓝牙 peripheral 的数据，必须包含[NSBluetoothPeripheralUsageDescription](https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW20)。

## Central 和 Peripheral 设备和它们在蓝牙通信中的角色

central 和 peripheral 蓝牙通信中不可或缺。基于比较传统的 C/S 结构，一个 periperal 的数据往往提供给其他设备。一个 central 使用 peripheral 提供的数据去完成一些特定的任务。如图1-1，例如一个心律测量仪有你的 Mac 或 iOS app 需要的数据，用来展示用户的心率状况。

![central 和 peripheral 设备](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBDevices1_2x.png)

### centrals 发现和连接正在广播的 peripherals

centrals 通过广播数据包广播一些它们拥有数据。一个*advertising packet(广播数据包)*是一个相对小的一捆数据，里面可能包含了有关 peripheral 必须提供的有用信息，例如 peripherals 的名字和主要的功能。举个例子，一个温度调节器会广播提供现在房间的室内温度。在蓝牙低功耗设备中，广播是 peripheral 基本的消息传递方式。

![广播和发现](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/AdvertisingAndDiscovery_2x.png)

### Peripheral 的数据怎么被解析

连接到 peripheral 的目的就是获取它提供的数据信息。在获取数据之前，我们先看看 peripheral 的数据是如何被解析的。

peripheral 可能拥有一个或多个 service 或者提供有关它们连接信号的强度。*service* 是用于实现设备(或该设备的部分)的功能或特征的数据和相关联的行为的集合。例如，一个心律监测器的 service 可能通过监测器的心率传感器得到心律数据。

service 由一个或以上的 characteristics(特性) 组成，或者它被包含在其他的 service 中。 *characteristic* 提供更进一步的信息。例如，刚刚描介绍过的心率 service 可能包含描述身体的心率传感器的位置的特性和传送心率测量值数据的特性。图1-3 说明了一种可能的心率监测器的 service 和 characteristic 结构。

![一个外设的 service 和 characteristics](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBPeripheralData_Example_2x.png)

### central 发现 peripheral 和与 peripheral 中的数据交互

在 central 成功与 peripheral 建立连接后，它可以访问所有 peripheral 开放的 service 和 characteristic (广播数据可能只包含一部分可用的 service)。

central 也可以和 peripheral 的 service 交互，通过读写它的 characteristic。例如，你的 app 可能请求当前房间的温度，或者它给温度调节器一个数值去设置房间的温度。

## centrals peripherals 和 peripheral 数据的表示

