---
layout: post
title:  "Make Windows Great Again"
tags: windows git zh-cn
---

# 让 Windows 伟大起来

## 如何使用 WSL 在 Windows 上优雅地安装 Linux

### 检视你电脑的版本

必须是运行 Windows 10 版本 2004 及更高版本（内部版本 19041 及更高版本）或 Windows 11 的电脑才能优雅。其他的版本可以参阅[手动安装页](https://learn.microsoft.com/zh-cn/windows/wsl/install-manual)，但是不优雅。

### 安装 WSL 优雅的命令

现在，可以使用单个命令安装运行 WSL 所需的一切内容。 在管理员模式下打开 PowerShell 或 Windows 命令提示符，方法是右键单击并选择“以管理员身份运行”，输入 `wsl --install` 命令，然后重启计算机。

```powershell
wsl --install
```

此命令将启用运行 WSL 并安装 Linux 的 Ubuntu 发行版所需的功能。 （可以更改此默认发行版）。

如果你运行的是旧版，或只是不想使用 install 命令并希望获得分步指引，请参阅旧版 WSL 手动安装步骤。

首次启动新安装的 Linux 发行版时，将打开一个控制台窗口，要求你等待将文件解压缩并存储到计算机上。 未来的所有启动时间应不到一秒。

>上述命令仅在完全未安装 WSL 时才有效，如果运行 wsl --install 并查看 WSL 帮助文本，请尝试运行 wsl --list --online 以查看可用发行版列表并运行 wsl --install -d <DistroName> 以安装发行版。 若要卸载 WSL，请参阅[卸载旧版 WSL](https://learn.microsoft.com/zh-cn/windows/wsl/troubleshooting#uninstall-legacy-version-of-wsl) 或[注销或卸载 Linux 发行版](https://learn.microsoft.com/zh-cn/windows/wsl/basic-commands#unregister-or-uninstall-a-linux-distribution)。

```shell
#!/bin/bash
hostip=$(cat /etc/resolv.conf | grep nameserver | awk '{ print $2 }')
wslip=$(hostname -I | awk '{print $1}')
port=7890

PROXY_HTTP="http://${hostip}:${port}"

set_proxy(){
    export http_proxy="${PROXY_HTTP}"
    export HTTP_PROXY="${PROXY_HTTP}"

    export https_proxy="${PROXY_HTTP}"
    export HTTPS_PROXY="${PROXY_HTTP}"

    git config --global http.proxy "${PROXY_HTTP}"
    git config --global https.proxy "${PROXY_HTTP}"

    # 设置APT代理
    echo "Acquire::http::Proxy \"${PROXY_HTTP}\";" | sudo tee /etc/apt/apt.conf.d/95proxies > /dev/null
    echo "Acquire::https::Proxy \"${PROXY_HTTP}\";" | sudo tee -a /etc/apt/apt.conf.d/95proxies > /dev/null
}

unset_proxy(){
    unset http_proxy
    unset HTTP_PROXY

    unset https_proxy
    unset HTTPS_PROXY

    git config --global --unset http.proxy
    git config --global --unset https.proxy

    # 移除APT代理设置
    sudo rm /etc/apt/apt.conf.d/95proxies
}

test_setting(){
    echo "Host ip:" ${hostip}
    echo "WSL ip:" ${wslip}
    echo "Current proxy:" $https_proxy
}

if [ "$1" = "set" ]
then
    set_proxy

elif [ "$1" = "unset" ]
then
    unset_proxy

elif [ "$1" = "test" ]
then
    test_setting
else
    echo "Unsupported arguments."
fi
```

## Git

```powershell
# PowerShell Script to Perform Git Operations with User-Inputted Path

# Prompt for the path to the Git repository
$repoPath = Read-Host -Prompt "Enter the path to your git repository"

# Change to the specified directory
cd $repoPath

# Update git
git pull

# Add all changes to staging
git add --all

# Prompt for a commit message
$commitMessage = Read-Host -Prompt "Enter your commit message"

# Commit the changes
git commit -m "$commitMessage"

# Push the changes to the remote repository
git push

# Back to the origin folder
cd ..
```

## 倒叙之前因后果

11月月初俺爹斥巨资给我换了个笔记本电脑

<center>
    <img src="http://icing.fun/img/post/2023/11/29/order.jpg" alt="Order" title="Order" width="300" />
</center>

可能有人要“素质三连”了

> Surface Pro 9 不是有酷睿版吗？SQ3 版不是很垃圾吗？有这个钱为什么不买苹果？

首先就是全功能 `Type-C` 接口，两个接口可以提供视频，音频，数据传输，更重要的是可以充电。这不比原装的那个方便？带个小米67W就可以搞定手机和电脑的充电了。酷睿版就八行，只能拿那个笨笨的充电器充电。

<center>
    <img src="http://icing.fun/img/post/2023/11/29/type-c.jpg" alt="Type-C" title="Type-C" width="300" />
</center>

5G 网络说实在真的很香，之前在学校办的移动的号，大流量卡，装这个上面，基本上是全天在线，很舒服。

之前去上海面试的时候，在高铁上就可以用电脑来办公，还能追追剧。

随身WiFi稍微麻烦了点，比如我现在也在用华为的。

<center>
    <img src="http://icing.fun/img/post/2023/11/29/5g.jpg" alt="5G" title="5G" width="300" />
</center>

再来就是这个屏幕了，这个屏幕香不香我不知道，主要是能触屏，MacBook那么多年也不能触屏。（估计有人就要骂我，说苹果触控板好用，我只能说，我不喜欢，我喜欢触屏）

<center>
    <img src="http://icing.fun/img/post/2023/11/29/touch.jpg" alt="Touch" title="Touch" width="300" />
</center>

说它是个大号平板也不为过，我现在就是拿着它在写这篇文章，很舒服。安卓平板根本打不过，如果说我要用安卓应用，Windows还有安卓子系统。

<center>
    <img src="http://icing.fun/img/post/2023/11/29/android.jpg" alt="Android" title="Android" width="300" />
</center>

8cx Gen 3 跑分啥的对我来说确实不重要，抛开实际体验的跑分都是耍流氓。

实际体验下来没啥用不了的应用，转译慢就慢我还能摸摸鱼。Python脚本可以跑，我用的是 Anaconda 的环境，没啥问题。IDE用的 VSCode，有原生 ARM64 版本，也没啥问题。VS也能用，还能编译x64/x86的程序，也没啥问题。

<center>
    <img src="http://icing.fun/img/post/2023/11/29/vscode.jpg" alt="VSCode" title="VSCode" width="300" />
</center>