---
layout: default
title:  "【安卓搞机记】Google Pixel XL 折腾记"
tags: android zh-cn
---

我在18年的时候，攒了好久的一笔钱，买了 Pixel XL，换掉了可怜的 iPhone5s，然后就开始了我的折腾之路。

# 前言

手机摔个稀碎，没法拍图了

# Pixel/Pixel XL V版解锁教程

- 退出谷歌账号，取消屏幕密码，删除指纹

- 把设备恢复出厂设置

- 跳过设置向导所有东西，不要连接wifi

- 进入开发者选项开启USB调试，并授权电脑访问

- 电脑使用adb命令

```shell
adb shell pm uninstall --user 0 com.android.phone 
adb reboot
```

- 比较关键的一步，国内很多人卡在这里，主要是要让手机可以访问谷歌并且消除wifi上的X号，我的方法是先使用adb命令（或者科学上网法）

```shell
adb shell settings put global captive_portal_https_url https://www.google.cn/generate_204 
```

然后开关飞行模式，再开启外网，再连接wifi，就可以消除信号x了

- 设置锁屏密码为图形，再通过桌面的google程序登录谷歌账号

- 这时候再回去开发者选项就会看到OEM解锁可以开了

- 打开OEM解锁重启进入BootLoader

电源+音量+ 或

```shell
adb reboot bootloader
```

- 使用adb命令

```shell
fastboot oem unlock
```

或

```shell
fastboot flashing unlock
```

- 然后手机会进入一个界面，音量键选择YES（将会清除所有数据）

- 清除完了之后手机会跳回到BootLoader界面，可以看到底下已经是UNLOCK状态了

- 然后重启手机，再进入开发者选项，就会看到OEM解锁那里已经是解锁状态了

# 如何进入Bootloader界面？

- 方法一：手机关机，等待3秒钟，按住音量下1秒，不要松开，然后按住电源键，等待手机振动，手机进入bootloader界面。

- 方法二：手机关机，等待3秒钟，按住音量下1秒，链接电源线，等待手机振动，手机进入bootloader界面。

- 方法三：手机开机状态下，打开usb调试，在cmd窗口中手动输入

```shell
adb reboot bootloader
```

# 线刷包刷机教程

