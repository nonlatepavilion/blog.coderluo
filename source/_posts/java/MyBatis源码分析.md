---
title: Mybatis源码分析
tags:
  - Mybatis
date: 2019-07-27 10:18:30
categories:
  - 源码阅读
cover: true
author: coderluo

---



> 这篇文章我们来深入阅读下Mybatis的源码，希望以后可以对底层框架不那么畏惧，学习框架设计中好的思想；



## 架构原理



### 架构图



![](http://media.coderluo.top/计算机系统漫游/6sf4i.png)



### 架构流程图

![架构流程图](http://media.coderluo.top/计算机系统漫游/mwc26.png)



上面这两幅图来源于网络，不过画的很好，基本说明了Mybatis的架构流程。



说明：

1. Mybatis配置文件

   - SqlMapConfig.xml，此文件作为mybatis的全局配置文件，配置了mybatis的运行环境等信息。

   - Mapper.xml，此文件作为mybatis的sql映射文件，文件中配置了操作数据库的sql语句。此文件需要在 

   SqlMapConfig.xml中加载。

2. SqlSessionFactory

   - 通过mybatis环境等配置信息构造SqlSessionFactory，即会话工厂。

3. SqlSession

   - 通过会话工厂创建sqlSession即会话，程序员通过sqlsession会话接口对数据库进行增删改查操作。

4. Executor执行器

   - mybatis底层自定义了Executor执行器接口来具体操作数据库，Executor接口有两个实现，一个是基本执行器 

   （默认）、一个是缓存执行器，sqlsession底层是通过executor接口操作数据库的。

5. MappedStatement

   - 它也是mybatis一个底层封装对象，它包装了mybatis配置信息及sql映射信息等。mapper.xml文件中一个 

   select\insert\update\delete标签对应一个Mapped Statement对象，select\insert\update\delete 

   标签的id即是Mapped statement的id。

   - Mapped Statement对sql执行输入参数进行定义，包括HashMap、基本类型、pojo，Executor通过Mapped 

   Statement在执行sql前将输入的java对象映射至sql中，输入参数映射就是jdbc编程中对



### 调用流程图

![结构](http://media.coderluo.top/计算机系统漫游/bwkyg.png)



**Executor**

​	MyBatis执行器，是MyBatis 调度的核心，负责SQL语句的生成和查询缓存的维护

**StatementHandler**

​	封装了JDBC Statement操作，负责对JDBC statement 的操作，如设置参数、将Statement结果集转换成List集 

合。

**ParameterHandler**

​	负责对用户传递的参数转换成JDBC Statement 所需要的参数

**ResultSetHandler**

​	负责将JDBC返回的ResultSet结果集对象转换成List类型的集合

**TypeHandler**

​	负责java数据类型和jdbc数据类型之间的映射和转换

**SqlSource**

​	负责根据用户传递的parameterObject，动态地生成SQL语句，将信息封装到BoundSql对象中，并返回BoundSql表 

示动态生成的SQL语句以及相应的参数信息





## 源码解析



### 加载全局配置文件

- 找入口：SqlSessionFactoryBuilder#build方法



```
SqlSessionFactoryBuilder#build 构建SqlSessionFactory

	XMLConfigBuilder#parse 全局配置文件解析，封装成Configuration对象

		#parseConfiguration 从根路径开始解析，加载的信息设置到Configuration对象中

			#mapperElement 解析mapper映射文件

				XMLMapperBuilder#parse 具体解析mapper映射文件

					SqlSessionFactoryBuilder#build：创建SqlSessionFactory接口的默认实现类

```





- 总结

```java
1.SqlSessionFactoryBuilder创建SqlsessionFactory时，需要传入一个Configuration对象。 
2.XMLConfigBuilder对象会去实例化Configuration。 
3.XMLConfigBuilder对象会去初始化Configuration对象。 
	通过XPathParser去解析全局配置文件，形成Document对象 
	通过XPathParser去获取指定节点的XNode对象。 
	解析Xnode对象的信息，然后封装到Configuration对象中
```

- 相关类和接口

```java
|--SqlSessionFactoryBuilder 
|--XMLConfigBuilder 
|--XPathParser 
|--Configuration
```





### 加载映射配置文件

- 找入口：XMLConfigBuilder#mapperElement方法



```java
XMLConfigBuilder#mapperElement:解析全局配置文件中的<mappers>标签 
	|--XMLMapperBuilder#构造方法：专门用来解析映射文件的 
		|--XPathParser#构造方法： 
			|--XPathParser#createDocument()：创建Mapper映射文件对应的Document对象 
			|--MapperBuilderAssistant#构造方法：用于构建MappedStatement对象的 
		|--XMLMapperBuilder#parse()： 
			|--XMLMapperBuilder#configurationElement：专门用来解析mapper映射文件 
				|--XMLMapperBuilder#buildStatementFromContext：用来创建MappedStatement对象的 
					|--XMLMapperBuilder#buildStatementFromContext 
						|--XMLStatementBuilder#构造方法：专门用来解析MappedStatement 
						|--XMLStatementBuilder#parseStatementNode: 
							|--MapperBuilderAssistant#addMappedStatement:创建 MappedStatement对象 
								|--MappedStatement.Builder#构造方法 
								|--MappedStatement.Builder#build方法：创建MappedStatement对象，并存储 到Configuration对象中
```



- 相关接口和类

```java
|--XMLConfigBuilder 
|--XMLMapperBuilder 
|--XPathParser 
|--MapperBuilderAssistant 
|--XMLStatementBuilder 
|--MappedStatement
```





### SqlSource创建流程



- 找入口：XMLLanguageDriver#createSqlSource



```java
XMLLanguageDriver#createSqlSource 创建SqlSource，解析SQL，封装SQL语句（出参数绑定）和入参信息

​	XMLScriptBuilder 构造函数：初始化动态SQL中的节点处理器集合
​		XMLScriptBuilder#parseScriptNode 
​			#parseDynamicTags 解析select\insert\ update\delete标签中的SQL语句，最终将解析到的SqlNode封装到MixedSqlNode中的List集合中
​			DynamicSqlSource 构造方法：如果SQL中包含${}和动态SQL语句，则将SqlNode封装到DynamicSqlSource
​			RawSqlSource 构造方法：如果SQL中包含#{}，则将SqlNode封装到RawSqlSource中，并指定parameterType
​				SqlSourceBuilder#parse
​					ParameterMappingTokenHandler 构造方法
​						GenericTokenParser#构造方法,指定待分析的openToken和closeToken并指定处理器
​							GenericTokenParser#parse 解析#{}
​								ParameterMappingTokenHandler#handleToken  处理token（#{}/${}）
​									#buildParameterMapping 创建ParameterMapping对象
​								StaticSqlSource 构造方法，将解析之后的sql信息，封装到StaticSqlSource 对象
```



- 相关类和接口

```java
|--XMLLanguageDriver 
|--XMLScriptBuilder 
|--SqlSource 
|--SqlSourceBuilder
```





### 创建Mapper代理对象

- 找入口：DefaultSqlSession#getMapper



```java
|--DefaultSqlSession#getMapper：获取Mapper代理对象 
​	|--Configuration#getMapper：获取Mapper代理对象 
​		|--MapperRegistry#getMapper：通过代理对象工厂，获取代理对象 
​			|--MapperProxyFactory#newInstance：调用JDK的动态代理方式，创建Mapper代理
```



### SqlSession执行主流程

- 找入口：DefaultSqlSession#selectList()



```java
DefaultSqlSession#selectList
​	CachingExecutor#query
​		BaseExecutor#query 委托给BaseExecutor执行
​			#queryFromDatabase
​			SimpleExecutor#doQuery  执行查询
​				Configuration#newStatementHandler  创建路由功能的StatementHandler，根据MappedStatement中的StatementType
​			SimpleExecutor#prepareStatement  设置PreapreStatement 的参数
​			BaseExecutor#getConnection 获取数据库连接
​				BaseStatementHandler#prepare 创建Statement PrepareStatement、Statement、CallableStatement）
​				PreparedStatementHandler#parameterize 设置参数
​				PreparedStatementHandler#query 执行SQL语句（已经设置过参数），并且映射结果集
​					com.mysql.jdbc.PreparedStatement#execute 调用JDBC的api执行Statement
​						DefaultResultSetHandler#handleResultSets  处理结果集
```



- 相关接口和类

```java
|--DefaultSqlSession 
|--Executor 
	|--CachingExecutor 
	|--BaseExecutor 
	|--SimpleExecutor 
|--StatementHandler 
	|--RoutingStatementHandler 
	|--PreparedStatementHandler 
|--ResultSetHandler 
	|--DefaultResultSetHandler 
```

​					

### BoundSql获取流程

- **找入口：MappedStatement#getBoundSql方法**

```java
MappedStatement#getBoundSql
​	DynamicSqlSource#getBoundSql 
​		SqlSourceBuilder#parse 执行解析：将带有#{}的SQL语句进行解析，然后封装到StaticSqlSource中
​			GenericTokenParser  #构造方法，指定待分析的openToken和closeToken，并指定处理器
​				GenericTokenParser#parse 解析SQL语句，处理openToken和closeToken中的内容 
​					ParameterMappingTokenHandler#handleToken 处理token（#{}/${}）  
​						#buildParameterMapping  创建 ParameterMapping对象
​							StaticSqlSource#构造方法 ： 将解析之后的SQL信息，封装到StaticSqlSource
|--RawSqlSource#getBoundSql 
​	|--StaticSqlSource#getBoundSql 
​		|--BoundSql#构造方法：将解析后的sql信息、参数映射信息、入参对象组合到BoundSql对象中 
```



### 参数映射流程

- 找入口： 其实就是SqlSession执行流程中的 PreparedStatementHandler#parameterize 

```

|--PreparedStatementHandler#parameterize：设置PreparedStatement的参数 
​	|--DefaultParameterHandler#setParameters：设置参数 
​		|--BaseTypeHandler#setParameter： 
​			|--xxxTypeHandler#setNonNullParameter:调用PreparedStatement的setxxx方法
```





### 处理结果集



- 找入口：其实就是SqlSession执行流程中的 DefaultResultSetHandler#handleResultSets



```
|--DefaultResultSetHandler#handleResultSets 
​	|--DefaultResultSetHandler#handleResultSet 
​		|--DefaultResultSetHandler#handleRowValues 
​			|--DefaultResultSetHandler#handleRowValuesForSimpleResultMap 
​				|--DefaultResultSetHandler#getRowValue 
​					|--DefaultResultSetHandler#createResultObject：创建映射结果对象 
​					|--DefaultResultSetHandler#applyAutomaticMappings 
​					|--DefaultResultSetHandler#applyPropertyMappings 
```



基本上Mybatis的流程就是这样了，其中还有很多实现细节暂时看不太懂。 我认为学习框架源码分为两步：

1. 抓住主线，掌握框架的原理和流程；
2. 理解了处理思路之后，再去理解面向对象思想和设计模式的用法；

目前第一步尚有问题，需要多走几遍源码，加深下理解，一起加油~~



















