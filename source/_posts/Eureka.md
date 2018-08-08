---
title: springcloud之服务注册与发现(Eureka)
date: 2018-08-08 16:13:19
tags: [SpringCloud,Eureka]
categories: SpringCloud
---
Eureka简介
---
Eureka是一个用于服务注册和发现的组件，分为服务注册中心Eureka Server和客户端Eureka Client。Spring Cloud Eureka是Spring Cloud Netflix项目下的服务治理模块。而Spring Cloud Netflix项目是Spring Cloud的子项目之一，它为Spring Boot应用提供了自配置的Netflix OSS整合。通过一些简单的注解，开发者就可以快速的在应用中配置一下常用模块并构建庞大的分布式系统。它主要提供的模块包括：服务发现（Eureka），断路器（Hystrix），智能路由（Zuul），客户端负载均衡（Ribbon）等。
**Spring Cloud Eureka的作用：实现服务治理（服务注册与发现）**
Eureka的基本架构
---
Eureka的基本架构图如下图所示：
Register Service:服务注册中心，是Eureka Service,提供服务注册和发现的功能
Provider Service:服务提供者，是Eureka Client,提供服务
Consumer Service:服务消费者，是Eureka Client,消费服务
<img src="/images/2018-7-31/springcloud_Eureka.png" width = "350" height = "200" />
消费服务的基本过程如下：
首先建立服务注册中心Eureka Service，服务提供者Eureka Client向服务注册中心注册，将自己的信息通过REST API的方式提交给Eureka Service。同样的，服务消费者Eureka Client也可以向服务中心注册，同时获取一份服务注册列表信息，该列表包含了所有向服务注册中心注册的服务信息。获取服务注册列表信息之后，服务消费者就知道服务提供者的IP地址，可通过Http远程调度来消费服务提供者的服务。
项目实战
---
下面我们就搭建基于Eueeka的注册服务客户端与服务端。项目采用Maven多Module的结构，即创建一个Maven主项目，其他项目以Module的形式存在。
项目的开发环境:Eclpse、jdk1.8、Maven3.5
项目的结构如下所示：
![](/images/2018-7-31/springcloud_eureka_project_structure.png)
**Maven主工程**
创建Mavne项目eureka-parent，并指定为pom工程。创建项目完成后在pom文件中添加依赖，包括指定Spring Boot版本，jdk版本。pom文件的代码如下：

    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
      <groupId>com.luis</groupId>
      <artifactId>eureka-parent</artifactId>
      <version>0.0.1-SNAPSHOT</version>
      <packaging>pom</packaging>

        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>1.5.2.RELEASE</version>
            <relativePath/> <!-- lookup parent from repository -->
        </parent>

        <properties>
            <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
            <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
            <java.version>1.8</java.version>
            <spring-cloud.version>Dalston.SR1</spring-cloud.version>
        </properties>

        <dependencyManagement>
            <dependencies>
                <dependency>
                    <groupId>org.springframework.cloud</groupId>
                    <artifactId>spring-cloud-dependencies</artifactId>
                    <version>${spring-cloud.version}</version>
                    <type>pom</type>
                    <scope>import</scope>
                </dependency>
            </dependencies>
        </dependencyManagement>
        <modules>
            <module>eureka-server</module>
            <module>eureka-client</module>
        </modules>
    </project>
注意：modules部分创建主项目时不存在，创建modules时会自动加上。

        <modules>
            <module>eureka-server</module>
            <module>eureka-client</module>
        </modules>
Eureka服务端—eureka-server
---
**项目创建**
主Maven项目创建完毕，创建jar的Maven module工程eureka-server，在eureka-server的pom文件中添加Eureka-Server依赖与Spring Boot的测试依赖。具体代码如下：

  	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka-server</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
**服务端搭建**
在src/java/resoures中添加资源文件：application.yml,指定端口为8761

    server:
      port: 8761

    eureka:
      instance:
        hostname: localhost
      client:
        registerWithEureka: false
        fetchRegistry: false
        serviceUrl:
          defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
最后编写启动类EurekaServerApplication.java

    @SpringBootApplication
    @EnableEurekaServer
    public class EurekaServerApplication {
        public static void main(String[] args) {
            SpringApplication.run(EurekaServerApplication.class, args);
        }
    }
**服务端测试**
运行EurekaServerApplication类，运行正常后，在浏览器输入：localhost:8761，若出现如下结果，则成功。
![](/images/2018-7-31/springcloud_eureka_server_show.png)
Eureka客户端—eureka-client
---
**项目创建**
同服务端一样，创建基于父项目的Maven Module子项目eureka-client，创建项目完成后在pom文件中添加依赖，具体代码如下：

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
**客户端搭建**
在src/java/resoures中添加资源文件：bootstrap.yml,指定端口为8762

	server:
      port: 8762
    spring:
      application:
        name: eureka-client

    eureka:
      client:
        serviceUrl:
          defaultZone: http://localhost:8761/eureka/
最后编写启动类EurekaClientApplication.java

    @SpringBootApplication
    @EnableEurekaClient
    public class EurekaClientApplication {

        public static void main(String[] args) {
            SpringApplication.run(EurekaClientApplication.class, args);
        }
    }
**客户端测试**
运行EurekaClientApplication类，运行正常后，刷新浏览器输入：localhost:8761，若出现如下结果，则成功。如图说明Eureka client已经向Eureka Server注册。
![](/images/2018-7-31/springcloud_eureka_client_show.png)
再使用API接口测试，在客户端项目中的controller层新建HiController类，并加上@RestController注解。并用@GetMapping设置Get请求的映射地址为“/hi",具体代码如下所示：

    @RestController
    public class HiController {

        @Value("${server.port}")
        String port;
        @GetMapping("/hi")
        public String home(@RequestParam String name) {
            return "Hi "+name+",I'm from port:" +port;
        }
    }
在浏览器输入：localhost:8762/hi?name=luis,结果如下图所示：
![](/images/2018-7-31/springcloud_eureka_client_web_show.png)

至此，已经成功搭建起Eureka注册中心，后面将会再次基础上进行服务的添加。

本文主要参考自《深入理解Spring Cloud与微服务构建》一书