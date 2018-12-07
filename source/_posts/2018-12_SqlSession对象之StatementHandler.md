---
title: SqlSession对象之StatementHandler
date: 2018-12-06 23:24:19
tags: [	mybatis]
categories: mybatis
---
上一篇讲了SqlSession对象中的Executor，接下来将对SqlSession的另一个对象StatementHandler进行讲解。

#### StatementHandler介绍

StatementHandler是Mybatis中最重要的一个对象，它负责操作Statement与数据库进行交流，在此过程中还会调用ParameterHandler进行参数配置，使用ResultHandler将查询结果与实体类对象进行绑定。因而ParameterHandler与ResultHandler的创建是与StatementHandler相关联的。

StatementHandler是一个顶级接口，它的类图如下所示：

![mybatis_statement_entends](/images/2018-12/mybatis_statement_entends.png)

StatementHandler接口下有两个直接实现类BaseStatementHandler和RoutingStatementHandler

##### RoutingStatementHandler类

RoutingStatementHandler：是一个具体实现类，在这个类中并没有对Statement对象进行具体使用，只是根据得到Executor类型决定创建何种类型StatementHandler对象。在MyBatis工作时，使用的StatementHandler接口对象实际上就是RoutingStatementHandler对象。

可以简单理解为：

```java
StatementHandler statmentHandler = new RountingStatementHandler();
```

#####  BaseStatementHandler抽象类

BaseStatementHandler：是StatementHandler接口的另一个实现类，本身是一个抽象类，用于简化StatementHandler接口实现的难度，属于适配器设计模式体现。它有三个实现类：SimpleStatementHandler，PreparedStatementHandler和CallableStatementHandler。

在RountingStatementHandler创建时,就根据接收的传递的SQL语句来创建这个三个类型的对象。

**SimpleStatementHandler**：管理Statement对象向数据库中推送不需要预编译的SQL语句。

**PreparedStatementHandler**：管理PreparedStatementHandler对象向数据库推送预编译的SQL语句。

**CallableStatementHandler**：管理CallableStatement对象调用数据库中构造函数，即有存储过程的SQL语句。

#### StatementHandler接口方法介绍

StatementHandler是一个接口，其代码如下：

```java 
public interface StatementHandler {

  Statement prepare(Connection connection, Integer transactionTimeout)throws SQLException;

  void parameterize(Statement statement)throws SQLException;

  void batch(Statement statement)throws SQLException;

  int update(Statement statement)throws SQLException;

  <E> List<E> query(Statement statement, ResultHandler resultHandler)throws SQLException;

  <E> Cursor<E> queryCursor(Statement statement)throws SQLException;

  BoundSql getBoundSql();

  ParameterHandler getParameterHandler();

}
```

我们主要关注StatementHandler中的四个重要方法：prepare()、parameterize()、update()、query()

##### prepare方法

prepare方法主要是在BaseStatementHandler类中实现的，其代码如下：

```java
  public Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException {
    ErrorContext.instance().sql(boundSql.getSql());
    Statement statement = null;
    try {
      statement = instantiateStatement(connection);
      setStatementTimeout(statement, transactionTimeout);
      setFetchSize(statement);
      return statement;
    } catch (SQLException e) {
      closeStatement(statement);
      throw e;
    } catch (Exception e) {
      closeStatement(statement);
      throw new ExecutorException("Error preparing statement.  Cause: " + e, e);
    }
  }
```

 prepare方法用于创建一个(Statement or PreparedStatement or CallableStatement)对象，并设置Statement对象的最大工作时间和一次性读取的最大数据量，然后将生成的Statement对象返回。

prepare方法只在BaseStatementHandler被实现，在其三个子类中没有被重写。它主要用于三个子类调用获得对应的Statement接口对象，依靠instantiateStatement(connection)方法来返回具体Statement接口对象。instantiateStatement方法是BaseStatementHandler中定义的抽象方法,由三个子类来具体实现，这里采用了模板方法模式。

