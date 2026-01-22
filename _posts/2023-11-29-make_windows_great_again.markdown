---
layout: default
title:  "【新品开箱】Make Windows Great Again"
tags: windows git zh-cn
---

## 补记

电源计划 卓越性能，启动！

```powershell
powercfg -duplicatescheme e9a42b02-d5df-448d-aa00-03f14749eb61
```

显示成功。但在电源计划里依然没有显示卓越性能。

此时需要修改注册表的CSEnabled值，但CSEnabled不见了(之前有的)，所以应该在管理员模式下的 cmdlet 敲

```powershell
reg add HKLM\System\CurrentControlSet\Control\Power /v PlatformAoAcOverride /t REG_DWORD /d 0
```

问了一下 Premier 事业部的同事，他们说目前据了解20H2取消了csenable更改来设置高性能，需要指令调用。

====================================

如何优雅地使用 `Windows` 作为开发环境。

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

>上述命令仅在完全未安装 WSL 时才有效，如果运行 wsl --install 并查看 WSL 帮助文本，请尝试运行 wsl --list --online 以查看可用发行版列表并运行 wsl --install -d <DistroName> 以安装发行版。 若要卸载 WSL，请参阅<a href="https://learn.microsoft.com/zh-cn/windows/wsl/troubleshooting#uninstall-legacy-version-of-wsl">卸载旧版 WSL</a> 或<a href="https://learn.microsoft.com/zh-cn/windows/wsl/basic-commands#unregister-or-uninstall-a-linux-distribution">注销或卸载 Linux 发行版</a>。

### 更改默认安装的 Linux 发行版

默认情况下，安装的 Linux 分发版为 Ubuntu。 可以使用 -d 标志进行更改。

- 若要更改安装的发行版，请输入：`wsl --install -d <Distribution Name>`。 将 `<Distribution Name>` 替换为要安装的发行版的名称。
- 若要查看可通过在线商店下载的可用 Linux 发行版列表，请输入：`wsl --list --online` 或 `wsl -l -o`。
- 若要在初始安装后安装其他 Linux 发行版，还可使用命令：`wsl --install -d <Distribution Name>`。

> 如果要通过 Linux/Bash 命令行（而不是通过 PowerShell 或命令提示符）安装其他发行版，必须在命令中使用 .exe：`wsl.exe --install -d <Distribution Name>` 或若要列出可用发行版，则使用：`wsl.exe -l -o`。

<s>Ubuntu 不够你用是吧？</s>

### 将版本从 WSL 1 升级到 WSL 2

使用 `wsl --install` 命令安装的新 Linux 安装将默认设置为 WSL 2。

`wsl --set-version` 命令可用于从 WSL 2 降级到 WSL 1，或将以前安装的 Linux 发行版从 WSL 1 更新到 WSL 2。

要查看 Linux 发行版是设置为 WSL 1 还是 WSL 2，请使用命令 `wsl -l -v`。

要更改版本，请使用 `wsl --set-version <distro name> 2` 命令将 `<distro name>` 替换为要更新的 Linux 发行版的名称。 例如，`wsl --set-version Ubuntu-20.04 2` 会将 Ubuntu 20.04 发行版设置为使用 WSL 2。

如果在 `wsl --install` 命令可用之前手动安装了 WSL，则可能还需要启用 WSL 2 所使用的虚拟机可选组件并安装内核包（如果尚未这样做）。

### WSL 子系统访问宿主机的 Proxy

在 WSL 里面新建一个 `setProxy.sh` , 然后把这个复制进去：

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

理论上是所有系统都可以这样做

然后 `chmod +x setProxy.sh` , 最后要设置的时候就 `source ./setProxy.sh set` 就行了。

### PowerShell 美化

#### Scoop包管理器：

