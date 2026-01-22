---
layout: default
title:  "【玩转树莓派】VSCode 远程登录树莓派并免密登录"
tags: raspberrypi vscode zh-cn
published: true
---

## 安装 VSCode 及 Remote SSH 插件

1. 安装 [Visual Studio Code](https://code.visualstudio.com/)

2. 安装插件：`Remote - SSH`

## 生成 SSH 密钥对（如果没有的话）

打开 PowerShell 或 CMD：

```powershell
ssh-keygen
```

默认会保存在：

```powershell
C:\Users\你的用户名\.ssh\id_rsa（私钥）
C:\Users\你的用户名\.ssh\id_rsa.pub（公钥）
```

直接按回车即可，不设密码。

## 登录树莓派并上传公钥

先用普通方式连接一次：

```powershell
ssh pi@raspberrypi.local
# 或 ssh pi@192.168.x.x
```

然后在树莓派上执行：

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

然后用如下命令将 Windows 上的公钥复制进去（在 Windows 端执行）：

```powershell
type $env:USERPROFILE\.ssh\id_rsa.pub | ssh pi@raspberrypi.local "cat >> ~/.ssh/authorized_keys"
```

再设置好权限（树莓派端）：

```bash
chmod 600 ~/.ssh/authorized_keys
```

## 编辑 SSH 配置文件（Windows）

打开：

```powershell
C:\Users\你的用户名\.ssh\config
```

（没有就创建）

添加如下内容：

```
Host rpi
    HostName 192.168.x.x     # 树莓派的 IP
    User pi
    IdentityFile ~/.ssh/id_rsa
```