SimpleStatementHandler的instantiateStatement方法

```java 
protected Statement instantiateStatement(Connection connection) throws SQLException {
    if (mappedStatement.getResultSetType() != null) {
      return connection.createStatement(mappedStatement.getResultSetType().getValue(), ResultSet.CONCUR_READ_ONLY);
    } else {
      return connection.createStatement();
    }
  }
```

PreparedStatementHandler的instantiateStatement方法

```java 
protected Statement instantiateStatement(Connection connection) throws SQLException {
    String sql = boundSql.getSql();
    if (mappedStatement.getKeyGenerator() instanceof Jdbc3KeyGenerator) {
      String[] keyColumnNames = mappedStatement.getKeyColumns();
      if (keyColumnNames == null) {
        return connection.prepareStatement(sql, PreparedStatement.RETURN_GENERATED_KEYS);
      } else {
        return connection.prepareStatement(sql, keyColumnNames);
      }
    } else if (mappedStatement.getResultSetType() != null) {
      return connection.prepareStatement(sql, mappedStatement.getResultSetType().getValue(), ResultSet.CONCUR_READ_ONLY);
    } else {
      return connection.prepareStatement(sql);
    }
  }
```

CallableStatementHandler的instantiateStatement方法

```java 
protected Statement instantiateStatement(Connection connection) throws SQLException {
    String sql = boundSql.getSql();
    if (mappedStatement.getResultSetType() != null) {
      return connection.prepareCall(sql, mappedStatement.getResultSetType().getValue(), ResultSet.CONCUR_READ_ONLY);
    } else {
      return connection.prepareCall(sql);
    }
  }
```

##### parameterize方法

parameterize方法用于传递参数，因此只在PreparedStatementHandler和CallableStatementHandler中被重写。

PreparedStatementHandler的parameterize方法

```java 
public void parameterize(Statement statement) throws SQLException {
    parameterHandler.setParameters((PreparedStatement) statement);
}
```

CallableStatementHandler的parameterize方法

```java
public void parameterize(Statement statement) throws SQLException {
    registerOutputParameters((CallableStatement) statement);
    parameterHandler.setParameters((CallableStatement) statement);
}
```

他们都调用了parameterHandler进行参数赋值。

##### query方法

query方法主要用于输送查询查询语句，并将查询结果转换对应的实体类对象。

SimpleStatementHandler的query方法：

```java
public <E> Cursor<E> queryCursor(Statement statement) throws SQLException {
    String sql = boundSql.getSql();
    statement.execute(sql);
    return resultSetHandler.<E>handleCursorResultSets(statement);
}
```

PreparedStatementHandler的query方法：

```java
public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
    PreparedStatement ps = (PreparedStatement) statement;
    ps.execute();
    return resultSetHandler.<E> handleResultSets(ps);
  }
```

CallableStatementHandler的query方法：

```java
public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
    CallableStatement cs = (CallableStatement) statement;
    cs.execute();
    List<E> resultList = resultSetHandler.<E>handleResultSets(cs);
    resultSetHandler.handleOutputParameters(cs);
    return resultList;
  }
```

由代码可知：在得到查询结果后，都是使用ResultSetHandler对结果进行转换的。

##### update方法

update方法用于输送insert、update、delete语句并返回处理数据行。

SimpleStatementHandler的update方法：

```java
public int update(Statement statement) throws SQLException {
    String sql = boundSql.getSql();
    Object parameterObject = boundSql.getParameterObject();
    KeyGenerator keyGenerator = mappedStatement.getKeyGenerator();
    int rows;
    if (keyGenerator instanceof Jdbc3KeyGenerator) {
      statement.execute(sql, Statement.RETURN_GENERATED_KEYS);
      rows = statement.getUpdateCount();
      keyGenerator.processAfter(executor, mappedStatement, statement, parameterObject);
    } else if (keyGenerator instanceof SelectKeyGenerator) {
      statement.execute(sql);
      rows = statement.getUpdateCount();
      keyGenerator.processAfter(executor, mappedStatement, statement, parameterObject);
    } else {
      statement.execute(sql);
      rows = statement.getUpdateCount();
    }
    return rows;
  }
```

