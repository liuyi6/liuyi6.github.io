---
title: Mybatis插件开发
date: 2018-12-08 09:24:19
tags: [mybatis]
categories: mybatis
---
前面几篇文章介绍了Mybtis中四个重要的对象，其中提到它们都是在Configuration中被创建的，我们一起看一下创建四大对象的方法，代码如下所示：

```java
public ParameterHandler newParameterHandler(MappedStatement mappedStatement, Object parameterObject, BoundSql boundSql) {
    ParameterHandler parameterHandler = mappedStatement.getLang().createParameterHandler(mappedStatement, parameterObject, boundSql);
    parameterHandler = (ParameterHandler) interceptorChain.pluginAll(parameterHandler);
    return parameterHandler;
  }

  public ResultSetHandler newResultSetHandler(Executor executor, MappedStatement mappedStatement, RowBounds rowBounds, ParameterHandler parameterHandler,
      ResultHandler resultHandler, BoundSql boundSql) {
    ResultSetHandler resultSetHandler = new DefaultResultSetHandler(executor, mappedStatement, parameterHandler, resultHandler, boundSql, rowBounds);
    resultSetHandler = (ResultSetHandler) interceptorChain.pluginAll(resultSetHandler);
    return resultSetHandler;
  }

  public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
    StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);
    statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);
    return statementHandler;
  }

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

重点关注每个方法中的这样一个语句：

```java
parameterHandler = (ParameterHandler) interceptorChain.pluginAll(parameterHandler);
resultSetHandler = (ResultSetHandler) interceptorChain.pluginAll(resultSetHandler);
statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);
executor = (Executor) interceptorChain.pluginAll(executor);
```

我们看到前面已将创建出了相关的对象，那么这里的pluginAll()的作用是什么？下面我们针对pluginAll()进行说明。

我们进入InterceptorChain中的pluginAll()方法，具体代码如下所示：

```java
  public Object pluginAll(Object target) {
    for (Interceptor interceptor : interceptors) {
      target = interceptor.plugin(target);
    }
    return target;
  }
```

在PluginAll()方法中，以循环的方式调用了plugin()，跟踪代码进入Interceptor接口，其代码如下：

```java
public interface Interceptor {

  Object intercept(Invocation invocation) throws Throwable;
    
  Object plugin(Object target);

  void setProperties(Properties properties);

}
```

Interceptor是一个顶级接口，且没有实现类，那么调用它的意义何在？

其实，这是Mybatis给开发者留下的“后门”，它采用了动态代理模式的思想，允许开发者进行插件开发，开发者可以通过实现Interceptor接口实现对四大对象的监控处理。

在PluginAll()方法中，要注意的是：

* 形参Object  target，这个是Executor、ParameterHandler、ResultSetHandler、StatementHandler接口的实现类，换句话说，plugin方法是要为Executor、ParameterHandler、ResultSetHandler、StatementHandler的实现类生成代理，从而在调用这几个类的方法的时候，其实调用的是InvocationHandler的invoke方法，这里应用动态代理模式的思想。

* 这里的target是通过for循环不断赋值的，也就是说如果有多个拦截器，那么如果用P表示代理，生成第一次代理为P(target)，生成第二次代理为P(P(target))，生成第三次代理为P(P(P(target)))，不断嵌套下去，这就得到一个重要的结论：**<plugins\>...</plugins\>中后定义的<plugin\>实际其拦截器方法先被执行**，因为根据这段代码来看，后定义的<plugin\>代理实际后生成，包装了先生成的代理，自然其代理方法也先执行。

plugin方法中调用MyBatis提供的现成的生成代理的方法Plugin.wrap(Object target, Interceptor interceptor)，接着看下wrap方法的源码实现，代码如下：

```java
public static Object wrap(Object target, Interceptor interceptor) {
    Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);
    Class<?> type = target.getClass();
    Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
    if (interfaces.length > 0) {
      return Proxy.newProxyInstance(
          type.getClassLoader(),
          interfaces,
          new Plugin(target, interceptor, signatureMap));
    }
    return target;
  }
```

其中：Proxy.newProxyInstance()就是动态代理模式的体现。动态代理模式的相关讲解具体参见设计模式系列。

首先看第二行代码：getSignatureMap(interceptor)，getSignatureMap()代码如下：

```java
private static Map<Class<?>, Set<Method>> getSignatureMap(Interceptor interceptor) {
    Intercepts interceptsAnnotation = interceptor.getClass().getAnnotation(Intercepts.class);
    if (interceptsAnnotation == null) {
      throw new PluginException("No @Intercepts annotation was found in interceptor " + interceptor.getClass().getName());      
    }
    Signature[] sigs = interceptsAnnotation.value();
    Map<Class<?>, Set<Method>> signatureMap = new HashMap<Class<?>, Set<Method>>();
    for (Signature sig : sigs) {
      Set<Method> methods = signatureMap.get(sig.type());
      if (methods == null) {
        methods = new HashSet<Method>();
        signatureMap.put(sig.type(), methods);
      }
      try {
        Method method = sig.type().getMethod(sig.method(), sig.args());
        methods.add(method);
      } catch (NoSuchMethodException e) {
        throw new PluginException("Could not find method on " + sig.type() + " named " + sig.method() + ". Cause: " + e, e);
      }
    }
    return signatureMap;
  }
```

getSignatureMap()先拿@Intercepts注解，如果没有定义@Intercepts注解，抛出异常，这意味着**使用MyBatis的插件，必须使用注解方式**。接着拿到@Intercepts注解下的所有@Signature注解，获取其type属性（表示具体某个接口），再根据method与args两个属性去type下找方法签名一致的方法Method（如果没有方法签名一致的就抛出异常，此签名的方法在该接口下找不到），能找到的话key=type，value=Set<Method\>，添加到signatureMap中，构建出一个方法签名映射。这里的作用就是通过定义的@Intercepts注解，要Executor、StatementHandler、ParameterHandler、ResultSetHandler下所有Method。

参考了：https://www.cnblogs.com/xrq730/p/6984982.html

根据上面分析总结Mybatis插件开发的步骤如下：

1. 开发一个Interceptor接口的实现类
2. 重写Interceptor接口方法
3. 在mybatis框架通过核心配置注册拦截器
4. 通过@Intercpts指定当前拦截器监听的对象类型和行为
