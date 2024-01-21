---
layout: post
title:  "【Intel x86 IoT系列】Intel Arduino/Genuino 101 完全开发指南"
tags: intel x86 zh-cn
---

*简评：移缓就急 劳而无功*

# 前言

*注：以下 Intel Arduino/Genuino 101 简称 Arduino 101*

Arduino 101是一款学习和开发板，它包含了Intel® Curie™模块，旨在将核心的低功耗和高性能与Arduino的易用性相结合。它还具有蓝牙®低功耗功能和内置的6轴加速度计/陀螺仪。

Arduino 101的是一款基于Intel® Curie™模组的低功耗开发板，包含一个x86的夸克核心和一个32bit的ARC架构核心（Zephyr）。其中，ARC架构是一种32位的RISC处理器架构，由Synopsys(新思科技)开发。

在2016年4月21日，英特尔发布 Arduino 101 固件源代码。包含用于 101 上 Curie 处理器的完整 BSP（板级支持包）。它允许您编译和修改核心操作系统和固件，以管理更新和引导加载程序。固件在 Curie 模块内的 x86 芯片上运行，并使用 *回调* 与 ARC 内核（运行 Arduino 程序）进行通信。x86 内核负责处理低功耗蓝牙® （BLE） 和 USB 通信，从而减轻 ARC 内核的负担。*[原网页](https://blog.arduino.cc/2016/04/21/intel-releases-the-arduino-101-firmware-source-code/)*

# 硬件

