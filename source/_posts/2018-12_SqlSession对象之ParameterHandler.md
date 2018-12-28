---
title: SqlSession对象之ParameterHandler
date: 2018-12-07 15:04:19
tags: [mybatis]
categories: mybatis
---
上一篇讲了StatementHandler，其中有ParameterHandler(参数处理器)是在StatementHandler被创建时被创建的。下面对ParameterHandler进行说明。其代码如下：

```java
public interface ParameterHandler {
  Object getParameterObject();
  void setParameters(PreparedStatement ps) throws SQLException;
}
```

它只有两个方法，其中getParameterObject()是获取参数的，而setParameters()是设置参数的。

ParameterHandler只有一个实现类DefaultParameterHandler，在查看DefaultParameterHandler代码之前首先了解DefaultParameterHandler中的相关属性。代码如下所示：

```java
//类型转换处理器：映射Java中数据类型与数据库中数据类型对应关系
private final TypeHandlerRegistry typeHandlerRegistry;
//映射sql语句与数据库操作对象关系以及sql关联的sql标签信息
private final MappedStatement mappedStatement;
//存储的需要进行赋值参数内容
private final Object parameterObject;
//输送的具体的sql语句
private final BoundSql boundSql;
//包含myBatis框架核心配置文件信息和sql映射文件信息
private final Configuration configuration;
```

下面查看DefaultParameterHandler中setParameters()的实现，代码如下所示：

```java
  public void setParameters(PreparedStatement ps) {
    ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
    //// parameterMappings是对#{}里给出的参数信息的封装，即这个SQL是个参数化SQL时会存在（SQL语句中带占位符?）
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    if (parameterMappings != null) {
      for (int i = 0; i < parameterMappings.size(); i++) {
        ParameterMapping parameterMapping = parameterMappings.get(i);
        //存储过程才存在输出参数，所以当参数不是输出参数的时候，就需要设置
        if (parameterMapping.getMode() != ParameterMode.OUT) {
          Object value;
            //这里的propertyName对应的是一个additionalParameters Map对象经过封装的key值，而不是Bean的属性
          String propertyName = parameterMapping.getProperty();
            //判断propertyName是additionalParameters中的key
          if (boundSql.hasAdditionalParameter(propertyName)) { 
            value = boundSql.getAdditionalParameter(propertyName);
          } else if (parameterObject == null) {
            value = null;
          } 
          //如果在typeHandlerRegistry中已经注册了这个参数的Class对象,value直接等于Method传进来的参数
          else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
            value = parameterObject;
          } 
          //否则是Map或Bean,List或Array被封装成了Map，通过Key得到Value；Bean是通过封装的MataObject对象，Bean通过反射得到，再用propertyName得到相应的Value。
          else {
            MetaObject metaObject = configuration.newMetaObject(parameterObject);
            value = metaObject.getValue(propertyName);
          }
          //得到parameterMapping的TypeHandler,均为IntegerTypeHandler
          TypeHandler typeHandler = parameterMapping.getTypeHandler();
          //得到相应parameterMapping的JdbcType,如果没有在#{}中显式的指定JdbcType，则为null
          JdbcType jdbcType = parameterMapping.getJdbcType();
          //当value和jdbcType都为null执行执行setNull操作时，会报OTHER的错误，因此指定JdbcType
          if (value == null && jdbcType == null) {
            jdbcType = configuration.getJdbcTypeForNull();
          }
          try {
            //具体实现ps.set***
            typeHandler.setParameter(ps, i + 1, value, jdbcType);
          } catch (TypeException e) {
            throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
          } catch (SQLException e) {
            throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
          }
        }
      }
    }
  }
```

上面的的setParameter()是在BaseTypeHandler中实现的，它是IntegerTypeHandler等的父类，这里采用了模板模式。setParameter()的代码如下：

```java
  public void setParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException {
    if (parameter == null) {
      if (jdbcType == null) {
        throw new TypeException("JDBC requires that the JdbcType must be specified for all nullable parameters.");
      }
      try {
        //执行PreparedStatement的setNull方法
        ps.setNull(i, jdbcType.TYPE_CODE);
      } catch (SQLException e) {
        throw new TypeException("Error setting null for parameter #" + i + " with JdbcType " + jdbcType + " . " +
                "Try setting a different JdbcType for this parameter or a different jdbcTypeForNull configuration property. " +
                "Cause: " + e, e);
      }
    } else {
      try {
        //进行参数设置，是抽象方法，具体的实现在子类中
        setNonNullParameter(ps, i, parameter, jdbcType);
      } catch (Exception e) {
        throw new TypeException("Error setting non null for parameter #" + i + " with JdbcType " + jdbcType + " . " +
                "Try setting a different JdbcType for this parameter or a different configuration property. " +
                "Cause: " + e, e);
      }
    }
  }
```

下面对上面所说的setNull()进行说名：

```java
public void setNull(int parameterIndex, int sqlType) throws SQLException{}
//parameterIndex:整形参数，第一个为1，第二个为2...
//sqlType必须为java.sql.Types 中定义的SQL类型代码 
//若parameterIndex 不对应于 SQL 语句中的参数标记；发生数据库访问错误；或在关闭的 PreparedStatement上调用此方法，会抛出SQLException异常。
```

由上面代码知，当SQL类型代码为1111(OTHER)时，就会出现报错：java.sql.SQLException: 无效的列类型: 1111，解决方案是在其后加上jdbcType，示例代码如下：

```xml
<insert id="insertUser" parameterType="com.luis.domain.User"> 
	insert into t_user(s_id,name,age) 
    values (
    #{SId,jdbcType=INTEGER},
    #{name,jdbcType=VARCHAR},
    #{age,jdbcType=INTEGER}
    );
</insert>
```

另有博主验证在ibatis2 可以正常的执行 数据库可以正常的插入数据。

参见：http://www.cnblogs.com/panxuejun/p/6163779.html
