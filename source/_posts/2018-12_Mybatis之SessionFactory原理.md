---
title: Mybatis之SessionFactory原理
date: 2018-12-04 21:13:19
tags: [	mybatis]
categories: mybatis
---
Mybatis在使用前需进行初始化，下面就针对Mybatis的初始化过程进行介绍。Mybatis的初始化过程有两种：基于XML和基于Java API两种方式，下面就针对基于XML的方式进行展开。

#### Mybatis初始化的基本过程

Mybatis的初始化过程如下图所示：

![mybatis_begin](/images/2018-12/mybatis_begin.png)

1. 调用 SqlSessionFactoryBuilder 对象的 build(inputStream) 方法；
2. SqlSessionFactoryBuilder 会根据输入流 inputStream 等信息创建XMLConfigBuilder 对象 ;
3. SqlSessionFactoryBuilder 调用 XMLConfigBuilder 对象的 parse() 方法；
4. XMLConfigBuilder 对象返回 Configuration 对象；
5. SqlSessionFactoryBuilder 根据 Configuration 对象创建一个DefaultSessionFactory 对象；
6. SqlSessionFactoryBuilder 返回 DefaultSessionFactory 对象给 Client ，供 Client使用。

上面涉及到的对象有：

- SqlSessionFactoryBuilder ：SqlSessionFactory的构造器，用于创建SqlSessionFactory，采用了Builder设计模式

- Configuration ：该对象是mybatis-config.xml文件中所有mybatis配置信息

- SqlSessionFactory：SqlSession工厂类，以工厂形式创建SqlSession对象，采用了Factory工厂设计模式

- XmlConfigParser ：负责将mybatis-config.xml配置文件解析成Configuration对象，供SqlSessonFactoryBuilder使用，创建SqlSessionFactory


下面针对这些对象进行展开。

####  SqlSessionFactory的创建

SqlSessionFactory是MyBatis中的一个主要接口，其主要负责MyBatis框架初始化操作及提供SqlSession对象。

```java
public interface SqlSessionFactory {

  SqlSession openSession();

  SqlSession openSession(boolean autoCommit);
  SqlSession openSession(Connection connection);
  SqlSession openSession(TransactionIsolationLevel level);

  SqlSession openSession(ExecutorType execType);
  SqlSession openSession(ExecutorType execType, boolean autoCommit);
  SqlSession openSession(ExecutorType execType, TransactionIsolationLevel level);
  SqlSession openSession(ExecutorType execType, Connection connection);

  Configuration getConfiguration();

}
```

它有两个实现类：**DefaultSqlSessionFactory**和**SqlSessionManager**，其中SqlSessionManager已被废弃。在Mybtis初始化的过程中首先会读取Mybatis的核心配置文件，一般创建SqlSessionFactory的代码如下：

```java
InputStream is = Resources.getResourceAsStream("myBatis-config.xml");
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
SqlSessionFactory factory = builder.build(is);
```

首先得到Mybatis核心配置文件的位置，得到InputStream，将其传给SqlSessionFactoryBuilder的build方法，以下为SqlSessionFactoryBuilde中创建SqlSessionFactory对象的方法：

```java
public SqlSessionFactory build(Reader reader);
public SqlSessionFactory build(Reader reader, String environment);
public SqlSessionFactory build(Reader reader, Properties properties);
public SqlSessionFactory build(Reader reader, String environment, Properties properties);

public SqlSessionFactory build(InputStream inputStream);
public SqlSessionFactory build(InputStream inputStream, String environment);
public SqlSessionFactory build(InputStream inputStream, Properties properties)
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties);

public SqlSessionFactory build(Configuration config);
```

SqlSessionFactoryBuilder是Builder模式中建造者，负责SqlSessionFactory对象的创建以及SqlSessionFactroy对象内部所需要内容的组装。其中，前八个方法都是通过读取XML文件得到XMLConfigBuilder，再调用parse()方法得到Configuration对象，从而产生SqlSessionFactory对象。核心代码如下：

