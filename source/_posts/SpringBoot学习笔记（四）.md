---
title: SpringBoot学习笔记（四）
date: 2019-03-17 15:08:28
tags:
- SpringBoot
- EnableAutoConfiguration
categories: 
- SpringBoot学习笔记
---

# SpringBoot自动配置

## 自动配置的原理

1. SpringBoot在启动的时候加载自动配置，开启自动配置的注解@EnableAutoConfigration

   ![](\img\2019-03-17\a9bc664b-aea8-4669-bca1-418665aa5ec7.png)

2. @EnableAutoConfigration的作用

   - 通过AutoConfigurationImportSelector给容器导入一些组件

   - 可以查看selectImports()方法

   - List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);

     查看候选的配置

     ![](\img\2019-03-17\8ef46a2f-be31-403f-8bad-5bc265affd8a.png)

     查看源码可以看到这里将类路径下的META-INF\spring.factories下的EnableAutoConfiguration的所有的值加入到了容器中

   ![](\img\2019-03-17\f21a137d-9743-4587-b6ca-cc0b215cdf60.png)

   ​	这里的每一个的XXXAutoConfigration类都是容器的一个组件，用他们来做自动配置。

3. 每一自动配置类进行自动配置功能

4. 以HttpEncodingAutoConfigration(HTTP自动编码配置类)为例

   ```java
   @Configuration//表示这是一个配置类，跟以前编写的配置文件一样，也可以给容器中添加组件
   @EnableConfigurationProperties(HttpProperties.class) //启动EnableConfigurationProperties功能，将配置文件中的配置与HttpProperties类绑定起来，同时加载到ioc容器当中去
   @ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)//Spring底层的注解Conditional，判断是不是Web应用程序，如果是的话生效
   @ConditionalOnClass(CharacterEncodingFilter.class)//判断当前的项目中有没有这个类，SpringMVC的进行乱码解决的过滤器
   @ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)//判断配置文件中是否有这个配置，spring.http.encoding.enabled如果不存在判断也是成立的，spring.http.encoding.enabled=true即使不配置也是默认生效的。
   public class HttpEncodingAutoConfiguration {
   	//和springboot的配置文件映射
   	private final HttpProperties.Encoding properties;
   	//只有一个有参构造器，参数的值就会从容器中获取
   	public HttpEncodingAutoConfiguration(HttpProperties properties) {
   		this.properties = properties.getEncoding();
   	}
   
   	@Bean//给容器中添加一个组件，这个组件的值是从properties中获取的
   	@ConditionalOnMissingBean//容器中如果没有组件 添加组件
   	public CharacterEncodingFilter characterEncodingFilter() {
   		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
   		filter.setEncoding(this.properties.getCharset().name());
   		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
   		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
   		return filter;
   	}
   	...省略
   
   }
   ```

   一旦这个配置类生效，就会将properties配置文件中的值配置到对应组件当中去。

5. properties配置文件中的所有配置都有对应的XXXProperties配置类，可以配置哪些属性，可以从对应的类中去查看

   ```java
   @ConfigurationProperties(prefix = "spring.http")//从配置文件中获取值和对应的属性进行绑定
   public class HttpProperties {
   
   	/**
   	 * Whether logging of (potentially sensitive) request details at DEBUG and TRACE level
   	 * is allowed.
   	 */
   	private boolean logRequestDetails;
   
   	/**
   	 * HTTP encoding properties.
   	 */
   	private final Encoding encoding = new Encoding();
   
   	public boolean isLogRequestDetails() {
   		return this.logRequestDetails;
   	}
   
   	public void setLogRequestDetails(boolean logRequestDetails) {
   		this.logRequestDetails = logRequestDetails;
   	}
   
   	public Encoding getEncoding() {
   		return this.encoding;
   	}
   
   	/**
   	 * Configuration properties for http encoding.
   	 */
   	public static class Encoding {
   		//默认配置编码是UTF-8
   		public static final Charset DEFAULT_CHARSET = StandardCharsets.UTF_8;
   
   		/**
   		 * Charset of HTTP requests and responses. Added to the "Content-Type" header if
   		 * not set explicitly.
   		 */
   		private Charset charset = DEFAULT_CHARSET;
   
   		/**
   		 * Whether to force the encoding to the configured charset on HTTP requests and
   		 * responses.是否在http请求和响应上进行强制编码
   		 */
   		private Boolean force;
   
   		/**
   		 * Whether to force the encoding to the configured charset on HTTP requests. 如果没有设定强制编码force，默认是true
   		 * Defaults to true when "force" has not been specified.
   		 */
   		private Boolean forceRequest;
   
   		/**
   		 * Whether to force the encoding to the configured charset on HTTP responses.
   		 */
   		private Boolean forceResponse;
   
   		/**
   		 * Locale in which to encode mapping.
   		 */
   		private Map<Locale, Charset> mapping;
   		...省略
   	}
   
   }
   ```

