---
title: hexo（jacman主题）修改导航链接为绝对路径
date: 2018-11-08 22:04:24
tags: 
- jacman 
- hexo 
- 导航链接 
- 绝对路径
categories: 
- 七七八八
---
# 起始
        一直想自己搭建一个博客，但是没想到坑是这么的多，磕磕绊绊总算是可以开始写了，本人做java开发的，博客也是
    作为简单的一个学习记录吧。从简单的事情做起,闲下来就会写一写，也算是督促自己吧。
# 关于jacman导航栏链接的修改
        hexo提倡使用绝对路径，但是我想在我的导航栏里面加上一些自己的链接，配置了_config.yml
```
        menu:
            主页: /
            归档: /archives
            GitHub: https://github.com/liujunit
            CSDN: https://blog.csdn.net/liujun_for_java
            关于: /about
```
        但是这样的配置没有生效，跳转链接的时候还是会在GitHub前加上一个斜杠/
    于是就是开始了漫漫的搜索调试路，无论是百度还是google都找不到自己满意的答案
    索性自己开始查找修改jacman主题下文件，还是真实找到了
# 修改的文件
        修改jacman文件夹下的themes\jacman\layout\_partial\header.ejs文件头部
```
        function root_for(path){
            path = path || '/'
            var root = config.root;
            if (path.substring(0, root.length) !== root){
                if (path.substring(0, 1) === '/'){
                path = root.substring(0, root.length - 1) + path;
                } else if(path.indexOf('http') == 0){
                    path = path;
                } else {
                    path = root + path;
                }
            }
        return path;
        }
```
        原先的js代码是没有对http协议是否包含的判断的，自己加个判断路径不变就可以了
# 后记
        可能大家都是高手高手高高手，没有遇到过这个问题（或许遇到也自己解决了）。。。希望遇到的能有幸看到这篇文正吧，
    样式后续慢慢调整吧。