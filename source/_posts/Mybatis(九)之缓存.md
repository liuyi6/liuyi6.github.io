---
title: Mybatis(九)之缓存
date: 2018-12-08 21:13:19
tags: [Mybatis]
categories: Mybatis
---
Mybatis提供缓存查询功能，用于减轻数据库压力，提升数据查询能力。

Mybatis中定义了两级缓存：包括一级缓存与二级缓存。示意图如下所示：

![1544410181880](/img/2018-12/937769970.png)

### 一级缓存

**一级缓存的特点**：

- 每一个SqlSession都有一个一级缓存，且它们的缓存数据区域之间互不影响。
- 一级缓存是默认开启的，开发人员不能关闭。

一级缓存又称为会话缓存，是SqlSession级别的缓存，它的作用域是同一个SqlSession，在查询数据时会创建SqlSession对象，其中会有一个HashMap进行数据的缓存，在同一个SqlSession执行两次相同的sql语句的过程中，第一次查询完毕会将查询的数据写到缓存中，第二次会直接从缓存中获取查询结果，不用去数据库进行查询，提升了查询效率。查询过程如下图所示：

![1544410557144](/img/2018-12/766089750.png)

如图所示，SqlSession进行查询的过程如下：

1. 在发起查询过程中，会首先去一级缓存进行查询，若没有，则去数据库进行数据查询。
2. 得到查询的数据结果，将其存储到一级缓存中。
3. 第二次进行相同sql的查询时，会先去缓存中进行数据查询，若有，则直接从缓存中获取数据。

Mybatis一级缓存的内部，是以HashMap的方式进行存储的，其中Key值为**hashCode+sqlId+Sql语句**，value为从数据库查出来的数据结果映射成的Java对象。

**注意**：

在上图中发现在进行了修改，添加，删除用户的操作时，SqlSession会执行commit操作刷新SqlSession中的一级缓存。目的是为了更新缓存中的数据信息，避免**脏读**。

一级缓存失效的情况：

1. 同一个用户使用两个SqlSession,

### 二级缓存

**二级缓存的特点**：

* 二级缓存默认是关闭的，需要开发人员开启。
* 二级缓存是多个SqlSession共享的，其作用域是mapper的同一个namespace，可以理解为它针对的是一个表的缓存
* 一个SqlSession对象将要被销毁时，MyBatis会自动将这个SqlSession中与当前表关联的数据存放到这个二级共享缓存中

二级缓存又称全局缓存，它是根据mapper的namespace划分的，相同namespace的mapper查询数据放在同一个区域，如果使用mapper代理方法每个mapper的namespace都不同，此时可以理解为二级缓存区域是根据mapper划分。它的查询过程如下图所示：

![1544412634659](/img/2018-12/316852350.png)

与一级缓存类似，每次查询会先从缓存区域找，如果找不到再从数据库查询，并将查询到数据将数据写入缓存。SqlSession执行insert、update、delete等操作commit提交后会刷新缓存区域。同时它的存储也是基于HashMap的，key值也为hashCode+sqlId+Sql语句。

### 缓存相关属性

#### 开启/禁用二级缓存

二级缓存的开启/禁用是在核心配置文件中的<settings\>标签中配置的。

```xml
<settings>
    <!-- 通知MyBatis框架开启二级缓存 -->
    <!-- <setting name="cacheEnabled" value="true"/> -->
    
    <!-- 通知MyBatis框架关闭二级缓存 -->
    <setting name="cacheEnabled" value="false"/>
 </settings>
```

#### useCache属性

useCache配置在mapper.xml文件中，且只会出现在select标签中。useCache默认值就是true，当置false时，对于一级缓存没有影响，只是与SqlSession相关联的查询结果在第一个SqlSession关闭时是不会放到二级缓存中。

```xml
<select id="findUsertById"  resultType="user" useCache="true">
```

#### 刷新缓存

flushCache是刷新缓存的属性，它配置在mapper.xml文件中，它用于决定当前sql执行完毕后，是否会刷新（清空）缓存。flushCache可以配置在添加，更新，删除，查询标签中。

* flushCache配置在添加，更新，删除标签中时，它的默认值为true。
  * 当flushCache为true时，在执行过程中会清空一级缓存数据/清空二级缓存对应的mapper.xml所有数据，会即时修改缓存中的数据，避免出现脏读。
  * 当flushCache为false时，在执行过程中会清空一级缓存数据/不会清空二级缓存数据

* flushCache配置在<select\>标签中时，它的默认值为false。
  * 当flushCache为false时，一级缓存和二级缓存都可以使用
  * 当flushCache为true时，查询结果不会保存到一级缓存和二级缓存

#### clearCache()方法

sqlSession.clearCache()只会清空当前会话的一级缓存，不会影响数据保存到二级缓存

参考了：https://www.cnblogs.com/zhangzongle/p/6211285.html