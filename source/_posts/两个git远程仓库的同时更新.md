---
title: 两个git远程仓库的同时更新
date: 2018-12-01 15:34:24
tags: 
- nginx
- git
- SourceTree
categories: 
- 七七八八
---

# 前言

原先的博客是放在github上的，但是github的站点不稳定，经常访问打不开，正好朋友租了个阿里的服务器，想想还是放到朋友的服务器上吧，然后就开始折腾，中途也是遇到了各种坑，搭建nginx，域名备案，gitlib安装，这里就简单的说下，gitlib前两天试着装上了，代码也可以上传了，但是发现这玩意占内存太大了，总是一卡一卡的，很是头疼，然后果断放弃，还是用最原始的git吧，但是也不想放弃github上的更新，那么问题来了，如何本地修改后同时更新两个库呢？

# 问题

## nginx配置

nginx这里的思路是直接监听80端口，然后跳转到博客的静态页面，自己配置了下http连接跳转https，然后发现需要域名备案，就这样来回的折腾，但是https是不需要备案的，这里贴出配置

```
server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  www.liujunit.com liujunit.com;
        rewrite     ^(.*)$ https://www.liujunit.com permanent;
        root         /usr/LjunBlog/public;
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
# Settings for a TLS enabled server.

server {
        listen       443 ssl http2 default_server;
        listen       [::]:443 ssl http2 default_server;
        server_name  www.liujunit.com liujunit.com;
        root         /usr/LjunBlog/public;

        ssl_certificate "/etc/pki/nginx/1547172_www.liujunit.com.pem";
        ssl_certificate_key "/etc/pki/nginx/1547172_www.liujunit.com.key";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

```

https的配置是需要证书的，阿里的域名可以免费申请一个证书的，然后下载下来放到服务器上就可以了,好像有效期是一年吧，完了继续申请就可以了。

## git配置

这里就不说git的安装以及公钥的上传配置，万能的百度和Google会告诉你的，就说下配置远程仓库。

```xml
cd /home/git
git init --bare blog.git
cd blog.git/hooks
vim post-receive
```

git init --bare blog.git创建一个远程裸仓库

然后新建一个配置文件post-receive，相当于指定工作空间吧

```
#!/bin/sh
git --work-tree=/usr/LjunBlog/public --git-dir=/home/git/blog.git checkout -f
```

:wq!保存退出

赋予这个配置文件权限

```
chmod +x post-receive
```

配置到这里基本上就可以了。

## 本地添加远程仓库地址

本人更新git远程仓库代码的时候一直用的是SourceTree，确实好用，选择原先的仓库在设置里添加远程仓库

![](\img\2018-12-01\c59562738e4a5b0caa6b789fd9ee99ae.jpg)

选择刚才新加的远程地址拉取代码

![](\img\2018-12-01\b9209e33627d5c5a9f3c317ebe30206e.png)

选择新增地址推送代

![](\img\2018-12-01\84f0a7e5bcd45d129434a7ce558d486b.png)

基本步骤就是这样，每次推送的时候两个仓库都推送一遍就可以了。

# 后记

一直用SourceTree，基本上就是傻瓜式的操作，看来得好好看看git的一些知识了。









