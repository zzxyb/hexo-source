---
title: treeland 事件分析
date: 2024/06/19 17:22:25
comments: true
toc: true
tags:
  - linux Desktop Environment
categories:
  - treeland
---

## treeland 事件到客户端流程
- libinput事件获取：
libinput是一个处理输入设备事件的库，用于从硬件设备（如键盘、鼠标、触摸板等）获取输入事件。
wlroots使用libinput来初始化和管理这些设备，并通过libinput_dispatch函数来获取新的输入事件。

- 事件处理：
wlroots设置了一个libinput事件循环，在事件循环中调用libinput_dispatch以处理新到达的事件。
当一个事件被获取时，wlroots会调用相应的回调函数，这些回调函数会处理具体类型的事件，如键盘事件（libinput_event_keyboard）、指针事件（libinput_event_pointer）等。

- 输入设备管理：
wlroots通过wlr_input_device结构体来表示输入设备。每个输入设备都有相应的事件处理函数。
例如，wlr_keyboard处理键盘事件，wlr_pointer处理鼠标事件。

- 事件分发：
treeland会将受到的键盘和鼠标事件经过Qt事件循环传递给wlr_seat。wlroots使用wlr_seat结构体来管理输入事件的分发，wlr_seat表示一个“座位”，即用户交互的一个上下文，它包含一个键盘、一个指针和一个触摸设备。会将事件分发到当前有焦点的Wayland客户端。焦点管理包括键盘焦点、指针焦点等。

- 事件转换：
wlroots会将这些事件转换为Wayland协议理解的事件。对于键盘事件，会生成wl_keyboard事件，对于指针事件，会生成wl_pointer事件。

- 客户端通信：
wlroots通过Wayland协议将事件发送给客户端。具体来说，通过wl_keyboard_send_key函数发送键盘事件，通过wl_pointer_send_motion、wl_pointer_send_button等函数发送指针事件。

![1.1 鼠标事件流转图](/img/treeland/pointer-motion.drawio.svg)
![1.2 键盘事件流转图](/img/treeland/keyboard_key.drawio.svg)