PreparedStatementHandler的update方法：

```java
public int update(Statement statement) throws SQLException {
    PreparedStatement ps = (PreparedStatement) statement;
    ps.execute();
    int rows = ps.getUpdateCount();
    Object parameterObject = boundSql.getParameterObject();
    KeyGenerator keyGenerator = mappedStatement.getKeyGenerator();
    keyGenerator.processAfter(executor, mappedStatement, ps, parameterObject);
    return rows;
  }
```

CallableStatementHandler的update方法：

```java
public int update(Statement statement) throws SQLException {
    CallableStatement cs = (CallableStatement) statement;
    cs.execute();
    int rows = cs.getUpdateCount();
    Object parameterObject = boundSql.getParameterObject();
    KeyGenerator keyGenerator = mappedStatement.getKeyGenerator();
    keyGenerator.processAfter(executor, mappedStatement, cs, parameterObject);
    resultSetHandler.handleOutputParameters(cs);
    return rows;
  }
```

#### StatementHandler对象创建

StatementHandler对象是在SqlSession对象接收到操作命令后由Configuraion中newStatementHandler方法负责调用的。代码如下：

```java
public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
    StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);
    statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);
    return statementHandler;
}
```

由代码知：statementHandler的创建调用了RoutingStatementHandler的构造方法。

RoutingStatementHandler构造方法,将会根据StatementType决定创建SimpleStatementHandler，PreparedStatementHandler，CallableStatementHandler实例对象。

```java
public enum StatementType {
  STATEMENT, PREPARED, CALLABLE
}
```

构造方法如下所示：

```java 
 public RoutingStatementHandler(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {

    switch (ms.getStatementType()) {
      case STATEMENT:
        delegate = new SimpleStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
        break;
      case PREPARED:
        delegate = new PreparedStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
        break;
      case CALLABLE:
        delegate = new CallableStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
        break;
      default:
        throw new ExecutorException("Unknown statement type: " + ms.getStatementType());
    }
  }
```

而创建三种StatementHandler的构造方法都采用的是BaseStatementHandler的构造方法，其代码如下：

```java
protected BaseStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
    this.configuration = mappedStatement.getConfiguration();
    this.executor = executor;
    this.mappedStatement = mappedStatement;
    this.rowBounds = rowBounds;

    this.typeHandlerRegistry = configuration.getTypeHandlerRegistry();
    this.objectFactory = configuration.getObjectFactory();

    if (boundSql == null) { // issue #435, get the key before calculating the statement
      generateKeys(parameterObject);
      boundSql = mappedStatement.getBoundSql(parameterObject);
    }

    this.boundSql = boundSql;

    this.parameterHandler = configuration.newParameterHandler(mappedStatement, parameterObject, boundSql);
    this.resultSetHandler = configuration.newResultSetHandler(executor, mappedStatement, rowBounds, parameterHandler, resultHandler, boundSql);
  }
```

这里需要注意的是最后两行代码，创建了parameterHandler和resultSetHandler，这正好和前面所说的StatementHandler、parameterHandler和resultSetHandler是在一起创建的说法保持一致。

创建好handler，执行器的方法会调用prepareStatement方法产生Statment.，代码如下：

```java
private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
    Statement stmt;
    BoundSql boundSql = handler.getBoundSql();
    String sql = boundSql.getSql();
    if (hasStatementFor(sql)) {
      stmt = getStatement(sql);
      applyTransactionTimeout(stmt);
    } else {
      Connection connection = getConnection(statementLog);
      stmt = handler.prepare(connection, transaction.getTimeout());
      putStatement(sql, stmt);
    }
    handler.parameterize(stmt);
    return stmt;
  }
```

此过程中会调用BaseStatementHandler的prepare方法，而在方法中又会调用instantiateStatement方法，通过Connection产生相应的Statment对象。