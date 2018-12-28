---
title: Mybatis中的事务
date: 2018-12-10 21:13:19
tags: [	mybatis]
categories: mybatis
---
## 1、Mybatis事务

### 1.1、事务管理方式

Mybatis中的事务管理方式有两种：

1、JDBC的事务管理机制，即使用JDBC事务管理机制进行事务管理

2、MANAGED的事务管理机制，Mybatis没有实现对事务的管理，而是通过容器来实现对事务的管理

其中，Mybatis提供了事务的接口：Transaction，其代码如下：

```java
public interface Transaction {

  /**
   * Retrieve inner database connection
   * @return DataBase connection
   * @throws SQLException
   */
  //获得数据库连接
  Connection getConnection() throws SQLException;

  /**
   * Commit inner database connection.
   * @throws SQLException
   */
  //提交
  void commit() throws SQLException;

  /**
   * Rollback inner database connection.
   * @throws SQLException
   */
  //回滚
  void rollback() throws SQLException;

  /**
   * Close inner database connection.
   * @throws SQLException
   */
  //连接关闭
  void close() throws SQLException;

  /**
   * Get transaction timeout if set
   * @throws SQLException
   */
  //获取事务timeout
  Integer getTimeout() throws SQLException;
  
}
```

Transaction有两个实现类：JdbcTransaction和ManagedTransaction，分别对应两种事务管理方式。

JdbcTransaction的代码如下：

```java
public class JdbcTransaction implements Transaction {

  private static final Log log = LogFactory.getLog(JdbcTransaction.class);

  //数据连接
  protected Connection connection;
  //数据源
  protected DataSource dataSource;
  //事务等级
  protected TransactionIsolationLevel level;
  //事务提交
  protected boolean autoCommmit;

  public JdbcTransaction(DataSource ds, TransactionIsolationLevel desiredLevel, boolean desiredAutoCommit) {
    dataSource = ds;
    level = desiredLevel;
    autoCommmit = desiredAutoCommit;
  }

  public JdbcTransaction(Connection connection) {
    this.connection = connection;
  }

  @Override
  public Connection getConnection() throws SQLException {
    if (connection == null) {
      openConnection();
    }
    return connection;
  }

  @Override
  public void commit() throws SQLException {
    if (connection != null && !connection.getAutoCommit()) {
      if (log.isDebugEnabled()) {
        log.debug("Committing JDBC Connection [" + connection + "]");
      }
      connection.commit();
    }
  }

  @Override
  public void rollback() throws SQLException {
    if (connection != null && !connection.getAutoCommit()) {
      if (log.isDebugEnabled()) {
        log.debug("Rolling back JDBC Connection [" + connection + "]");
      }
      connection.rollback();
    }
  }

  @Override
  public void close() throws SQLException {
    if (connection != null) {
      resetAutoCommit();
      if (log.isDebugEnabled()) {
        log.debug("Closing JDBC Connection [" + connection + "]");
      }
      connection.close();
    }
  }

  protected void setDesiredAutoCommit(boolean desiredAutoCommit) {
    try {
      //事务提交状态不一致时修改
      if (connection.getAutoCommit() != desiredAutoCommit) {
        if (log.isDebugEnabled()) {
          log.debug("Setting autocommit to " + desiredAutoCommit + " on JDBC Connection [" + connection + "]");
        }
        connection.setAutoCommit(desiredAutoCommit);
      }
    } catch (SQLException e) {
      // Only a very poorly implemented driver would fail here,
      // and there's not much we can do about that.
      throw new TransactionException("Error configuring AutoCommit.  "
          + "Your driver may not support getAutoCommit() or setAutoCommit(). "
          + "Requested setting: " + desiredAutoCommit + ".  Cause: " + e, e);
    }
  }

  protected void resetAutoCommit() {
    try {
      if (!connection.getAutoCommit()) {
        if (log.isDebugEnabled()) {
          log.debug("Resetting autocommit to true on JDBC Connection [" + connection + "]");
        }
        connection.setAutoCommit(true);
      }
    } catch (SQLException e) {
      if (log.isDebugEnabled()) {
        log.debug("Error resetting autocommit to true "
          + "before closing the connection.  Cause: " + e);
      }
    }
  }

  protected void openConnection() throws SQLException {
    if (log.isDebugEnabled()) {
      log.debug("Opening JDBC Connection");
    }
  	//从数据源中获得连接
    connection = dataSource.getConnection();
    if (level != null) {
      connection.setTransactionIsolation(level.getLevel());
    }
    setDesiredAutoCommit(autoCommmit);
  }

  @Override
  public Integer getTimeout() throws SQLException {
    return null;
  }
}
```

JdbcTransaction通过使用jdbc提供的方式来管理事务，通过Connection提供的事务管理方法来进行事务管理，只是将JDBC的事务管理进行了封装。

ManagedTransaction实现类是通过容器来进行事务管理，所有它对事务提交和回滚并不会做任何操作。其代码如下：

