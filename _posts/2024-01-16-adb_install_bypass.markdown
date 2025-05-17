---
layout: default
title:  "【安卓搞机记】通过 adb 侧载低版本 SDK 的安卓应用"
tags: adb android zh-cn
---

前些日子翻出来自己高中写的安卓应用，安装上去看看，但是发现小米的 HyperOS 装不上去，一直报错：

> 安装失败 (-29) 
> 失败原因
> 安装包与系统不兼容

我第一反应应该是因为应用的目标 SDK 版本（19）太低了，先用 [App Cloner](https://appcloner.app/) 改改参数，改到 SDK26 看看。

还是不行，打开是能打开但是白屏。

最后看看用 adb 侧载算了：

## [安全性](https://developer.android.google.cn/about/versions/14/behavior-changes-all?hl=zh-cn#security)

### 最低可安装的目标 API 级别

从 Android 14 开始，targetSdkVersion 低于 23 的应用无法安装。要求应用满足这些最低目标 API 级别要求有助于提高用户的安全性和隐私性。

恶意软件通常会以较旧的 API 级别为目标平台，以绕过在较新版本 Android 中引入的安全和隐私保护机制。例如，有些恶意软件应用使用 targetSdkVersion 22，以避免受到 Android 6.0 Marshmallow（API 级别 23）在 2015 年引入的运行时权限模型的约束。这项 Android 14 变更使恶意软件更难以规避安全和隐私权方面的改进限制。尝试安装以较低 API 级别为目标平台的应用将导致安装失败，并且 Logcat 中会显示以下消息：

```shell
INSTALL_FAILED_DEPRECATED_SDK_VERSION: App package must target at least SDK version 23, but found 7
```

在升级到 Android 14 的设备上，targetSdkVersion 低于 23 的所有应用都将继续保持安装状态。

如果您需要测试以旧版 API 级别为目标平台的应用，请使用以下 ADB 命令：

```shell
adb install --bypass-low-target-sdk-block FILENAME.apk
```
