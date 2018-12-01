---
title: SpringBoot学习笔记（三）
date: 2018-12-01 13:30:58
tags:
- SpringBoot
- 注解
categories: 
- SpringBoot学习笔记
---

# SpringBoot配置文件操作

## @Value注解

不管是yaml还是properties配置文件@Value都支持

上一次的时候我们使用的是@ConfigurationProperties进行的全部注入，这次使用@Value

**两者的区别：**

|                             | @ConfigurationProperties | @Value |
| --------------------------- | ------------------------ | ------ |
| 批量注入                    | 支持                     | 不支持 |
| 松散语法                    | 支持                     | 不支持 |
| SpEl（Spring表达式）        | 不支持                   | 支持   |
| JSR303数据校验              | 支持                     | 不支持 |
| 复杂类型封装（map，list等） | 支持                     | 不支持 |

**松散语法：**

假定这个bean中有个属性是lastName，我们配置文件中可以写成last_name或者last-name

**SpEl：**

Spring的表达式，例如下面的#{10*10}输出的注入后就会成为100，“false”直接指定属性boolean类型

**JSR303：**

Spring的数据校验@ConfigurationProperties支持，而使用@Value的时候会跳过校验，使用数据校验的时候添加@Validated注解

```java
/**
 * 将配置文件中的每一个配置映射到这个组件中来
 * @ConfigurationProperties 告诉SpringBoot将本类中的数据与配置文件中的数据进行绑定
 * @Component 只有将这个类放到容器中，才能使用@ConfigurationProperties功能
 * @Validated 数据校验只针对@ConfigurationProperties注解，使用@Value的时候不生效
 * @author liujun
 */
@Component
//@ConfigurationProperties(prefix = "person")
@Validated
public class Person {
    @Value("${person.name}")
    //name必须是邮箱格式才能注入
    @Email
    private String name;
    @Value("#{10*10}")
    private Integer age;
    @Value("false")
    private boolean boss;
    @Value("${person.birth}")
    private Date birth;
    private Map<String, Object> map;
    private List<Object> list;
    private Dog dog;
    ...
```

通过上面的对比我们可以发现@ConfigurationProperties比@Value要好使许多，但是@Value也是有自己的用途的，例如我们要在我们的程序获取配置文件中的一个单个的值，这时候就用@Value要方便很多。

比如下面的代码

```java
@RestController
public class HelloController {

    @Value("${person.name}")
    private String name;

    @RequestMapping("/hello")
    public String hell() {
        return "Hello " + name;
    }

}
```

在浏览器中访问就可以看到name的值。

## @PropertySource注解

@PropertySource用来指定获取拿个配置文件

```properties
person.name=Ljun
person.age=28
person.birth=1990/1/1
person.boss=true
person.map.k1=v1
person.map.k2=12
person.list=1,lilis,nihao
person.dog.name=pipi
person.dog.age=2
```

```java
@PropertySource(value = "classpath:person.properties")
@Component
@ConfigurationProperties(prefix = "person")
//@Validated
public class Person {
//    @Value("${person.name}")
//    @Email
    private String name;
//    @Value("#{10*10}")
    private Integer age;
//    @Value("false")
    private boolean boss;
//    @Value("${person.birth}")
    private Date birth;
    private Map<String, Object> map;
    private List<Object> list;
    private Dog dog;
    ...
```

**注意：**这里的classpath：后面直接写文件名不要加空格，不然会找不到这个文件

## @ImportResource注解

@ImportResource注解用来引入Spring的配置文件，在启动类上添加这个注解就可以了

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="helloService" class="com.liujunit.springboothelloworld02.service.HelloService"></bean>
</beans>
```

```java
/**
 * @ImportResource导入配置文件使其生效
 */
@ImportResource(value = "classpath:beans.xml")
@SpringBootApplication
public class SpringBootHelloworld02Application {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootHelloworld02Application.class, args);
    }
}
```

在测试类中测试是否引入成功

```java
import com.liujunit.springboothelloworld02.bean.Person;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.ApplicationContext;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * SpringBoot的测试类 测试是否注入成功
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootHelloworld02ApplicationTests {

    @Autowired
    private Person person;
    @Autowired
    private ApplicationContext ioc;

    @Test
    public void testBeans() {
        boolean b = ioc.containsBean("helloService");
        System.out.println(b);
    }

    @Test
    public void contextLoads() {
        System.out.println(person);
    }

}
```

**注意：**注入ApplicationContext的时候一定要引用的是Spring的包而不是servelt的

但是SpringBoot不推荐这种方式去引用Spring的配置的文件，而是推荐使用注解的方式@Configuration和@Bean

## @Configuration和@Bean注解

代替@ImportResource注解，注入Spring的配置文件

```java
package com.liujunit.springboothelloworld02.config;

import com.liujunit.springboothelloworld02.service.HelloService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @Configuration添加配置文件
 * 代替原先的beans.xml方式的注入
 */
@Configuration
public class MyAppConfig {

    /**
     * @Bean相当与原先配置文件中的<bean></bean>标签，方法名helloService01相当于bean标签中的id
     * @return
     */
    @Bean
    public HelloService helloService01() {
        System.out.println("配置@Bean给容器中添加组件");
        return new HelloService();
    }

}
```

修改测试类中的获取的id进行测试，注入成功

```java
boolean b = ioc.containsBean("helloService01");
```

## 配置文件中的占位符

```java
${random.value}、${random.int}、${random.long}
${random.int(10)}、${random.int[1024,65536]}
```

```properties
person.name=Ljun${random.uuid}
person.age=${random.int(20)}
person.birth=1990/1/1
person.boss=true
person.map.k1=v1
person.map.k2=12
person.list=1,lilis,nihao
person.dog.name=${person.sex:hello--}pipi
person.dog.age=2
```

person.sex不存在则会引用默认值hello--

# 后记

贵在坚持！





