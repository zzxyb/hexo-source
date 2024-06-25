---
title: treeland 显示器设备分析
date: 2024/06/19 17:22:25
comments: true
toc: true
tags:
  - linux Desktop Environment
categories:
  - treeland
---

## 显示器初始化及热插拔
- 初始化与设置
首先，wlroots 的 wlr_backend 和 wlr_output 模块需要初始化。wlr_backend 负责与底层硬件交互，而 wlr_output 则表示具体的显示输出（即显示器）。

- 监控硬件事件
wlroots 使用 udev 来监控硬件事件（包括显示器的插拔）。当有硬件事件发生时，wlroots 会通过 udev 提供的接口接收这些事件。

- 事件处理
当一个显示器插入或拔出时，wlroots 会触发相应的事件：new_output、destroy。treeland连接事件，处理QPA中存储的Screen数据，同步硬件动作信息，如下图1.1。

![1.1 treeland显示器热插拔流程](/img/treeland/monitor-init-hotplug.drawio.svg)