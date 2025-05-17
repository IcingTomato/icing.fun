---
layout: default
title:  "【老物新用】Mojo V3 FPGA 在 Ubuntu 上的开发环境搭建"
tags: fpga linux zh-cn
published: true
---

今天翻出一个12年前老物件（虽说是12年前，其实是两三年前买的），是一款 FPGA 开发板，还记得自己当时因为修这个开发板,使用烙铁风枪的水平突飞猛进。
这款板子还是比较坑的，DC接口居然只能接5V，当时公司里面只有9-12V的适配器，插上去就一股电子元器件的清香。拿万用表打了一下，好家伙直接一带一路，烧掉一个稳压（AMS1117）、两个LED、一个SPI Flash、一个FPGA芯片（XC6SLX9）。很感谢当时在M5的同事，帮我找到了这么多坏的元器件，不然我还以为只是一个稳压坏了。

这老物件，生产它的公司 [Embedded Micro](https://twitter.com/embeddedmicro)已经无了（貌似是转生成[Alchitry](https://alchitry.com/)了，有机会把板子买过来试试），官网也无了，这些资料也只能在[Internet Archive](https://archive.org/)上找到，还是很感谢这个网站的存在，已经捐了10刀作为感谢，希望这个网站能继续运营下去。

<img src="http://icing.fun/img/post/2025/03/10/1.png" alt="捐赠照片">
<i>捐赠照片</i>

对我来说，FPGA（现场可编程逻辑门阵列）一直是电气工程设计中排行老二的领域（老大是制造芯片，那个我还远远做不到），在高中的时候就开始接触，直到大学的时候第一次去Seeed Studio实习，跟同事们聊起FPGA时，我们一致认为FPGA是一个很有意思的领域，但是门槛太高，学习成本太高：

> —— “我搞过FPGA” \
> —— “歪日，牛的，兄弟” \
> —— “这碉板子贵的一批” \
> —— “多少钱” \
> —— “便宜的都得好几百” \
> —— “……我还是琢磨公司的XIAO吧”

这玩意儿不是简单地修改代码就能运行起来的，不仅仅是“玩”，FPGA 使用自己的语言（通常是 Verilog 或 VHDL）进行编程，使用基于同步硬件的技术，并使用极其复杂的软件开发。

## Mojo V3 FPGA 开发板概况

- 1 个 Xilinx Spartan6 FPGA
- 8 个可编程的 LED
- 84 个 I/O 引脚。
- 1 个 Atmel AVR 32U4 MCU。
- 1 个 flash memory chip 用于存储 FPGA 配置数据

XC6SLX9 FPGA 是开发板的核心，它的内部有：

- 9,152 个逻辑块
- 16 个 DSP 切片
- 576 KB RAM
- 102 个 I/O 引脚

我估摸着这应该是世界上第一个把FPGA和MCU结合在一起的开发板，之后的都是什么ARM+FPGA、ESP32+FPGA的设计，也都是参照这个设计的，让MCU可以把比特流文件从SPI Flash中烧录到FPGA里面，就不需要额外的 JTAG 线了。（赛灵思一个原厂JTAG实在是贵的不行，就相当于买一个FPGA一样，仿制的也贵的离谱）

我也想借着闲赋在家的时候，整理整理这个FPGA的基础使用方法。板子的环境搭建我就放在【老物新用】这个栏目里面，具体的知识我再另外开设一个栏目。

## Xilinx ISE WebPACK 安装

[AMD FPGA ISE Archive](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools/archive-ise.html)

这个软件是 Xilinx 早期的 FPGA 开发软件 ISE，现在已经不再维护了。也是神奇，之前接触FPGA还是赛灵思，转眼就被按摩店收购了。收购赛灵思之后感觉按摩店都起飞了，估计是用了不少赛灵思的IP核。

进入到这个网页，找到14.7（14.7的Win10版本就是VirtualBox虚拟机），ISE Design Suite - 14.7  Full Product Installation，Full Installer for Linux (TAR/GZIP - 6.09 GB) MD5 SUM Value : e8065b2ffb411bb74ae32efa475f9817，点击该选项，跳转到AMD登录界面，登录完成后即可下载。

下载完成后，解压到一个目录，然后进入到该目录，执行 `./xsetup`，然后一路下一步，到选择产品的时候需要选择ISE WebPACK即可，等待安装完成。

注意需要添加License，这个License可以在Xilinx官网申请，也可以在网上找到，我这里就<s>不</s>[提供](http://icing.fun/img/post/2025/03/10/vivadolicense.zip)了。

<s>
安装完之后，需要添加环境变量，打开 `/etc/environment` 文件，添加如下内容：
</s>
<s>
```shell
# Xilinx ISE WebPACK
. /opt/Xilinx/14.7/ISE_DS/settings64.sh >/dev/null 2>&1
```
</s>
<s>
其中 `. /opt/Xilinx/14.7/ISE_DS/settings64.sh` 是官方给的，安装完之后会提示你运行这个命令，这个命令会设置一些环境变量，然后就可以在终端中使用 `ise` 命令打开软件了。我就是图方便加到 `.bashrc` 里面的。但是每次开终端都会有运行结果显示，所以我加了 `>/dev/null 2>&1`。
<p></p>
<p><i>
至于为什么不加到 `/etc/environment` 和 `.bashrc` 文件里面，是因为加了之后不知道为啥会影响到其他软件的运行，导致其他软件无法正常运行
</i></p>
</s>

建议是使用shell脚本启动环境：

```shell
# /opt/start_ise.sh
#!/bin/bash
# Xilinx ISE WebPACK
. /opt/Xilinx/14.7/ISE_DS/settings64.sh >/dev/null 2>&1
ise & >/dev/null 2>&1
```

然后给这个脚本添加执行权限：

```shell
sudo chmod +x /opt/start_ise.sh
```

## Mojo Loader 和 Mojo IDE 安装

[IcingTomato/MojoFPGA - GitHub](https://github.com/IcingTomato/MojoFPGA)

这个是我以前下载的 Mojo Loader 和 Mojo IDE 的程序，上传到了 GitHub 上留作备份。这两个程序一个是用来烧录 FPGA 的，另一个是用来编写代码的。

将 `mojo-ide-B1.3.6-linux64.tgz` 和 `mojo-loader-1.3.0-linux64.tgz` 下载到本地，解压到一个目录，然后进入到该目录，执行 `./mojo-ide` 和 `./mojo-loader` 即可。

要注意的是，Mojo IDE 需要JDK8，如果没有安装的话，可以使用 `sudo apt install openjdk-8-jdk` 安装。

我这边为了方便，把这两个程序解压到了 `/opt/` 目录下，然后创建软链接：

```shell
sudo chmod +x /opt/mojo-ide-B1.3.6-linux64/mojo-ide
sudo chmod +x /opt/mojo-loader-1.3.0-linux64/mojo-loader
sudo ln -s /opt/mojo-ide-B1.3.6-linux64/mojo-ide /usr/bin/mojo-ide
sudo ln -s /opt/mojo-loader-1.3.0-linux64/mojo-loader /usr/bin/mojo-loader
``` 

这样就可以在终端中使用 `mojo-ide` 和 `mojo-loader` 命令了。

## Mojo Loader 使用

打开终端，输入 `mojo-loader`，然后选择你的 FPGA 开发板，然后选择你的比特流文件，然后点击 `Erase`，等待擦除完成后即可点击 `Load` 将bin文件加载到Flash。

<img src="http://icing.fun/img/post/2025/03/10/2.png" alt="Mojo Loader">
<i>Mojo Loader</i>

## Mojo IDE 使用

建议是使用shell脚本启动环境：

```shell
# /opt/start_mojo_ide.sh
#!/bin/bash
# Xilinx ISE WebPACK
. /opt/Xilinx/14.7/ISE_DS/settings64.sh >/dev/null 2>&1
mojo_ide & >/dev/null 2>&1
```

然后给这个脚本添加执行权限：

```shell
sudo chmod +x /opt/start_mojo_ide.sh
```

<img src="http://icing.fun/img/post/2025/03/10/3.png" alt="Mojo IDE">
<i>Mojo IDE</i>

## 补充记录

针对 ISE 和 Mojo IDE/Loader 在开始菜单的显示问题，我在后续使用的时候琢磨出来了，找到一个相对比较好的解决方案：

### ISE 配置

先安装完 ISE WebPACK，打开终端，输入：

```shell
cd /usr/share/applications/
```

然后创建一个 `ise.desktop` 文件：

```shell
sudo vim ise.desktop
```

然后输入以下内容：

```shell
[Desktop Entry] 
Version=1.0 
Name=ISE 
Exec=bash -c "unset LANG && unset QT_PLUGIN_PATH && source /opt/Xilinx/14.7/ISE_DS/settings64.sh && ise" 
Icon=/opt/Xilinx/14.7/ISE_DS/ISE/data/images/pn-ise.png
Terminal=false 
Type=Application
Categories=Development;
```

保存退出。

### Mojo IDE 配置

同样的，打开终端，输入：

```shell
cd /opt/MojoFPGA/mojo-ide-B1.3.6/
sudo vim mojo-ide.sh
``` 

然后输入以下内容：

```shell
#!/bin/bash
# Xilinx ISE WebPACK
. /opt/Xilinx/14.7/ISE_DS/settings64.sh >/dev/null 2>&1
exec /opt/MojoFPGA/mojo-ide-B1.3.6/mojo-ide & >/dev/null 2>&1
```

保存退出。

然后给这个脚本添加执行权限：

```shell
sudo chmod +x /opt/MojoFPGA/mojo-ide-B1.3.6/mojo-ide.sh
```

然后添加软链接：

```shell
sudo ln -s /opt/MojoFPGA/mojo-ide-B1.3.6/mojo-ide.sh /usr/sbin/mojo-ide
```

再回到 `/usr/share/applications/` 目录，创建一个 `mojo-ide.desktop` 文件：

```shell
sudo vim mojo-ide.desktop
```

然后输入以下内容：

```shell
[Desktop Entry] 
Version=1.0 
Name=Mojo IDE 
Exec=/usr/sbin/mojo-ide
Icon=/opt/MojoFPGA/mojo-ide-B1.3.6/icon.png
Terminal=false
Type=Application
Categories=Development;
```

保存退出。

这样就可以在开始菜单中找到 ISE 和 Mojo IDE 了。

### Mojo Loader 配置

先创建软链接：

```shell
sudo ln -s /opt/MojoFPGA/mojo-loader-1.3.0/mojo-loader /usr/sbin/mojo-loader
```

同样的，打开终端，输入：

```shell
cd /usr/share/applications/
```

然后创建一个 `mojo-loader.desktop` 文件：

```shell
[Desktop Entry] 
Version=1.0 
Name=Mojo Loader 
Exec=/usr/sbin/mojo-loader
Icon=/opt/MojoFPGA/mojo-loader-1.3.0/icon.png
Terminal=false
Type=Application
Categories=Development;
```

保存退出。

这样就可以在开始菜单中找到 Mojo Loader 了。

<img src="http://icing.fun/img/post/2025/03/10/4.png" alt="Desktop Shortcut">
<i>Desktop Shortcut</i>
