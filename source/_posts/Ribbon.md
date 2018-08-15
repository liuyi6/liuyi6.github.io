---
title: springcloud之负载均衡(Ribbon)
date: 2018-08-15 22:13:19
tags: [SpringCloud]
categories: SpringCloud
---
RestTemplate简介
---
RestTemplate是Spring Resources中访问第三方RESTful API接口的网络请求框架。RestTemplate是是用来消费REST服务的，所以RestTemplate主要方法与参与REST的Http协议的方法紧密连接，如headForHeaders(),getFoeObject(),postForObject(),put(),delete()等。
RestTemplate很容易构建RESTful API,它支持常见的Http协议请求方法，如Put,Post,Delete等。它支持Xml、JSON数据格式，默认实现了序列化，克自动将JSON字符串转换为实体。如下为示例代码：

	User user = restTemplate.getForObject("https://www.xxx.com/",User.class);
Ribbon简介
---
负载均衡是指将负载分摊到多个执行单元上常见的负载均衡有：一是独立进程单元，通过负载均衡策略，将请求转发到不同的执行单元上，如Niginx。二是降幅在均衡逻辑以代码的方式封装到服务消费者客户端将消费正客户端维护的服务提供者信息列表通过负载均衡策略分摊给多个服务提供者，从而实现负载均衡。Ribbon属于第二种，它是Netflix公司开源的负载均衡组件。它将负载均衡逻辑封装在客户端，运行在客户端进程中。Ribbon死一个经过云端测试的额IPC库，可以很好的控制HTTP和TCP客户端的负载均衡行为。
Spring Cloud微服务系统中，Robbin作为服务消费者的负载均衡器，有两种使用方式：一是与RestTemplate相结合，二是和Feign相结合，Feign已经默认集成了Ribbon.
目前Ribbon的子模块如下：

 1. ribbon-loadbalancer:可以独立使用或与其他模块一起使用的负载均衡API
 2. ribbon-eureka:Ribbon结合Eureka客户端API，为负载均衡器提供动态服务注册列表信息。
 3. ribbon-core:Ribbon的核心API。

项目实战
---
此项目依旧采用多模块的项目组成方式，首先依照eureka的过程搭建起eureka-server服务端与eureka-client客户端，具体参照上一节。
在上节项目基础上，新建eureka-ribbon-client，作为服务消费者，通过RestTemplate进行远程调用eureka-client服务。
项目结构如下：
![](/images/2018-7-31/springcloud_ribbon_project_structure.png)
**搭建服务消费者**
新建maven module项目，并在pom文件中引入依赖包括Eureka Client的依赖pring-cloud-starter-eureka,Ribbon的依赖spring-cloud-starter-ribbon，web的依赖spring-boot-starter-web，其代码如下所示：

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-ribbon</artifactId>
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

在src/main/resources中添加application.yml，指定服务端口，以及服务注册地址，具体代码如下：

    spring:
      application:
        name: eureka-ribbon-client
    server:
      port: 8764

    eureka:
      client:
        serviceUrl:
          defaultZone: http://localhost:8761/eureka/

在src/mian/java中新建启动类EurekaRibbonClientApplication，添加注解@EnableEurekaClient作为服务消费者。代码如下所示：

    @SpringBootApplication
    @EnableEurekaClient
    public class EurekaRibbonClientApplication {
        public static void main(String[] args) {
            SpringApplication.run(EurekaRibbonClientApplication.class, args);
        }
    }

**添加负载均衡**
在config包中建立RibbonConfig类，注入restTemplate的Bean，并在此Bean加上@LoadBalanced注解，以达到结合RestTemplate与Ribbon开启负载均衡的目的。代码如下：

    @Configuration
    public class RibbonConfig {

        @Bean
        @LoadBalanced
        RestTemplate restTemplate() {
            return new RestTemplate();
        }
    }

在service包中建立RibbonService类，其中的hi方法用于利用restTemplate调用eureka-client的API接口，，代码如下所示：

    @Service
    public class RibbonService {
        @Autowired
        RestTemplate restTemplate;

        public String hi(String name) {
            return restTemplate.getForObject("http://eureka-client/hi?name="+name,String.class);
        }
    }

最后建立RibbonController类，加上@RestController注解，开启RestController功能，添加“/hi”，用于调用RibbonService的hi(),代码如下：

 	@Autowired
    RibbonService ribbonService;
    @GetMapping("/hi")
    public String hi(@RequestParam(required = false,defaultValue = "forezp") String name){
        return ribbonService.hi(name);
    }

**项目测试**
启动eureka-server工程，端口为8761
启动eureka-client工程，端口为8762，修改eureka-client端口为8763，再次启动，使出现两个eureka-client服务
启动eureka-ribbon-cient工程，端口为8764，如下图所示：
![](/images/2018-7-31/springcloud_ribbon_project_show.png)
在浏览器多次访问：localhost:8764/hi?name=luis,浏览器轮流显示：

    hi luis,i am from port:8762
    hi luis,i am from port:8763

**项目分析**
负载均衡器的核心是LoadBalancerClient，它可以获取负载均衡服务者提供的信息。现在RibbonController类中新建方法：testRibbon，以此来选择eureka-client的服务实例信息，并返回信息，代码如下：

	@Autowired
    private LoadBalancerClient loadBalancer;

    @GetMapping("/testRibbon")
    public String  testRibbon() {
        ServiceInstance instance = loadBalancer.choose("eureka-client");
        return instance.getHost()+":"+instance.getPort();
    }

重新启动工程eureka-ribbon-cient，访问localhost:8764/testRibbon,浏览器轮流显示：

	localhost:8762
	localhost:8763

由上知loadBalancer.choose("eureka-client")可以轮流得到eureka-client的信息。
原来，LoadBalancerClient从Eueka Client获取服务注册列表信息时，会将服务注册信息列表缓存一份，当loadBalancer调用choose（）方法时，根据负载均衡策略选择服务实例。

Ribbon维护注册列表
---
当然，如果禁止Ribbon获取注册列表信息，则Ribbon自己维护注册列表信息，也是可以实现负载均衡的。
现新建maven moudle项目ribbon-client，pom文件添加依赖如下：

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-ribbon</artifactId>
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

application.yml文件如下：

    stores:
      ribbon:
        listOfServers: example.com,google.com

    ribbon:
      eureka:
       enabled: false
    server:
      port: 8769

启动类RibbonClientApplication代码如下：

    @SpringBootApplication
    public class RibbonClientApplication {

        public static void main(String[] args) {
            SpringApplication.run(RibbonClientApplication.class, args);
        }
    }

web层的RibbonController如下：

    @RestController
    public class RibbonController {

        @Autowired
        private LoadBalancerClient loadBalancer;

        @GetMapping("/testRibbon")
        public String  testRibbon() {
            ServiceInstance instance = loadBalancer.choose("stores");
            return instance.getHost()+":"+instance.getPort();
        }
    }

多次访问localhost:8769/testRibbon，会轮流显示：

	google.com:80
	example.com:80

即，Ribbon可自己维护注册列表信息，并实现负载均衡。

本文主要参考自《深入理解Spring+Cloud与微服务构建》一书。