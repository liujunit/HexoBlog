---
title: SpringBoot学习笔记（二）
date: 2018-11-25 15:08:28
tags:
- SpringBoot
- yaml
categories: 
- SpringBoot学习笔记
---

# SpringBoot的配置文件

SpringBoot支持两种配置文件，一种是我们常用properties文件，另一种是yaml文件，properties配置文件的使用相信大家已经很熟悉了，在这里我们只讨论新增的yaml配置文件。

SpringBoot已经默认配置文件的名称可以为(其中yml的后缀也可以改成yaml)：

> application.yml
>
> application.properties

# yaml配置文件

## 简介

YAML是一另一种标记性语言，这种格式的配置文件更注重的是一数据为中心的进行配置，不像我们以前所用的XXX.xml配置文件有许多的标签格式，YAML更是将空格发挥到了极致的作用，数据层级的关系，以空格和左对齐的方式去判别，例如：

```yaml
server:
  port: 80
```

## yaml语法格式	

### 基本语法

K:(空格)V：用来表示一对键值对，空格必须有

以左对齐加空格的方式去判断是否是同一层级的数据

```yaml
server:
  port: 80
  path: /hello
```

### 值的写法

#### 普通字面量（字符串，数字，布尔值）：

k: v 普通的值以键值对中间加空格的形式就可以，字符串默认不用添加双引号或者单引号的。

双引号：如果添加了双引号，会对双引号中的带有转义的字符进行转义

name: "zhangsan \n lisi"：输出zhangsan 换行 lisi

单引号：如果添加了单引号，单引号中的内容是什么就会输出什么，转义无效

name:'zhangsan \n lisi'：输出zhangsan \n lisi

#### 对象、Map(属性和值) （键值对）：

首行是这个对象或者map的名称，下一行依然是对象和k: v的显示

```yaml
friends:
  name: lili
  age: 20
```

也可以写到一行

```yaml
friends: {name: lili, age: 20}
```



#### 数组（list, set）:

用 -(空格)值 的方式表示数组中的元素

```yaml
pets:
  - cat
  - dog
  - pig
```

行内写法

```yaml
pets: [cat, dog, pig]
```



### 配置文件的注入

yaml：

```yaml
person:
  name: lili
  age: 28
  boss: true
  birth: 1990/02/03
  map:
    k1: v1
    k2: v2
  list:
    - lisi
    - wangwu
    - 24
  dog:
    name: xixi
    age: 3
```

JavaBean：

```java
/**
 * 将配置文件中的每一个配置映射到这个组件中来
 * @ConfigurationProperties 告诉SpringBoot将本类中的数据与配置文件中的数据进行绑定
 * @Component 只有将这个类放到容器中，才能使用@ConfigurationProperties功能
 * @author liujun
 */
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Integer age;
    private boolean boss;
    private Date birth;
    private Map<String, Object> map;
    private List<Object> list;
    private Dog dog;
    ...省略get set toString
```

当我们在写JavaBean添加@ConfigrationProperties注解的时候，如果使用的IDEA，会在这个类的上方出现提醒没有找到配置文件处理器，点击open documents，将处理器引入到pom文件中即可

pom文件添加配置文件处理器

```xml
<!-- 导入配置文件处理器，这样配置文件进行绑定时就会有提示 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

### 测试是否能读取数据

**注意：**自己新建的类的层级关系，SpringBoot的启动类的层级必须大于等于我们所创建的类，不能再启动类的外层新建JavaBean，否则会注入失败。

```java
/**
 * SpringBoot的测试类 测试是否注入成功
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootHelloworld02ApplicationTests {

    @Autowired
    private Person person;

    @Test
    public void contextLoads() {
        System.out.println(person);
    }

}
```



# 后记

时间是挤出来的，这段时间真的是忙997，后面会继续抽时间充电。













