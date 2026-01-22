---
layout: default
title:  "【赛博丁真】让你的笔电也加入 5G "
tags: laptop cellular zh-cn
published: true
---

<s>都说 2019 年是 5G 商用元年，但我一直都有点困惑，4G 已经够快了，5G 到底有什么用呢？</s>

手机和 Pad 上都有 5G，但是笔电制造商很不热衷于在笔电主板上加一个 WWAN 接口或者是直接集成，就连高通处理器的笔电很多也没有集成。

为了探究这个问题，我在 2023 年让我爸斥巨资买了个 Surface Pro 9 5G 版，来体验一下 5G 的快感。

<center>
    <img src="http://icing.fun/img/post/2023/11/29/order.jpg" alt="Order" title="Order" width="50%" />
</center>

随后在进入联想工作后，又弄了一台 ThinkPad X1 Carbon Gen 9，原装是 LTE 模块，上淘宝搜一下轻轻松松就搞到一个高通 X55 的 5G 模块，换上装 Fedora（过程还是比较麻烦的，下一期补上这个）。

先说结论，如果说模块支持修改 IMEI 的话，买个天际通流量卡，基本上就可以做到随时随地在线，办公追剧两不误。

讲真，天际通的流量卡真的很香，2000G 流量才 99 元（我直接包年，599），5G 网络覆盖也还不错。

其实能改 IMEI 的模块也不多，ThinkPad 的模块需要魔改才能暴露串口，使用 AT 指令来修改 IMEI，Surface Pro 9 的模块貌似用的广和通的，也是屏蔽了部分 COM 口。

所以，我直接在淘宝的微雪旗舰店买了个 [USB 3.2 Gen1 5G DONGLE](https://www.waveshare.net/wiki/USB_3.2_Gen1_5G_DONGLE)，再配上 Sub-6 GHz 和 毫米波 模块 [RM530N-GL](https://www.waveshare.net/wiki/RM530N-GL)。

把黑盒的 USB 线接到笔电上，再在电脑上安装驱动：

- [RM520N-GL Windows NDIS驱动](https://www.waveshare.net/w/upload/f/f5/Quectel_Windows_USB_DriverQ_NDIS_V2.4.6.zip)
- [RM520N-GL Windows MBIM驱动](https://www.waveshare.net/w/upload/9/94/Quectel_Windows_USB_Driver%28Q%29_MBIM_V1.3.1.zip)

<center>
    <img src="http://icing.fun/img/post/2025/09/04/Capture.PNG" alt="Devices Manager" title="Devices Manager" width="50%" />
</center>

看到这样就成。然后下载 [QCOM](https://www.waveshare.net/w/upload/c/ca/QCOM_V1.6.zip)，用作为修改 IMEI 的工具。

在设备管理器中，我们能看到三个 COM 口，记下 AT 指令口的 COM 号，比如我的是 `Quectel USB AT Port (COM11)`。

打开 QCOM，选择对应的 COM 口，波特率 115200，然后点击 `Open Port`；`Operation` 下勾选 `RTS` 和 `Send with Enter`：

<center>
    <img src="http://icing.fun/img/post/2025/09/04/Capture1.PNG" alt="QCOM setup" title="QCOM setup" width="50%" />
</center>

输入 `ATE1`，回车，打开回显，看到 `OK` 说明串口通信正常。

输入 `AT+CGSN`，回车，看到返回的结果是当前的 IMEI 号。

移远模组的文档里面没写修改 IMEI 的 AT 指令，网上也有愣头青跑去移远论坛下问，结果人家说不支持修改 IMEI。微雪的文档也说不支持，乐。

国内版可能用不了，但是全球版应该会遵循3GPP TS 27.007 和 TS 27.005 标准，这两个标准规定了常见的蜂窝网络AT命令集（如拨号、短信、网络注册、信号质量查询等）。

放心大胆使用 `AT+EGMR=1,7,"你的华为随身WiFi的IMEI号"` 来修改 IMEI 吧。最后可以用 `AT+CGSN` 来查询修改后的 IMEI 号。

修改完 IMEI 之后，拔掉黑盒，插上 SIM 卡，就可以联网了。

<center>
    <img src="http://icing.fun/img/post/2025/09/04/IMG_4009.jpeg" alt="BlackBox" title="BlackBox" width="50%" />
</center>

如果要使用 MBIM 模式的话，使用以下 AT 指令：

```sh
AT+QCFG="usbnet",2
AT+CFUN=1,1
```

想换回 NDIS 模式的话，使用以下 AT 指令：

```sh
AT+QCFG="usbnet",1
AT+QNETDEVCTL=2,3,1
AT+CFUN=1,1
```

<center>
    <img src="http://icing.fun/img/post/2025/09/04/Capture2.PNG" alt="BlackBox" title="BlackBox" width="50%" />
</center>

<s>不觉得这很酷吗？作为一名理工男我觉得这太酷了，很符合我对未来生活的想象，科技并带着趣味。</s>

突然想起来，我刚上大学的时候，在常州认识了常州创客中心的创始人，当时他就跟我说过一句话：

> 何同学很厉害。他做的 5G 视频你看了没有，我觉得那就是创客。

哦。啧。唉。