```java
public class ManagedTransaction implements Transaction {

  private static final Log log = LogFactory.getLog(ManagedTransaction.class);

  private DataSource dataSource;
  private TransactionIsolationLevel level;
  private Connection connection;
  private final boolean closeConnection;

  public ManagedTransaction(Connection connection, boolean closeConnection) {
    this.connection = connection;
    this.closeConnection = closeConnection;
  }

  public ManagedTransaction(DataSource ds, TransactionIsolationLevel level, boolean closeConnection) {
    this.dataSource = ds;
    this.level = level;
    this.closeConnection = closeConnection;
  }

  @Override
  public Connection getConnection() throws SQLException {
    if (this.connection == null) {
      openConnection();
    }
    return this.connection;
  }

  @Override
  public void commit() throws SQLException {
    // Does nothing
  }

  @Override
  public void rollback() throws SQLException {
    // Does nothing
  }

  @Override
  public void close() throws SQLException {
    if (this.closeConnection && this.connection != null) {
      if (log.isDebugEnabled()) {
        log.debug("Closing JDBC Connection [" + this.connection + "]");
      }
      this.connection.close();
    }
  }

  protected void openConnection() throws SQLException {
    if (log.isDebugEnabled()) {
      log.debug("Opening JDBC Connection");
    }
    this.connection = this.dataSource.getConnection();
    if (this.level != null) {
      this.connection.setTransactionIsolation(this.level.getLevel());
    }
  }

  @Override
  public Integer getTimeout() throws SQLException {
    return null;
  }

}
```

此外，当Mybatis与Spring一起使用时，Spring会提供一个Transaction的实现类SpringManagedTransaction进行事务管理，这会在后面的Spring源码中说到。

可参考：https://blog.csdn.net/qq924862077/article/details/525997851.2

### 1.2、事务配置方式

Mybatis的事务管理方式的配置是在核心配置文件中进行的，它是在configuration标签下的environments中与数据源一起配置的，可以配置多个，如下所示：

```xml
 <environments default="development">
    <environment id="development">
      //配置事务管理方式
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
      </dataSource>
    </environment>
</environments>
```

## 2、事务隔离级别

在上面的源码中，我们看到TransactionIsolationLevel—事务隔离级别，Mybaais中定义了五种隔离级别，代码如下所示：

```java
public enum TransactionIsolationLevel {
  NONE(Connection.TRANSACTION_NONE),
  READ_COMMITTED(Connection.TRANSACTION_READ_COMMITTED),
  READ_UNCOMMITTED(Connection.TRANSACTION_READ_UNCOMMITTED),
  REPEATABLE_READ(Connection.TRANSACTION_REPEATABLE_READ),
  SERIALIZABLE(Connection.TRANSACTION_SERIALIZABLE);

  private final int level;

  private TransactionIsolationLevel(int level) {
    this.level = level;
  }

  public int getLevel() {
    return level;
  }
}
```

其中四种是一般数据库的事务隔离级别，从高到底以此为：Read uncommitted(读未提交)、Read committed（读已提交)、Repeatable read(可重复读)、Serializable(可串行化)，这几个级别主要用于解决脏读、不可重复读、幻读等问题，其总结如下表：

|       级别       | 脏读  | 不可重复读 | 幻读  | 说明                                                         |
| :--------------: | :---: | :--------: | :---: | ------------------------------------------------------------ |
| Read uncommitted | true  |    true    | true  | 一个更新语句没有提交,但是别的事务可以读到这个改变.这是很不安全的。允许任务读取数据库中未提交的数据更改，也称为脏读。 |
|  Read committed  | false |    true    | true  | 语句提交以后即执行了COMMIT以后别的事务就能读到这个改变. 只能读取到已经提交的数据。Oracle等多数数据库默认都是该级别,可防止脏读。 |
| Repeatable read  | false |   false    | true  | 同一个事务里面先后执行同一个查询语句的时候,得到的结果是一样的.在同一个事务内的查询都是事务开始时刻一致的，InnoDB默认级别。在SQL标准中，该隔离级别消除了不可重复读，但是还存在幻象读。 |
|   Serializable   | false |   false    | false | 事务执行的时候不允许别的事务并发执行. 完全串行化的读，每次读都需要获得表级共享锁，读写相互都会阻塞。 |

注：true表示出现，false表示不出现

参考自：https://blog.csdn.net/qq924862077/article/details/52599961

Mybatis添加的隔离级别NONE，可在DefaultSqlSessionFactory中创建SqlSession时，设置数据库的事务隔离级别，以及通过设置autoCommit来设置事务的提交方式。代码参见DefaultSqlSessionFactory，示例代码如下：

```java
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit)
```

## 3、事务工厂

此外，在Mybatis中还提供了事务工厂：TransactionFactory，代码如下所示：

```java
public interface TransactionFactory {

  //配置工厂属性
  void setProperties(Properties props);

  //通过Connection获取事务
  Transaction newTransaction(Connection conn);
  
  //通过数据库，事务等级,是否自动提交创建事务
  Transaction newTransaction(DataSource dataSource, TransactionIsolationLevel level, boolean autoCommit);

}
```

类似的，TransactionFactory也有两个实现类：JdbcTransactionFactory和ManagedTransactionFactory

JdbcTransactionFactory代码如下：

```java
public class JdbcTransactionFactory implements TransactionFactory {

  @Override
  public void setProperties(Properties props) {
  }

  @Override
  public Transaction newTransaction(Connection conn) {
    return new JdbcTransaction(conn);
  }

  @Override
  public Transaction newTransaction(DataSource ds, TransactionIsolationLevel level, boolean autoCommit) {
    return new JdbcTransaction(ds, level, autoCommit);
  }
}
```

ManagedTransactionFactory代码如下：

```java
public class ManagedTransactionFactory implements TransactionFactory {

  private boolean closeConnection = true;

  @Override
  public void setProperties(Properties props) {
    if (props != null) {
      String closeConnectionProperty = props.getProperty("closeConnection");
      if (closeConnectionProperty != null) {
        closeConnection = Boolean.valueOf(closeConnectionProperty);
      }
    }
  }

  @Override
  public Transaction newTransaction(Connection conn) {
    return new ManagedTransaction(conn, closeConnection);
  }

  @Override
  public Transaction newTransaction(DataSource ds, TransactionIsolationLevel level, boolean autoCommit) {
    return new ManagedTransaction(ds, level, closeConnection);
  }
}
```

