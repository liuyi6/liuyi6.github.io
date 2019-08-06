---
title: Mybatis(二)之架构简介
date: 2018-12-01 11:45:19
tags: [Mybatis]
categories: Mybatis
---
### Mybatis与ORM

对象关系映射（即Object Relational Mapping，简称ORM），主要用于关系型数据库和实体之间的映射，主要为了解决对象与关系数据库存在的互不匹配的现象,ORM通过使用描述对象和数据库之间映射的元数据，将程序中的对象自动持久化到关系数据库中。Java典型的ORM中间件有:Hibernate,Mybatis。

ORM是Object与Relation之间映射，包括Object->Relation和Relation->Object两方面。Hibernate是一个完整ORM框架。而MyBatis完成的是Relation->Object，MyBatis并不刻意于完成ORM(对象映射)的完整概念，而是旨在让开发人员使用更简单、更方便的方式完成数据库操作功能。

### Mybatis架构

Mybatis的功能架构设计如下图所示：

![mybatis_structure](/img/2018-12/mybatis_structure.png)

**接口层**

接口层主要提供和数据库交互的方式，包括sqlSession和Mapper接口两种方式。通过SqlSession接口和Mapper接口开发人员可以通知MyBatis框架调用那一条SQL命令以及SQL命令关联参数。

**数据处理层**

数据处理层可以说是MyBatis的核心，从大的方面上讲，它要完成三个功能： 通过参数构建动态SQL语句、SQL语句的执行以及查询结果类型转换List

**基础支撑层**

支撑层用来完成MyBaits与数据库基本连接方式以及SQL命令与配置文件对应.主要负责：连接方式管理、事务管理、配置方式管理、缓存管理

### Mybatis调用流程

![mybaatis_structure_stream](/img/2018-12/mybaatis_structure_stream.png)

如上图所示，MyBatis的主要的核心部件有以下几个：

- SqlSession          

  作为MyBatis工作的主要顶层API，表示和数据库交互的会话，完成必要数据库增删改查功能 

- Executor              

  MyBatis执行器，是MyBatis 调度的核心，负责SQL语句的生成和查询缓存的维护

- StatementHandler   

  封装了JDBC Statement操作，负责对JDBC statement 的操作，如设置参数、将Statement结果集转换成List集合。

- ParameterHandler   

  负责对用户传递的参数转换成JDBC Statement 所需要的参数，

- ResultSetHandler     

  负责将JDBC返回的ResultSet结果集对象转换成List类型的集合；

- TypeHandler             

  负责java数据类型和jdbc数据类型之间的映射和转换

- MappedStatement   

  MappedStatement维护了一条<select|update|delete|insert>节点的封装， 

- SqlSource                  

  负责根据用户传递的parameterObject，动态地生成SQL语句，将信息封装到BoundSql对象中，并返回

- BoundSql             

  表示动态生成的SQL语句以及相应的参数信息

- Configuration        

  MyBatis所有的配置信息都维持在Configuration对象之中。

参考https://blog.csdn.net/qq_27886997/article/details/78073379

### Mybatis框架使用

Mybatis的使用方式有两种，一种是**基于XML配置文件**，它将SQL命令声明在XML配置文件中；两一种是**基于注解方式**，它将SQL命令声明在注解中。