- 在[谷歌Pixel镜像](https://developers.google.cn/android/images)下载对应的线刷包

- 然后把所有的文件放到同一个文件夹内

- 手机进入Bootloader界面，链接电脑

- 这时候可以确认驱动是不是安装好了，打开设备管理器

- 双击刚才那个文件夹下的 flash-all.bat，开始刷机（刷机会清除所有数据哦）

- 刷机过程中手机会重启多次，请静静等待结束

# 为什么要解锁 Bootloader？

解锁 Bootloader 是打开玩机之门的钥匙。只有在 Bootloader 解锁的前提下，我们接下来的步骤才得以顺利进行：安装 TWRP、获取 Root 权限……最后，通过一些需要 Root 权限的特殊手段，打开位置历史记录功能，获得近乎完整的 Android 体验。

让我们直奔正题吧：

（注意：解锁 Bootloader 重置设备数据，请注意备份）

首先，我们需要安装必（ni）备（dong）的工具并成功接入互联网。

前往「设置-关于手机-版本号」，猛击版本号 7 次开启「开发者选项」。

在开发者选项中开启「OEM 解锁」。若该选项显示为灰色，请检查你的网络连接是否正（ke）常（xue）USB 调试。

在 Pixel 设备中，开启 OEM 解锁需要连网

通过数据线将手机连接至电脑，以管理员身份运行命令提示符（CMD），输入 adb shell 并回车。

此时，手机端会弹出 USB 调试申请，点击「允许」。

回到命令提示符窗口，键入 

```shell
adb reboot bootloader 
```

并回车，手机会立即重启至 Bootloader 模式。

在 Bootloader 解锁界面中，使用音量键 +/- 来控制光标，选择「Yes」并按下电源键来进行 Bootloader 解锁。

稍等片刻之后，你的设备会自动重启，开机时屏幕下方出现一把打开的小锁，那 Bootloader 就解锁成功啦。

# 刷入第三方 Recovery —— TWRP

如果说 Bootloader 是玩机大门上的那把锁，那么 TWRP 就是你打开玩机之门后的领路人。

所以在解锁 Bootloader 之后，紧接着要做的事情就是刷入第三方 Recovery —— TWRP。由于 Pixel 与 Nexus 的系统分区方式不同，刷入 TWRP 的方式也略为复杂一些：

首先，前往 TWRP 官网下载最新版 TWRP 压缩包（.zip）和临时 TWRP 镜像文件（.img）。

将 .img 文件留在电脑上，同时将 .zip 文件拷贝至内置储存。

重启手机至 Fastboot 模式（参考上面的方法或关机后长按「电源」和「音量 -」），在电脑以管理员身份运行命令提示符。

在命令提示符窗口中输入 cd 你的 .img 文件路径 来进行定向（比如我的 .img 文件放在 E:\Android 下，那就键入 cd E:\Android 并回车），然后输入

```shell
fastboot boot xxx.img //xxx 为具体的文件名
```

此时手机会重启至临时 TWRP。

在临时 TWRP 中，选择「Install」，找到我们事先放在内置储存中的压缩包文件，点击刷入。

至此，我们就已经用第三方 TWRP 替换了系统自带的 Recovery，接下来的 Root 操作也得以进行：

前往 SuperSU 官网下载最新版 SuperSU 压缩包。

将压缩包拷贝至手机内置储存。

重启手机至 TWRP（关机后长按「电源」和「音量+」），找到 SuperSU 压缩包并刷入。

完成后重启手机，首次启动过程会自动中断并再次自动重启，不用担心。开机后，手机便已获得 Root 权限，SuperSU 权限管理应用也已经安装至系统当中。

做好这些工作之后，我们就可以进一步使用 LocationReportEnabler 等需要 Root 权限的应用来开启位置记录报告功能，最终获得完整的 Android 生态体验了。

# 如何进行系统更新

非 Root 用户很难对系统文件进行修改，但在进行系统更新时则较为轻松，在保证网络条件畅（ke）通（xue）的前提下，只需前往「设置-关于-系统更新」，便可自动检查、下载并升级至最新版本的 Android 操作系统。

但对 Root 用户而言，手动刷入工厂镜像进行系统更新的方法则更为稳妥。

首先，我们需要前往 Google 的 Nexus/Pixel 工厂镜像网站找到并下载最新版本的 Android 系统镜像，然后解压。

完整工厂镜像文件压缩包包含了这些内容

如果你是拿到手机想要优先进行系统升级的用户，将手机重启至 Fastboot 模式并连接电脑后，直接运行 flash-all.bat 即可全自动升级至最新版本；如果你想保留升级前的系统数据，则需要在运行 flash-all.bat 前对其进行一些处理。

使用文本文档、Notepad++ 等工具打开 flash-all.bat，找到

```shell
fastboot -w update image-marlin-xxxxxx.zip
```

字段，将其改为

```shell
fastboot update image-marlin-nof27b.zip //即去掉 -w
```

然后保存，即可利用修改后的 flash-all.bat 文件在保留数据的前提下进行系统更新。

另外，在运行 flash-all.bat 进行更新的过程中，很有可能会出现报错。报错文本类似于

```shell
archive does not contain 'boot.sig'
archive does not contain 'recovery.sig'
archive does not contain 'system.sig'
```

当出现这样的字段时，千万不要终止操作。耐心等待耐心等待耐心等待！只要最后出现

```shell
finished. total time: 128.109s 
Press any key to exit...
```

即是升级成功，手机也会自动重启进入新系统。

另外，在这个过程中所出现的错误往往和 platform-tools（包含 ADB、fastboot 等）版本过旧有关。所以遇到其他形式的报错也不用惊慌，前往 Google 官方网站 下载安装最新版本的 platform-tools 后，再次执行以上操作步骤即可。

Google 从年初开始提供独立的 platform-tools 下载

与非 root 系统自动更新相比，手动刷入完整版工厂镜像尽管要麻烦不少，但也更加灵活。

举个例子，我的 Pixel XL 到手时的系统版本是 Android 7.1.1，但安全更新补丁停留在去年 10 月。如果采用非 root 系统自动更新的方法，那我总计需要更新五次才能更新至最新的 3 月安全更新补丁。

手动 Fastboot 刷入工厂镜像则简单许多，只要我们刷入的是完整版工厂镜像，就可以无视版本跨度，一次性升级至最新版本。

# 原生安卓WiFi信号去叹号去叉教程5.0-Android P

> Captive Portal是安卓5引入的一种检测网络是否正常连接的机制，制作的非常有创意，通过HTTP返回的状态码是否是204来判断是否成功，如果访问得到了200带网页数据，那你就可能处在一个需要登录验证才能上网的环境里，比如说校园网，再比如说一些酒店提供的客户才能免费使用的WiFi（其实是通过DNS劫持实现的），如果连接超时（根本就连接不上）就在WiFi图标和信号图标上加一个标志，安卓5和6是叹号，安卓7改成一个叉了。只不过默认访问的是谷歌自家的验证服务器，然而由于你懂的原因，就算你连接上了网络也连不上这个服务器... 嗯...那其实还是没有连接上网络嘛... 噫....
>
> 谷歌设计了一个开关来控制是否启用这个特性，同时也提供了一个变量来控制待验证的服务器地址，国内的修改版ROM通常都改成了高通中国的地址，还有一些ROM设计了代码在重启的时候恢复这个设置，不知道是出于什么目的。
>
> 没更新7.0的时候，一直用小狐狸的叹号杀手，很不错的应用，可惜当时他已经很久不更新了，当时安卓N不能用，后来自己做了个小工具，想了想就干脆上架酷安吧，也能帮助大家，这样有了CaptiveMgr工具，这分明就是个没有名字的名字嘛...根本就是foo, bar一样...好像也没什么好叫的了？现在代码还比较乱，要是哪天有空把这堆代码整理出来就开源了算了，毕竟纯粹体力活。
>
> 具体的原理不在这里写了，这里主要写如何去掉叹号或者叉标志。
>
> 如果有root权限直接用我这个工具算了，比较方便，毕竟用命令也就是检测一下系统然后代替执行命令而已嘛。
>
> (PS: 如果使用SS/SSR可以通过NAT模式让系统直接连接，其内部是通过iptables实现的)
>
> 如果没有root权限就得按下面操作了，做好配置以后重启WiFi和数据流量（打开再关闭飞行模式即可）就可以看到效果了。
>
> 以下需要ADB调试，配置不赘述

- 5.0 - 6.x教程

5和6还不支持HTTPS，直接修改即可

检测开关相关：

先处理开关状态，这个变量删除就是默认开启的，删除操作随意执行，反正没影响，删除状态下获取这个变量会返回null。

> 注意：如果关闭，则无法判断当前网络是否需要登录，无法自动弹出登录页面

```shell
adb shell settings delete global captive_portal_server //删除
adb shell settings put global captive_portal_server 0 //禁用
adb shell settings get global captive_portal_server  //查询状态
```

服务器地址相关：

```shell
adb shell settings delete global captive_portal_server  //删除地址就可以恢复默认的谷歌服务器
adb shell settings put global captive_portal_server captive.v2ex.co  //设置一个可用地址（高通/V2EX都推荐）
adb shell settings get global captive_portal_server  //查询当前地址
```

- 7.0 - 7.1教程

这两个版本相比5和6没有大的更改，只是默认连接服务器的时候使用HTTPS，但是提供了一个开关用以指定是否使用HTTPS

检测开关相关：同5.0 - 6.x

HTTPS开关相关：

```shell
adb shell settings delete global captive_portal_use_https  //删除（直接删除则默认使用HTTPS）
adb shell settings put global captive_portal_use_https 0  //禁用HTTPS（写1启用 写0禁用）
adb shell settings get global captive_portal_use_https  //查询HTTPS开关状态
```

服务器地址相关：（如果启用了HTTPS需要先确定地址是否支持HTTPS）同5.0 - 6.x

- 7.1.1教程

这个版本把HTTPS和HTTP两个地址分开保存，并通过7.0加入的HTTPS开关来控制使用哪一个地址。

检测开关相关：同5.0 - 6.x

HTTPS开关相关：同7.0 - 7.1

服务器地址相关：

```shell
adb shell settings delete global captive_portal_https_url  //删除（删除默认用HTTPS）
adb shell settings delete global captive_portal_http_url
```

分别修改两个地址

```shell
adb shell settings put global captive_portal_http_url http://captive.v2ex.co/generate_204
adb shell settings put global captive_portal_https_url https://captive.v2ex.co/generate_204
```

- 7.1.2教程

此版本服务器地址判断逻辑相比7.1.1没有更改，但是检测的开关却变了。

检测开关：

```shell
#删除变量：（删除以后默认启用）
adb shell settings delete global captive_portal_mode
#关闭检测：
adb shell settings put global captive_portal_mode 0
#查看当前状态：
adb shell settings get global captive_portal_mode
```

服务器地址相关（同7.1.1）：

```shell
#删除（删除默认用HTTPS）
adb shell settings delete global captive_portal_https_url
adb shell settings delete global captive_portal_http_url  
#分别修改两个地址
adb shell settings put global captive_portal_http_url http://captive.v2ex.co/generate_204
adb shell settings put global captive_portal_https_url https://captive.v2ex.co/generate_204
```

- 8.0.0和8.1.0和9.0(Android P)同上7.1.2，未做修改

# 隐藏导航栏和状态栏

```shell
#全屏沉浸：
adb shell settings put global policy_control immersive.full=*

#沉浸状态栏：
adb shell settings put global policy_control immersive.status=*

#沉浸导航栏：
adb shell settings put global policy_control immersive.navigation=*

#单独控制某一个app不沉浸，比如以下代码设定google实时界面不沉浸，其他程序沉浸：
adb shell settings put global policy_control immersive.full=apps,-com.google.android.googlequicksearchbox

#恢复到正常方式：
adb shell settings put global policy_control null
```

# adb 查看android手机中应用的包名和安装位置

```shell
#查看是否连接手机
adb devices

#进入指定的device的shell
adb shell
#或者
adb -s ********* shell

#adb 查看所有安装的包
adb shell pm list packages

#根据某个关键字查找包
pm list packages | grep tencent

#查看包安装位置
pm list packages -f

#同样可以进行筛选
pm list packages -f | grep tencent

#将apk拉到pc中
adb pull /data/app/com.tencent.tbs-1/base.apk ~/Downloads
```

# 黑阈

```shell
adb -d shell sh /data/data/me.piebridge.brevent/brevent.sh
```

# Google Pixel 电信破解

> 仅适用于一代，无需双清，无需QPST，摆脱繁琐的刷机

在Android Pie下测试通过，重启后不丢失

开发者模式中开启 OEM 解锁后，重启手机进入 bootloader 模式，在电脑端运行 fastboot oem unlock

> 注：此过程会清除手机数据，请及时备份。

[下载modem.img](https://onedrive.live.com/embed?cid=A62E298A214E48C7&resid=A62E298A214E48C7%2125763&authkey=ANct6CYbGcn2HhM)

刷入modem.img

```shell
fastboot flash modem_a modem.img
fastboot flash modem_b modem.img
```

理论上不支持移动和联通，如更换请重刷官方底包

- 注意（如果有路由器级别kexue可以忽略）

> 你需要2018年3月或之前的系统。原因是这样的：
> 有一个叫frp的分区，它的全称是——Factory Reset Protection。
> 开机设置读取frp，得知是否首次开机。清除frp，伪装成首次开机。
> 不幸的是，4月的官方镜像里的开机设置没有这种操作。开机设置才不管frp呢，必须联网。
> 所以在恢复出厂设置前一定要看看你的安全补丁日期是哪个月的！！如果是4月的需要先刷回3月。

# 2021年2月21日补记

忘记是哪一次刷机的时候`Pixel`和`Pixel XL`的包搞混了，在`Pixel`上刷了`XL`的包，结果还能用？就是屏幕的`最小宽度`（可以在`设置 -> 系统 -> 开发者选项 -> 绘图` 下面更改）变了好像？