**总结**：

1. **SpringBoot启动的时候会加载大量的自动配置类**
2. **我们可以查看我们需要的配置类是否已经写好了**
3. **可以查看配置类中配置了哪些组件，只要我们要用的组件有就不需要再来配置**
4. **我们可以在properties中配置这些属性，从对应的XXXProperties类中查看有哪些属性可以配置**

[官方参考配置](https://docs.spring.io/spring-boot/docs/2.1.3.RELEASE/reference/htmlsingle/#appendix)

## Spring的原生注解@Conditional

SpringBoot中的只要有@Conditional扩展的地方说明必须条件满足，才能给容器中添加组件，配置里的能容才会生效

| @Conditional扩展注解            | 作用（判断条件是否满足）                           |
| ------------------------------- | -------------------------------------------------- |
| @ConditionalOnJava              | 系统的java版本是否符合要求                         |
| @ConditionalOnBean              | 容器中存在指定的Bean                               |
| @ConditionalOnMissingBean       | 容器中不存在指定Bean                               |
| @ConditionalOnExpression        | 满足SpEL表达式                                     |
| @ConditionalOnClass             | 系统中有指定的类                                   |
| @ConditionalOnMissingClass      | 系统中没有指定的类                                 |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选的Bean |
| @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                     |
| @ConditionalOnResource          | 类路径下是否有指定                                 |
| @ConditionalOnWebApplication    | 当前是web环境                                      |
| @ConditionalOnNotWebApplication | 当前不是web环境                                    |
| @ConditionalOnJndi              | JNDI存在指定项                                     |

自动配置必须是一定的条件才能生效，可以通过properties设置debug=true，让控制台打印查看生效的配置类。

```java
============================
CONDITIONS EVALUATION REPORT
============================


Positive matches://生效的配置类
-----------------

   CodecsAutoConfiguration matched:
      - @ConditionalOnClass found required class 'org.springframework.http.codec.CodecConfigurer' (OnClassCondition)

   CodecsAutoConfiguration.JacksonCodecConfiguration matched:
      - @ConditionalOnClass found required class 'com.fasterxml.jackson.databind.ObjectMapper' (OnClassCondition)

   CodecsAutoConfiguration.JacksonCodecConfiguration#jacksonCodecCustomizer matched:
      - @ConditionalOnBean (types: com.fasterxml.jackson.databind.ObjectMapper; SearchStrategy: all) found bean 'jacksonObjectMapper' (OnBeanCondition)

   DispatcherServletAutoConfiguration matched:
      - @ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet' (OnClassCondition)
      - found 'session' scope (OnWebApplicationCondition)
	...省略

Negative matches://没有生效的配置类
-----------------

   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'javax.jms.ConnectionFactory' (OnClassCondition)

   AopAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'org.aspectj.lang.annotation.Aspect' (OnClassCondition)

   ArtemisAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'javax.jms.ConnectionFactory' (OnClassCondition)
	...省略
```

# 后记

2019每周至少一篇，不能再懒散下去了，加油吧！