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

Executor是一个接口，主要有两个实现类：分别是[BaseExecutor]和[CachingExecutor]。[BaseExecutor]是一个抽象类，这种通过抽象类实现接口的方式是[适配器设计模式]的体现，主要用于方便次一级子类对接口中方法的实现。

[BaseExecutor]主要有三个实现类[SimpleExecutor]、[ ReuseExecutor]和[ BatchExecutor]。

[SimpleExecutor]被称为[简单执行器]，是MyBatis中默认使用的执行器，每执行一次update或select，就开启一个Statement对象，用完立刻关闭Statement对象。（可以是Statement或PrepareStatement对象）。

[ ReuseExecutor]被称为[可重用执行器]，重用指的是重复使用Statement。它会在内部利用一个Map把创建的Statement都缓存起来，每次在执行一条SQL语句时，它都会去判断之前是否存在基于该SQL缓存的Statement对象，存在而且之前缓存的Statement对象对应的Connection还没有关闭的时候就继续用之前的Statement对象，否则将创建一个新的Statement对象，并将其缓存起来。因为每一个新的SqlSession都有一个新的Executor对象，所以我们缓存在ReuseExecutor上的Statement的作用域是同一个SqlSession。

[ BatchExecutor]称为[批处理执行器]，用于将多个sql语句一次性输送到数据库执行。

[CachingExecutor]称为[缓存执行器]，先从缓存中获取查询结果，存在就返回，不存在，再委托给Executor delegate去数据库取，delegate可以是上面任一的SimpleExecutor、ReuseExecutor、BatchExecutor。

#### Excecutor对象创建

创建Executor是在创建sqlSession之前创建的。代码如下：

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

在创建sqlSession之前需要创建Executor对象，创建对象的过程是由Configuration完成的，创建Executor的方法newExecutor()的代码如下所示：

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

由代码可见：创建何种类型的Executor是由ExecutorType决定的。

#### ExecutorType的选择

ExecutorType是一个枚举对象，其代码如下：

```java
public enum ExecutorType {
  SIMPLE, REUSE, BATCH
}
```

ExecutorType的类型选择，可以在两个地方进行赋值。

1、通过<settings>标签来设置当前工程中所有SqlSession对象使用的默认Executour。

![mybatis_executor_seeting1](/images/2018-12/mybatis_executor_seeting1.png)

2、通过SqlSessoinFactory中openSession方法来指定具体的SqlSession使用的执行器。

![mybatis_executor_seeting2](/images/2018-12/mybatis_executor_seeting2.png)

