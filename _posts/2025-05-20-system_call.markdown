---
layout: default
title:  "【Linux 从入门到重修C语言】系统调用的一种奇技淫巧"
tags: linux raspberrypi zh-cn
published: true
---

今天<s>手欠</s>手滑在赋权的时候把 `chmod` 的权限改成了 `000`，就在想怎么解决。

突然灵光乍现，没有 `chmod` 程序不是还有 `chmod()` 系统调用么，直接写个 C 代码改一下就完事了：

```c
#include <sys/stat.h>

int main(void)
{
    chmod("/usr/bin/chmod", 0755);
    return 0;
}
```

编译后直接执行就可以了。

```bash
gcc ./mian.c && ./a.out
```

参考[此处](https://man7.org/linux/man-pages/man2/chmod.2.html)。

```
NAME
       chmod, fchmod, fchmodat - change permissions of a file
LIBRARY
       Standard C library (libc, -lc)
SYNOPSIS
       #include <sys/stat.h>

       int chmod(const char *pathname, mode_t mode);
       int fchmod(int fd, mode_t mode);

       #include <fcntl.h>           /* Definition of AT_* constants */
       #include <sys/stat.h>

       int fchmodat(int dirfd, const char *pathname, mode_t mode, int flags);
```

那有大聪明问了，如果 `gcc` 也 `000` 了怎么办？

那直接用 `cc1 + as + ld` 组合就可以了。系统中仍然保留有 `cc1`（GCC 的前端编译器）、`as`（汇编器）和 `ld`（链接器），仍可以手动完成编译流程。

`gcc` 在编译过程中（比如 `gcc shit.c -o shit`）实际上隐式做了如下几步：

- 预处理（cpp）：`shit.c` -> `shit.i`
- 编译（cc1）：`shit.i` -> `shit.s`
- 汇编（as）：`shit.s` -> `shit.o`
- 链接（ld）：`shit.o` -> `shit`

当然，嫌麻烦 `clang` 也可以。

看样子想学好 Linux，得先学明白C。