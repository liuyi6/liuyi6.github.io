---
title: SqlSession对象之Executor
date: 2018-12-05 07:59:19
tags: [	mybatis]
categories: mybatis
---
Executor是Mybatis的一个核心接口，每一个SqlSession对象都会拥有一个Executor(执行器对象)；这个执行对象负责[增删改查]的具体操作，我们可以简单的将它理解为JDBC中Statement的封装版。它的代码如下：

```java
public interface Executor {
  ResultHandler NO_RESULT_HANDLER = null;
  int update(MappedStatement ms, Object parameter) throws SQLException;
  <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey cacheKey, BoundSql boundSql) throws SQLException;
  <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException;
  <E> Cursor<E> queryCursor(MappedStatement ms, Object parameter, RowBounds rowBounds) throws SQLException;
  List<BatchResult> flushStatements() throws SQLException;
  void commit(boolean required) throws SQLException;
  void rollback(boolean required) throws SQLException;
  CacheKey createCacheKey(MappedStatement ms, Object parameterObject, RowBounds rowBounds, BoundSql boundSql);
  boolean isCached(MappedStatement ms, CacheKey key);
  void clearLocalCache();
  void deferLoad(MappedStatement ms, MetaObject resultObject, String property, CacheKey key, Class<?> targetType);
  Transaction getTransaction();
  void close(boolean forceRollback);
  boolean isClosed();
  void setExecutorWrapper(Executor executor);
}
```

#### Executor的继承结构

Executor的类图如下所示：

![mybatis_exectour_class](/images/2018-12/mybatis_exectour_class.png)

Executor是一个接口，主要有两个实现类：分别是BaseExecutor和CachingExecutor。

##### BaseExecutor类

BaseExecutor是一个抽象类，这种通过抽象类实现接口的方式是适配器设计模式的体现，主要用于方便次一级子类对接口中方法的实现。BaseExecutor主要有三个实现类**SimpleExecutor**、**ReuseExecutor**和**BatchExecutor**。三个实现类分别对应executor对Statement对象管理方案。

* **简单方案**：一个Statement接口对象只执行一次，执行完毕就会Statement接口对象进行销毁。对应SimpleExecutor，被称为简单执行器，是MyBatis中默认使用的执行器，每执行一次update或select，就开启一个Statement对象，用完立刻关闭Statement对象（可以是Statement或PrepareStatement对象）。
* **可重用方案**：对应ReuseExecutor，被称为可重用执行器，重用指的是重复使用Statement，它会在内部利用一个Map把初始化的Statement都缓存起来，每次在执行一条SQL语句时，它都会去判断之前是否存在基于该SQL缓存的Statement对象，存在而且之前缓存的Statement对象对应的Connection还没有关闭的时候就继续用之前的Statement对象，否则将初始化一个新的Statement对象，并将其缓存起来。因为每一个新的SqlSession都有一个新的Executor对象，所以我们缓存在ReuseExecutor上的Statement的作用域是同一个SqlSession。
* **批量处理方案**：对应BatchExecutor，称为批处理执行器，用于将多个Statement对应的SQL语句，交给一个Statement对象一次输送到数据库，进行批处理操作。

##### CachingExecutor类

CachingExecutor称为缓存执行器，MyBatis框架默认情况下使用执行器缓存执行器，可以提高查询效率，先从缓存中获取查询结果，存在就返回，不存在，再委托给Executor delegate去数据库取，delegate可以是SimpleExecutor、ReuseExecutor、BatchExecutor中任意一个。

#### Executor对象初始化

Executor的初始化是在SqlSessionFactory调用openSession方法期间初始化的，首先看一下SqlSessionFactory中的方法，具体代码如下：

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

由代码可见，SqlSessionFactory中的openSession方法分为两类：带ExecutorType和不带ExecutorType的，此外每个类型还有四种，这里采用了重载的方法，方法名称相同，参数列表不同。下面进入到SqlSessionFactory的实现类DefaultSqlSessionFactory中查看openSession()d的实现，代码如下：

```java
  @Override
  public SqlSession openSession(ExecutorType execType) {
    return openSessionFromDataSource(execType, null, false);
  }

  @Override
  public SqlSession openSession(TransactionIsolationLevel level) {
    return openSessionFromDataSource(configuration.getDefaultExecutorType(), level, false);
  }
...
```

通过查看代码发现，所有的openSession方法都会走向openSessionFromDataSource方法中，openSessionFromDataSource的代码如下：

```java
private SqlSession openSessionFromConnection(ExecutorType execType, Connection connection) {
    try {
      boolean autoCommit;
      try {
        autoCommit = connection.getAutoCommit();
      } catch (SQLException e) {
        autoCommit = true;
      }      
      final Environment environment = configuration.getEnvironment();
      final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
      final Transaction tx = transactionFactory.newTransaction(connection);
      final Executor executor = configuration.newExecutor(tx, execType);
      return new DefaultSqlSession(configuration, executor, autoCommit);
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
  }
```

主要看其中的两行代码：

```java
 final Executor executor = configuration.newExecutor(tx, execType);
 return new DefaultSqlSession(configuration, executor, autoCommit);
```

这两行代码先产生executor，再产生了DefaultSqlSession对象，也验证了前面所说的执行器的初始化是在SqlSession初始化之前进行的。下面我们继续查看执行器的初始化过程，它的初始化是由Configuration完成的，初始化Executor的方法newExecutor()的代码如下所示：

```java
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Executor executor;
    if (ExecutorType.BATCH == executorType) {
      executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
      executor = new ReuseExecutor(this, transaction);
    } else {
      executor = new SimpleExecutor(this, transaction);
    }
    if (cacheEnabled) {
      executor = new CachingExecutor(executor);
    }
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
}
```

```java
protected ExecutorType defaultExecutorType = ExecutorType.SIMPLE;
```

由代码可见：首先判断初始化ExecutorType是否为空，若不为空，则默认执行器为SimpleExecutor。而这里的ExecutorType类型则是由openSession方法传入的，因而SqlSessionFactory中没有ExecutorType参数的openSession()则默认为SimpleExecutor。此外关注以下代码：

```java
protected boolean cacheEnabled = true;

if (cacheEnabled) {
      executor = new CachingExecutor(executor);
}
```

这里是初始化CachingExecutor执行器，可见默认情况下CachingExecutor执行器是开启的。

#### ExecutorType的选择

ExecutorType是枚举类型，其代码如下：

```java
public enum ExecutorType {
  SIMPLE, REUSE, BATCH
}
```

ExecutorType的枚举值分别对应三种执行器类型：简单执行器、可重用执行器、批量执行器。

ExecutorType的类型选择，有两种方式对其进行赋值。

##### <settings\>标签设置

通过<settings\>标签来设置当前工程中所有SqlSession对象使用的默认Executour。

![mybatis_executor_seeting1](/images/2018-12/mybatis_executor_seeting1.png)

##### openSession方法指定

通过SqlSessoinFactory中openSession方法来指定具体的SqlSession使用的执行器。

![mybatis_executor_seeting2](/images/2018-12/mybatis_executor_seeting2.png)