[Scoop](https://scoop.sh/) 是 Windows 的命令行安装程序。使用 Scoop，可以为终端安装程序和插件。

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
irm get.scoop.sh | iex

scoop search - 搜索软件
scoop install - 安装软件
scoop info - 查看软件详细信息
scoop list - 查看已安装软件
scoop uninstall - 卸载软件，-p删除配置文件。
scoop update - 更新 scoop 本体和软件列表
scoop update - 更新指定软件
scoop update * - 更新所有已安装的软件
scoop checkup - 检查 scoop 的问题并给出解决问题的建议
scoop help - 查看命令列表
scoop help - 查看命令帮助说明*
```

#### gsudo（提权）

[gsudo](https://gerardog.github.io/gsudo/) 是 sudo 在 Windows 的等价物，具有与原始 Unix/Linux sudo 相似的用户体验。允许在当前控制台窗口或新控制台窗口中以提升的权限运行命令，或提升当前 shell。

只需 gsudo 将（或sudo别名）添加到您的命令中，它就会以提升的方式运行。

```powershell
###GSUDO
#安装：
scoop install gsudo

#导入配置文件内$PROFILE：
# 添加以下这行
Import-Module (Get-Command 'gsudoModule.psd1').Source
# 或者运行这行命令:
Get-Command gsudoModule.psd1 | % { Write-Output "`nImport-Module `"$($_.Source)`"" | Add-Content $PROFILE }

#在PowerShell中执行notepad $PROFILE，打开文件。
#在其中添加：
Set-alias 'su' 'gsudo'
Set-alias 'sudo' 'gsudo'

.$PROFILE使配置生效

# gsudo如果用户选择启用缓存，则可以提升多次，仅显示一个 UAC 弹出窗口。
# 凭据缓存
使用 手动启动/停止缓存会话gsudo cache {on | off}。
使用 停止所有缓存会话gsudo -k。
可用的缓存模式：
Disabled:每次提权都会显示一个 UAC 弹出窗口。
Explicit:（默认）每次提权都会显示一个 UAC 弹出窗口，除非缓存会话以gsudo cache on
Auto:类似于 unix-sudo。第一个提权显示 UAC 弹出窗口并自动启动缓存会话。
使用更改缓存模式gsudo config CacheMode Disabled|Explicit|Auto
```

#### Git 和 posh-git

```powershell
###SCOOP安装GIT
scoop bucket add main
scoop install git
#验证GIT安装：
git --version

#模块安装posh-git
Install-Module posh-git -Scope AllUser -Force
#notepad $PROFILE打开配置档案,添加以下：
Import-Module posh-git
#配置完成后，运行.$PROFILE,使得修改生效

# 克隆任何 GitHub 存储库，以测试在输入常用 git 命令时 posh-git的反映：
git clone https://github.com/dahlbyk/posh-git
cd posh-git
```

#### PSReadline 语法定制

PSReadLine 是微软创建的一个模块，用于自定义 PowerShell 中的命令行编辑环境。它提供了大量的定制，可以改变命令行编辑器以多种方式呈现数据的方式。

```powershell
# 安装
Install-Module PSReadLine
# 配置文件：
Import-Module PSReadLine
```

#### fzf

fzf 是一个用于命令行的模糊文件查找器。

这将为当前文件夹层次结构中的文件启用搜索机制，或者能够查看您之前使用的先前命令。

要使用 PSFzf，您必须先安装 fzf。

```powershell
# 首先，需要通过SCOOP安装
scoop install fzf
# 第二步，安装模块：
Install-Module PSFzf
# 配置文件：
Import-Module PSFzf
Set-PsFzfOption -PSReadLineChordProvider ‘Ctrl+f’ -PSReadLineChordReverseHistory ‘Ctrl+r’
###
#Ctrl+f您可以在当前文件夹和子文件夹中搜索文件。
#您可以通过Ctrl+r列表查看使用过的命令的历史记录。
```

#### z

z 可让您根据`cd`命令历史记录在 PowerShell 中快速浏览文件系统。

z 也是经常使用的模块之一，它有助于快速导航到常用目录，

```powershell
Install-Module -Name z
```

#### Neovim

```powershell
scoop install neovim
```

#### Powershell 升级

```powershell
iex "& { $(irm https://aka.ms/install-powershell.ps1) } -UseMSI"
```

### Git 一键推送

新建一个 `git_push.ps1`：

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
    <img src="../img/post/2023/11/29/order.jpg" alt="Order" title="Order" width="50%" />
</center>

可能有人要“素质三连”了

> Surface Pro 9 不是有酷睿版吗？SQ3 版不是很垃圾吗？有这个钱为什么不买苹果？

首先就是全功能 `Type-C` 接口，两个接口可以提供视频，音频，数据传输，更重要的是可以充电。这不比原装的那个方便？带个小米67W就可以搞定手机和电脑的充电了。酷睿版就八行，只能拿那个笨笨的充电器充电。

<center>
    <img src="../img/post/2023/11/29/type-c.png" alt="Type-C" title="Type-C" width="50%" />
</center>

5G 网络说实在真的很香，之前在学校办的移动的号，大流量卡，装这个上面，基本上是全天在线，很舒服。

之前去上海面试的时候，在高铁上就可以用电脑来办公，还能追追剧。

随身WiFi稍微麻烦了点，比如我现在也在用华为的。

<center>
    <img src="../img/post/2023/11/29/5g.png" alt="5G" title="5G" width="50%" />
</center>

再来就是这个屏幕了，这个屏幕香不香我不知道，主要是能触屏，MacBook那么多年也不能触屏。（估计有人就要骂我，说苹果触控板好用，我只能说，我不喜欢，我喜欢触屏）

<center>
    <img src="../img/post/2023/11/29/touch.png" alt="Touch" title="Touch" width="50%" />
</center>

说它是个大号平板也不为过，我现在就是拿着它在写这篇文章，很舒服。安卓平板根本打不过，如果说我要用安卓应用，Windows还有安卓子系统。

<center>
    <img src="../img/post/2023/11/29/android.png" alt="Android" title="Android" width="50%" />
</center>

8cx Gen 3 跑分啥的对我来说确实不重要，抛开实际体验的跑分都是耍流氓。

实际体验下来没啥用不了的应用，转译慢就慢我还能摸摸鱼。Python脚本可以跑，我用的是 Anaconda 的环境，没啥问题。IDE用的 VSCode，有原生 ARM64 版本，也没啥问题。VS也能用，还能编译x64/x86的程序，也没啥问题。
