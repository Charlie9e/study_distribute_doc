---
layout: post
title:  "Spring Boot使用与理解"
date:   2021-04-12 23:24:10 +0800
categories: springboot
---

# 基础问题

## spring boot的功能?
- 嵌入tomcat/jetty容器
- 提供starter，简化构建配置

## spring boot由哪些部分组成？


## spring cloud与spring boot的关系？
- Spring Cloud是一整套基于Spring Boot的微服务解决方案: 配置管理、注册中心、服务发现、限流、网关、链路追踪等

# SSM配置&启动&请求
## 配置
### WEB-INF/web.xml
- 配置DispatcherServlet
- 配置controller/service/dao配置xml路径
### controller层配置
- controller路径scan
- HttpMessageConverter配置
- 基础配置路径
### service层配置
- service路径scan
### dao层配置
- DataSouce/SessionFactory/TransactionManager
## 启动
- 项目打包为war
- tomcat启动
- 查看tomcat启动日志
- 查看war启动日志
## 请求
### tomcat接收请求
- 根据url需要对应war项目
- 调用WEB-INF/web.xml配置的DispatcherServlet
### DispatcherServlet处理请求
- ![](https://raw.githubusercontent.com/Charlie9e/study_distribute_doc/master/_images/DispatcherServlet_Flow.webp)
# SpringBoot配置&启动&请求
## 配置
### 

## 启动

## 请求
### tomcat接口请求

### 



