---
layout: post
title:  "Spring Boot使用与理解"
date:   2021-04-12 23:24:10 +0800
categories: springboot
---
## SSM配置&启动&请求

### 配置
- WEB-INF/web.xml
  - 配置DispatcherServlet
  - 配置controller/service/dao配置xml路径
- controller层配置
  - controller路径scan
  - HttpMessageConverter配置
  - 基础配置路径
- service层配置
	- service路径scan
- dao层配置
	- DataSouce/SessionFactory/TransactionManager

### 开发
- controller->service->dao

### 启动
- 项目打包为war
- tomcat启动
- 查看tomcat启动日志
- 查看war启动日志

### 请求
-  tomcat接收请求
  - 根据url需要对应war项目
  - 调用WEB-INF/web.xml配置的DispatcherServlet
- DispatcherServlet处理请求
	- ![](https://raw.githubusercontent.com/Charlie9e/study_distribute_doc/master/_images/DispatcherServlet_Flow.webp)

## SpringBoot配置&启动&请求

### 配置

#### 引入starter依赖
spring-boot-starter-web(Spring MVC+tomcat)

#### 启动Application配置
@SpringbootApplication
```
等价于以下的三个配置: 
@SpringBootConfiguration # Spring会自动扫描到添加了@Configuration的类，读取其中的配置信息，@SpringBootConfiguration是来声明当前类是SpringBoot应用的配置类，项目中只能有一个

@EnableAutoConfiguration # 开启自动配置，告诉SpringBoot基于所添加的依赖，去“猜测”你想要如何配置Spring

@ComponentScan # 默认从声明这个注解的类所在的包开始，扫描包及子包
```
### 开发
- controller->service->dao

### 启动
- java -jar /usr/src/xxx.jar

### 请求
- tomcat接口请求
	- 调用WEB-INF/web.xml配置的DispatcherServlet
- DispatcherServlet处理请求
	- 同上

### SpringBoot VS SSM
- SpringBoot可以轻易实现SSM的完整功能
- SpringBoot使用更加简洁

### SpringCloud VS SpringBoot
- Spring Cloud是一整套基于Spring Boot的微服务解决方案: 配置管理、注册中心、服务发现、限流、网关、链路追踪等

## SpringBoot启动流程

### Application Context的Bean生命周期
![](https://raw.githubusercontent.com/Charlie9e/study_distribute_doc/master/_images/ApplicationContext_Bean_Lifecycle.jpeg)

### ApplicationContext启动过程
- 概述
主要是通过spring.factories/listener/Initializer来实现starter的自动加载
- 启动过程(简述)
SpringApplication.run  
从spring.factories读取配置 e.g. listener/Initializer......  
listener初始化  
send ApplicationStartingEvent  
应用环境信息封装  
send ApplicationEnvironmentPreparedEvent  
Initializer启动  
send ApplicationContextInitializedEvent  
ApplicationContext准备  
send ApplicationPreparedEvent  
启动tomcat/注册bean......(AnnotationConfigServletWebServerApplicationContext.refresh)  
send ApplicationReadyEvent  
- spring.factories重要配置

通过spring.factories实现对依赖jar的配置进行初始化

```
// 一般用于bean初始化
org.springframework.boot.autoconfigure.EnableAutoConfiguration=
// 在“Initializer启动”这一步启动; 
org.springframework.context.ApplicationContextInitializer=
// 用于监听ApplicationXXXEvent
org.springframework.context.ApplicationListener=
......
```

