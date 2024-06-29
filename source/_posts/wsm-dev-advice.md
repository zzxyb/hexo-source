---
title: wsm开发建议
date: 2024/06/23 16:07:25
comments: true
toc: true
tags:
  - linux Desktop Environment
categories:
  - wsm
---

## 笔者开发环境
```shell
操作系统： Arch Linux 
KDE Plasma 版本： 6.1.1
KDE 程序框架版本： 6.3.0
Qt 版本： 6.7.2
内核版本： 6.9.6-arch1-1 (64 位)
图形平台： Wayland
处理器： 12 × 12th Gen Intel® Core™ i7-1250U
内存： 15.3 GiB 内存
图形处理器： Mesa Intel® Graphics，Intel Corporation Alder Lake-UP4 GT2 [Iris Xe Graphics] (rev 0c)
制造商： TIMI
产品名称： Xiaomi Book Air 13 2022
```
&nbsp;&nbsp;&nbsp;&nbsp;笔者推荐使用Arch Linux + KDE Wayland桌面环境开发。

## 硬件建议
&nbsp;&nbsp;&nbsp;&nbsp;基于现有的 Linux 驱动现状，笔者推荐使用 X86 平台电脑设备进行开发，具体建议如下：
- 使用 Intel 或 AMD 的 CPU 和 GPU 进行功能开发，这些硬件的驱动稳定且维护积极，适配 Linux 良好。
- 不推荐使用闭源且维护不积极的 CPU 和 GPU 设备，这类硬件在开发过程中会带来诸多麻烦，且遇到问题时没有硬件厂商的支持，难以有效解决。
- 对于喜欢折腾的同学，可以在英伟达、树莓派等驱动闭源设备上进行适配开发，建议联系相关技术人员提供技术支持。

## 软件建议
&nbsp;&nbsp;&nbsp;&nbsp;建议在Arch Linux或Ununtu-debian上开发，wsm 会紧跟 wlroots 的发展，需要系统保持最新，紧跟上游发展。

## 核心要求列表
- udev
- libinput
- OpenGL、Vulkan、Pixman
- Mesa X11 and Wayland Platform（可选择屏蔽wsm Xwayland,即可去掉 X11 platform 必要支持）
- linux-dma-buf（可选支持）
- KMS、DRM

对于X86电脑，以上基础要求都会满足，如果存在问题可联系笔者协助解决。非X86请联系硬件厂商。

## 结语
&nbsp;&nbsp;&nbsp;&nbsp;wsm 将会以树莓派为最低硬件标准进行测试，当整个框架逐渐成熟时，笔者会考虑适配更多的硬件平台。