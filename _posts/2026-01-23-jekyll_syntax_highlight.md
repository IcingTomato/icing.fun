---
layout: default
title:  "【博客搭建】Jekyll 语法高亮配置"
tags: ruby jekyll zh-cn
published: true
---

一个月前看 X 上有人分享了一个很简约的网站，[U.S. Graphics Company](https://usgraphics.com/)，想着自己需要一个简约的主题，于是顺手扒了他们网页研究一下做个 Jekyll 主题出来。前前后后一个月的时间，把主题做出来了。但总感觉差强人意：

> 代码高亮很难受！

代码高亮要做层叠样式，很烦。还好 Jekyll 中默认的语法高亮是 Rouge。

```yaml
markdown: kramdown
kramdown:
  input: GFM
  hard_wrap: false
  syntax_highlighter: rouge
```

搂了一下文档，发现这个可以直接生成层叠样式文件！

首先得确保安装了 Rouge：

```bash
gem install rouge
```

安装完成后，可以看看自带的样式有哪些：

```bash
rougify help style
```

会告诉你有如下样式：

```powershell
PS Z:\tomato\Codes\Web\Blog> rougify help style
usage: rougify style [<theme-name>] [<options>]

Print CSS styles for the given theme.  Extra options are
passed to the theme. To select a mode (light/dark) for the
theme, append '.light' or '.dark' to the <theme-name>
respectively. Theme defaults to thankful_eyes.

options:
  --scope       (default: .highlight) a css selector to scope by
  --tex         (default: false) render as TeX
  --tex-prefix  (default: RG) a command prefix for TeX
                implies --tex if specified

available themes:
  base16, base16.dark, base16.light, base16.monokai, base16.monokai.dark, base16.monokai.light, base16.solarized, base16.solarized.dark, base16.solarized.light, bw, colorful, github, github.dark, github.light, gruvbox, gruvbox.dark, gruvbox.light, igorpro, magritte, molokai, monokai, monokai.sublime, pastie, thankful_eyes, tulip
```

那么有聪明的小伙伴要问了，番茄哥，它那么多样式，到底哪个适合呢？

我推荐个在线预览网站：[Rouge Theme Preview Page](https://spsarolkar.github.io/rouge-theme-preview/)。

看上哪个样式，就直接生成对应的层叠样式文件：

```bash
rougify style base16.solarized > assets/css/syntax-base16-solarized.css
```

比如我就生成了 `base16.solarized` 这个样式的层叠样式文件。

然后就该把生成的层叠样式文件引入到网页中了，在 `_includes/head.html` 里加一行：

```html
<link rel="stylesheet" href="{{ '/assets/css/syntax-base16-solarized.css' | relative_url }}"/>
```

当然，你要是不想生成，可以直接去那个在线预览网站对应的 [GitHub 仓库](https://github.com/spsarolkar/rouge-theme-preview/tree/gh-pages/css)里下载对应的层叠样式文件。

看看效果吧！

##### Java

```java
int total = 0;
for(int i = 0; i < list.length; i++)
{	total += list[i];
System.out.println( list[i] );
}
return total;
```

##### Scala

```scala
  def findNums(n: Int): Iterable[(Int, Int)] = {

    // a for comprehension using two generators
    for (i <- 1 until n;
         j <- 1 until (i-1);
         if isPrime(i + j)) yield (i, j)
  }
```

##### C++
```c++
#include <iostream>
using namespace std;

int main() 
{    
    cout << "Size of char: " << sizeof(char) << " byte" << endl;
    cout << "Size of int: " << sizeof(int) << " bytes" << endl;
    cout << "Size of float: " << sizeof(float) << " bytes" << endl;
    cout << "Size of double: " << sizeof(double) << " bytes" << endl;

    return 0;
}
```

##### Python
```python
# line comment
v = 1
s = "string"

for i in range(-10, 10):
    print(i + 1)

class LinkedList(object):
    def __init__(self, x):
        self.val = x
        self.next = None
```