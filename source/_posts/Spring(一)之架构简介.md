---
title: Spring(一)之Spring简介
date: 2018-12-27 23:24:19
tags: [Spring]
categories: Spring
---
Spring是一个分层的JavaEE/SE的一站式轻量级开源框架。

## Spring简介

Spring的结构如下图所示：

![1545405107378](/img/2018-12/1545405107378.png)

Spring核心概念介绍：

* **Spring容器：就是IoC容器。**Ioc容器就是BeanFactory工厂（**DefaultListableBeanFactory**）。BeanFactory有一个子接口叫ApplicationContext（应用上下文接口）。

* **IoC（核心）：Inverse of Control，控制反转**。即对象的创建由Spring框架进行。

* **DI：Dependency Injection，依赖注入**。在Spring框架负责创建Bean对象时，动态的将依赖对象注入到Bean组件中。

* **AOP：Aspect Oriented Programming，面向切面编程**。在不修改目标对象的源代码情况下，增强IoC容器中Bean的功能。
  * 面向切面编程，是一种编程思想，有很多的实现：AspectJ、Spring AOP、Spring 整合了AspectJ。

  * AOP的主要作用：横向抽取重复代码，实现代码的重用（事务、日志监控等）。

    * 纵向抽取（继承）

    * 横向抽取（AOP）


## Spring 入口

所谓的spring入口，指的就是如何启动spring容器。

- 基于XML

  - Java应用

    ```java
    ApplicationContext ctx = new ClasspathXmlApplicationContext("spring.xml");
    ```

  - web应用

    **web.xml**

    ```xml
    <context-param>
    	<param-name>contextConfigLocation</param-name>
    	<param-value>classpath:spring.xml</param-value>
    </context-param>
    <listener>
    	<listener-class>
    		ContextLoaderListener
    	</listener-class>
    </listener>
    ```

    ContextLoaderListener监听器中，会去调用getWebApplicationContext()—> AbstractApplicationContext()

- 基于注解

  - Java应用

    ```java
    ApplicationContext ctx = new AnnotationConfigApplicationContext(@Configuration配置类);
    ```

  - web应用

    **web.xml**

    ```xml
    <context-param>
    	<param-name>contextConfigLocation</param-name>
    	<param-value>@Configuration配置类</param-value>
    </context-param>
    <listener>
    	<listener-class>
    		ContextLoaderListener
    	</listener-class>
    </listener>
    ```

    ContextLoaderListener监听器中，会去调用getWebApplicationContext()—> AbstractApplicationContext()

    getWebApplicationContext得到的默认实现类AnnotationConfigWebApplicationContext

