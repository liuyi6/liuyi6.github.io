---
title: Mybatis(一)之JDBC流程
date:  2018-12-01 08:35:19
tags: [Mybatis]
categories: Mybatis
---
最近要学习Mybatis的源码，因此对JDBC的使用进行总结。

#### JDBC概述

##### **JDBC的定义**

JDBC（Java DataBase Connectivity，java数据库连接）是一种用于执行SQL语句的Java API，可以为多种关系数据库提供统一访问，它由一组用Java语言编写的类和接口组成。JDBC提供了一种基准，据此可以构建更高级的工具和接口，使数据库开发人员能够编写数据库应用程序。

##### **JDBC的作用**

让我们通过我们的程序操作数据库，是java的核心技术之一。

#### JDBC流程详解

JDBC操作数据库共包括6个步骤：

* 注冊驱动 (仅仅做一次)

* 建立连接(Connection)

* 创建传输器Statement

* 运行SQL语句

* 处理运行结果(ResultSet)

* 释放资源

下面就分别针对六个步骤进行说明。

##### 第一步：注册驱动

注冊驱动有三种方式：

1、 Class.forName(“com.mysql.jdbc.Driver”);

推荐这样的方式，不会对详细的驱动类产生依赖

2、DriverManager.registerDriver(com.mysql.jdbc.Driver);

会对详细的驱动类产生依赖

3、System.setProperty(“jdbc.drivers”, “driver1:driver2”);

尽管不会对详细的驱动类产生依赖，但注冊不太方便，所以非常少使用

##### 第二步：建立连接

通过Connection建立连接，Connection是一个接口类。其功能是与数据库进行连接（会话）。

　　建立Connection接口类对象：

```java
Connection conn =DriverManager.getConnection(url, user, password);
```

user为登录数据库的username，如root

password为登录数据库的密码，为空填””

当中url的格式要求为：

```java 
//子协议:子名称//主机名:port/数据库名？属性名=属性值&…
url = "jdbc:mysql://localhost:3306/test“
```

##### 第三步：创建传输器Statement

运行对象Statement负责运行SQL语句。由Connection对象产生。

```java
Statement st = connection.createStatement();
```

Statement接口类还派生出两个接口类PreparedStatement和CallableStatement，这两个接口类对象为我们提供了更加强大的数据訪问功能。

**PreparedStatement接口**

　PreparedStatement能够对SQL语句进行预编译，这样防止了SQL注入，提高了安全性。

```java
PreparedStatement ps=connection.prepareStatement( "update user set id = ? where username = ?”); 
//sql语句中用？作为通配符，变量值通过参数设入：ps.setObject(1, object);
```

此外预编译结果能够存储在PreparedStatement对象中，当多次运行SQL语句时能够提高效率。作为Statement的子类，PreparedStatement继承了Statement的全部函数。

**CallableStatement接口**

CallableStatement类继承了PreparedStatement类，他主要用于运行SQL存储过程。

在JDBC中运行SQL存储过程须要转义。

JDBC API提供了一个SQL存储过程的转义语法：

```java
{call<procedure-name>[<arg1>,<arg2>, ...]}
//procedure-name：是所要运行的SQL存储过程的名字
//[<arg1>,<arg2>, ...]：是相相应的SQL存储过程所须要的參数
```

##### 第四步：运行SQL语句

运行对象Statement 或 PreparedStatement 提供两个经常使用的方法来运行SQL语句：executeQuery和executeUpdate。

1、executeQuery(Stringsql),该方法用于运行实现查询功能的sql语句。返回类型为ResultSet（结果集）。

```java
ResultSet  rs =st.executeQuery(sql);
```

2、executeUpdate(Stringsql),该方法用于运行实现增、删、改功能的sql语句，返回类型为int，即受影响的行数。

```java
int flag = st.executeUpdate(sql);
```

##### 第五步：处理运行结果

