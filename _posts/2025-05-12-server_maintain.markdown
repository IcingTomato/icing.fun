---
layout: default
title:  "【服务器维护】时隔425天的一次维护记录"
tags: AliCloud ECS zh-cn
published: true
---

- [删除阿里云助手Agent](#title1)
- [阻止脚本小子偷偷撞我SSH密码](#title2)
- [定时清扫一下 buff/cache](#title3)

## <span id="title1">删除阿里云助手Agent</span>

今天正好在玩树莓派的时候，心血来潮登录自己<s>部落格</s>博客的服务器看看。

熟练地 `htop` 一下，结果……

<img src="../img/post/2025/05/12/1.png" alt="htop" width="50%">
*惊不惊喜？意不意外？*

`/usr/local/share/assist-daemon/assist_daemon` 很扎眼，我就知道是阿里云干的事情。

首先说结论：参阅[卸载云助手Agent](https://help.aliyun.com/zh/ecs/user-guide/start-stop-or-uninstall-the-cloud-assistant-agent#4f295d821anjm)的文档，跟着删就行。

```bash
# 停止云助手守护进程
/usr/local/share/assist-daemon/assist_daemon --stop

# 卸载云助手守护进程
/usr/local/share/assist-daemon/assist_daemon --delete

# 删除云助手守护进程目录
rm -rf /usr/local/share/assist-daemon

# 卸载云助手Agent
sudo dpkg -r aliyun-assist # Debian/Ubuntu
sudo rpm -qa | grep aliyun_assist | xargs sudo rpm -e # CentOS/RHEL

# 删除云助手Agent目录
rm -rf /usr/local/share/aliyun-assist
```

## <span id="title2">阻止脚本小子偷偷撞我SSH密码</span>

跟前年hvv一样，阿里不知道从哪里找来一些野鸡选手，当时我的博客用的是<s>硬屎</s>极简风，用Jekyll生成的类似Apache/Nginx默认页面，具体看[这](https://web.archive.org/web/20231003163648/http://icing.fun/)。发短信告诉我80端口开了，都看到里面的文件了，很危险，需要我注意。

<img src="../img/post/2025/05/12/2.png" alt="?" width="50%">
*我请问呢？*

现在看来，他们很喜欢扫 `ssh` 端口。<s>抓不到漏洞急了吧？随便写一下就能输出报告？我那个时候高位端口都是开着的咋不扫一下？看ssh日志时候才发现有个IP是镇江的江苏科技大学的一直在扫我服务器，光扫5000以下的，有点难绷了兄弟。</s>

使用 `cat /var/log/auth.log |grep Failed` 这个命令（文件可能是auth.log或者是auth.log.数字，可以进目录看看）看了一下登录失败的记录。

<img src="../img/post/2025/05/12/3.png" alt="ssh" width="50%">
*ssh登录失败的记录*

蔚为壮观……

可以看看这个[Treemap](https://icingtomato.github.io/Anti-SSH-Attack/)，可以清晰看到IP地址和源地址以及攻击手段。

自己搓了个脚本，提取 `/var/log/auth.log` 登录失败的IP地址，添加到 `/etc/hosts.deny` 里面。

```bash
#!/bin/bash

# 脚本功能：从/var/log/auth.log和/var/log/auth.log.N提取SSH失败登录的IP地址并添加到/etc/hosts.deny

# 检查是否有root权限
if [ "$(id -u)" -ne 0 ]; then
    echo "Need root!"
    exit 1
fi

# 查找所有匹配的日志文件
log_files=(/var/log/auth.log /var/log/auth.log.*)

# 检查是否找到至少一个有效的日志文件
valid_files=0
for log_file in "${log_files[@]}"; do
    if [ -f "$log_file" ]; then
        valid_files=$((valid_files+1))
    fi
done

if [ $valid_files -eq 0 ]; then
    echo "Cannot find any auth.log files"
    exit 1
fi

# 计数器
added=0
skipped=0

# 处理每个找到的日志文件
for log_file in "${log_files[@]}"; do
    # 跳过不存在的文件
    if [ ! -f "$log_file" ]; then
        continue
    fi

    echo "Processing $log_file..."
    
    # 查找包含"port"但不包含正常连接行为的记录，提取IP地址
    grep "port" "$log_file" | grep -v -E "(Accepted password|Received disconnect|Disconnected from user)" | while read -r line; do
        # 使用正则表达式提取from/by与port之间的IP地址
        if [[ $line =~ (from|by)[[:space:]]+([0-9]+\.[0-9]+\.[0-9]+\.[0-9]+)[[:space:]]+port ]]; then
            ip="${BASH_REMATCH[2]}"
            
            # 检查是否已经在hosts.deny文件中
            if grep -q "sshd: $ip" /etc/hosts.deny; then
                echo "Pass $ip (exist)"
                ((skipped++))
            else
                # 将IP地址追加到hosts.deny
                echo "sshd: $ip" >> /etc/hosts.deny
                echo "Block $ip"
                ((added++))
            fi
        fi
    done
done

echo "Done! Add: $added, Exist: $skipped"
```

再加上定时任务，[每分钟执行一次](https://crontab.guru/every-1-minute)。

```bash
* * * * * /root/anti_attack.sh
```

<img src="../img/post/2025/05/12/4.png" alt="block_ip" width="50%">
*尝试运行，不错，确实在自动运行*

自己尝试拿手机热点登录一下，获取一个IP地址，看看能不能被封。

<img src="../img/post/2025/05/12/5.png" alt="block_ip" width="50%">
*可以清晰看到第一次登录成功，加了deny之后封IP成功，ssh登录失败了*

## <span id="title3">定时清扫一下 buff/cache</span>

有时候服务器运行久了，内存会被占满，导致服务器变得很卡。可以使用 `free -h` 命令查看一下内存的使用情况。

可以看到 `buff/cache` 占用的内存很大。一般可以用以下命令清理一下：

```bash
# Writing to this will cause the kernel to drop clean caches, as well as
# reclaimable slab objects like dentries and inodes.  Once dropped, their
# memory becomes free.

# To free pagecache:
	echo 1 > /proc/sys/vm/drop_caches
# To free reclaimable slab objects (includes dentries and inodes):
	echo 2 > /proc/sys/vm/drop_caches
# To free slab objects and pagecache:
	echo 3 > /proc/sys/vm/drop_caches
```

来源：[Documentation for /proc/sys/vm/*](https://www.kernel.org/doc/Documentation/sysctl/vm.txt)

但是有时候运行的时候会报错 Permission denied，可以用如下命令：

```bash
sudo sh -c 'echo 3 > /proc/sys/vm/drop_caches'
# Or
echo 3 | sudo tee /proc/sys/vm/drop_caches
```

再使用 `free -h` 命令查看一下内存的使用情况，buff/cache已经清理掉了。

其实可以在 /etc/cron.daily 目录下，创建一个脚本 `/etc/cron.daily/clear_cache.sh`，内容如下：

```bash
#!/bin/sh
sync
echo 3 > /proc/sys/vm/drop_caches
```

然后给这个脚本加上可执行权限： `chmod +x /etc/cron.daily/clear_cache.sh`。

这样就可以每天定时清理缓存了。不过这个脚本会清理掉所有的缓存，包括文件系统缓存和页面缓存，所以在使用时要谨慎。

## 转移到 Cloudflare

2020年7月27日 19:20:00

我在阿里云上创建了这台1C512M的小水管。

<img src="../img/post/2025/05/12/6.png" alt="ECS" width="50%">
*阿里云ECS*
<img src="../img/post/2025/05/12/7.png" alt="ECS" width="50%">
*创建时间*

于2025年5月16日 12:00:00，转移到 Cloudflare。

别了，阿里云尼亚。
