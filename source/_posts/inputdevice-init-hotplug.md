---
title: treeland 输入设备分析
date: 2024/06/19 17:22:25
comments: true
toc: true
tags:
  - linux Desktop Environment
categories:
  - treeland
---

## 输入初始化及热插拔
- 设备检测
wlroots 使用 libinput 库来处理输入设备的检测。libinput 会在设备插入或移除时生成相应的事件。
当一个新的输入设备插入时，libinput会生成一个LIBINPUT_EVENT_DEVICE_ADDED事件。
当一个输入设备被移除时，libinput会生成一个LIBINPUT_EVENT_DEVICE_REMOVED事件。

- 事件处理
wlroots有一个事件循环，它会监听libinput生成的事件。当检测到设备添加或移除事件时，wlroots会调用相应的回调函数来处理这些事件，并且发送new_input、destory。

- 设备添加流程
在处理LIBINPUT_EVENT_DEVICE_ADDED事件时，wlroots会创建一个wlr_input_device结构来表示新插入的设备。然后，wlroots会将这个新的wlr_input_device添加到其内部管理的设备列表中。接着，wlroots会发送一个wlr_signal信号，通知其他可能依赖于输入设备的部分（如treeland）有新设备插入。treeland通过相关信号将设备对接到其上下文环境与QPA中。

- 设备移除流程
在处理LIBINPUT_EVENT_DEVICE_REMOVED事件时，wlroots会找到相应的wlr_input_device结构。然后，wlroots会从其内部管理的设备列表中移除这个设备。同样地，wlroots会发送一个wlr_signal信号，通知其他部分有设备被移除。treeland可以通过监听这些信号来更新其状态或进行相应的清理工作（例如，从输入处理链中移除设备，释放相关资源等）。

![1.1 treeland输入设备热插拔流程](/img/treeland/input-device-init-hotplug.drawio.svg)