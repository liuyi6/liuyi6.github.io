---
title: SqlSession对象之ResultSetHandler
date: 2018-12-07 17:24:19
tags: [mybatis]
categories: mybatis
---
ResultSetHandler是Mybatis中的另一重要接口，它的代码如下所示：

```java
public interface ResultSetHandler {

  <E> List<E> handleResultSets(Statement stmt) throws SQLException;

  <E> Cursor<E> handleCursorResultSets(Statement stmt) throws SQLException;

  void handleOutputParameters(CallableStatement cs) throws SQLException;

}
```

ResultSetHandler的作用有：

* 处理Statement执行后产生的结果集，生成结果列表
* 处理存储过程执行后的输出参数

这里只讨论处理Statement执行后产生的结果集，生成结果列表这一作用。

ResultSetHandler只有一个实现类-DefaultResultSetHandler，其中重要的方法是handleResultSets()，其代码如下：

```java
public List<Object> handleResultSets(Statement stmt) throws SQLException {
    ErrorContext.instance().activity("handling results").object(mappedStatement.getId());

    final List<Object> multipleResults = new ArrayList<Object>();

    int resultSetCount = 0;
    //获取第一个结果集
    ResultSetWrapper rsw = getFirstResultSet(stmt);

    List<ResultMap> resultMaps = mappedStatement.getResultMaps();
    int resultMapCount = resultMaps.size();
    //判断是否有resultMap,没有的话抛出异常
    validateResultMapsCount(rsw, resultMapCount);
    while (rsw != null && resultMapCount > resultSetCount) {
      //获得resultMap，实体类和表中数据字段的对应关系  
      ResultMap resultMap = resultMaps.get(resultSetCount);
      //将值设置成对应的resultmap对象
      handleResultSet(rsw, resultMap, multipleResults, null);
      rsw = getNextResultSet(stmt);
      cleanUpAfterHandlingResultSet();
      resultSetCount++;
 }
```

详细参见：http://www.cnblogs.com/damens/p/6460763.html

handleResultSets中的getFirstResultSet方法获取第一个结果集，将传统JDBC的ResultSet包装成一个包含结果列元信息的ResultSetWrapper对象。之所以称之为第一个结果集是因为Mybatis考虑到了一个Statement执行了多个sql语句的情况，后面用while循环处理每个结果集，multipleResults中每个元素就是一个结果集的List对象，这也就解释了为什么mappedStatement.getResultMaps()返回的是一个List<ResultMap\>。

其中将值设置成对应的resultmap对象的handleResultSet()代码如下：

```java
private void handleResultSet(ResultSetWrapper rsw, ResultMap resultMap, List<Object> multipleResults, ResultMapping parentMapping) throws SQLException {
    try {
      if (parentMapping != null) {
        handleRowValues(rsw, resultMap, null, RowBounds.DEFAULT, parentMapping);
      } else {
        if (resultHandler == null) {
          DefaultResultHandler defaultResultHandler = new DefaultResultHandler(objectFactory);
          handleRowValues(rsw, resultMap, defaultResultHandler, rowBounds, null);
          multipleResults.add(defaultResultHandler.getResultList());
        } else {
          handleRowValues(rsw, resultMap, resultHandler, rowBounds, null);
        }
      }
    } finally {
      // issue #228 (close resultsets)
      closeResultSet(rsw.getResultSet());
    }
  }
```

参考了：https://blog.csdn.net/shenchaohao12321/article/details/80069600