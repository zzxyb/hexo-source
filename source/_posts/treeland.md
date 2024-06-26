---
title: treeland简介
date: 2024/06/19 17:22:25
comments: true
toc: true
tags:
  - linux Desktop Environment
categories:
  - treeland
---

## 什么是 treeland？
在讲 treeland 是什么前，先介绍一下现有的 Wayland 合成器类型 
- 系统合成器： 可用于启动系统、处理多用户切换、可能的控制台终端仿真器等。系统合成器可以从早期启动一直运行到关机。它有效地取代了内核 vt 系统，并可以与系统图形启动设置和多座席支持相结合。
系统合成器可以承载不同类型的会话合成器，并让我们在多个会话之间切换（快速用户切换或安全/个人桌面切换）。
- 会话合成器： 会话合成器负责单个用户会话。如果存在系统合成器，会话合成器将嵌套在系统合成器下运行。嵌套是可行的，因为协议是异步的；当涉及嵌套时，往返成本太高。如果不存在系统合成器，会话合成器可以直接在硬件上运行。常见会话合成器有：gnome-shell、kwin、weston。
- 嵌入合成器： X11允许客户端嵌入来自其他客户端的窗口，或者允许客户端将另一个客户端渲染的像素图内容复制到其窗口中。这通常用于面板中的小程序、浏览器插件等。Wayland 不直接允许这样做，但客户端可以在带外传递 GEM 缓冲区名称，例如，使用 D-Bus 或在面板启动小程序时使用命令行参数。另一种选择是使用嵌套的 Wayland 实例。为此，Wayland 服务器必须是主机应用程序链接到的库。然后，主机应用程序将 Wayland 服务器套接字名称传递给嵌入式应用程序，并需要实现 Wayland 合成器接口。主机应用程序将客户端表面合成为其窗口的一部分，即在网页或面板中。嵌套 Wayland 服务器的好处是它提供了嵌入式客户端需要通知主机有关缓冲区更新的请求以及从主机应用程序转发输入事件的机制。

Treeland 的设计目标是实现多会话合成器，这与现有的 Wayland 合成器有着显著不同。通常，传统的会话合成器会为每个用户创建一个独立的合成器进程，并在用户切换时通过 systemd 相关组件进行切换。而 Treeland 无论有多少用户，其进程始终保持一个，能够在多用户场景下统一管理显示服务器的输入和输出。

## 核心功能
waylib 提供了一组与 Qt 、显示管理器密集的功能。其核心功能包括：
- 多用户支持：在 ddm 项目的加持下，以一个合成器进程统一管理显示服务器的输入和输出
- Xwayland支持：支持 X11 应用更好地在 Wayland 下工作。
- 动画/渲染分级：针对不同性能的机器采用不同的渲染、动画策略。

## 使用场景
- treeland 未来会成为 Deepin V23 默认的 Wayland 合成器

## 结语
treeland 作为一个基于 QtQuick、QML 的合成器，它的设计目标是 Muti-Session Compositor，为产品化、个性化定制提供了坚实的技术，其高效的动画和渲染性能可以方便后期的产品迭代和功能更新。

更多信息和详细文档可以访问 [treeland的Github 页面](https://github.com/linuxdeepin/treeland)。