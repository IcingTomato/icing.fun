---
layout: default
title:  "【博客搭建】Jekyll Learning"
tags: linux jekyll zh-cn
---

Jekyll/Liquid 博客学习记录（连载）

# Jekyll博客配置教程

我是LNMP派，因为我不会用Apache配置。

然而静态博客没必要MySQL, MariaDB, PHP。所以我们只安装Jekyll所有必需的依赖项。

```shell
sudo apt-get install ruby-full build-essential zlib1g-dev nginx
sudo apt-get install gcc g++ make
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update && sudo apt-get install yarn
```

然而Ubuntu 16.04太老了，ruby的版本不支持Jekyll 4。所以我们要手动安装Ruby 3.0.0。

第一步是为Ruby安装一些依赖项。一步一步运行。

```shell
sudo apt install curl
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update
sudo apt-get install git-core zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev software-properties-common libffi-dev nodejs yarn
```

接下来，我们将使用以下三种方法之一安装Ruby。每个都有自己的好处，如今大多数人都喜欢使用rbenv，但是如果您熟悉rvm，也可以按照这些步骤进行操作。我也提供了从源代码安装的说明，但是一般而言，您需要选择rbenv或rvm。

- 方法一：使用rbenv安装。首先安装rbenv，然后安装ruby-build：

```shell
cd
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
exec $SHELL

git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
exec $SHELL

rbenv install 3.0.0
rbenv global 3.0.0
ruby -v
```

- 方法二：RVM安装

```shell
sudo apt-get install libgdbm-dev libncurses5-dev automake libtool bison libffi-dev
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
curl -sSL https://get.rvm.io | bash -s stable
source ~/.rvm/scripts/rvm
rvm install 3.0.0
rvm use 3.0.0 --default
ruby -v
```

- 方法三：Ruby源码安装

```shell
cd
wget http://ftp.ruby-lang.org/pub/ruby/3.0/ruby-3.0.0.tar.gz
tar -xzvf ruby-3.0.0.tar.gz
cd ruby-3.0.0/
./configure
make
sudo make install
ruby -v
```

Ruby服务器在国外，我的博客服务器在国内，所以换个Ruby源

```shell
gem source -r https://rubygems.org/
gem source --add https://gems.ruby-china.com/
gem source -u
```

最后一步是安装Bundler

```shell
gem install bundler jekyll github-pages jekyll-paginate webrick
```

安装完之后就可以将博客从`GitHub`或`Gitee`上`git clone`下来了。

接着完成对`nginx`的配置

安装`nginx`

```shell
sudo apt-get install nginx
```

然后定位到`nginx`的配置文件：

```shell
sudo vim /etc/nginx/sites-enabled/default
```

配置文件在下面：

```shell
pi@ubuntu:~cat /etc/nginx/sites-enabled/default 
server {
	# 监听80端口，我嫌麻烦就懒得加SSL了
	listen 80 default_server;
	listen [::]:80 default_server;
  # 网页在本地的根目录
	root /var/www/html;
  # 显示的主页
	index index.html;
  # 你的域名
	server_name domain.yours;
  # 404，403配置文件
  # 主要目的是当出现404或403时直接回显200
	error_page 404 =200 /404.html;
        # 404
        location /404.html {
                root /var/www/html/;
                internal;
        }        
        # 403
        error_page 403 =200 /offline.html;
        # 404
        location /offline.html {
                root /var/www/html/;
                internal;
        }        
}
```

保存好之后重启`nginx`

```shell
sudo systemctl restart nginx
```

# Jekyll博客维护教程

