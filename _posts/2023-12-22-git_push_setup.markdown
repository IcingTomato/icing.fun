---
layout: default
title:  "【Git】Git 推送的配置"
tags: git github zh-cn
sticky: true
---

- [`Git` 初次设定](#title1)
- [`Git` 代理设置](#title2)
- [`Git` 回滚操作](#title3)
- [`git push` 到两个地址的仓库](#title4)
- [`Git` 不公开邮箱账户](#title5)
- [`Git` 取消文件跟踪](#title6)
- [`Git` 记住密码](#title7)

## <span id="title1">`Git` 初次设定</span>

```shell
git config --global user.name "IcingTomato"
git config --global user.email ""
```

参阅[1.6 開始 - 初次設定 Git](https://git-scm.com/book/zh-tw/v2/%E9%96%8B%E5%A7%8B-%E5%88%9D%E6%AC%A1%E8%A8%AD%E5%AE%9A-Git)

## <span id="title2">`Git` 代理设置</span>

```shell
git config --global http.proxy http://proxyUsername:proxyPassword@proxy.server.com:port
git config --global https.proxy http://proxyUsername:proxyPassword@proxy.server.com:port
```

## <span id="title3">`Git` 回滚操作</span>

```shell
git reset --hard HEAD^ # 回退到上个版本。
git reset --hard HEAD^ <filename> # 回退指定文件到上个版本。
git reset --hard HEAD^^ # 回退到上上个版本。
git reset --hard HEAD~n # 回退到前n次提交之前，若n=3，则可以回退到3次提交之前。
git reset --hard <commit_sha> # 回滚到指定commit的SHA码，一般使用这种方式。
```

`--hard` 参数撤销工作区中所有未提交的修改内容，将暂存区与工作区都回到上一次版本，并删除之前的所有信息提交。
`--soft` 参数撤销工作区中所有未提交的修改内容，将暂存区与工作区都回到上一次版本，但保留之前的所有信息提交。

回滚完之后就可以

```shell
git push -f origin <branch_name>
``` 

## <span id="title4">`git push` 到两个地址的仓库</span>

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

## <span id="title5">`Git` 不公开邮箱账户</span>

之前貌似更新了什么，好像是将保密你的邮箱地址，并在执行基于 Web WebIDE 的 Git 操作中，使用 xxxx@user.noreply.xxx.com 作为你的邮箱地址。如果你希望命令行 Git 操作使用你的私人邮箱地址，你必须在 Git 中设置你的邮箱地址。

```powershell
# 我经常用 GitHub 的邮箱，全局就设置成这个了
git config --global user.email "xxxx@users.noreply.github.com"

# cd 到 icing.fun 仓库目录下
git config user.email "xxxx@users.noreply.github.com"
git config user.email "xxxx@users.noreply.gitee.com"
```

## <span id="title6">`Git` 取消文件跟踪</span>

参见[【Git】Git 取消文件跟踪](http://icing.fun/2025/05/02/git_rm/)

## <span id="title7">`Git` 记住密码</span>

在使用 `git push` 时，可能会遇到需要输入用户名和密码的情况。为了避免每次都输入，可以使用以下命令来记住密码：

```shell
git config --global credential.helper store
```

这将会在本地存储你的 Git 凭据，之后在推送时就不需要再次输入用户名和密码了。
如果你希望在每次推送时都输入密码，可以使用以下命令：

```shell
git config --global credential.helper cache
```

这将会在内存中缓存你的凭据，默认缓存时间为 15 分钟。
如果你希望更改缓存时间，可以使用以下命令：

```shell
git config --global credential.helper 'cache --timeout=3600'
```

这将会将缓存时间设置为 1 小时（3600 秒）。
