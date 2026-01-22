---
layout: default
title:  "【Blog Build】Plan of Jekyll Blog - Episode 1: Jekyll-3 on Ubuntu and Nginx"
tags: linux en-us
---

This tutorial you will learn how to configure Jekyll-3 on Ubuntu 16.04 and Nginx.

# Jekyll On Ubuntu

The release of the Ubuntu is [Ubuntu 16.04 LTS (Xenial Xerus)](https://ubuntu.com/16-04).

You can follow this link [Jekyll on Ubuntu](https://jekyllrb.com/docs/installation/ubuntu/) to configure Jekyll.

Before we install Jekyll, we need to install Ruby and other dependencies:

```shell
sudo apt-get install ruby-full build-essential zlib1g-dev
```

<strong>Caution: </strong>`sudo apt-get install ruby-full` command on Ubuntu 16.04 even 18.04 can only install Ruby2.5, it might take some unsuspected problem.

Avoid installing RubyGems packages (called gems) as the root user. (Well, AliCloud only provides `root` account and I am lazy about adding a new user account. So I think it doesn't matter to use `root` account.) Instead, set up a gem installation directory for your user account. The following commands will add environment variables to your `~/.bashrc` file to configure the gem installation path:

```shell
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Finally, install Jekyll and Bundler:

```shell
gem install jekyll bundler
```

You can make a quick start for surfing the Jekyll blog:

```shell
jekyll new my-awesome-site
cd my-awesome-site
bundle exec jekyll serve
# => Now browse to http://localhost:4000
```

Or, clone the repository from GitHub and build Jekyll blog:

```shell
git clone https://github.com/IcingTomato/blog.old.git
cd blog.old
jekyll serve
# => Now browse to http://localhost:4000
```

# Liquid warning

If you follow the step, congratulation, you cannot see any blog in your website.

Just like this:

![](../img/post/2020/05/04/unsuspected_error.jpg)

Well, because Ruby2.5 made its Bundler incompatible with Jekyll-4.0, so we have to uninstall Jekyll-4.0 and install the previous versions.

```shell
gem uninstall jekyll
gem install jekyll -v 3.1.3
```

# Jekyll Serve

`jekyll serve` will publish your site on localhost:4000. But you can change the port:

```shell
jekyll serve --port 4001
```

But you can't visit it on public network. So you can:

```shell
jekyll serve -w --host=0.0.0.0
```

After doing these jobs, we can't visit our website ethier.

# Nginx Configuration

Because Jekyll uses port 4000, so we will make a Nginx "proxy" from port 4000 to port 80.

First, install Nginx:

```shell
sudo apt-get install nginx
```

Then, use your favourite text editor to edit `/etc/nginx/conf.d/jekyll.conf`:

```shell
root@blog:~# sudo nano /etc/nginx/conf.d/jekyll.conf

server {
    listen 80;
    server_name your.domain;
    location / {
        proxy_pass http://localhost:4000;
    }
}
```

![](../img/post/2020/05/04/your_domain.jpg)

Finally, `Ctrl+O` and `Ctrl+X` to save and exit, then restart `nginx.service`:

```shell
sudo systemctl restart nginx
jekyll serve --detach
```