ResultSet对象负责保存Statement运行后所产生的查询结果。结果集ResultSet是通过游标来操作的，游标就是一个可控制的、能够指向随意一条记录的指针。有了这个指针我们就能轻易地指出我们要对结果集中的哪一条记录进行改动、删除，或者要在哪一条记录之前插入数据。一个结果集对象中仅仅包括一个游标。另外，借助ResultSetMetaData ，可以将数据表的结构信息都查出来。

```java 
ResultSetMetaData rsmd= resultSet.getMetaData();
```

##### 第六步：释放资源

　数据库资源不关闭，其占用的内存不会被释放，消耗资源，影响系统。倒序释放资源：倒叙释放资源resultSet—>preparedStatement—>connection。

#### 代码实现

数据库删除，修改和插入，使用executeUpdate()方法。

```java
public class JDBCTest {

	/**
	 * 加载的mysql的驱动
	 */
	public static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";

	/**
	 * 连接数据库的url
	 */
	public static final String JDBC_URL = "jdbc:mysql://localhost:3306/test";

	/**
	 * 连接数据库用户名
	 */
	public static final String JDBC_USERNAME = "root";

	/**
	 * 数据库密码
	 */
	public static final String JDBC_PASSWORD = "123456";

	/**
	 * 测试数据库删除，修改和插入
	 */
	@Test
	public void testUpdate() {
		Connection connection = null;
		PreparedStatement preparedStatement = null;

		try {
			// 加载Driver类，注册数据库驱动
			Class.forName(JDBC_DRIVER);
			// 通过DriverManager,使用url，用户名和密码建立连接(Connection)
			connection = DriverManager.getConnection(JDBC_URL, JDBC_USERNAME, JDBC_PASSWORD);
			// 通过Connection，使用sql语句打开Statement对象；
			preparedStatement = connection.prepareStatement("UPDATE student SET age=20 WHERE name=?");
			// 传入参数，防止sql注入
			preparedStatement.setString(1, "xiaoming");
			// 执行语句，将结果返回resultSet
			int count = preparedStatement.executeUpdate();
			// 对结果进行处理
			System.out.println(count);
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			// 倒序释放资源
			try {
				if (preparedStatement != null && !preparedStatement.isClosed()) {
					preparedStatement.close();
				}
			} catch (SQLException e) {
				e.printStackTrace();
			}
			try {
				if (connection != null && connection.isClosed()) {
					connection.close();
				}
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
	}

	/**
	 * 测试批量操作
	 */
	@Test
	public void testBatch() {
		Connection connection = null;
		PreparedStatement preparedStatement = null;
		String insertSql = "insert into student values(?,?)";
		try {
			// 加载Driver类，注册数据库驱动
			Class.forName(JDBC_DRIVER);
			// 通过DriverManager,使用url，用户名和密码建立连接(Connection)
			connection = DriverManager.getConnection(JDBC_URL, JDBC_USERNAME, JDBC_PASSWORD);
			// 通过Connection，使用sql语句打开Statement对象；
			preparedStatement = connection.prepareStatement(insertSql);
			// 传入参数，防止sql注入
			for (int i = 0; i < 10; i++) {
				preparedStatement.setString(1, 100 + i + "user");
				preparedStatement.setInt(2, 100 + i);
				preparedStatement.addBatch();
			}
			// 执行语句，将结果返回resultSet
			int[] count = preparedStatement.executeBatch();
			// 对结果进行处理
			System.out.println(count);
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			// 倒序释放资源
			try {
				if (preparedStatement != null && !preparedStatement.isClosed()) {
					preparedStatement.close();
				}
			} catch (SQLException e) {
				e.printStackTrace();
			}

			try {
				if (connection != null && connection.isClosed()) {
					connection.close();
				}
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
	}
}

```

参考了：

https://www.cnblogs.com/homejim/p/8068199.html

https://www.cnblogs.com/lightandtruth/p/9473862.html