博客的维护教程修改自 [Hux](https://github.com/Huxpro/huxpro.github.io) 和 [BY_Blog](https://github.com/qiubaiying/qiubaiying.github.io)

## 环境

如果你安装了 [Jekyll](http://jekyllcn.com/)，那你只需要在命令行输入`jekyll serve` 或 `jekyll s`就能在本地浏览器中输入`http://127.0.0.1:4000/`预览主题，对主题的修改也能实时展示（需要强刷浏览器，Ctrl+F5）。

## 开始

你可以通用修改 `_config.yml`文件来轻松的开始搭建自己的博客:

```yml
# Site settings
title: 啥玩意儿啊这                 # 你的博客网站标题
SEOTitle: 啥玩意儿 | What's this    # SEO 标题
description: "Hey"                 # 随便说点，描述一下

# SNS settings      
github_username: null     # 你的github账号

# Build settings
# paginate: 100              # 一页你准备放几篇文章
```

Jekyll官方网站还有很多的参数可以调，比如设置文章的链接形式...网址在这里：[Jekyll - Official Site](http://jekyllrb.com/) 中文版的在这里：[Jekyll中文](http://jekyllcn.com/).

## 撰写博文

要发表的文章一般以 **Markdown** 的格式放在这里`_posts/`，你只要看看这篇模板里的文章你就立刻明白该如何设置。

yaml 头文件长这样:

```yml
---
layout:     post
title:      硕人其颀 衣锦褧衣 
subtitle:   Quark核心板的GPIO推算过程
date:       2021-02-01
author:     null
header-img: img/2021/02/01/01/title.jpg
catalog: true
tags:
    - Quark
    - GPIO
    - Linux
    - Project-Quantum
    - Nano Pi
---
```

## 侧边栏

看右边

设置是在 `_config.yml`文件里面的`Sidebar settings`那块。

```yml
# Sidebar settings
sidebar: true  #添加侧边栏
sidebar-about-description: "简单的描述一下你自己"
sidebar-avatar: /img/avatar-by.jpg     #你的大头贴，请使用绝对地址.注意：名字区分大小写！后缀名也是
```

侧边栏是响应式布局的，当屏幕尺寸小于992px的时候，侧边栏就会移动到底部。具体请见[bootstrap栅格系统](http://v3.bootcss.com/css/)

## Mini About Me

Mini-About-Me 这个模块将在你的头像下面，展示你所有的社交账号。这个也是响应式布局，当屏幕变小时候，会将其移动到页面底部，只不过会稍微有点小变化，具体请看代码。

## Featured Tags

看到这个网站 [Medium](http://medium.com) 的推荐标签非常的炫酷，所以我将他加了进来。
这个模块现在是独立的，可以呈现在所有页面，包括主页和发表的每一篇文章标题的头上。

```yml
# Featured Tags
featured-tags: true  
featured-condition-size: 1     # A tag will be featured if the size of it is more than this condition value
```

唯一需要注意的是`featured-condition-size`: 如果一个标签的 SIZE，也就是使用该标签的文章数大于上面设定的条件值，这个标签就会在首页上被推荐。
 
内部有一个条件模板 `｛% if tag[1].size > ｛｛ site.featured-condition-size｝｝ %｝` 是用来做筛选过滤的.

## Social-media Account

```yml
# SNS settings
RSS: true
bilibili_username:  bilibili_uid 
zhihu_username:     username
facebook_username:  username
github_username:    username
weibo_username:     username
```

## Friends

友链部分。这会在全部页面显示。

设置是在 `_config.yml`文件里面的`Friends`那块，自己加吧。

```yml
# Friends
friends: [
    {
        title: "huohuo",
        href: "http://null.fun/"
    },
    {
        title: "Apple",
        href: "https://apple.com/"
    }
]
```

## Keynote Layout

HTML5幻灯片的排版：

这部分是用于占用html格式的幻灯片的，一般用到的是 `Reveal.js`, `Impress.js`, `Slides`, `Prezi` 等等.我认为一个现代化的博客怎么能少了放html幻灯的功能呢~

其主要原理是添加一个 `iframe`，在里面加入外部链接。你可以直接写到头文件里面去，详情请见下面的yaml头文件的写法。

```yml
---
layout:     keynote
iframe:     "http://huangxuan.me/js-module-7day/"
---
```

iframe在不同的设备中，将会自动的调整大小。保留内边距是为了让手机用户可以向下滑动，以及添加更多的内容。

## Comment

博客不仅支持 [Disqus](http://disqus.com) 评论系统,还加入了 [Gitalk](https://gitalk.github.io/) 评论系统，[支持 Markdwon 语法](https://guides.github.com/features/mastering-markdown/)，cool~

### Disqus

优点：国际比较流行，界面也很大气、简洁，如果有人评论，还能实时通知，直接回复通知的邮件就行了；

缺点：评论必须要去注册一个disqus账号，分享一般只有Facebook和Twitter，另外在墙内加载速度略慢了一点。想要知道长啥样，可以看以前的版本点[这里](http://brucezhaor.github.io/about.html) 最下面就可以看到。

> Note：有很多人反映 Disqus 插件加载不出来，可能墙又架高了，有条件的话翻个墙就好了~

**使用：**

**首先**，你需要去注册一个Disqus帐号。

**其次**，你只需要在下面的 yaml 头文件中设置一下就可以了。

```yml
# 评论系统
# Disqus（https://disqus.com/）
disqus_username: 
```

### Gitalk

优点：界面干净简洁，利用 Github issue API 做的评论插件，使用 Github 帐号进行登录和评论，最喜欢的支持 Markdown 语法，对于程序员来说真是太 cool 了。

缺点：配置比较繁琐，每篇文章的评论都需要初始化。

**使用：**

参考我的这篇文章：[《屡顾尔仆 不输尔载-Jekyll博客迁移计划：Gitalk 插件与 Google Analytics 的配置》](http://panzhifei.fun/2020/08/18/%E5%B1%A1%E9%A1%BE%E5%B0%94%E4%BB%86%E4%B8%8D%E8%BE%93%E5%B0%94%E8%BD%BD/)


## Analytics

网站分析，现在支持百度统计和Google Analytics。需要去官方网站注册一下，然后将返回的code贴在下面：

```yml
# Baidu Analytics
ba_track_id: 
#
# Google Analytics
ga_track_id: 'UA-'            # 你用Google账号去注册一个就会给你一个这样的id
ga_domain:                    # 默认的是 auto, 这里我是自定义了的域名，你如果没有自己的域名，需要改成auto。
```

## Customization

如果你喜欢折腾，你可以去自定义这个模板的 Code。

**如果你可以理解 `_include/` 和 `_layouts/`文件夹下的代码（这里是整个界面布局的地方），你就可以使用 Jekyll 使用的模版引擎 [Liquid](https://github.com/Shopify/liquid/wiki)的语法直接修改/添加代码，来进行更有创意的自定义界面啦！**

## Header Image

博客每页的标题底图是可以自己选的，看看几篇示例post你就知道如何设置了。
  
标题底图的选取完全是看个人的审美了。每一篇文章可以有不同的底图，你想放什么就放什么，最后宽度要够，大小不要太大，否则加载慢啊。

> 上传的图片最好先压缩，这里推荐 imageOptim 图片压缩软件，让你的博客起飞。

但是需要注意的是本模板的标题是**白色**的，所以背景色要设置为**灰色**或者**黑色**，总之深色系就对了。当然你还可以自定义修改字体颜色，总之，用github pages就是可以完全的个性定制自己的博客。

## SEO Title

我的博客标题是 **呐啥** ，但是我想要在搜索的时候显示 **哦豁** ，这个就需要 SEO Title 来定义了。

其实这个 SEO Title 就是定义了 `<head><title>标题</title></head>` 这个里面的东西和多说分享的标题，可以自行修改的。

## 增加阅读时间和字数统计

> 注意：以下代码中出现的全角花括号要改成半角花括号。不要问我为什么，问就是这个影响我全局代码编译。

Displaying a post's word count is rather common when creating a blog, but usually those techniques rely on JavaScript to work. The script reads the post's text, counts the words and displays the result accordingly. That was the way I did things on this blog first as well, but then I set out to find a better way.

### Showing the word count

Luckily Jekyll provides a handy liquid filter called `number_of_words`. So displaying the actual word count is as simple as that:

```yaml
｛｛ page.content | number_of_words ｝｝
```

While this works just nicely it's not very solid. You might want to hide word counts on shorter posts, for example as they're of little value in such posts. This is a little more complex as you can not directly use Liquid filters in a conditional block.

### Variables in Liquid

In Liquid there are two ways to create variables. You can `｛% assign %｝` a variable and you can `｛% capture %｝` a variable. The difference might not be obvious, but it's simple once you get it.

Assigning a value to a variable means that you take any kind of data (e.g. a string, a number, a boolean) and Liquid knows that you want to access that exact data when you refer to this variable. An assigned variable is fixed, that means you can not use the value returned from other Liquid tags.

```yml
｛% assign awesome = true %｝

｛% if awesome %｝
  <p>Yay, awesome!</p>
｛% endif %｝
```

But what if you want to store a Liquid tags's return value in a variable? That's exactly what the `｛% capture %｝` block is for. Unlike assigned variables, captured variables can only hold strings — which will cause us some trouble later on. This is simply because Liquid tags return strings by default.

```yml
｛% capture value %｝
  ｛｛ page.title | upcase ｝｝ from ｛｛ page.date | date: "%b %d, %y" ｝｝
｛% endcapture %｝
```

As you can see in the above example, you can capture any number of strings into a variable, be it strings returned from a Liquid tag or fixed strings.

### Making the word count conditional

Now that you know about `｛% assign %｝` and `｛% capture %｝` we can move on to store our word count in a variable. The question remains, do we assign the variable or do we capture it?

It should be clear by now that we'll have to capture the value as it's returned from a Liquid tag. That gives us something like this:

```yml
｛% capture words %｝
  ｛｛ page.content | number_of_words ｝｝
｛% endcapture %｝
```

Let's say we considered posts that are shorter than 250 words not worthy of getting the word count. A good example for this would be 'link list'-style post that consist of mostly a quote from the original article and a comment spanning a sentence or two. Ideally, this would be taken care of using a simple conditional block.

```yml
｛% if words > 250 %｝
  ｛｛ words ｝｝
｛% endif %｝
```

But you'll soon see that this won't work as intended as Jekyll will throw you this error an error saying you've attempted to compare a string (the words) with a number (250), which is entirely true, and also, sadly, entirely not possible. There is, however, a simple workaround.

```yml
｛% capture words %｝
  ｛｛  page.content | number_of_words | minus: 250 ｝｝
｛% endcapture %｝
｛% unless words contains "-" %｝
  ｛｛  words | plus: 250 ｝｝
｛% endunless %｝
```

You can use Liquid filters to substract your minimum number from the word count to see if it falls below 0. If it does it will contain a '-' at the beginning, which means the post is too short and won't get the word number displayed. If our variable doesn't contain a '-' we can simply add our minimum number back to the word count and display it. Quite simple, right?

### Customising the output

Now that we finally have our word number along with the conditional to hide it from short posts we can move on to make the output a bit nicer. You do this using Liquid filters like `append` or `prepend`. For a complete list of available filters you can check Shopify's [Liquid for Designers guide](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers#standard-filters).

```yml
｛｛  words | plus: 250 | append: " words" ｝｝
```

The above snippet results in something like 'There are 250 words in this post'. You can go crazy with filters, they offer lots of possibilities.

### Calculating the reading time

You might have noticed that I display an estimated reading time on this blog instead of just a word count. Personally, I just think this is a more useful guideline. Doing this is as easy as putting the `divided_by` filter into our final word count construct. The number to divide by is arbitrary, but 180 is the avarage number of words a person reads per minute.

```yml
｛｛  words | plus: 250 | divided_by: 180 | append: " minutes to read" ｝｝
```

### Summing it up

```yml
｛% capture words %｝
  ｛｛  page.content | number_of_words | minus: 250 ｝｝
｛% endcapture %｝
｛% unless words contains "-" %｝
  ｛｛  words | plus: 250 | append: " words" ｝｝
｛% endunless %｝
```

```yml
｛% capture words %｝
  ｛｛  page.content | number_of_words | minus: 250 ｝｝
｛% endcapture %｝
｛% unless words contains "-" %｝
  ｛｛  words | plus: 250 | divided_by: 180 | append: " minute read" ｝｝
｛% endunless %｝
```

### 这是我用的参数

在`/_include`下新建`read_time.html`和`word_count.html`

```yml
<!-- Add Read Time and word count, by chiya 2021.02.06-->
｛% capture words %｝
｛｛  content | number_of_words | minus: 0 ｝｝
｛% endcapture %｝
｛% unless words contains '-' %｝
｛｛  words | plus: 200 | divided_by: 100 | append: ' minute(s)' ｝｝
｛% endunless %｝
```

```yml
<!-- Add Read Time and word count, chiya 2021.02.06-->
｛% capture words %｝
  ｛｛  page.content | number_of_words | minus: 10 ｝｝
｛% endcapture %｝
｛% unless words contains "-" %｝
  ｛｛  words | plus: 10 | append: " words" ｝｝
｛% endunless %｝
```

然后编辑`./_layouts/post.html`，大概在43行处。

```yml
<header class="intro-header" >
    <div class="header-mask"></div>
    <div class="container">
        <div class="row">
            <div class="col-lg-8 col-lg-offset-2 col-md-10 col-md-offset-1">
                <div class="post-heading">
                    <div class="tags">
                        ｛% for tag in page.tags %｝
                        <a class="tag" href="｛｛  site.baseurl ｝｝/tags/#｛｛  tag ｝｝" title="｛｛  tag ｝｝">｛｛  tag ｝｝</a>
                        ｛% endfor %｝
                    </div>
                    <h1>｛｛  page.title ｝｝</h1>
                    ｛% comment %｝
                        always create a h2 for keeping the margin , Hux
                    ｛% endcomment %｝
                    ｛% comment %｝ if page.subtitle ｛% endcomment %｝
                    <h2 class="subheading">｛｛  page.subtitle ｝｝</h2>
                    ｛% comment %｝ endif ｛% endcomment %｝
                    <!-- Add Read Time and word count, by chiya 2021.02.06-->
                    <p class="meta">
                        <span>Posted by ｛% if page.author %｝｛｛  page.author ｝｝｛% else %｝｛｛  site.title ｝｝｛% endif %｝ on ｛｛  page.date | date: "%B %-d, %Y" ｝｝</span>
                        <span>- ｛% include word_count.html %｝, ｛% include read_time.html %｝ to read</span>
                    </p>
                </div>
            </div>
        </div>
    </div>
</header>
```

# 最终使用

如果配置好nginx和Jekyll的话

```shell
cd 文件夹
git clone 仓库地址
git pull origin master
jekyll build -d /var/www/html/
```

现在我单独写了个脚本用于自动拉取编译以及发布：

```shell
#!/bin/bash

cd /var/www/html/
rm -rf /var/www/html/blog/
cd /root/icing.fun
git pull origin master
jekyll build -s /root/icing.fun -d /var/www/html/blog/
```

然后

```shell
chmod +x deploy.sh
rvm cron setup
```

```shell
crontab -e

# 编辑
* * * * * /bin/bash /root/push_blog.sh
```

每分钟就能拉取一次编译发布。
