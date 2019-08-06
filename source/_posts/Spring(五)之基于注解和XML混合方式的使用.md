---
title: Spring(五)之基于注解和XML混合方式的使用
date: 2019-01-03 23:45:19
tags: [Spring]
categories: Spring
---
首先要明白，基于注解和XML两种方式的实现功能是一样的，只是两种不同的配置方式。

## IoC配置

### 配置xml

在使用注解与xml结合的方式配置IoC之前，首先要引入context标签：

```xml
xmlns:context="http://www.springframework.org/schema/context" 

http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd 
```

再配置包的扫描：

```xml
<?xml version="1.0" encoding="UTF-8"?> <beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx" xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
	<!-- 扫描com.luis包下所有使用注解的类型 --> 
    <context:component-scan base-package="com.luis" />
</beans>
```

此配置起的作用是扫描com.luis包下所有带@Component及其衍生注解的类

### 配置注解

#### @Component注解

@Component注解作用是将资源交给Spring进行管理，相当于xml中配置的bean。

通过属性value指定bean的id，若不指定则默认bean的名称为类的名称，首字母小写。

#### @Component衍生注解

@Component产生三个衍生注解-@Controller、@Service、@Repository。它们与@Component的作用及用法一致，习惯上表达更为清晰的意义：

* @Controller：一般用于表现层(controller)的注解。
* @Service：一般用于业务层(service)的注解。
* @Repository：一般用于持久层(dao)的注解。

## DI注解配置

DI的装配方式有两种：按类型装配，按名称装配。这里的装配相当于xml配置方式中的:

```xml
<property name="" ref="">
```

### 按类型装配

#### @Autowired

@Autowired默认按**类型装配**（byType），它是由AutowiredAnnotationBeanPostProcessor类实现。

@Autowired是spring自带的注解，它默认情况下要求依赖对象必须存在，如果需要允许null值，可以设置它的required属性为false，如：@Autowired(required=false)。

#### @Inject

@Inject是**根据类型进行自动装配**的，如果需要按名称进行装配，则需要配合@Named；

 @Inject是JSR330中的规范，需要导入javax.inject.Inject;实现注入，它可以作用在变量、setter方法、构造函数上。

### 按名称装配

#### @Resource

@Resource默认**按名称装配**（byName），可以通过@Resource的name属性指定名称， @Resource属于J2EE JSR250规范的实现。

 @Resource如果没有指定name属性，当注解写在字段上时，默认取字段名进行按照名称查找，当找不到与名称匹配的bean时才按照类型进行装配。如果name属性一旦指定，就只会按照名称进行装配。

推荐使用@Resource注解，这个注解属于J2EE的，减少了与spring的耦合。相当于xml中的`<property name="" value="">`

#### 注解组合

其实我们的按名称装配与按类型装配两种方式之间并没有明显的分界线，如@Resource也可以通过类型进行装配，而我们的按照类型装配也可以和其他注解结合的方式实现按照名策划那个装配。

@Autowired可以与@Qualifier结合实现按名称装配。在自动按照类型注入的基础之上，再按照 Bean 的 id 注入，@Qualifier在给**字段注入**时不能独立使用，**必须和@Autowire 一起使用**；但是给方法参数注入时，可以独立使用。

**注意，@Autowired、@Resource、@Inject区别**：

* @Autowired是spring自带的，@Inject是JSR330规范实现的，@Resource是JSR250规范实现的，需要导入不同的包
* @Autowired、@Inject用法基本一样，不同的是@Autowired有一个request属性
* @Autowired、@Inject是默认按照类型匹配的，@Resource是按照名称匹配的
* @Autowired如果需要按照名称匹配需要和@Qualifier一起使用，@Inject和@Name一起使用

#### 其他注解

常用的注解如@Value，用于给基本类型和String类型注入值、使用占位符获取属性文件中的值

```java
@Value(“${name}”)//name是properties文件中的key
private String name;
```

bean作用范围注解@Scope，用于指定 bean 的作用范围，通过value进行取值，其值可取：singleton、prototype、request、session、globalsession

生命周期注解@PostConstruct、@PreDestroy，作用相当于xml中的`<bean id="" class="" init-method="" destroy-method=""/>`

另外还有一大批注解，将会在下一篇中进行说明。

## 注解和xml两种配置方式对比

两种配置方式各有优点：注解配置简单，维护方便（找到类，就相当于找到了对应的配置）；xml修改时，不用改源码，不涉及重新编译和部署。因而具体的配置方式由个人进行选择。

Spring管理bean方式对比：

|                        | 基于xml                                | 基于注解                                                  |
| :--------------------: | -------------------------------------- | --------------------------------------------------------- |
|        Bean定义        | `<bean id="" calss=""/>`               | @Component及其衍生注解                                    |
|        Bean名称        | 通过id或name指定                       | @Component("person")                                      |
|        Bean注入        | `<property>`或p命名空间                | @Autowired、@Resource                                     |
| Bean作用范围、生命周期 | init-method、destory-method、scope属性 | @PostConstruct初始化，@PreDestroy销毁，@Scope作用范围设置 |