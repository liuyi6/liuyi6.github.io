---
title: springcloud之声明式调用(Feign)
date: 2018-08-16 21:47:19
tags: [SpringCloud]
categories: SpringCloud
---

Feign采用了API接口的风格，将Java http客户端绑定到内部。Feign的首要目标是将Java Http客户端调用过程变得简单。
Feign的工作原理
---
FeignRibbonClientAutoConfiguration类配置了Client类型（包括HttpURLConnection、OkHttp、HttpClient）,最终向容器注入的是Client的实现类LoadBalancerFeignClient,即负载均衡客户端。
项目实战
---
**搭建Feign项目**
在建立Eureka服务的基础上，加入Feihn服务，实现远程调用。新建Spring Boot的Moudle工程eureka-feign-client，pom文件中引入依赖，代码如下：

	<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-feign</artifactId>
		</dependency>

		<dependency>
			<groupId>com.netflix.feign</groupId>
			<artifactId>feign-httpclient</artifactId>
			<version>RELEASE</version>
		</dependency>

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
在src/main/resources中新建文件：application.yml,代码如下：

    spring:
      application:
        name: eureka-feign-client
    server:
      port: 8765

    eureka:
      client:
        serviceUrl:
          defaultZone: http://localhost:8761/eureka/
在src/main/java的包com.luis中新建Feign的启动类，代码如下：

    @SpringBootApplication
    @EnableEurekaClient
    @EnableFeignClients
    public class EurekaFeignClientApplication {

        public static void main(String[] args) {
            SpringApplication.run(EurekaFeignClientApplication.class, args);
        }
    }
**搭建Feign客户端**
以上，成功搭建了Feign功能的项目，下面进行客户端的搭建：
新建EurekaClientFeign接口，并添加@FeignClient注解，声明Feign Client，其中的value = "eureka-client"为远程调用的服务的名称，FeignConfig为Feign Client的配置类.在EurekaClientFeign接口内部建立方法用来调用Feign Client的"/hi"API接口.代码如下所示:

    @FeignClient(value = "eureka-client",configuration = FeignConfig.class)
    public interface EurekaClientFeign {
        @GetMapping(value = "/hi")
        String sayHiFromClientEureka(@RequestParam(value = "name") String name);
    }
在FeignConfig类中添加@Configuration注解,并注入feignRetryer的Bean,Feign远程调用失败后会重试.代码如下:

    @Configuration
    public class FeignConfig {

        @Bean
        public Retryer feignRetryer() {
            return new Retryer.Default(100, SECONDS.toMillis(1), 5);
        }
    }
在Service层的HiService类中注入EurekaClientFeign的Bean,以通过EurekaClientFeign调用sayHiFromClientEureka()方法.代码如下:

    @Service
    public class HiService {

        @Autowired
        EurekaClientFeign eurekaClientFeign;
        public String sayHi(String name){
            return  eurekaClientFeign.sayHiFromClientEureka(name);
        }
    }
在HiController中加上@RestController注解,开启RestController功能,并编写API接口"/hi",调用HiService的方法.进而远程调用eureka-client的API接口.代码如下:

    @RestController
    public class HiController {
        @Autowired
        HiService hiService;
        @GetMapping("/hi")
        public String sayHi(@RequestParam( defaultValue = "forezp",required = false)String name){
            return hiService.sayHi(name);
        }
    }
**功能测试**
开启eureka-server,端口为8761,开启两个eureka-client服务端口分别为8762和8763,最后开启eureka-feign-client,端口为8765
项目启动成功后,在浏览器多次访问:http://localhost:8765/hi交替出现

    hi luis,I'm from port:8762
    hi luis,I'm from port:8763
由此知,Feign Client远程调用了eureka-client服务的接口,并具有负载均衡能力.由于Feign的以来中默认引入了Ribbon与Hystrix的依赖,因而居于负载均衡功能,下节将对熔断器进行了解.

本文主要参考自《深入理解Spring+Cloud与微服务构建》一书.