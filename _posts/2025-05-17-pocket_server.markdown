---
layout: default
title:  "【玩转树莓派】制作一个口袋服务器"
tags: raspberrypi zh-cn
published: true
---

手头上有个闲置的树莓派 Zero 2 W，想着能不能把它做成一个口袋服务器，随时随地都能用。于是就有了这篇文章。

树莓派 Zero 2 W 是一款小巧的单板计算机，搭载了四核 ARM Cortex-A53 处理器，内存为 512MB。它的体积小巧，功耗低，非常适合用作轻量级的服务器。

同时支持 Wi-Fi 和蓝牙，可以方便地连接到网络。树莓派 Zero 2 W 的价格也非常亲民，适合 DIY 爱好者和学生使用。

树莓派的操作系统是基于 Debian 的 Raspberry Pi OS，支持多种编程语言和开发工具，非常适合用来学习和实验。

那么树莓派的优点讲完了，缺点也很明显：PWR接口和USB接口是 `Micro USB`，所以需要一个转换头。HDMI接口是 `Mini HDMI`，又要一个转换头。

看了一下[官方文档](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#test-pad-locations)，发现他们预留了测试点位，里面有一个 `5V`、`GND`、`USB_DP`、`USB_DM` 的测试点位，直接焊接上去就可以了。这样就可以用 `USB` 供电了。

<img src="http://icing.fun/img/post/2025/05/17/zero2-pad-diagram.png" alt="pin" width="50%">

如果嫌自己焊的太捞了，可以直接去搜树莓派USB转接器，这种产品多的很。

<img src="http://icing.fun/img/post/2025/05/17/pi-zero-usb-adapter-3.jpg" alt="pin" width="50%">

买到手装好之后就该开始配置了。

## 烧录镜像

最新的树莓派64位 Lite 镜像（笔者截稿时是2025年5月13日发行的），使用会有些问题，所以我翻了一下仓库，建议下载这个镜像[raspios_lite_arm64-2022-01-28](https://downloads.raspberrypi.com/raspios_lite_arm64/images/raspios_lite_arm64-2022-01-28/2022-01-28-raspios-bullseye-arm64-lite.zip)。

解压后直接烧录到TF卡上，然后需要修改 `/boot/config.txt` 文件，添加以下内容：

```bash
enable_uart=1
```

因为我们手边没有HDMI线，没办法接屏幕，所以需要通过配置串口来访问树莓派，这种叫法叫做 `headless` 模式。（很奇怪，中文翻译成 **无头** 模式，实际上就是不需要任何外设来启动树莓派<s>开什么玩笑不是还是需要串口工具么</s>）

接下来我们需要从<s>烤箱</s>里面拿出来一个串口工具，连接到树莓派的 `UART` 接口上。树莓派 Zero 2 W 的 `UART` 接口在下面的图中可以看到（Pin1的焊盘是方形的）：

<img src="http://icing.fun/img/post/2025/05/17/GPIO-Pinout-Diagram-2.png" alt="pin" width="50%">

## UART 连接

连接好之后，打开 PuTTY，设置波特率为 `115200`，数据位 `8`，停止位 `1`，无奇偶校验。然后插入TF卡，接上电源，就可以看到树莓派的启动信息了。

以下信息可以参阅[Turning your Raspberry Pi Zero into a USB Gadget - Adafruit](https://learn.adafruit.com/turning-your-raspberry-pi-zero-into-a-usb-gadget/ethernet-gadget)

用户名 `pi`，密码 `raspberry`。登录后可以看到树莓派的命令行界面。

接着编辑config.txt & cmdline.txt

```bash
sudo nano /boot/config.txt
```

在文件末尾添加以下内容：

```bash
dtoverlay=dwc2
```

然后编辑 `cmdline.txt` 文件：

```bash
sudo nano /boot/cmdline.txt
```

在 `rootwait` 后添加一个空格，然后键入 `modules-load=dwc2,g_ether`。

保存并退出。

然后就是熟门熟路地进入 `raspi-config`，设置一下 `SSH` 和 `Wi-Fi` 的配置。

> 树莓派 Zero 2 W 的 Wi-Fi 是 `2.4GHz` 的，不能连接 `5GHz` 的 Wi-Fi。

## 配置USB网络

如果使用的是 Mac 或 Linux，则很可能已经安装了 Bonjour。在 Windows 上，需要安装 `iTunes` 以添加 Bonjour 支持，以便它知道如何处理 `.local` 名称。

关闭树莓派，把USB口插到电脑上，等待树莓派启动，电脑会有设备插入的提示音。

但是可怜的 Windows 会自动识别成串口设备，这时候我们 去下载[USB Ethernet/RNDIS Gadget](https://www.catalog.update.microsoft.com/Search.aspx?q=usb%5Cvid_0525%26pid_a4a2)的驱动，手动安装上，就能正常识别了。

接下来需要配置固定IP，方便我们使用。

在树莓派上执行以下命令：

```bash
sudo nano /etc/network/interfaces
```

在文件中添加以下内容：

```bash
allow-hotplug usb0
iface usb0 inet static
        address 192.168.100.2
        netmask 255.255.255.0
        network 192.168.100.0
        broadcast 192.168.100.255
        gateway 192.168.100.1
```

保存并退出。这将为 Raspberry Pi 提供 IP 地址 `192.168.100.2`。

然后重启网络接口

```bash
sudo ifdown usb0
sudo ifup usb0
ifconfig usb0
```

接着回到自己的电脑，因为我是牛马的ThinkPad用户，我安装了Windows，所以我需要打开网络和共享中心，然后单击更改适配器设置。找到 RNDIS 适配器并将其重命名为 `pizero`。

接着右键单击它，选择属性，然后选择 Internet 协议版本 4 (TCP/IPv4)，然后单击属性。选择使用下面的 IP 地址，并输入 `192.168.100.1` 作为计算机的 IP 地址和网关，子网掩码为 `255.255.255.0` 与树莓派的相同。（网关后来被擦除了，Adafruit的作者认为Windows是自动使用IP地址，但是我认为是电脑与树莓派之间是二层设备，二层之间不需要网关）

## 共享电脑网络

打开网络和共享中心，然后单击我已经连接的网络（比如我现在连接的是Wi-Fi）去更改适配器设置。右键单击 Wi-Fi 适配器，选择属性，然后选择共享选项卡。选中允许其他网络用户通过此计算机的 Internet 连接来连接的复选框，并从下拉列表中选择 `pizero` 适配器。然后单击确定。

使用 `sudo reboot` 重新启动树莓派，然后使用 `ssh pi@raspberrypi.local` 通过 SSH 将其重新连接到它。可以尝试 ping 百度。

## IP寻址

在我们烧录的这个版本的 Raspbian 上，所有网卡的 IP 寻址都是通过名为 dhcpcd 的程序在树莓派上完成的。但是我们现在不需要 dhcpcd，所以我们需要禁用它。

```bash
sudo systemctl disable dhcpcd
```

> 注意：禁用前请使用串口工具登录树莓派。

然后设置接口：

```bash
sudo nano /etc/network/interfaces
```

然后添加以下内容：

```bash
auto lo
iface lo inet loopback

auto usb0
allow-hotplug usb0
iface usb0 inet static
        address 192.168.100.2
        netmask 255.255.255.0
        #network 192.168.100.0
        #broadcast 192.168.100.255
        #gateway 192.168.100.1

allow-hotplug wlan0
iface wlan0 inet dhcp
wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
```

接下来修改 `/etc/wpa_supplicant/wpa_supplicant.conf` 文件，添加以下内容：

```bash
network={
    scan_ssid=1
    ssid="我家的Wi-Fi"
    psk="我家的Wi-Fi密码"
    key_mgmt=WPA-PSK
}
```

`scan_ssid=1` 是因为我家的 Wi-Fi 是隐藏的，所以需要添加这个选项。

接着，我们需要安装 dnsmasq，它允许我们使用 DHCP 为连接到 Pi 上的 USB 端口的 PC 或 Mac 分配 IP 地址。如果树莓派默认的源下载速度太慢，可以参考[USTC](https://mirrors.ustc.edu.cn/help/debian.html)。

```bash
sudo sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
sudo sed -i -e 's|security.debian.org/\? |security.debian.org/debian-security |g' \
            -e 's|security.debian.org|mirrors.ustc.edu.cn|g' \
            -e 's|deb.debian.org/debian-security|mirrors.ustc.edu.cn/debian-security|g' \
            /etc/apt/sources.list
sudo sed -i 's|raspbian.raspberrypi.org|mirrors.ustc.edu.cn/raspbian|g' /etc/apt/sources.list
sudo apt update
```

安装 dnsmasq：

```bash
sudo apt-get install -y dnsmasq
```

然后编辑 `/etc/dnsmasq.conf` 文件，添加以下内容：

```bash
dhcp-range=192.168.100.1,192.168.100.254,24h
dhcp-option=3
dhcp-option=6
```

DHCP 范围需要与我们分配给 usb0 接口的接口 IP 地址匹配，此选项将分配 192.168.100.2 和 .254 之间的地址，租期为 24 小时。这应该绰绰有余。如果由于某种原因需要更改 IP 范围，请确保将 usb0 的配置与这些项目匹配。我们还使用了 DHCP 选项 3 和 6 - 它们在配置文件中进行了注释，但它们阻止 dnsmaq 通告默认路由或 DNS - 在本教程中，我们不需要树莓派作为 DNS 服务器或路由器。

最后重启树莓派。就可以使用 SSH 连接我们的树莓派了：

```bash
ssh -l pi 192.168.100.2
```

## 添加 swap 分区

树莓派的内存只有 512MB，运行一些程序会很吃力，所以我们需要添加一个 swap 分区。

```bash
sudo nano /etc/fstab
```

看到文件末尾有以下内容：

```bash
# a swapfile is not a swap partition, no line here
#   use  dphys-swapfile swap[on|off]  for that
```

我就知道了，树莓派的 swap 分区是使用 `dphys-swapfile` 来管理的。我们可以直接使用它来创建一个 swap 分区。

```bash
sudo nano /etc/dphys-swapfile
```

修改以下内容：

```bash
CONF_SWAPSIZE=512
```

重启 `dphys-swapfile` 服务：

```bash
sudo systemctl restart dphys-swapfile
```

查看 swap 分区：

```bash
free -h
```

可以看到 swap 分区已经添加成功了。


## 降低功耗

有些电脑 USB 可能带不动，所以我们需要降低以下功耗。

```bash
sudo nano /boot/config.txt
```

在文件末尾添加以下内容：

```bash
dtparam=act_led_trigger=actpwr
gpu_mem=16
arm_freq=700
```

其中 `dtparam=act_led_trigger=actpwr` 是让树莓派的电源指示灯变成硬盘活动指示灯。

然后修改 `/boot/cmdline.txt` 文件，在 `console=tty1` 后面加上 `maxcpus=2`。

重启，这将限制树莓派的 CPU 核心数为 2 个。

## 定时清除 buff/cache

参见[【服务器维护】时隔425天的一次维护记录 - 冰冰番茄的归档](https://icing.fun/2025/05/12/server_maintain/#title3)。

也可以将清理脚本 `clear_cache` 放在 `/usr/local/bin` 目录下，确保权限 `+x`，然后添加到 `/etc/cron.d` 中：

```bash
sudo nano /etc/cron.d/clear_cache
```

在 `/etc/cron.d/clear_cache` 中添加以下内容：

```bash
*/10 * * * * root /usr/local/bin/clear_cache
```

## 连接家里的 NAS

可以参考[此处](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/storage_administration_guide/mounting_an_smb_share#unix_extensions_support)。

先安装必要组件：

```bash
sudo apt install cifs-utils
```

然后直接挂载：

```bash
mount -t cifs -o vers=1.0,username=user_name //server_name/share_name /mnt/
```

如果需要密码，可以使用 `-o password=your_password` 选项。

卸载：

```bash
sudo umount /mnt/
```
