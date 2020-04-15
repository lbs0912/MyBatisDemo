
# README


此处给出一个 MyBatis 入门Demo，并结合 MyBatis 的工作流程进行分析。


## 学习资料汇总
* [MyBatis 维基百科](https://zh.wikipedia.org/wiki/MyBatis)
* [MyBatis入门 | 掘金](https://juejin.im/post/5aa5c6fb5188255587232e5a)
* [MyBatis | github](https://github.com/mybatis)
* [MyBatis | W3CSchool](https://www.w3cschool.cn/mybatis/)



## Overview

MyBatis是一个Java持久化框架，它通过 XML 描述符或注解把对象与存储过程或 SQL 语句关联起来。



MyBatis 是支持定制化 SQL、存储过程以及高级映射的优秀的持久层框架。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以对配置和原生 Map 使用简单的 XML 或注解，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象) 映射成数据库中的记录。


与其他的对象关系映射框架不同，MyBatis 并没有将 Java 对象与数据库表关联起来，而是将 Java 方法与 SQL 语句关联。MyBatis 允许用户充分利用数据库的各种功能，例如存储过程、视图、各种复杂的查询以及某数据库的专有特性。如果要对遗留数据库、不规范的数据库进行操作，或者要完全控制 SQL 的执行，MyBatis是一个不错的选择。

与 JDBC 相比，MyBatis 简化了相关代码：SQL 语句在一行代码中就能执行。MyBatis 提供了一个映射引擎，声明式的把SQL语句执行结果与对象树映射起来。通过使用一种内建的类 XML 表达式语言，或者使用 Apache Velocity 集成的插件，SQL 语句可以被动态的生成。

MyBatis与Spring Framework和Google Guice集成，这使开发者免于依赖性问题。


![](https://image-bed-20181207-1257458714.cos.ap-shanghai.myqcloud.com/back-end-2020/mybatis-basic-1.png)

如上图所示，无论是 Mybatis、Hibernate 都是 对象关系映射 （`ORM`）的一种实现框架，都是对 JDBC 的一种封装！

持久层中几种常用的技术包括
* Hibernate
* jdbc
* SpringDAO

那我们为啥还要学 Mybatis 呢？现在 Mybatis 在业内大行其道，那为啥他能那么火呢？

Hibernate 是一个比较老旧的框架，用过他的同学都知道，只要你会用，用起来十分舒服...啥 sql 代码都不用写...但是呢，它也是有的缺点：处理复杂业务时，灵活度差, 复杂的 HQL 难写难理解，例如多表查询的 HQL 语句。

JDBC 很容易理解，就那么几个固定的步骤，就是开发起来太麻烦了，因为什么都要我们自己干。

而 SpringDAO 其实就是 JDBC 的一层封装，就类似于 dbutils 一样，没有特别出彩的地方....

我们可以认为，Mybatis 就是 jdbc 和 Hibernate 之间的一个平衡点。



## MyBatis 工作流程

1. 通过 Reader 对象读取 Mybatis 映射文件
2. 通过 `SqlSessionFactoryBuilder` 对象创建 `SqlSessionFactory` 对象
3. 获取当前线程的 `SQLSession`
4. 事务默认开启
5. 通过 `SQLSession` 读取映射文件中的操作编号，从而读取 SQL 语句
6. 提交事务
7. 关闭资源


## Mybatis 入门 Demo

* [一个简单的mybatis入门demo | CSDN](https://blog.csdn.net/magi1201/article/details/85078561)
* [MyBatisDemo 源码 | github-lbs0912](https://github.com/lbs0912/MyBatisDemo)


下面创建一个 MyBatis 入门 Demo，理解 MyBatis 的使用流程，源码可以从 [MyBatisDemo 源码 | github-lbs0912](https://github.com/lbs0912/MyBatisDemo) 获取。

### 数据库创建

使用下述 `sql` 脚本创建数据库，用于后续测试。


```
//DB.sql

create database mybatis default character set utf8 collate utf8_general_ci;

use mybatis;

create table country 
(id int primary key auto_increment,
 countryname varchar(255) null,
 countrycode varchar(255) null
);

show tables from mybatis;

desc country;

insert country(countryname, countrycode) values('中国', 'CN'),('美国', 'US'),('俄罗斯','RU'),('英国', 'GB'),('法国', 'FR'),('中国香港', 'HK');
```

查看数据中的数据如下所示。

```
mysql> select * from country;
+----+--------------+-------------+
| id | countryname  | countrycode |
+----+--------------+-------------+
|  1 | 中国         | CN          |
|  2 | 美国         | US          |
|  3 | 俄罗斯       | RU          |
|  4 | 英国         | GB          |
|  5 | 法国         | FR          |
|  6 | 中国香港     | HK          |
+----+--------------+-------------+
6 rows in set (0.00 sec)
```


### Java 工程

1. 创建一个基于Maven的Java工程。
2. 在 `pom.xml` 配置文件中引入 `mybatis` 和 `mysql` 依赖，并完成必要的配置。


```xml
//pom.xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>MybatisDemo01</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!-- 设置源代码编码格式为utf-8 -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!-- mybatis依赖 -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.6</version>
        </dependency>

        <!-- mysql 依赖 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.46</version>
        </dependency>

        <!-- slf4j依赖 -->
        <!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-api -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-log4j12 -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.25</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/log4j/log4j -->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>

        <!-- junit依赖 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

3. 在资源文件目录 `src/main/resources` 下创建 MyBatis 配置文件 `mybatis-config.xml`


```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 引入外部资源文件，默认同resources目录下 -->
    <properties resource="jdbc.properties"></properties>

    <settings>
        <!-- 设置驼峰匹配 -->
        <setting name="mapUnderscoreToCamelCase" value="true" />
        <!-- 配置指定使用log4j输出日志 -->
        <setting name="logImpl" value="LOG4J" />
    </settings>

    <!-- 设置包扫描(别名) -->
    <typeAliases>
        <package name="com.lbs0912.model" />
    </typeAliases>

    <!-- 配置环境：可以配置多个环境，default：配置某一个环境的唯一标识，表示默认使用哪个环境 -->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC" />
            <dataSource type="POOLED">
                <!-- 配置连接信息 -->
                <property name="driver" value="${jdbc.driverClass}" />
                <property name="url" value="${jdbc.url}" />
                <property name="username" value="${jdbc.username}" />
                <property name="password" value="${jdbc.password}" />
            </dataSource>
        </environment>
    </environments>

    <!-- 配置映射文件：用来配置sql语句和结果集类型等，路径与resources下面路径相同 -->
    <mappers>
        <mapper resource = "com/lbs0912/model/mapper/CountryMapper.xml"/>
    </mappers>
</configuration>
```

4. 在上述第3步中，使用了外部资源文件 `jdbc.properties` 进行数据库的配置。因此，需要在 `mybatis-config.xml` 的同级目录下创建 `jdbc.properties` 配置文件。


```
//jdbc.properties

jdbc.driverClass=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis
jdbc.username=root
jdbc.password=mysql113459
```


5. 在上述第3步中，使用了外部资源文件 `log4j.properties` 进行数据库的配置。因此，需要在 `mybatis-config.xml` 的同级目录下创建 `log4j.properties` 配置文件。


```
#全局配置
log4j.rootLogger=ERROR, stdout
#mybatis日志配置
log4j.logger.cn.mybatis.mapper=TRACE
#控制台输出配置
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```


6. 下面创建一个实体类对象 `Country`，对应数据库中每条记录。


```
//Country.java

package com.lbs0912.model;


public class Country {

    private int id;
    private String countryname;
    private String countrycode;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getCountryname() {
        return countryname;
    }

    public void setCountryname(String countryname) {
        this.countryname = countryname;
    }

    public String getCountrycode() {
        return countrycode;
    }

    public void setCountrycode(String countrycode) {
        this.countrycode = countrycode;
    }

}
```

7. 最后创建主类 `Main.java`，用于测试 MyBatis 功能

```
//Main.java

package com.lbs0912.app;

import java.io.IOException;
import java.io.Reader;
import java.util.List;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import com.lbs0912.model.Country;


public class Main {

    private static SqlSessionFactory sqlSessionFactory;

    /**
     * 初始化sqlSessionFactory
     */
    public static void init() {
        try {
            Reader reader = Resources.getResourceAsReader("mybatis-config.xml");
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
            reader.close();
        } catch (IOException ignore) {
            ignore.printStackTrace();
        }
    }

    public static void main(String[] args) {

        // 连接数据库
        init();

        SqlSession sqlSession = sqlSessionFactory.openSession();
        try {
            List<Country> countryList = sqlSession.selectList("selectAll");
            printCountryList(countryList);
        } finally {
            sqlSession.close();
        }
    }

    /**
     * 打印country信息
     * @param countryList
     */
    private static void printCountryList(List<Country> countryList) {
        for (Country country : countryList) {
            System.out.printf("%-4d%4s%4s\n", country.getId(), country.getCountryname(), country.getCountrycode());
        }
    }
}
```

8. 运行 `Main.java` ，在控制台可以看到如下打印信息


```
/Library/Java/JavaVirtualMachines/jdk1.8.0_231.jdk/Contents/Home/bin/java ...

1     中国  CN
2     美国  US
3    俄罗斯  RU
4     英国  GB
5     法国  FR
6   中国香港  HK

Process finished with exit code 0
```


### 程序分析




通过SqlSessionFactory工厂对象获取一个SqlSession
通过SqlSession的selectList方法查找到CountryMapper.xml中id="selectAll" 的方法，执行SQL进行查询
mybatis底层使用jdbc执行SQL，获得查询结果集ResultSet后，根据resultType的配置将结果映射为Country类型的集合，返回查询结果。



下面结合上一章节 `MyBatis 工作流程` 中的几个步骤进行分析。

1. 通过 `Reader` 对象读取 `Mybatis` 映射文件 `mybatis-config.xml`。再通过 `SqlSessionFactoryBuilder` 建造类使用 `Reader` 创建 `SqlSessionFactory` 工厂对象。


```
//main.java

import java.io.Reader;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;

public class Main {

    private static SqlSessionFactory sqlSessionFactory;

    /**
     * 初始化sqlSessionFactory
     */
    public static void init() {
        try {
            Reader reader = Resources.getResourceAsReader("mybatis-config.xml");
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
            reader.close();
        } catch (IOException ignore) {
            ignore.printStackTrace();
        }
    }

    // ...
}
```

在创建 `SqlSessionFactory` 对象的过程中，首先解析 `mybatis-config.xml` 配置文件，读取配置文件中的 `mappers` 配置后会读取全部的 `CountryMapper.xml` 进行具体方法的解析，在这些解析完成后，`SqlSessionFactory` 就包含了所有的属性配置和执行 SQL 的信息。



2. 通过 `SqlSessionFactory` 工厂对象获取一个 `SqlSession`。通过 `SqlSession` 的 `selectList` 方法查找到 `CountryMapper.xml` 中 `id="selectAll"` 的方法，执行 `SQL` 进行查询。

```
public static void main(String[] args) {
    // 连接数据库
    init();

    SqlSession sqlSession = sqlSessionFactory.openSession();
    try {
        List<Country> countryList = sqlSession.selectList("selectAll");
        printCountryList(countryList);
    } finally {
        sqlSession.close();
    }
}
```

3. MyBatis 底层使用 jdbc 执行 SQL，获得查询结果集 ResultSet 后，根据 `resultType` 的配置将结果映射为 `Country`类型的集合，返回查询结果。其中 `resultType` 是在 `CountryMapper.xml` 配置中指定的。


```xml
// CountryMapper.xml

<!-- 定义当前XML的命名空间 -->
<mapper namespace="com.lbs0912.model.mapper.CountryMapper">
    <!-- ID属性，定义当前select查询的唯一ID；resultType，定义当前查询的返回值类型，如果没有设置别名，则需要写成cn.mybatis.model.Country -->
    <select id="selectAll" resultType="Country">
		select id, countryname, countrycode from country
	</select>
</mapper>
```

4. 最后关闭会话资源


```
 sqlSession.close();
```


