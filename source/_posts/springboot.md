---
title: springboot入门
date: 2018-08-02 16:13:19
tags: [SpringBoot]
categories: SpringBoot
---

最近几天，学习Spring Boot相关知识，下面就快速搭建Spring Boot项目。
搭建项目工具：Eclipse、maven

创建maven project
---
如下图：
![](/images/2018-8-1/springboot_create_mavenproject.png)
![](/images/2018-8-1/springboot_create_mavenchoose.png)
![](/images/2018-8-1/springboot_create_mavenfinish.png)
项目创建完成。
项目配置
---
**pom文件**
添加spring boot的起步依赖

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

  	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

		<dependency>
	        <groupId>org.springframework.boot</groupId>
	        <artifactId>spring-boot-starter-web</artifactId>
	    </dependency>
    </dependencies>
**程序启动类**
在工程主目录下创建程序启动类——HelloApplication，代码如下：

    package com.luis;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;

    @SpringBootApplication
    public class HelloApplication {
        public static void main(String[] args) {
            SpringApplication.run(HelloApplication.class, args);
        }
    }
其中@SpringBootApplication注解包含@SpringBootConfiguration、@EnableAuto-Configuration、@ComponentScan，用于开启包扫描、配置、自动配置的功能。
**Controller文件**
为演示效果，此web工程，因而需要创建controller,代码如下：

    package com.luis.web;

    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;

    @RestController
    public class HelloController {
        @RequestMapping("/hello")
        public String index() {
            return "Hello World";
        }
	}
其中的@RestController用于表明类为Controller.
工程结构图如图所示：
![](/images/2018-8-1/springboot_project_structure.png)
项目测试
---
运行程序启动类HelloApplication，如下图则启动成功！
![](/images/2018-8-1/springboot_project_begin.png)
启动成功后，在浏览器访问：localhost:8080/hello,浏览器出现“Hello World”,如下图所示：
![](/images/2018-8-1/springboot_show_hello.png)