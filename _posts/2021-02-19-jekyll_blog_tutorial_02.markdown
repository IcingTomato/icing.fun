---
layout: default
title:  "【Blog Build】Plan of Jekyll Blog - Episode 2: Ruby-3, Jekyll-4, Git, Ubuntu and Nginx"
tags: linux en-us
---

This tutorial you will learn how to configure Jekyll-4 on Ubuntu 16.04/18.04.

# Catalogue

- [Jekyll-4 Installation Ubuntu 16.04](#1)
- [Nginx Setup](#2)
- [GitHub Local Repository Setup](#3)
- [Git roll back to some commit](#4)
- [Jekyll Maintaining Tutorial](#5)
- [Jekyll Code Block for Liquid-Like](#6)
- [Finally...](#7)
- [My Second Jekyll Blog](#8)
- [Git Dual Repository Setup](#9)

Here is the blog repository: [blog.old](https://github.com/IcingTomato/blog.old)/[blog.old3](https://github.com/IcingTomato/blog.old3)

# <span id="1">Jekyll-4 Installation Ubuntu 16.04</span>

First, we should install Ruby3.0.0 for Ubuntu 16.04/18.04:

```shell
sudo apt-get install ruby-full build-essential zlib1g-dev nginx
sudo apt-get install gcc g++ make
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update && sudo apt-get install yarn
sudo apt install curl
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

sudo apt-get update
sudo apt-get install git-core zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev software-properties-common libffi-dev nodejs yarn
```

Then, we can use <strong>one of these methods</strong> to install Ruby-3:

1. Install `rbenv`

```shell
cd ~
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

2. Install `RVM`

```shell
sudo apt-get install libgdbm-dev libncurses5-dev automake libtool bison libffi-dev
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
curl -sSL https://get.rvm.io | bash -s stable
source ~/.rvm/scripts/rvm
rvm install 3.0.0
rvm use 3.0.0 --default
ruby -v
```

3. Ruby Source Codes Installation

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

<strong>Caution: </strong>In some area, it's difficult to visit RubyGems, so you might be change a Ruby Source.

```shell
gem source -r https://rubygems.org/
gem source --add https://gems.ruby-china.com/
gem source -u
```

Finally, install `Bundler`

```shell
gem install bundler jekyll github-pages jekyll-paginate webrick
```

After installation, you can clone your repository from `GitHub` or `GitLab`.

```shell
git clone https://github.com/IcingTomato/blog.old.git
```

# <span id="2">Nginx Setup</span>

Well, you can also use `Apache` which use `sudo apt-get install apache2` on Debian/Ubuntu or `sudo yum install httpd` on CentOS/RHEL/Fedora.

First, install Nginx:

```shell
sudo apt-get isntall nginx
```

Then, configure nginx profile:

```shell
root@blog:~# sudo nano /etc/nginx/sites-enabled/default

server {
        listen 80 default_server;
        listen [::]:80 default_server;

        # SSL configuration
        #
        listen 443 ssl default_server;
        listen [::]:443 ssl default_server;

        # Disable SSL, because when it enabled, it blocked 80.
        #ssl on;
        ssl_certificate path/to/your_key.pem;
        ssl_certificate_key path/to/your_key.key;
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;

        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html;

        server_name your.domain;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }


        # Rewrited the 404 page to 200
        error_page 404 =200 /404.html;

        location /404.html {
                root /var/www/html/;
                internal;
        }        

        # Rewrited the 403 page to 200
        error_page 403 =200 /403.html;

        location /403.html {
                root /var/www/html/;
                internal;
        }
}
```

`Ctrl+O` and `Ctrl+X` to save and exit.

Finally, restart `nginx.service`

```shell
sudo systemctl restart nginx

# On AWS, you must use systemctl to start, stop, enable, disable, restart a service.
```

# <span id="3">GitHub Local Repository Setup</span>

```shell
git init
git add README.md
git commit -m "first commit"
git branch -M master
git remote add origin repo_address
git push -u origin master
```

# <span id="4">Git roll back to some commit</span>

```shell
git reset --hard 某个commit id
git push -f -u origin master
```

# <span id="5">Jekyll Maintaining Tutorial</span>

You can make a quick start for surfing the Jekyll blog:

```shell
git clone -b blank https://github.com/IcingTomato/blog.old.git
cd blog.old
jekyll serve # jekyll s
# => Now browse to http://localhost:4000
```

![quick_start](../img/post/2021/02/19/quick_start.jpg)

## Setup `_config.yml`

```yml
# Site settings
title: Your Blog                        # Your blog title
SEOTitle: Jekyll | Ruby                 # SEO title
description: "Hello from Seattle"	    # Your blog description

# SNS settings      
github_username: John Doe               # GitHub account
RSS: true                               # RSS On
weibo_username:                         # Weibo  
zhihu_username:                         # Zhihu
facebook_username:                      # Facebook
bilibili_username:                      # Bilibili  
twitter_username:                       # Twitter

# Build settings
paginate: 100                          # Contents in a page
```

You can visit [Jekyll-Docs](https://jekyllrb.com/docs/)/[Jekyll中文网-文档](http://jekyllcn.com/docs/home/) for more support.

### Sidebar

On PCs, tablet devices or ultra-wide scale screen there is a sidebar to display your personal profile.

```yml
# Sidebar settings
sidebar: true                                           # Add Sidebar
sidebar-about-description: "describe yourself"          # Description
sidebar-avatar: /img/avatar.jpg                         # Your Avatar
```

<strong>Sidebar</strong> is a Responsive Layout*, when the display size is below 992px, the sidebar will move to the page bottom. You can visit [Bootstrap3 for global CSS settings](https://getbootstrap.com/docs/3.4/css/)/[Bootstrap 设置全局 CSS 样式](https://v3.bootcss.com/css/)

### Mini About Me

This module is under your avatar. It will display all your social media account. It is also a Responsive Layout. When the screen size become small, it will move to the bottom of the page. But there is a litte change when it move to the bottom.

![mini_about_me](../img/post/2021/02/19/mini_about_me.jpg)

![mini_about_me_bottom](../img/post/2021/02/19/mini_about_me_bottom.jpg)

### Featured Tags

```yml
# Featured Tags
featured-tags: true  
featured-condition-size: 15     
# A tag will be featured if the size of it is more than this condition value
```

`featured-condition-size` means if the tags' count over 15(or the number you set), it will display on home page.

![tags](../img/post/2021/02/19/tags.jpg)

### Friends

```yml
# Friends
friends: [
    {
        title: "John Doe",
        href: "http://john.doe/"
    },{
        title: "村財　師内",
        href: "https://sonzai.shinai/"
    },{
        title: "鵜祖田　陽",
        href: "https://usoda.you/"
    }
]
```

### Comment

This Jekyll Blog supports both [Disqus](http://disqus.com/) and [Gitalk](https://gitalk.github.io/), it also supports Markdown Grammar.

#### Disqus

```yml
# Disqus（https://disqus.com/）
disqus_username:
```

#### Gitalk

First, we should create a new repository for saving our comments:

![gitalk_repo_create](../img/post/2021/02/19/gitalk_repo_create.jpg)

![gitalk_repo_setup](../img/post/2021/02/19/gitalk_repo_setup.jpg)

And then open `Issues` option:

![open_issues](../img/post/2021/02/19/open_issues.jpg)

Besides, register a new OAuth application:

![oauth_setup](../img/post/2021/02/19/oauth_setup.jpg)

![oauth_id](../img/post/2021/02/19/oauth_id.jpg)

### Google Analytics

```yml
# Google Analytics
ga_track_id: 'UA-'
ga_domain:         # Default as 'auto', you can use your domain
```

![google_analytics](../img/post/2021/02/19/google_analytics.jpg)

## Write your first blog

We must put the article, which named as `YYYY-MM-DD-your_first_article.markdown`, in `_post` folder. And you can change parameter in the front of the article:

```yml
---
layout:     post
title:      Your First Blog 
subtitle:   Jekyll is Хорошо
date:       1900-01-01
author:     John Doe
header-img: img/title.jpg
catalog: true
tags:
    - Jekyll
---
```

# <span id="6">Jekyll Code Block for Liquid-Like</span> 

In Jekyll, some Ruby code block can't generate perfectly.

So, we can use [raw tag](https://shopify.github.io/liquid/tags/template/) to avoid execute liquid or liquid-like code block:

`{% raw %}{% raw %}{% endraw %}`

You can use `{% raw %}{% highlight ruby %}{% endhighlight %}{% endraw %}` for code highlight.

# <span id="7">Finally...<span>

```shell
cd your.folder
git clone repo.address
git pull origin master
jekyll build -d /var/www/html/
```

# <span id="8">My Second Jekyll Blog</span>

Here is the blog repository: [blog.old2](https://github.com/IcingTomato/blog.old2)

You can use these command line for configuring the blog:

```shell
cd <the folder>
git clone https://github.com/IcingTomato/blog.old2.git
bundle add webrick rake
bundle install --path vendor/cache
git pull origin master
bundle exec jekyll build -d /var/www/html/
```

First, you need to install some Dependence like **webrick** **rake** **jekyll**.

And then, in my case, I had a PATH problem.

```
Bundler::GemNotFound: Could not find rake-10.3.2 in any of the sources
~/.rvm/gems/ruby-2.0.0-p451/gems/bundler-1.6.2/lib/bundler/spec_set.rb:92:in `block in materialize'
~/.rvm/gems/ruby-2.0.0-p451/gems/bundler-1.6.2/lib/bundler/spec_set.rb:85:in `map!'
~/.rvm/gems/ruby-2.0.0-p451/gems/bundler-1.6.2/lib/bundler/spec_set.rb:85:in `materialize'
~/.rvm/gems/ruby-2.0.0-p451/gems/bundler-1.6.2/lib/bundler/definition.rb:133:in `specs'
~/.rvm/gems/ruby-2.0.0-p451/gems/bundler-1.6.2/lib/bundler/definition.rb:178:in `specs_for'
Show 28 more lines
```

I used `gem install rake` but it was no use fixing the matter.

SO, `bundle install --path vendor/cache` can help you fix the problem.

This command line generally fixes it as that is the more common problem. Basically, my bundler path configuration is messed up. See their [documentation](https://bundler.io/v1.11/man/bundle-config.1.html) (first paragraph) for where to find those configurations and change them manually if needed.

# <span id="9">Git Dual Repository Setup</span>

Everytime I finish my blog, I have to push to two different platform: [GitHub](https://github.com/) and [Gitee](https://gitee.com/).

[Gitee](https://gitee.com/) is a collaboration platform for software development & code hosting in China Mainland.

Because I use Ali ECS Area Hangzhou, it's hard to visit GitHub, so I have to use Gitee.

So

```shell
cd 'your-repository-path'
git remote add gitee 'your-gitee-repository-address'
git push gitee master    # Push to 'gitee'
git push origin master   # Push to 'github' 
```

You can also change `your-repository/.git/config`

```
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[remote "origin"]
	url = 'your-github-repository-address'
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
[remote "gitee"]
	url = 'your-gitee-repository-address'
	fetch = +refs/heads/*:refs/remotes/gitee/*
```

