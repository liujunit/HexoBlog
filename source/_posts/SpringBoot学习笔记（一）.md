---
title: SpringBoot学习笔记（一）
date: 2018-11-18 17:24:02
tags: 
- SpringBoot
- Maven 
categories: 
- SpringBoot学习笔记
---

# Spring Boot 简介

> 简化Spring应用开发的一个框架；
>
> 整个Spring技术栈的一个大整合；
>
> J2EE开发的一站式解决方案；

​	微服务是一种架构风格，每一个单个应用应该是一个小型服务，每个应用之间通过http进行相互之间的通信，这样每个单独的应用可以进行独立的升级和替换而不影响整体的功能。

​	SpringBoot简化了开发的繁琐的环境搭建，以及应用部署，我们不必过多的关注环境的准备以及上线时服务器环境的配置，SpringBoot可以将程序打成jar包，服务器上只要使用java -jar的命令就能将程序启动起来，而不必过多的去配置tomcat，因为SpringBoot已经将tomcat集成到了程序里。

# 环境准备

> jdk1.8	SpringBoot官方要求jdk的版本不得低于1.7
>
> maven3.X	maven3.3及以上版本
>
> IntelliJIDEA2018	开发工具用的IDEA，目前应该算是最流行也是最好用的，版本只要是16以上的都没问题
>
> SpringBoot2.1.0.RELEASE	2.1.0我这里用的是最新的版本

## maven配置

### 配置阿里云镜像

​	引入大批量的jar没有国内镜像，这个下载的过程总是痛苦的，所以我推荐大家最好还是用阿里的镜像，体验下什么是波音747的感觉。在mirrors标签里添加如下：

```xml
<mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>central</mirrorOf>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```



### 配置profiles

​	添加本地的运行打包环境, 在profiles中添加如下：

```xml
<profile>
    <id>jdk-1.8</id>
    <activation>
        <!-- 激活配置的环境 -->
        <activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
    </activation>
    <properties> 
        <maven.compiler.source>1.8</maven.compiler.source> 
        <maven.compiler.target>1.8</maven.compiler.target> 
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion> 
    </properties>
</profile>
```

## IDEA的设置

​	我们使用的IDEA可以在默认配置中修改关于maven的引用，将自带的maven替换成我们自己的，首先寻找默认配置：

![](\img\2018-11-21\8718cb52753c4883b5c87f02d9718b5e.png)

修改默认maven配置点击保存：

![](\img\2018-11-21\d3c5d3085cb844248874b98e2c9584da.png)



# 两种快速搭建的方式

​	快速搭建有两种方式，一种是本地自己手动maven构建，一种是从官方下载模板构建

## 本地maven构建

​	创建一个maven工程，不使用任何的预设好的原型

![](\img\2018-11-21\df29629f35ae4497a58fca23f950e5be.png)

​	maven中添加依赖，指定版本，从导入的jar包中可以看出SpringBoot已经将我们平常开发所需要的基本的jar包都添加到了父依赖当中，省去了我们不必要的一些整合，包扩一些默认的配置（例如tomca的端口号），在以后的文章中会提到如何如何修改这些默认配置。

```xml
<!-- Inherit defaults from Spring Boot 导入父依赖-->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.0.RELEASE</version>
</parent>
<!-- Add typical dependencies for a web application 导入我们需要的模块-->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
<!-- Package as an executable jar 导入maven打包的插件-->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

​	编写启动类

```java
package com.liujunit;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @SpringBootApplication 用来标注这是一个SpringBoot的应用程序
 */
@SpringBootApplication
public class HelloWorldMainApplication {

    public static void main(String[] args) {
        //用来启动这个应用程序
        SpringApplication.run(HelloWorldMainApplication.class, args);
    }

}
```

​	写一个HelloController来进行测试

```java
package com.liujunit.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloController {

    @RequestMapping("/hello")
    @ResponseBody
    public String hello() {
        return "hello";
    }

}
```

​	运行启动类，浏览器访问http://localhost:8080/hello

![](\img\2018-11-21\ac3e84dfa5cb49cb83c5a4bfc3aaf73b.png)

​	是不是很快！！！在这整个的过程中我们没有进行任何的配置，默认启用tomcat的8080端口。

## 使用Spring Initializr联网构建

​	IDEA新建项目的时候选用Spring Initializr，默认官方网址就可以了

![](\img\2018-11-21\6468595841f846b0b09d88b918ab0741.png)

​	填写组织和项目标识就可以了，其他默认，打包方式是jar包的形式

![](\img\2018-11-21\c3dc6480e71847bf870e2bf17ea71942.png)

​	这里我们初步只勾选web就可以了，这里面有许多我们可以预选择的一些集成

![](\img\2018-11-21\6155b7e779ee40aea4615a92b4ecba03.png)

​	删掉一些没有用的文件，可以看下目录结构都给我们建好了包扩启动类，其中resource下的static中主要放一些前台静态文件，templates主要放一些前端的模板，SpringBoot不支持jsp，推荐使用thymeleaf 或者freemarker。

![](\img\2018-11-21\5be70f95253d4f2690f86f76488f8757.png)

​	同第一种方式一样，运行启动，在浏览器看到同样的结果, 而且控制台是彩色的有木有很装逼的样子。

# 打包部署

​	打包部署更是简单，点击package就会在target文件下生成一个jar包。

![](\img\2018-11-21\6bd57a4467eb43efa8f7a410d7f5114e.png)

​	将这个jar包粘贴到任意文件夹下使用命令行java -jar的就可以启动，真的简单好用。

# 后记

​	SpringBoot是趋势，觉得还是很有必要学一学的。后面会继续更新，图片粘贴复制真的麻烦。