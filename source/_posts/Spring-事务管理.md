---
title: Spring 事物管理
date: 2018-03-25 22:59:27
tags: Spring
categories: 
- JAVA WEB
- Spring
---

## 事务管理

<!--more-->

事务有四个特性：ACID

- 原子性（Atomicity）：事务是一个原子操作，由一系列动作组成。事务的原子性确保动作要么全部完成，要么完全不起作用。
- 一致性（Consistency）：一旦事务完成（不管成功还是失败），系统必须确保它所建模的业务处于一致的状态，而不是部分完成部分失败。在现实中的数据不应该被破坏。
- 隔离性（lsolation）：可能有许多事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。
- 持久性（Durability）：一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响，这样就能从任何系统奔溃中恢复过来。通常情况下，事务的结果被写到持久化存储器中。

![](http://img.blog.csdn.net/20160324011156424)





### 事务管理器

Spring并不直接管理事务，而是提供了多种事务管理器，他们将事务管理的职责委托给Hibernate或者JTA等持久化机制所提供的相关平台框架的事务来实现。

Spring事务管理器的接口是 **org.springframework.transaction.PlatformTransactionManager ** ,通过这个接口，sping 为各个平台如JDBC、Hibernate等都提供了对应的事务管理器，但是具体的实现就是各个平台自己的事情。接口的内容是

```java
public interface PlatformTransactionManager(){
	 // 由TransactionDefinition得到TransactionStatus对象
    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException; 
    // 提交
    Void commit(TransactionStatus status) throws TransactionException;  
    // 回滚
    Void rollback(TransactionStatus status) throws TransactionException;  
}
```



- JDBC事务

  如果应用程序直接使用JDBC来进行持久化，DataSourceTransactionManager会为你处理事务边界，为了使用DataSourceTransactionManager , 需要使用如下的 xml 将其装配到应用程序的上下文定义中，

  ```xml
  <bean id ="transactionManager" class ="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  	<property name ="dataSource"  ref ="dataSource" />
  </bean>
  ```

  实际上，DataSourceTransactionManager 是通过调用java.sql.Connection来管理事务，而后者是通过DataSource获取到的。通过调用连接的commit()方法来提交事务，同样，事务失败则通过调用rollback()方法进行回滚。