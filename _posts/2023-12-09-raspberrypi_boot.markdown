---
layout: default
title:  "【阅板无数】RaspberryPi 启动流程探究"
tags: linux raspberrypi zh-cn
---

树莓派学习第一步，探究树莓派的启动流程。

# 树莓派的启动流程

相较于一般/通常的 ARM SoC 来说，树莓派1/2/3/Zero的启动流程有些不同，这里简单的记录一下。

## 树莓派1/2/3/Zero的启动流程

在开机时，CPU是离线的，由GPU上的一个小型RISC核心负责启动SoC，因此大部分启动组件实际上是在GPU代码上运行，而不是CPU上。

启动顺序如下：

- 第一阶段引导程序：用于挂载SD卡上的FAT32启动分区，以便可以访问第二阶段引导程序。它在制造树莓派时已经烧录到SoC本身，并且用户无法重新编程。
- 第二阶段引导程序（`bootcode.bin`）：用于从SD卡检索GPU固件，加载固件，然后启动GPU。
- GPU固件（`start.elf`）：一经加载，允许GPU启动CPU。另一个文件`fixup.dat`用于配置GPU和CPU之间的SDRAM分区。此时，CPU从`复位`模式释放并转移到CPU上执行。
- 用户代码（`User Code`）：这可以是任何数量的二进制文件之一。默认情况下，它是Linux内核（通常命名为`kernel.img`），但它也可以是另一个引导程序（例如`U-Boot`）或一个简单的应用程序。
*在2012年10月19日之前，曾经还有第三阶段引导程序（loader.bin），现已废除。*

下面是原文[elinux.org/RPi_Software](https://elinux.org/RPi_Software)

> At power-up, the CPU is offline, and a small RISC core on the GPU is responsible for booting the SoC, therefore most of the boot components are actually run on the GPU code, not the CPU.
>
>The boot order and components are as follows:
>
> - First stage bootloader - This is used to mount the FAT32 boot partition on the SD card so that the second stage bootloader can be accessed. It is programmed into the SoC itself during manufacture of the RPi and cannot be reprogrammed by a user.
> - Second stage bootloader (bootcode.bin) - This is used to retrieve the GPU firmware from the SD card, program the firmware, then start the GPU.
> - GPU firmware (start.elf) - Once loaded, this allows the GPU to start up the CPU. An additional file, fixup.dat, is used to configure the SDRAM partition between the GPU and the CPU. At this point, the CPU is release from reset and execution is transferred over.
> - User code - This can be one of any number of binaries. By default, it is the Linux kernel (usually named kernel.img), but it can also be another bootloader (e.g. U-Boot), or a bare-bones application.
>
> *Prior to 19th October 2012, there was previously also a third stage bootloader (loader.bin) but this is no longer required.<a href="https://github.com/raspberrypi/firmware/commit/c57ea9dd367f12bf4fb41b7b86806a2dc6281176">[1]</a>*

## 树莓派4B（BCM2711）的启动流程

树莓派4B（BCM2711）因为某些硬件升级导致启动流程复杂了不是一丁半点<a href="https://hackaday.io/page/6372-raspberry-pi-4-boot-sequence">[2]</a>。

当树莓派4开机时，引导过程涉及BCM2711 VideoCore VI 处理器单元（VPU）的操作。以下是引导过程的步骤概述：

1. **VPU核心0的初始化：**
   - 开机时，BCM2711芯片的VPU核心0开始运行。
   - 程序计数器设置为`0x60000000`，映射到片上启动ROM。

2. **VPU频率和启动ROM：**
   - 最初，VPU的频率设置为振荡器频率，树莓派4上为54.0 MHz。
   - 这个低频率足以完成初始引导任务。

3. **读取OTP寄存器：**
   - 启动ROM读取某些一次性可编程（OTP）寄存器（17/18, 66, 和 67）。
   - 这些寄存器与树莓派3相比有不同的含义。更多信息可以在[这个GitHub问题](https://github.com/raspberrypi/firmware/issues/1169)中找到。

4. **引导指示引脚：**
   - 如果配置了“引导指示引脚”，则它会被激活几毫秒。

5. **USB设备模式恢复：**
   - 如果配置了USB设备模式强制引导引脚且激活，某些步骤会被跳过。
   - 这是一种从USB启动恢复的机制。

6. **SD卡引导：**
   - 如果SD卡引导引脚处于激活状态或未配置，启动ROM尝试从SD卡的第一个FAT分区加载`recovery.bin`。
   - 这种机制有助于在EEPROM损坏时恢复板子。更多细节可以在[树莓派硬件文档](https://www.raspberrypi.org/documentation/hardware/raspberrypi/booteeprom.md)中找到。

7. **从EEPROM加载固件：**
   - BootROM尝试从EEPROM芯片加载固件，通常是通过SPI0在GPIO 40–43接口的Winbond W25X40芯片。

8. **USB设备模式：**
   - 如果前面的步骤失败，芯片进入通过Type-C连接器的USB设备模式。
   - 它等待从USB主机获取恢复映像，需要在主机上更新usbboot。更多信息可以在[这个拉取请求](https://github.com/raspberrypi/firmware/pull/1169)中找到。
   - 在这种模式下，VPU的频率提升到100 MHz。

9. **固件映像签名：**
   - 需要在固件映像中附加一个20字节的签名，这是使用存储在OTP中的通用密钥的映像的HMAC-SHA1散列。
   - 这个密钥无法通过树莓派提供的固件的API访问。
   - 这种机制最初在树莓派3上是可选的，现在则是强制性的，尽管它并不提供安全保护，因为存在一个通用的recovery.bin映像。

这个引导过程突出显示了确保树莓派4能够成功引导并从潜在的固件问题中恢复的各种机制。

> When the Raspberry Pi 4 is powered on, the BCM2711 VPU core 0 gets started. The program counter is set to 0x60000000. This address maps to the on-chip boot ROM. Note that the VPU frequency is set to the oscillator (54.0 MHz on RPi4), so it's not very fast, but it does not do a lot of things either:
>
> Read boot-related OTP registers (17/18, 66 and 67). Note that these registers have a different meaning than on Raspberry Pi 3. More information can be found [here](https://github.com/raspberrypi/firmware/issues/1169).
>
> If a “boot indication pin” is configured, it is activated for some milliseconds.
>
> If USB device-mode recovery force boot pin is configured and active, steps 4 and 5 are skipped.
>
> If SD card boot pin is either active or not configured, the boot ROM tries to load recovery.bin from the first FAT partition. This provides a mechanism to unbrick your board if the EEPROM gets corrupted. More details can be found [here](https://www.raspberrypi.org/documentation/hardware/raspberrypi/booteeprom.md).
>
> Next, the boot ROM tries to load firmware from the EEPROM chip. This is a standard Winbond W25X40 chip, access through SPI0 on GPIO 40–43.
>
> If everything else fails, the chip enters USB device mode (on the Type-C connector) and waits to get the recovery image from the USB host. You need an updated usbboot on the host to use this method. See also this pull request.
>
> The VPU frequency is increased to 100 MHz for the USB device mode.
>
> Note that a 20-byte signature must be appended to the firmware image. It is an HMAC-SHA1 hash of the image using a universal key that is also stored in the OTP, but not accessible using the Foundation firmware APIs. A similar mechanism was available as an optional feature on Raspberry Pi3. I originally assumed that the key is unique for each Raspberry Pi, but since there is one universal recovery.bin image that works for all, there must be only one key. Given that it provides no security, I'm not sure why this hash is now mandatory.