```java
XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
return build(parser.parse());
```

当得到一个XMLConfigBuilder后，调用parse()方法,这里采用了建造者设计模式，将XML文件中的配置读取出来，得到Configuration对象。

```java
public Configuration parse() {
    if (parsed) {
      throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    parsed = true;
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
  }

  private void parseConfiguration(XNode root) {
    try {
      //issue #117 read properties first
      propertiesElement(root.evalNode("properties"));
      Properties settings = settingsAsProperties(root.evalNode("settings"));
      loadCustomVfs(settings);
      typeAliasesElement(root.evalNode("typeAliases"));
      pluginElement(root.evalNode("plugins"));
      objectFactoryElement(root.evalNode("objectFactory"));
      objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
      reflectorFactoryElement(root.evalNode("reflectorFactory"));
      settingsElement(settings);
      // read it after objectFactory and objectWrapperFactory issue #631
      environmentsElement(root.evalNode("environments"));
      databaseIdProviderElement(root.evalNode("databaseIdProvider"));
      typeHandlerElement(root.evalNode("typeHandlers"));
      mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
  }
```

注：Builder模式,称为[建造者设计模式]，使用多个简单的对象一步一步构建一个复杂的对象。这种模式属于[创建型模式]，目前它是创建对象的最佳模式。

得到Configuration对象后，将得到的Configuration对象调用build方法创建DefaultSqlSessionFactory对象。

```java
public SqlSessionFactory build(Configuration config) {
    return new DefaultSqlSessionFactory(config);
}
```

在实际调用时，SqlSessionFactory创建过程的基本写法如下：

```java
InputStream is = Resources.getResourceAsStream("myBatis-config.xml");
SqlSessionFactoryBuilder builderObj = new SqlSessionFactoryBuilder();
SqlSessionFactory factory = builderObj.build(is);
```

#### SqlSession原理

SqlSession提供select/insert/update/delete方法，在旧版本中使用SqlSession接口的这些方法，新版本中建议使用Mapper接口的方法。从底层实现来说，Mapper接口方法的实现底层还是采用SqlSession接口方法实现的。SqlSession四个重要对象：

* Execute：调度执行StatementHandler、ParmmeterHandler、ResultHandler执行相应的SQL语句；

* StatementHandler：使用数据库中Statement（PrepareStatement）执行操作，即底层是封装好了的prepareStatement；

* ParammeterHandler：处理SQL参数；

* ResultHandler：结果集ResultSet封装处理返回。

后面会针对四个对象分别进行说明。

#### Configuration介绍

MyBatis框架支持开发人员通过配置文件与其进行交流，在配置文件所配置的信息,在框架运行时会被XMLConfigBuilder解析并存储在一个Configuration对象中。Configuration对象会被作为参数传送给DeFaultSqlSessionFactory，而DeFaultSqlSessionFactory根据Configuration对象信息为Client创建对应特征的SqlSession对象。

#### Mybatis生命周期

* **SqlSessionFactoryBuilder**：用于创建SqlSessionFactory，一旦SqlSessionFactory创建成功，就会被回收。

* **SqlSessionFactory**：用于创建SqlSession，类似于JDBC的Connection对象，而每次应用程序需要访问数据库，都要创建SqlSession，所以SqlSessionFactory在整Mybatis整个生命周期中。(每个数据库对应一个SqlSessionFactory，是单例产生的)

* **SqlSession**：生命周期是存在于请求数据库处理事务的过程中，是一个线程不安全的对象（在多线程的情况下，需要特别注意），即存活于一个应用的请求和申请，可以执行多条SQL保证事务的一致性。

* **Mapper**：是一个接口，没有实现类。它的作用是发送SQL，返回结果，或修改数据库表，所以它存活于一个SqlSession内，是一个方法级别的东西。当SqlSession销毁的时候，Mapper也会销毁。






