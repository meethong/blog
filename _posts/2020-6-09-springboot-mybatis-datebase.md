---
layout: post
title: Mybatis获取国产数据库厂商标识
categories: [Springboot, Mybatis,DatabaseIdProvider]
description: Springboot Mybatis DatabaseIdProvider
keywords: Springboot, Mybatis,DatabaseIdProvider
---

由于国产数据库没有相关文档说明，所以并不知道他的数据库标识是啥,刚好看到Mybatis官网说可以根据`DatabaseMetaData#getDatabaseProductName()` 返回的字符串去获取数据库的厂商标识！！那就直接获取看一看

## 关于

MyBatis官网的介绍是MyBatis 可以根据不同的数据库厂商执行不同的语句，这种多厂商的支持是基于映射语句中的 `databaseId` 属性。 MyBatis 会加载带有匹配当前数据库 `databaseId` 属性和所有不带 `databaseId` 属性的语句。 如果同时找到带有 `databaseId` 和不带 `databaseId` 的相同语句，则后者会被舍弃。 为支持多厂商特性。

### test类

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;

@SpringBootTest
class Springboot03WebApplicationTests {
    @Autowired
    DataSource dataSource;
    @Test
    public void conn() throws SQLException{
        //获得连接
        Connection connection =   dataSource.getConnection();
        String dbName = connection.getMetaData().getDatabaseProductName();
        String dbVersion=connection.getMetaData().getDatabaseProductVersion();
        System.out.println("数据库名称是：" + dbName + "；版本是：" + dbVersion);
        //关闭连接
        connection.close();
    }
```

### 相关对应

```
金仓数据库8:KingbaseES
神通数据库7:OSCAR
达梦数据库8:DM DBMS
```

## 参考

[mybatis官方文档](https://mybatis.org/mybatis-3/zh/configuration.html#databaseIdProvider)