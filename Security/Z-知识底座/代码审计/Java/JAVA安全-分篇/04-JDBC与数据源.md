---
type: 查阅
status: active
created: 2026-11-13
updated: 2026-11-13
tags: [Java, 代码审计, SQL注入, JDBC]
本线: 只查不读
源: JAVA安全-全本（2026-11-13 拆）
---

# 04 · JDBC 与数据源

> 查阅区 · **只查不读**，不进学习路径。原文拆自 `JAVA安全-全本`，正文一字未改，仅调标题层级与图片相对路径。
> 什么时候翻：判断项目走 DriverManager 还是 DataSource，SQL 拼接点在哪一层

## JDBC基础

`JDBC(Java Database Connectivity)`:Java抽象出的对数据库进行存在的API接口

Java通过`java.sql.DriverManager`来管理所有数据库的驱动注册，所以如果想要建立数据库连接需要先在`java.sql.DriverManager`中注册对应的驱动类，然后调用`getConnection`方法才能连接上数据库。

JDBC定义了一个叫`java.sql.Driver`的接口类负责实现对数据库的连接，所有的数据库驱动包都必须实现这个接口才能够完成数据库的连接操作。

- 连接数据库

  ```java
  String CLASS_NAME = "com.mysql.jdbc.Driver";
  String URL = "jdbc:mysql://localhost:3306/mysql"
  String USERNAME = "root";
  String PASSWORD = "root";
  
  Class.forName(CLASS_NAME);// 用数据库驱动名注册JDBC驱动类
  Connection connection = DriverManager.getConnection(URL, USERNAME, PASSWORD);
  ```

### DataSource

在实际的Java项目中用数据源(`javax.sql.DataSource`)来代替`DriverManager`管理数据库的连接。

- 数据库源

  - Web容器自带
    -  `Tomcat JNDI DataSource`
    - `Resin JNDI DataSource`
  - 第三方
    - `DBCP`
    - `C3P0`
    - `Druid`
    - `Mybatis DataSource`

  - 在Spring MVC中可以自由的选择第三方数据源，通常会定义一个`DataSource Bean`用于配置和初始化数据源对象，然后在Spring中通过Bean注入的方式获取数据源对象。
