---
layout: post
title:  "【Git】Git 推送的配置"
tags: git github zh-cn
---

## `Git` 代理设置

```shell
git config --global http.proxy http://proxyUsername:proxyPassword@proxy.server.com:port
git config --global https.proxy http://proxyUsername:proxyPassword@proxy.server.com:port
```

## `git push` 到两个地址的仓库

好比我这个博客仓库，在国内因为某些不可抗力，`github` 无法访问，所以我在 `gitee` 上也有一个仓库，这样就可以在国内访问了。但是我每次都要推送两次，很麻烦。或者要登陆 `gitee` 的仓库，然后按一下同步按钮，也很麻烦。

所以我就想，能不能只推送一次，然后两个仓库都更新呢？

```powershell
# 原始推送地址 https://github.com/IcingTomato/icing.fun.git

# 添加第二个推送地址
git remote set-url --add origin https://gitee.com/IcingTomato/icing.fun.git

# 查看推送/拉取地址
git remote -v

# origin  https://github.com/IcingTomato/icing.fun.git (fetch)
# origin  https://github.com/IcingTomato/icing.fun.git (push)
# origin  https://gitee.com/IcingTomato/icing.fun.git (push)

# 推送
git push 
```

## `Git` 不公开邮箱账户

之前貌似更新了什么，好像是将保密你的邮箱地址，并在执行基于 Web WebIDE 的 Git 操作中，使用 xxxx@user.noreply.xxx.com 作为你的邮箱地址。如果你希望命令行 Git 操作使用你的私人邮箱地址，你必须在 Git 中设置你的邮箱地址。

```powershell
# 我经常用 GitHub 的邮箱，全局就设置成这个了
git config --global user.email "xxxx@users.noreply.github.com"

# cd 到 icing.fun 仓库目录下
git config user.email "xxxx@users.noreply.github.com"
git config user.email "xxxx@users.noreply.gitee.com"
```