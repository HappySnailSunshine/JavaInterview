# Mybatis

自己总结这个过程是非常重要的，自己总结自己思考才知道自己学会了没有。



> 学习进度，第三个 10分钟

# Bilibili  狂神说

## 简介

### 什么是 MyBatis

![MyBatis logo](../media/pictures/Mybatis.assets/mybatis-logo.png)

- MyBatis 是一款优秀的**持久层框架**

- 它支持自定义 SQL、存储过程以及高级映射

- MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作

- MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，

  普通老式 Java 对象）为数据库中的记录。

- MyBatis 本是[apache](https://baike.baidu.com/item/apache/6265)的一个开源项目[iBatis](https://baike.baidu.com/item/iBatis), 2010年这个项目由apache software foundation 迁移到了google code，并且改名为MyBatis 。2013年11月迁移到Github

- 2013年11月迁移到Github。



#### 如何获取

- Maven仓库：

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.6</version>
</dependency>
```

- Github：https://github.com/mybatis/mybatis-3.git
- 中文文档：https://mybatis.org/mybatis-3/zh/index.html



### 持久化

数据持久化

- 持久化就是将程序的数据在持久状态和瞬时状态转换的过程
- 内存  断电即失去
- 数据库（JDBC）：IO文件持久化

为什么要持久化？

- 有一些对象不能让他丢掉
- 内存太贵啦



### 持久层

Dao层，Service层，Controller层....

- 完成持久化工作的代码块
- 层界限十分明显



### 为什么要用Mybatis？

- 帮助程序员将数据存入数据库。

- 方便
- 传统的JDBC太复杂了，简化，框架，自动化。
- 不用Mybatis也行，使用了更加方便。技术本身没有高低之分，只有使用技术的人有高低之分。
- 优点
  - 简单易学
  - 灵活
  - 解除sql与程序代码的耦合
  - 提供映射标签
  - 提供对象关系映射标签，支持对象关系组建维护
  - 提供xml标签，支持编写动态sql。



**最重要的一点：使用的人多。**

42分钟



## 第一个Mybatis程序

### Mybatis配置文件

```xml
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--加载配置文件-->
    <properties resource="db.properties">
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://47.92.208.93:3306/mybatis?useUnicode=true&amp;characterEncoding=utf-8"/>
    </properties>
    <!--别名-->
    <typeAliases>
        <!--<typeAlias type="com.cskaoyan.bean.User" alias="userz"/>-->
        <!--别名是类名的小写形式 user car UserDetail userdetail-->
        <package name="com.cskaoyan.bean"/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/><!--MANAGED-->
            <dataSource type="POOLED"><!--池-->
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <!--mybatis的映射文件 sql和代码进行分离-->
    <!--下面是常使用的几种方式,都可以找到UserMapper-->
    <mappers>
        <!--下面的一个一个放开测试-->
        <!--相对于classpath下的映射文件的路径-->
        <mapper resource="com/cskaoyan/mapper/UserMapper.xml"/>
        <!--class→对应接口的全类名-->
        <!--<mapper class="com.cskaoyan.mapper.UserMapper"/>-->
        <!--url-->
        <!--<mapper url="file:\\\D:\JavaEEWorkplace\mybatis2\demo01_mapper\src\main\resources\com\cskaoyan\mapper\UserMapper.xml"/>-->
        <!--使用扫描包的实行加载映射文件-->
        <!--<package name="com.cskaoyan.mapper"/>-->   <!--使用的配置-->
    </mappers>
</configuration>
```

里面吧数据库需要的驱动，URL，用户名，密码弄上。可以写死，也可以写在配置文件中，参考mybatis2。



### 实体类

```java
public class User {
    private int id;
    private String username;
    private String password;
    private int age;
    private String gender;
    //省略setter和getter
}
```



### mapper

接口



### UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--namespace要等于接口的全类名-->
<mapper namespace="com.cskaoyan.mapper.UserMapper">
    
    <!--根据id查询用户-->
    <select id="queryUserById" resultType="com.cskaoyan.bean.User">
        select id,username,password,age,gender from user where id = #{id}
    </select>
    
    <!--插入用户-->
    <insert id="insertUser">
        insert into user (username,password,age,gender) values (
        #{username},
        #{password},
        #{age},
        #{gender}
        )
    </insert>
    
    <!--根据id删除用户-->
    <delete id="deleteUserById">
        delete from user where id = #{id}
    </delete>
</mapper>
```



## CRUD

### namespace

namespace中的包名要和Dao/mapper接口的包名一致。

### select

选择，查询语句

- id：就是对相应namespace中的方法名
- resultType：SQL执行的返回值
- parameterType： 参数类型









# Mybatis

## Mybatis入门

### Mybatis入门案例2

![](../media/pictures/Mybatis.assets/28ece45410fbf7d17f27d1ec0e7c350a.png)

写单元测试的时候，可以直接通过SqlSession获得dao层接口的实例

![](../media/pictures/Mybatis.assets/07abbc5e21e0462b1b2b74fabb35b645.png)

当我们去写一个接口中的方法，一定要有对应的标签与之对应

通过，如果新增标签可以不写对应的接口中的方法

### Mybatis配置文件

#### Properties

![](../media/pictures/Mybatis.assets/609b43213c289be20ad486b7f59a9aab.png)

#### Setting

缓存

懒加载→多表查询

#### typeAliases（别名→小名）

com.cskaoyan.bean.User→user

resultType、parameterType（不写）

![](../media/pictures/Mybatis.assets/c7c61d3c0288b956d84059be94f0dc33.png)

![](../media/pictures/Mybatis.assets/a31f6e293881fa06826ef6a64006cc1a.png)

#### typeHandler（类型转换）

#### plugins(插件)

pageHelper分页的插件

#### mappers的配置

![](../media/pictures/Mybatis.assets/1bb3eafcc70899559d13b54797db7054.png)

## log4j 日志

springboot日志

日志级别

格式

1. 导包2、引入日志的配置文件

### 导包

![](../media/pictures/Mybatis.assets/f0f0daeabd97d1af29ce03e509eba810.png)

### 引入配置文件

![](../media/pictures/Mybatis.assets/907147e6f24cbfdad6393ed0f4c9e151.png)

![](../media/pictures/Mybatis.assets/e9dade919d4c54288f1a44ec08ceb3d8.png)

![](../media/pictures/Mybatis.assets/d3afe2d67e9dc6f6fb2cdff8f801968a.png)

其中他们之间的优先级是有一个等级划分的!如果输出的是优先级为7的debug,则,控制台会输出他和他以下优先级的所有日志! 下面的优先级来自互联网,和上面的优先级有出入!以后看书学习正确的!......

![log4j配置详解 - stone - stonexmx 的博客](../media/pictures/Mybatis.assets/None.gif)FATAL       0 
![log4j配置详解 - stone - stonexmx 的博客](../media/pictures/Mybatis.assets/None.gif)ERROR     3 
![log4j配置详解 - stone - stonexmx 的博客](../media/pictures/Mybatis.assets/None.gif)WARN      4 
![log4j配置详解 - stone - stonexmx 的博客](../media/pictures/Mybatis.assets/None.gif)INFO         6 
![log4j配置详解 - stone - stonexmx 的博客](../media/pictures/Mybatis.assets/None.gif)DEBUG     7 

### 格式

![](../media/pictures/Mybatis.assets/244de120464061b43dbcdf17b45e2b7b.png)

%d 日期（date）ABSOLUTE(这是一个格式名)

%5p 优先级（privilege）5代表使用5个占位符

%c{1}+方法名 顺序从后往前取

%L 行号

%m 消息 输出的日志信息

%n 换行

### 输出自定义的日志

Logger

![](../media/pictures/Mybatis.assets/a6e9a9d169cee645ce576cb5bf107158.png)

## 映射

### 输入映射（重点）

#### 没有注解

##### 基本类型

![](../media/pictures/Mybatis.assets/83a1b468fd392ccecf45acaa8879f41f.png)

##### Javabean

![](../media/pictures/Mybatis.assets/37312c9c3f613393976da42d00ec722e.png)

##### Map

![](../media/pictures/Mybatis.assets/c0687928338743ade3cc064fe332d5e9.png)

##### 多个参数

![](../media/pictures/Mybatis.assets/f9826a60fa8f8a4028020ce77a40519c.png)

![](../media/pictures/Mybatis.assets/07e93ba9c8453a42468f50aeb3fbe90e.png)

##### List或数组（讲sql标签的时候补充这一部分）

##### 使用注解来输入映射

\@Param 出现在dao层，在方法对应的参数前

![](../media/pictures/Mybatis.assets/e94e381542ce6a5276cf2d2389bb664d.png)

在@param注解里面写了什么,在映射文件中就可以用什么

### 输出映射（重点）

#### 简单类型

#### Pojo类型

![](../media/pictures/Mybatis.assets/4e6420072e38bee4e048ade8b96570e2.png)

查询结果的列名,(而不是表的列名),和javabean的成员变量名对应

#### List或者数组

##### 基本类型 list或array

![](../media/pictures/Mybatis.assets/947889ffe1930948f992d26b0a0130c7.png)

##### Javabean list或array

![](../media/pictures/Mybatis.assets/eb9964748b317c1a37fce090209214c5.png)

#### resultMap

输出结果的封装

将查询结果的列名和javabean的成员变量名建立联系

![](../media/pictures/Mybatis.assets/f582a299b13c116d463544dd6b29ae19.png)

## Sql标签、动态sql

### Where标签

成对存在，两个标签之间放条件

![](../media/pictures/Mybatis.assets/7915434e572267fbdd05dfda7616849a.png)

Where标签可以帮我们去掉他紧跟着的第一个关系词

![](../media/pictures/Mybatis.assets/9431ff5ca80dbd4b2fde230f4778ec58.png)

### If

这个标签中,如果if不成立,前面的and,标签会自动维护,就会取消的!不会出现异常!

如果写在标签中,&gt;

```xml
&gt; 表示>
```

![](../media/pictures/Mybatis.assets/78cacd391dd0f8b386b0488b2ef714be.png)

### Trim

![](../media/pictures/Mybatis.assets/55a5a5ab34a1c18b13c2c18499f1dc05.png)

### Choose when

If和else

![](../media/pictures/Mybatis.assets/9d2ff7a2dd1f6a76a6257a943be86343.png)

### Sql

![](../media/pictures/Mybatis.assets/f3d6d4a17da3d855d8a8d438810b96c3.png)

### SelectKey

主要做的是赋值的操作   怎么找到返回来的这个id在?????

![](../media/pictures/Mybatis.assets/1d00ae35bb5bc44f2cd85a2750f73703.png)

上面是以前学习的时候的笔记



下面是代码中的具体使用 : 有效

1.mapper.xml层是这么写的没有问题

![image-20191216174837924](../media/pictures/Mybatis.assets/image-20191216174837924.png)

返回来怎么用呢?

返回结果是直接不是返回给id  而是将id 直接赋值给了goods里面的id,你在使用的时候直接从goods里面拿出来就可以啦.

![image-20191216174946253](../media/pictures/Mybatis.assets/image-20191216174946253.png)

测试结果:是有效的

![image-20191216175119454](../media/pictures/Mybatis.assets/image-20191216175119454.png)

### Foreach

![](../media/pictures/Mybatis.assets/6e0394a9103c568b3fb0075524f22d51.png)

![](../media/pictures/Mybatis.assets/4355909065f11967e9ab248791a726e9.png)

## 多表查询

### 一对一

#### Javabean关系维护

![](../media/pictures/Mybatis.assets/ae945170f5b70dfbf4968e6d57d39ba8.png)

#### 表关系维护

![](../media/pictures/Mybatis.assets/c253afc16472579d6a348f02beb754b7.png)

#### 执行查询

##### 分次查询

![](../media/pictures/Mybatis.assets/0ecfc7ec550d8290c38976c8db02f11c.png)

##### 连接查询

![](../media/pictures/Mybatis.assets/5b971953fb4207d0ad9f52848df12626.png)

### 一对多

#### Javabean关系维护

![](../media/pictures/Mybatis.assets/b6a4beebca70bf67b560101339a2f39d.png)

#### 表关系维护

![](../media/pictures/Mybatis.assets/4055431d51a31460cbf19c85e5905a9d.png)

#### 执行查询

##### 分次查询

![](../media/pictures/Mybatis.assets/907dd4724c1663afba2cf56003d7810d.png)

##### 连接查询

首先查出全部的数据，接着进行封装

![](../media/pictures/Mybatis.assets/9990ab27686737fc6052494f7957453d.png)

![](../media/pictures/Mybatis.assets/932464a4c36a81623230e35c8bc60fdb.png)

### 多对多

双向的一对多

#### Javabean关系维护

维护双向一对多，在各自bean中维护另一个bean的list

![](../media/pictures/Mybatis.assets/2175527c4b42f78cde1317a577cb28e5.png)

#### 表关系的维护

通过一张中间表维护互相之间的关系

![](../media/pictures/Mybatis.assets/72bd009f2bb1a62911d758c9484da5e2.png)

#### 执行查询

##### 分次查询

![](../media/pictures/Mybatis.assets/002e4fd10a98b6dcd662f4f31e0a217e.png)

![](../media/pictures/Mybatis.assets/637b5afe7e505750a622a17ed54c2701.png)

##### 连接查询

![](../media/pictures/Mybatis.assets/79f8d130e00c3f6823ff3919ae5f2b03.png)

![](../media/pictures/Mybatis.assets/2d7102052a7edb398b3155ee7fa0f647.png)

### 分次和连接查询的limit

一对多查询中

分次查询limit限制的是左一的数据数

连接查询limit限制的是右多的数据数

## Orm层的优化

### 懒加载

分次查询的使用场景、默认没有开启懒加载、需要手动开启

当setting中lazyLoadingEnabled= true时，分次查询已经开启了懒加载

![](../media/pictures/Mybatis.assets/e33537b9eed9d9df7fcaa6e766f20d18.png)

开启懒加载开关后，默认分次查询都为懒加载；如果需要开启立即加载，需要让association或collection标签的fetchType属性为eager

![](../media/pictures/Mybatis.assets/f95c474594476e754aff3a8baa86a31f.png)

### 缓存

#### 一级缓存

会话级别：同一个sqlSession

默认开启一级缓存

![](../media/pictures/Mybatis.assets/481fd8a8a889846fe082b60045554ce4.png)

![](../media/pictures/Mybatis.assets/e6223b2cf07ed10cb80c4ed2bd04f615.png)

#### 二级缓存

3件事情：

1. 开启缓存开关
2. Javabean增加序列化的接口
3. 映射文件中增加缓存标签

![](../media/pictures/Mybatis.assets/3d3dd59f92df2f503a91242f244843a1.png)

> sqlSession.commit时存入二级缓存

> 执行cud操作时，清空二级缓存

## Alias

在类上出现定义别名

![](../media/pictures/Mybatis.assets/df734a7802b24ace9ba7d5a20627ae9b.png)

## 使用注解替代部分标签

CRUD

## 逆向工程mybatis generator

数据库中的表→Bean mapper mapper.xml

![](../media/pictures/Mybatis.assets/9df4ce47122f68305de1ac5ad7acaccd.png)

要想用mybatis代码生成器生成对象  要导入这两个依赖

```xml
<dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
</dependency>

<dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.3.7</version>
</dependency>
```

需要注意的点为:逆向工程生成器,所指定的generatorConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <context id="testTables" targetRuntime="MyBatis3">
        <commentGenerator>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        <!--数据库连接的信息：驱动类、连接地址、用户名、密码 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://192.168.3.200:3306/litemalltest"
                        userId="root"
                        password="123456">
            <!--是否去除同名表  这句话一定要加上-->
            <property name="nullCatalogMeansCurrent" value="true"/>
        </jdbcConnection>
        <!--下面这些是oracle中用的配置-->
        <!--&lt;!&ndash;
            for oracle
           &ndash;&gt;
        <jdbcConnection driverClass="oracle.jdbc.OracleDriver"
            connectionURL="jdbc:oracle:thin:@127.0.0.1:1521:yycg"
            userId="yycg"
            password="yycg">
        </jdbcConnection>-->

        <!-- 默认false，
            为false把JDBC DECIMAL 和 NUMERIC 类型解析为Integer，
            为 true把JDBC DECIMAL 和 NUMERIC 类型解析为java.math.BigDecimal -->
        <!--<javaTypeResolver>
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>-->

        <!-- javaModelGenerator javaBean生成的配置信息
             targetProject:生成PO类的位置
             targetPackage：生成PO类的类名-->
        <javaModelGenerator targetPackage="com.cskaoyan.bean"
                            targetProject=".\src\main\java">
            <!-- enableSubPackages:是否允许子包,是否让schema作为包的后缀
                 即targetPackage.schemaName.tableName -->
            <property name="enableSubPackages" value="true"/>
            <!-- 从数据库返回的值是否清理前后的空格 -->
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <!--sqlMapGenerator Mapper映射文件的配置信息
            targetProject:mapper映射文件生成的位置
            targetPackage:生成mapper映射文件放在哪个包下-->
        <sqlMapGenerator targetPackage="com.cskaoyan.mapper"
                         targetProject=".\src\main\resources">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>

        <!--javaClientGenerator 生成 Model对象(JavaBean)和 mapper XML配置文件 对应的Dao代码
           targetProject:mapper接口生成的位置
           targetPackage:生成mapper接口放在哪个包下

           ANNOTATEDMAPPER
           XMLMAPPER
           MIXEDMAPPER
           -->

        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.cskaoyan.mapper"
                             targetProject=".\src\main\java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator><!---->
        <!-- 指定数据库表 -->
        <!-- 指定所有数据库表 -->
        <!--<table tableName="%"
               enableCountByExample="false"
               enableUpdateByExample="false"
               enableDeleteByExample="false"
               enableSelectByExample="false"
               enableInsert="false"
               enableDeleteByPrimaryKey="true"
               enableSelectByPrimaryKey="true"
               selectByExampleQueryId="false" ></table>-->

        <!-- 指定数据库表，要生成哪些表，就写哪些表，要和数据库中对应，不能写错！ -->
        
        //这下面 如果将ByExample全部false，那么生成的代码，就没有example 
        <!--<table tableName="j16_user_t"
               enableCountByExample="false"
               enableUpdateByExample="false"
               enableDeleteByExample="false"
               enableSelectByExample="false"
               enableInsert="true"
               enableDeleteByPrimaryKey="true"
               enableSelectByPrimaryKey="true"
               selectByExampleQueryId="false"
               domainObjectName="User"
        ></table>
        <table tableName="j16_student_t" domainObjectName="Studentz"/>-->
        <table tableName="litemall_goods" domainObjectName="LitemallGoods"/>
        <table tableName="litemall_admin_store" domainObjectName="LitemallAdminStore"/>

        <!--<table schema="" tableName="orders"></table>
             <table schema="" tableName="items"></table>
             <table schema="" tableName="orderdetail"></table>
      -->
        <!-- 有些表的字段需要指定java类型
         <table schema="" tableName="">
            <columnOverride column="" javaType="" />
        </table> -->
    </context>
</generatorConfiguration>

```

![1589373999929](../media/pictures/Mybatis.assets/1589373999929.png)

生成的bean对象是这个样子的，没有example。

![1589374030409](../media/pictures/Mybatis.assets/1589374030409.png)

只写了一个相对路径,这里要将下面的项目配置改成 如下: 

![image-20191214111544826](../media/pictures/Mybatis.assets/image-20191214111544826.png)

![1569397723500](../media/pictures/Mybatis.assets/1569397723500.png)

generatorConfig.xml放在module下面就可以,配置里面,要将工作路径设置成module目录下面!

然后运行Generator中的main方法就可以啦!就可以啦!



//下面这是Mybatis-plus 代码逆向工程:

```java
package com.cskaoyan;
import org.mybatis.generator.api.MyBatisGenerator;
import org.mybatis.generator.config.Configuration;
import org.mybatis.generator.config.xml.ConfigurationParser;
import org.mybatis.generator.internal.DefaultShellCallback;

import java.io.File;
import java.util.ArrayList;
import java.util.List;

public class Generator {
    public void generator() throws Exception{
        List<String> warnings = new ArrayList<String>();
        boolean overwrite = true; //指向逆向工程配置文件
        File configFile = new File("generatorConfig.xml");
        System.out.println(configFile.getAbsolutePath());
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator =
                new MyBatisGenerator(config, callback, warnings);

        myBatisGenerator.generate(null);
    }

    public static void main(String[] args) throws Exception {
        try {
            Generator generatorSqlmap = new Generator();
            generatorSqlmap.generator();
        }
        catch (Exception e) {
            e.printStackTrace();
        }

    }
}

```

在实际项目开发中,不要在当前项目中使用逆向工程,会报错.

在其他项目中用逆向工程生成代码,然后拷进来

拷进来要注意:

1.修改包名和导入的包名等:

![image-20191214114937228](../media/pictures/Mybatis.assets/image-20191214114937228.png)

2.正确的mapper是这样的:

![image-20191214115029500](../media/pictures/Mybatis.assets/image-20191214115029500.png)

如果下面买有画红线,说明正确.

如果错误:

需要修改mapper.xml中的命名空间:

![image-20191214115204849](../media/pictures/Mybatis.assets/image-20191214115204849.png)

这些地方都修改好,修改好的标志是:

![image-20191214115246938](../media/pictures/Mybatis.assets/image-20191214115246938.png)

对应mapper的接口不是红色 说明正确.



### 逆向工程中的example(牛皮)

![1569844094508](../media/pictures/Mybatis.assets/1569844094508.png)

### example实例解析

mybatis的逆向工程中会生成实例及实例对应的example，example用于添加条件，相当where后面的部分 
xxxExample example = new xxxExample(); 
Criteria criteria = new Example().createCriteria();

![1569844136788](../media/pictures/Mybatis.assets/1569844136788.png)

### 应用举例:

#### 1.查询

1.selectByPrimaryKey()

```java
User user = XxxMapper.selectByPrimaryKey(100); //相当于select * from user where id = 100
```

2.selectByExample() 和 selectByExampleWithBLOGs()

```java
UserExample example = new UserExample();
Criteria criteria = example.createCriteria();
criteria.andUsernameEqualTo("wyw");
criteria.andUsernameIsNull();
example.setOrderByClause("username asc,email desc");
List<?>list = XxxMapper.selectByExample(example);
//相当于：select * from user where username = 'wyw' and  username is null order by username asc,email desc
```

注：在iBator逆向工程生成的文件XxxExample.java中包含一个static的内部类Criteria，Criteria中的方法是定义SQL 语句where后的查询条件。

#### 2.插入数据

1.insert()

```java
User user = new User();
user.setId("dsfgsdfgdsfgds");
user.setUsername("admin");
user.setPassword("admin")
user.setEmail("wyw@163.com");
XxxMapper.insert(user);
//相当于：insert into user(ID,username,password,email) values ('dsfgsdfgdsfgds','admin','admin','wyw@126.com');
```

#### 3.更新数据

1.updateByPrimaryKey()

```java
User user =new User();
user.setId("dsfgsdfgdsfgds");
user.setUsername("wyw");
user.setPassword("wyw");
user.setEmail("wyw@163.com");
XxxMapper.updateByPrimaryKey(user);
//相当于：update user set username='wyw', password='wyw', email='wyw@163.com' where id='dsfgsdfgdsfgds'

```

2.updateByPrimaryKeySelective()

```java
User user = new User();
user.setId("dsfgsdfgdsfgds");
user.setPassword("wyw");
XxxMapper.updateByPrimaryKey(user);
//相当于：update user set password='wyw' where id='dsfgsdfgdsfgds'

```

3.updateByExample() 和 updateByExampleSelective()

```java
UserExample example = new UserExample();
Criteria criteria = example.createCriteria();
criteria.andUsernameEqualTo("admin");
User user = new User();
user.setPassword("wyw");
XxxMapper.updateByPrimaryKeySelective(user,example);
//相当于：update user set password='wyw' where username='admin'

```

updateByExample()更新所有的字段，包括字段为null的也更新，建议使用 updateByExampleSelective()更新想更新的字段

#### 4.删除数据

1.deleteByPrimaryKey()

```java
XxxMapper.deleteByPrimaryKey(1);  //相当于：delete from user where id=1

```

2.deleteByExample()

```java
UserExample example = new UserExample();
Criteria criteria = example.createCriteria();
criteria.andUsernameEqualTo("admin");
XxxMapper.deleteByExample(example);
//相当于：delete from user where username='admin'
```

#### 5.查询数据数量

1.countByExample()

```java
UserExample example = new UserExample();
Criteria criteria = example.createCriteria();
criteria.andUsernameEqualTo("wyw");
int count = XxxMapper.countByExample(example);
//相当于：select count(*) from user where username='wyw'
```



# Mybatis-plus 

idea中插件叫mybatisX

这个插件没有安装上!

大佬说,首先要让market加载出来,才可以搜索的!



舍友说,分页插件要和生成器生成的mapper , .xml等一起用!不能单独用!然后有时间试试!

**这个分页和逆向工程很像,wrapper后面的,下面代码不完整!然后看官方文档!**

```java
Page<SteveOrderInfo> page = new Page<>(steveOrder.getNewPage(),steveOrder.getPageSize());

 EntityWrapper<SteveOrderInfo> wrapper = new EntityWrapper<>();
wrapper.eq("user_id", userId);
  //根据当前登录人username获取订单信息
mapper.(page,wrapper)
```

## 分页 要试试使用!

## 条件构造器(EntityWrapper)：

以上基本的 CRUD 操作，我们仅仅需要继承一个 BaseMapper 即可实现大部分单表 CRUD 操作。BaseMapper 提供了多达 17 个方法供使用, 可以极其方便的实现单一、批量、分页等操作，极大的减少开发负担。但是mybatis-plus的强大不限于此，请看如下需求该如何处理：
**需求：**
我们需要分页查询 tb_employee 表中，年龄在 18~50 之间性别为男且姓名为 xx 的所有用户，这时候我们该如何实现上述需求呢？
**使用MyBatis :** 需要在 SQL 映射文件中编写带条件查询的 SQL,并用PageHelper 插件完成分页. 实现以上一个简单的需求，往往需要我们做很多重复单调的工作。
**使用MP:** 依旧不用编写 SQL 语句，MP 提供了功能强大的条件构造器 ------ EntityWrapper。

**接下来就直接看几个案例体会EntityWrapper的使用。**

**1、分页查询年龄在18 - 50且gender为0、姓名为tom的用户：**

```php
List<Employee> employees = emplopyeeDao.selectPage(new Page<Employee>(1,3),
     new EntityWrapper<Employee>()
        .between("age",18,50)
        .eq("gender",0)
        .eq("last_name","tom")
);
```

**注：**由此案例可知，分页查询和之前一样，new 一个page对象传入分页信息即可。至于分页条件，new 一个EntityWrapper对象，调用该对象的相关方法即可。between方法三个参数，分别是column、value1、value2，该方法表示column的值要在value1和value2之间；eq是equals的简写，该方法两个参数，column和value，表示column的值和value要相等。注意column是数据表对应的字段，而非实体类属性字段。

**2、查询gender为0且名字中带有老师、或者邮箱中带有a的用户：**

```php
List<Employee> employees = emplopyeeDao.selectList(
                new EntityWrapper<Employee>()
               .eq("gender",0)
               .like("last_name","老师")
                //.or()//和or new 区别不大
               .orNew()
               .like("email","a")
);
```

**注：**未说分页查询，所以用selectList即可，用EntityWrapper的like方法进行模糊查询，like方法就是指column的值包含value值，此处like方法就是查询last_name中包含“老师”字样的记录；“或者”用or或者orNew方法表示，这两个方法区别不大，用哪个都可以，可以通过控制台的sql语句自行感受其区别。

**3、查询gender为0，根据age排序，简单分页：**

```php
List<Employee> employees = emplopyeeDao.selectList(
                new EntityWrapper<Employee>()
                .eq("gender",0)
                .orderBy("age")//直接orderby 是升序，asc
                .last("desc limit 1,3")//在sql语句后面追加last里面的内容(改为降序，同时分页)
);
```

**注：**简单分页是指不用page对象进行分页。orderBy方法就是根据传入的column进行升序排序，若要降序，可以使用orderByDesc方法，也可以如案例中所示用last方法；last方法就是将last方法里面的value值追加到sql语句的后面，在该案例中，最后的sql语句就变为`select ······ order by desc limit 1, 3`，追加了`desc limit 1,3`所以可以进行降序排序和分页。

**4、分页查询年龄在18 - 50且gender为0、姓名为tom的用户：**
条件构造器除了EntityWrapper，还有Condition。用Condition来处理一下这个需求：

```php
 List<Employee> employees = emplopyeeDao.selectPage(
                new Page<Employee>(1,2),
                Condition.create()
                        .between("age",18,50)
                        .eq("gender","0")
 );
```

**注：**Condition和EntityWrapper的区别就是，创建条件构造器时，EntityWrapper是new出来的，而Condition是调create方法创建出来。

**5、根据条件更新：**

```java
@Test
public void testEntityWrapperUpdate(){
        Employee employee = new Employee();
        employee.setLastName("苍老师");
        employee.setEmail("cjk@sina.com");
        employee.setGender(0);
        emplopyeeDao.update(employee,
                new EntityWrapper<Employee>()
                .eq("last_name","tom")
                .eq("age",25)
        );
}
```

**注：**该案例表示把last_name为tom，age为25的所有用户的信息更新为employee中设置的信息。

**6、根据条件删除：**

```cpp
emplopyeeDao.delete(
        new EntityWrapper<Employee>()
        .eq("last_name","tom")
        .eq("age",16)
);

```

**注：**该案例表示把last_name为tom、age为16的所有用户删除。



总结：

以上便是mybatis-plus的入门教程，介绍了其如何与spring整合、通用crud的使用、全局策略的配置以及条件构造器的使用，但是这并不是MP的所有内容，其强大不限于此。避免篇幅过长，想了解mybatis-plus的更多用法请参考《[mybatis-plus的使用 ------ 进阶](https://www.jianshu.com/p/a4d5d310daf8)》，同时在文末会给出案例的源码。







# Mybatis-plus

## 分页:

```java
 //Mybatis-plus 自带分页查询
 IPage<ActivityEntity> ipage = new Page<>(dto.getPageNum(),dto.getPageSize());
 IPage<ActivityEntity> activityPage = activityService.page(ipage);
```



## CRUD接口:

### services CRUD 接口 

好多业务需求中,不需要mapper层东西,直接就可以在controller层用services层的CRUD接口.

项目中用到的 **查询分页**和**模糊查询**结合的情况:

```java
QueryWrapper<ActivityEntity> queryWrapper = new QueryWrapper<>();
queryWrapper.like(SqlConst.TITLE, dto.getTitle())
    .or().
    like(SqlConst.LANG, dto.getLang());
IPage<ActivityEntity> ipage = new Page<>(dto.getPageNum(),dto.getPageSize());
IPage<ActivityEntity> iPage = activityService.page(ipage,queryWrapper);
```

如果需求是返回一个list :

这个方法里面的多数都是返回有所有信息的对象所组成的list\

官方文档:

```java
// 查询所有
List<T> list();
// 查询列表
List<T> list(Wrapper<T> queryWrapper);   //上次用到这个 直接services层调用
// 查询（根据ID 批量查询）
Collection<T> listByIds(Collection<? extends Serializable> idList);
// 查询（根据 columnMap 条件）
Collection<T> listByMap(Map<String, Object> columnMap);
// 查询所有列表
List<Map<String, Object>> listMaps();
// 查询列表
List<Map<String, Object>> listMaps(Wrapper<T> queryWrapper);
// 查询全部记录
List<Object> listObjs();
// 查询全部记录
<V> List<V> listObjs(Function<? super Object, V> mapper);
// 根据 Wrapper 条件，查询全部记录
List<Object> listObjs(Wrapper<T> queryWrapper);
// 根据 Wrapper 条件，查询全部记录
<V> List<V> listObjs(Wrapper<T> queryWrapper, Function<? super Object, V> mapper);
```

##### 参数说明 

|                类型                |    参数名    |              描述               |
| :--------------------------------: | :----------: | :-----------------------------: |
|             Wrapper<T>             | queryWrapper | 实体对象封装操作类 QueryWrapper |
| Collection<? extends Serializable> |    idList    |           主键ID列表            |
|        Map<?String, Object>        |  columnMap   |         表字段 map 对象         |
|    Function<? super Object, V>     |    mapper    |            转换函数             |

需要注意的地方:

```java
QueryWrapper<DictionaryEntity> queryWrapper = new QueryWrapper<>();
            queryWrapper.eq("state", BizConst.BIZ_STATUS_VALID)
                    .eq("type", dto.getType());

            List<DictionaryEntity> dicts = dictService.list(queryWrapper);

```

如果这里想用service默认的方法,原来的service需要继承IService

```java
public interface IDictionaryService extends IService<DictionaryEntity>

```

泛型里面最好指定 Entity ,同时实现类 也要继承基础实现类:

```java
extends ServiceImpl<DictionaryMapper, DictionaryEntity>

```

完整的是这个样子的:

![image-20191213123658159](../media/pictures/Mybatis.assets/image-20191213123658159.png)

这里的Mapper也要继承基础的mapper:

```java
extends BaseMapper<DictionaryEntity>

```

这样的:

![image-20191213123801283](../media/pictures/Mybatis.assets/image-20191213123801283.png)

这些都有了以后,service层才可以用  为什么要mapper????  能不能不指定泛型中的bean 



### mapper CRUD 接口







## MybatisPlus中的注解:

@TableName：数据库表相关
 @TableId：表主键标识
 @TableField：表字段标识
 @TableLogic：表字段逻辑处理注解（逻辑删除）

@TableId(type= IdType.ID_WORKER_STR)

@TableField(exist = false)：表示该属性不为数据库表字段，但又是必须使用的。

@TableField(exist = true)：表示该属性为数据库表字段。

@TableField(condition = SqlCondition.LIKE)：表示该属性可以模糊搜索。

@TableField(fill = FieldFill.INSERT)：注解填充字段 ，生成器策略部分也可以配置！



项目中用到的注解是@TableField(exist = false),表示这个属性不为数据库表字段,但是又是必须使用的,所以这样写.



Mybatis-Plus的基本使用(用法来自于官方文档):

关于**ActiveRecord**的操作,这里搜索一段关于ActiveRecord的说明.

## ActiveRecord介绍

- 是一种领域模型模式
- 特点
  1、作用于模型层
  2、一个模型类对应关系型数据库中的一个表
  3、模型类的一个实例对应表中的一行数据
  4、封装了对数据库的访问，CURD操作
  5、ActiveRecord特别适合web快速开发
- ActiveRecord是JFinal最核心的组成部分之一
- 通过ActiveRecord来操作数据库，将极大地减少代码量，极大地提升开发效率



```java
@TableName("sys_user") // 注解指定表名
public class User extends Model<User> {

  ... // fields

  ... // getter and setter

  /** 指定主键 */
  @Override
  protected Serializable pkVal() {
      return this.id;
  }
}

```

## 条件构造器:

## [条件参数说明](https://baomidou.gitee.io/mybatis-plus-doc/#/wrapper?id=条件参数说明)

| 查询方式     | 说明                              |
| ------------ | --------------------------------- |
| setSqlSelect | 设置 SELECT 查询字段              |
| where        | WHERE 语句，拼接 + `WHERE 条件`   |
| and          | AND 语句，拼接 + `AND 字段=值`    |
| andNew       | AND 语句，拼接 + `AND (字段=值)`  |
| or           | OR 语句，拼接 + `OR 字段=值`      |
| orNew        | OR 语句，拼接 + `OR (字段=值)`    |
| eq           | 等于=                             |
| allEq        | 基于 map 内容等于=                |
| ne           | 不等于<>                          |
| gt           | 大于>                             |
| ge           | 大于等于>=                        |
| lt           | 小于<                             |
| le           | 小于等于<=                        |
| like         | 模糊查询 LIKE                     |
| notLike      | 模糊查询 NOT LIKE                 |
| in           | IN 查询                           |
| notIn        | NOT IN 查询                       |
| isNull       | NULL 值查询                       |
| isNotNull    | IS NOT NULL                       |
| groupBy      | 分组 GROUP BY                     |
| having       | HAVING 关键词                     |
| orderBy      | 排序 ORDER BY                     |
| orderAsc     | ASC 排序 ORDER BY                 |
| orderDesc    | DESC 排序 ORDER BY                |
| exists       | EXISTS 条件语句                   |
| notExists    | NOT EXISTS 条件语句               |
| between      | BETWEEN 条件语句                  |
| notBetween   | NOT BETWEEN 条件语句              |
| addFilter    | 自由拼接 SQL                      |
| last         | 拼接在最后，例如：last("LIMIT 1") |

TeamController 中 ,   用的是ne,意思是不等于. 



## Mybatis-plus的代码生成器:

使用之前要导入依赖:

```xml
<!--mybatis-plus 上面这个Mybatis-plus的依赖是包含下面的,如果只用生成器,用下面的即可-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus</artifactId>
    <version>2.1.9</version>
</dependency>

<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.3.0</version>
</dependency>

<!--生成器放在test中,就要导入这个-->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
<!--如果是spring 项目 还需要下面这个依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<!--如果单独测试 而不是在spring项目中,这里还要这个依赖-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-nop</artifactId>
    <version>1.7.24</version>
</dependency>
<!--mysql的依赖也需要-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.47</version>
</dependency>

```

代码生成器:

```java
package generator;

import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.InjectionConfig;
import com.baomidou.mybatisplus.generator.config.DataSourceConfig;
import com.baomidou.mybatisplus.generator.config.GlobalConfig;
import com.baomidou.mybatisplus.generator.config.PackageConfig;
import com.baomidou.mybatisplus.generator.config.StrategyConfig;
import com.baomidou.mybatisplus.generator.config.converts.MySqlTypeConvert;
import com.baomidou.mybatisplus.generator.config.rules.DbColumnType;
import com.baomidou.mybatisplus.generator.config.rules.DbType;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import org.junit.Test;

import java.util.HashMap;
import java.util.Map;

/**
 * 实体生成
 *
 * @author fengshuonan
 * @date 2017-08-23 12:15
 */
public class EntityGenerator {

    @Test
    public void entityGenerator() {
        AutoGenerator mpg = new AutoGenerator();

        // 全局配置
        GlobalConfig gc = new GlobalConfig();
        //gc.setOutputDir("/Users/ciggar/cskaoyan/csworkspace/guns/guns-promo/src/main/java");//这里写你自己的java目录
        gc.setOutputDir("D:\\Steve\\Code\\JavaEEWorkPlace\\mybatis4\\demo04_generator\\src\\main\\java");//这里写你自己的java目录
        gc.setFileOverride(true);//是否覆盖
        gc.setActiveRecord(true);
        gc.setEnableCache(false);// XML 二级缓存
        gc.setBaseResultMap(true);// XML ResultMap
        gc.setBaseColumnList(false);// XML columList
        gc.setAuthor("ciggar");
        mpg.setGlobalConfig(gc);

        // 数据源配置
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setDbType(DbType.MYSQL);
        dsc.setTypeConvert(new MySqlTypeConvert() {
            // 自定义数据库表字段类型转换【可选】
            @Override
            public DbColumnType processTypeConvert(String fieldType) {
                return super.processTypeConvert(fieldType);
            }
        });
        dsc.setDriverName("com.mysql.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("123456");
        dsc.setUrl("jdbc:mysql://192.168.3.200:3306/litemalltest?autoReconnect=true&useUnicode=true&useSSL=true&characterEncoding=utf8&serverTimezone=GMT%2B8");
        mpg.setDataSource(dsc);

        // 策略配置
        StrategyConfig strategy = new StrategyConfig();
        //strategy.setTablePrefix(new String[]{"_"});// 此处可以修改为您的表前缀
        strategy.setNaming(NamingStrategy.underline_to_camel);// 表名生成策略
        //下面这里是表名  放到数组里面即可
        strategy.setInclude(new String[]{"litemall_store","litemall_user_wallet"});
        mpg.setStrategy(strategy);

        // 包配置
        PackageConfig pc = new PackageConfig();
        pc.setParent(null);
        pc.setEntity("com.cskaoyan.entity");
        pc.setMapper("com.cskaoyan.dao");
        pc.setXml("com.cskaoyan.mapping");
        pc.setService("com.cskaoyan.service");       
        pc.setServiceImpl("com.cskaoyan.service.iml");   
        pc.setController("TTT");    //TTT 一般不生成controller层  生成以后 删除
        mpg.setPackageInfo(pc);

        // 注入自定义配置，可以在 VM 中使用 cfg.abc 设置的值
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                Map<String, Object> map = new HashMap<>();
                map.put("abc", this.getConfig().getGlobalConfig().getAuthor() + "-mp");
                this.setMap(map);
            }
        };
        mpg.setCfg(cfg);

        // 执行生成
        mpg.execute();

        // 打印注入设置
        System.err.println(mpg.getCfg().getMap().get("abc"));
    }
}

```

## 模糊查询

```xml
<resultMap id="BaseResultMapAddSales" type="org.linlinjava.litemall.db.domain.LitemallGoodsForSales">
    <id column="id" jdbcType="INTEGER" property="id" />
    <result column="goods_sn" jdbcType="VARCHAR" property="goodsSn" />
    <result column="store_id" jdbcType="INTEGER" property="storeId" />
    <result column="name" jdbcType="VARCHAR" property="name" />
    <result column="category_id" jdbcType="INTEGER" property="categoryId" />
    <result column="brand_id" jdbcType="INTEGER" property="brandId" />
    <result column="gallery" jdbcType="VARCHAR" property="gallery" typeHandler="org.linlinjava.litemall.db.mybatis.JsonStringArrayTypeHandler" />
    <result column="keywords" jdbcType="VARCHAR" property="keywords" />
    <result column="brief" jdbcType="VARCHAR" property="brief" />
    <result column="is_on_sale" jdbcType="BIT" property="isOnSale" />
    <result column="sort_order" jdbcType="SMALLINT" property="sortOrder" />
    <result column="pic_url" jdbcType="VARCHAR" property="picUrl" />
    <result column="share_url" jdbcType="VARCHAR" property="shareUrl" />
    <result column="is_new" jdbcType="BIT" property="isNew" />
    <result column="is_hot" jdbcType="BIT" property="isHot" />
    <result column="unit" jdbcType="VARCHAR" property="unit" />
    <result column="counter_price" jdbcType="DECIMAL" property="counterPrice" />
    <result column="retail_price" jdbcType="DECIMAL" property="retailPrice" />
    <result column="add_time" jdbcType="TIMESTAMP" property="addTime" />
    <result column="update_time" jdbcType="TIMESTAMP" property="updateTime" />
    <result column="deleted" jdbcType="BIT" property="deleted" />
    <result column="sales" jdbcType="INTEGER" property="sales" />
  </resultMap>
  <select id="saleList"  resultMap="BaseResultMapAddSales">
    SELECT lg.id, lg.goods_sn,lg.store_id,lg.name,lg.category_id,lg.brand_id,lg.gallery,lg.keywords,
			lg.brief,lg.is_on_sale,lg.pic_url,lg.share_url,lg.is_new,lg.is_hot,lg.unit,
			lg.counter_price,lg.retail_price,lg.detail,lg.add_time,lg.update_time,lg.deleted ,lgs.number as sales
        FROM litemall_goods lg
        INNER JOIN litemall_goods_sales lgs ON lg.id = lgs.good_id 
        WHERE lg.deleted = 0 and name like #{name} ORDER BY #{sort} limit #{limit} offset #{page}
  </select>

```



传参数的时候要注意:  name这里拼接一个 page -1  这里要注意

![image-20191225105630546](../media/pictures/Mybatis.assets/image-20191225105630546.png)



## github上面一个开源的mybatis逆向工程生成器

https://github.com/zouzg/mybatis-generator-gui

这是地址 ，测试发现，真好用，真香。

![1591604313901](../media/pictures/Mybatis.assets/1591604313901.png)







# Interviewe

## 1.Mybatis有什么关键作用？

MyBatis是持久层框架，支持JDBC，简化了持久层开发。

使用MyBatis时，只需要通过接口指定数据操作的抽象方法，然后配置与之关联的SQL语句，即可。



持久化存储：在程序运行过程中，数据都是在内存（RAM，即内存条）中的，内存中的数据不是永久存储的，例如程序可以对这些数据进行销毁，或者由于断电也会导致内存中所有数据丢失！而把数据存储到硬盘中的某个文件中，会使得这些数据永久的存储下来，常见做法是存储到数据库中，当然，也可以使用其他技术把数据存储到文本文件、XML文件等其他文件中。



## 2.Mybatis和Herbanate的区别

MyBatis（前身是iBatis）是一个支持普通SQL查询、存储过程以及高级映射的持久层框架，它消除了几乎所有的JDBC代码和参数的手动设置以及对结果集的检索，并使用简单的XML或注解进行配置和原始映射，用以将接口和Java的POJO（Plain Old Java Object，普通Java对象）映射成数据库中的记录，使得Java开发人员可以使用面向对象的编程思想来操作数据库。

MyBatis 框架也被称之为 ORM（Object/Relational Mapping，即对象关系映射）框架。所谓的 ORM 就是一种为了解决面向对象与关系型数据库中数据类型不匹配的技术，它通过描述Java对象与数据库表之间的映射关系，自动将Java应用程序中的对象持久化到关系型数据库的表中。ORM框架的工作原理如下图所示。

![1588578489385_数据库、.png](../media/pictures/Mybatis.assets/1588578489385_%E6%95%B0%E6%8D%AE%E5%BA%93%E3%80%81.png)


从上图可以看出，使用ORM框架后，应用程序不再直接访问底层数据库，而是以面向对象的方式来操作持久化对象（Persisent Object，PO），而ORM框架则会通过映射关系将这些面向对象的操作转换成底层的SQL操作。

当前的ORM框架产品有很多，常见的ORM框架有Hibernate和MyBatis。这两个框架的主要区别如下。

- **Hibernate：**是一个全表映射的框架。通常开发者只需定义好持久化对象到数据库表的映射关系，就可以通过 Hibernate 提供的方法完成持久层操作。开发者并不需要熟练地掌握 SQL语句的编写，Hibernate会根据制定的存储逻辑，自动的生成对应的SQL，并调用JDBC接口来执行，所以其开发效率会高于MyBatis。然而Hibernate自身也存在着一些缺点，例如它在多表关联时，对 SQL 查询的支持较差；更新数据时，需要发送所有字段；不支持存储过程；不能通过优化 SQL 来优化性能等。这些问题导致其只适合在场景不太复杂且对性能要求不高的项目中使用。

- **MyBatis：**是一个半自动映射的框架。这里所谓的“半自动”是相对于Hibernate全表映射而言的，MyBatis 需要手动匹配提供 POJO、SQL和映射关系，而Hibernate只需提供POJO 和映射关系即可。与Hibernate相比，虽然使用MyBatis手动编写 SQL 要比使用Hibernate的工作量大，但MyBatis可以配置动态SQL并优化SQL，可以通过配置决定SQL的映射规则，它还支持存储过程等。对于一些复杂的和需要优化性能的项目来说，显然使用MyBatis更加合适。推荐了解传智播客[java](http://java.itcast.cn/)中级程序员课程。



## 3.Mybatis如何实现批量插入？ 

**之所以要批量插入是为了避免数据库和程序之间建立多次连接，增加服务器的负荷。**



### 正常的插入多条数据是这样子的:

```sql
insert into user 
		(username, `password`, age, gender)
VALUES 
		('杨豆豆','123456',11,'male'), 
		('杨豆豆1','123456',11,'male')
```



### 动态sql foreach:

```xml
<insert id="insertBatch" >
    insert into user 
    	( <include refid="Base_Column_List" /> ) 
    values 
        <foreach collection="list" item="item" index="index" separator=",">    <!--用逗号分开-->
            (#{item.username},#{item.password},#{item.age},#{item.gender})
        </foreach>
</insert>
```

参考：https://www.jianshu.com/p/97e484b55d04



## 4.Mybatis里插入一条数据，如何返回这条记录的主键?

统一都用selectKey, 只是在Oracle里面用的话，这里是返回序列号

Oracle：

```xml
<selectKey keyProperty="id" resultType="Long" order="BEFORE"> 
    SELECT SEQUENCE_1.nextval AS id FROM dual
</selectKey>
```

Mysql：

```xml
<selectKey keyProperty="id" resultType="java.lang.Integer" order="AFTER">
    SELECT LAST_INSERT_ID()
</selectKey>
```



## 5.Mybatis 十万条 数据 五万条数据更新,五万条新增 怎么去实(DUPLICATE)

在 mysql 中，当存在主键冲突或唯一键冲突的情况下，根据插入策略不同，一般有以下三种避免方法：

- `insert ignore into`：若没有则插入，若存在则忽略

- `replace into`：若没有则正常插入，若存在则先删除后插入

- `insert into ... on duplicate key update`：若没有则正常插入，若存在则更新

```sql
#数据库原来2中有数据，4中没有数据。
insert into ljtest
	(id, emp_no, title, from_date) 
values 
 	(2, 20002, 'insert test 2', '0000-00-00'), 
	(4, 40004, 'insert test 4', '0000-00-00') 
on duplicate key update emp_no=values(emp_no), title=values(title); 

#结果
Query OK, 3 rows affected (0.00 sec)
Records: 2  Duplicates: 1  Warnings: 0
```



参考：https://blog.csdn.net/lijing742180/article/details/90243470?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.channel_param



## 6.Mybatis #{} 和 ${}的区别是什么?  具体#的底层是怎么样做的？为什么#{}可以防止SQL注入？Mybatis如何防注入的? PrepareStatement底层原理是怎么样的，预编译做了什么？

- `${}`是 Properties 文件中的变量占位符，它可以用于标签属性值和 sql 内部，属于静态文本替换，比如\${driver}会被静态替换为`com.mysql.jdbc.Driver`。
- `#{}`是 sql 的参数占位符，Mybatis 会将 sql 中的`#{}`替换为?号，在 sql 执行前会使用 PreparedStatement 的参数设置方法，按序给 sql 的?号占位符设置参数值，比如 ps.setInt(0, parameterValue)，`#{item.name}` 的取值方式为使用反射从参数对象中获取 item 对象的 name 属性值，相当于 `param.getItem().getName()`。



参考：JavaGuide和下面这篇文章 写的很好。

https://blog.csdn.net/yizhenn/article/details/52384601



## 7.Mybatis里Left join inner join有什么区别？他们使用之后分别是什么结果？

参考Mysql。



## 8.Mybatis中，xml和mapper接口是如何映射的?



你知道如何批量插入吗？用Mybatis-plus

Mybatis的缓存原理？

mybatis用的反射(关于mybatis的细节)

介绍逆向工程：Generator和Mybatis-Plus

项目mybatis怎么使用？用mapper映射文件还是注解?

mybatis的xml文件中有什么标签?

mybatis 逆向工程，mybatis 的批量操作，比如大批量数据更新(这个问题卡住了)：在应用里面用 mybatis 实现大批量数据(上万条)的更新，常规做法：Java 的 for 循环，慢，然后他说你可以去了解一下。



mybatis的实现过程?怎么配置的?

mybait-start，了解过他什么时候生效的吗?

JPA了解吗？知道Hibernate吗？为何选择Mybatis，它和Hibernate的区别是什么



## 9.Mybatis的常用标签

Mybatis的常用标签有哪些，如果DAO层的集合对象和<#foreache>标签名字不一样怎么解决的？(当时一时半会儿想不起来，面试官已经提示用@Param注解了，然而我听成了@Parable……)





10.Hibernate用过吗，JDBCTemplate用过吗？介绍一下里面具体的方法

11.where和where标签的区别

12.${}和#{}有什么区别





> 参考自：https://github.com/Snailclimb/JavaGuide.git

## JavaGuide

### 1、#{}和\${}的区别是什么？

注：这道题是面试官面试我同事的。

答：

- `${}`是 Properties 文件中的变量占位符，它可以用于标签属性值和 sql 内部，属于静态文本替换，比如\${driver}会被静态替换为`com.mysql.jdbc.Driver`。
- `#{}`是 sql 的参数占位符，Mybatis 会将 sql 中的`#{}`替换为?号，在 sql 执行前会使用 PreparedStatement 的参数设置方法，按序给 sql 的?号占位符设置参数值，比如 ps.setInt(0, parameterValue)，`#{item.name}` 的取值方式为使用反射从参数对象中获取 item 对象的 name 属性值，相当于 `param.getItem().getName()`。

### 2、Xml 映射文件中，除了常见的 select|insert|updae|delete 标签之外，还有哪些标签？

注：这道题是京东面试官面试我时问的。

答：还有很多其他的标签，`<resultMap>`、`<parameterMap>`、`<sql>`、`<include>`、`<selectKey>`，加上动态 sql 的 9 个标签，`trim|where|set|foreach|if|choose|when|otherwise|bind`等，其中<sql>为 sql 片段标签，通过`<include>`标签引入 sql 片段，`<selectKey>`为不支持自增的主键生成策略标签。

### 3、最佳实践中，通常一个 Xml 映射文件，都会写一个 Dao 接口与之对应，请问，这个 Dao 接口的工作原理是什么？Dao 接口里的方法，参数不同时，方法能重载吗？

注：这道题也是京东面试官面试我时问的。

答：Dao 接口，就是人们常说的 `Mapper`接口，接口的全限名，就是映射文件中的 namespace 的值，接口的方法名，就是映射文件中`MappedStatement`的 id 值，接口方法内的参数，就是传递给 sql 的参数。`Mapper`接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为 key 值，可唯一定位一个`MappedStatement`，举例：`com.mybatis3.mappers.StudentDao.findStudentById`，可以唯一找到 namespace 为`com.mybatis3.mappers.StudentDao`下面`id = findStudentById`的`MappedStatement`。在 Mybatis 中，每一个`<select>`、`<insert>`、`<update>`、`<delete>`标签，都会被解析为一个`MappedStatement`对象。

Dao 接口里的方法，是不能重载的，因为是全限名+方法名的保存和寻找策略。

Dao 接口的工作原理是 JDK 动态代理，Mybatis 运行时会使用 JDK 动态代理为 Dao 接口生成代理 proxy 对象，代理对象 proxy 会拦截接口方法，转而执行`MappedStatement`所代表的 sql，然后将 sql 执行结果返回。

### 4、Mybatis 是如何进行分页的？分页插件的原理是什么？

注：我出的。

答：Mybatis 使用 RowBounds 对象进行分页，它是针对 ResultSet 结果集执行的内存分页，而非物理分页，可以在 sql 内直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成物理分页。

分页插件的基本原理是使用 Mybatis 提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的 sql，然后重写 sql，根据 dialect 方言，添加对应的物理分页语句和物理分页参数。

举例：`select _ from student`，拦截 sql 后重写为：`select t._ from （select \* from student）t limit 0，10`

### 5、简述 Mybatis 的插件运行原理，以及如何编写一个插件。

注：我出的。

答：Mybatis 仅可以编写针对 `ParameterHandler`、`ResultSetHandler`、`StatementHandler`、`Executor` 这 4 种接口的插件，Mybatis 使用 JDK 的动态代理，为需要拦截的接口生成代理对象以实现接口方法拦截功能，每当执行这 4 种接口对象的方法时，就会进入拦截方法，具体就是 `InvocationHandler` 的 `invoke()`方法，当然，只会拦截那些你指定需要拦截的方法。

实现 Mybatis 的 Interceptor 接口并复写` intercept()`方法，然后在给插件编写注解，指定要拦截哪一个接口的哪些方法即可，记住，别忘了在配置文件中配置你编写的插件。

### 6、Mybatis 执行批量插入，能返回数据库主键列表吗？

注：我出的。

答：能，JDBC 都能，Mybatis 当然也能。

### 7、Mybatis 动态 sql 是做什么的？都有哪些动态 sql？能简述一下动态 sql 的执行原理不？

注：我出的。

答：Mybatis 动态 sql 可以让我们在 Xml 映射文件内，以标签的形式编写动态 sql，完成逻辑判断和动态拼接 sql 的功能，Mybatis 提供了 9 种动态 sql 标签 `trim|where|set|foreach|if|choose|when|otherwise|bind`。

其执行原理为，使用 OGNL 从 sql 参数对象中计算表达式的值，根据表达式的值动态拼接 sql，以此来完成动态 sql 的功能。

### 8、Mybatis 是如何将 sql 执行结果封装为目标对象并返回的？都有哪些映射形式？

注：我出的。

答：第一种是使用`<resultMap>`标签，逐一定义列名和对象属性名之间的映射关系。第二种是使用 sql 列的别名功能，将列别名书写为对象属性名，比如 T_NAME AS NAME，对象属性名一般是 name，小写，但是列名不区分大小写，Mybatis 会忽略列名大小写，智能找到与之对应对象属性名，你甚至可以写成 T_NAME AS NaMe，Mybatis 一样可以正常工作。

有了列名与属性名的映射关系后，Mybatis 通过反射创建对象，同时使用反射给对象的属性逐一赋值并返回，那些找不到映射关系的属性，是无法完成赋值的。

### 9、Mybatis 能执行一对一、一对多的关联查询吗？都有哪些实现方式，以及它们之间的区别。

注：我出的。

答：能，Mybatis 不仅可以执行一对一、一对多的关联查询，还可以执行多对一，多对多的关联查询，多对一查询，其实就是一对一查询，只需要把 `selectOne()`修改为 `selectList()`即可；多对多查询，其实就是一对多查询，只需要把 `selectOne()`修改为 `selectList()`即可。

关联对象查询，有两种实现方式，一种是单独发送一个 sql 去查询关联对象，赋给主对象，然后返回主对象。另一种是使用嵌套查询，嵌套查询的含义为使用 join 查询，一部分列是 A 对象的属性值，另外一部分列是关联对象 B 的属性值，好处是只发一个 sql 查询，就可以把主对象和其关联对象查出来。

那么问题来了，join 查询出来 100 条记录，如何确定主对象是 5 个，而不是 100 个？其去重复的原理是`<resultMap>`标签内的`<id>`子标签，指定了唯一确定一条记录的 id 列，Mybatis 根据<id>列值来完成 100 条记录的去重复功能，`<id>`可以有多个，代表了联合主键的语意。

同样主对象的关联对象，也是根据这个原理去重复的，尽管一般情况下，只有主对象会有重复记录，关联对象一般不会重复。

举例：下面 join 查询出来 6 条记录，一、二列是 Teacher 对象列，第三列为 Student 对象列，Mybatis 去重复处理后，结果为 1 个老师 6 个学生，而不是 6 个老师 6 个学生。

 t_id t_name s_id

| 1 | teacher | 38 |
| 1 | teacher | 39 |
| 1 | teacher | 40 |
| 1 | teacher | 41 |
| 1 | teacher | 42 |
| 1 | teacher | 43 |

### 10、Mybatis 是否支持延迟加载？如果支持，它的实现原理是什么？

注：我出的。

答：Mybatis 仅支持 association 关联对象和 collection 关联集合对象的延迟加载，association 指的就是一对一，collection 指的就是一对多查询。在 Mybatis 配置文件中，可以配置是否启用延迟加载 `lazyLoadingEnabled=true|false。`

它的原理是，使用` CGLIB` 创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，比如调用 `a.getB().getName()`，拦截器 `invoke()`方法发现 `a.getB()`是 null 值，那么就会单独发送事先保存好的查询关联 B 对象的 sql，把 B 查询上来，然后调用 a.setB(b)，于是 a 的对象 b 属性就有值了，接着完成 `a.getB().getName()`方法的调用。这就是延迟加载的基本原理。

当然了，不光是 Mybatis，几乎所有的包括 Hibernate，支持延迟加载的原理都是一样的。

### 11、Mybatis 的 Xml 映射文件中，不同的 Xml 映射文件，id 是否可以重复？

注：我出的。

答：不同的 Xml 映射文件，如果配置了 namespace，那么 id 可以重复；如果没有配置 namespace，那么 id 不能重复；毕竟 namespace 不是必须的，只是最佳实践而已。

原因就是 namespace+id 是作为 `Map<String, MappedStatement>`的 key 使用的，如果没有 namespace，就剩下 id，那么，id 重复会导致数据互相覆盖。有了 namespace，自然 id 就可以重复，namespace 不同，namespace+id 自然也就不同。

### 12、Mybatis 中如何执行批处理？

注：我出的。

答：使用 BatchExecutor 完成批处理。

### 13、Mybatis 都有哪些 Executor 执行器？它们之间的区别是什么？

注：我出的

答：Mybatis 有三种基本的 Executor 执行器，**`SimpleExecutor`、`ReuseExecutor`、`BatchExecutor`。**

**`SimpleExecutor`：**每执行一次 update 或 select，就开启一个 Statement 对象，用完立刻关闭 Statement 对象。

**``ReuseExecutor`：**执行 update 或 select，以 sql 作为 key 查找 Statement 对象，存在就使用，不存在就创建，用完后，不关闭 Statement 对象，而是放置于 Map<String, Statement>内，供下一次使用。简言之，就是重复使用 Statement 对象。

**`BatchExecutor`：**执行 update（没有 select，JDBC 批处理不支持 select），将所有 sql 都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个 Statement 对象，每个 Statement 对象都是 addBatch()完毕后，等待逐一执行 executeBatch()批处理。与 JDBC 批处理相同。

作用范围：Executor 的这些特点，都严格限制在 SqlSession 生命周期范围内。

### 14、Mybatis 中如何指定使用哪一种 Executor 执行器？

注：我出的

答：在 Mybatis 配置文件中，可以指定默认的 ExecutorType 执行器类型，也可以手动给 `DefaultSqlSessionFactory` 的创建 SqlSession 的方法传递 ExecutorType 类型参数。

### 15、Mybatis 是否可以映射 Enum 枚举类？

注：我出的

答：Mybatis 可以映射枚举类，不单可以映射枚举类，Mybatis 可以映射任何对象到表的一列上。映射方式为自定义一个 `TypeHandler`，实现 `TypeHandler` 的 `setParameter()`和 `getResult()`接口方法。`TypeHandler` 有两个作用，一是完成从 javaType 至 jdbcType 的转换，二是完成 jdbcType 至 javaType 的转换，体现为 `setParameter()`和 `getResult()`两个方法，分别代表设置 sql 问号占位符参数和获取列查询结果。

### 16、Mybatis 映射文件中，如果 A 标签通过 include 引用了 B 标签的内容，请问，B 标签能否定义在 A 标签的后面，还是说必须定义在 A 标签的前面？

注：我出的

答：虽然 Mybatis 解析 Xml 映射文件是按照顺序解析的，但是，被引用的 B 标签依然可以定义在任何地方，Mybatis 都可以正确识别。

原理是，Mybatis 解析 A 标签，发现 A 标签引用了 B 标签，但是 B 标签尚未解析到，尚不存在，此时，Mybatis 会将 A 标签标记为未解析状态，然后继续解析余下的标签，包含 B 标签，待所有标签解析完毕，Mybatis 会重新解析那些被标记为未解析的标签，此时再解析 A 标签时，B 标签已经存在，A 标签也就可以正常解析完成了。

### 17、简述 Mybatis 的 Xml 映射文件和 Mybatis 内部数据结构之间的映射关系？

注：我出的

答：Mybatis 将所有 Xml 配置信息都封装到 All-In-One 重量级对象 Configuration 内部。在 Xml 映射文件中，`<parameterMap>`标签会被解析为 `ParameterMap` 对象，其每个子元素会被解析为 ParameterMapping 对象。`<resultMap>`标签会被解析为 `ResultMap` 对象，其每个子元素会被解析为 `ResultMapping` 对象。每一个`<select>、<insert>、<update>、<delete>`标签均会被解析为 `MappedStatement` 对象，标签内的 sql 会被解析为 BoundSql 对象。

### 18、为什么说 Mybatis 是半自动 ORM 映射工具？它与全自动的区别在哪里？

注：我出的

答：Hibernate 属于全自动 ORM 映射工具，使用 Hibernate 查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以它是全自动的。而 Mybatis 在查询关联对象或关联集合对象时，需要手动编写 sql 来完成，所以，称之为半自动 ORM 映射工具。

面试题看似都很简单，但是想要能正确回答上来，必定是研究过源码且深入的人，而不是仅会使用的人或者用的很熟的人，以上所有面试题及其答案所涉及的内容，在我的 Mybatis 系列博客中都有详细讲解和原理分析。







## 附加:一文了解JPA、Hibernate、Spring Data JPA之间的爱恨情仇

**前言**

我们都知道Java 持久层框架访问数据库的方式大致分为两种。一种以 SQL 核心，封装一定程度的 JDBC 操作，比如： MyBatis。另一种是以 Java 实体类为核心，将实体类的和数据库表之间建立映射关系，也就是我们说的ORM框架，如：Hibernate、Spring Data JPA。今天咱们就先来了解一下什么是Spring Data JPA?

**JPA是啥**

在开始学习Spring Data JPA之前我们首先还是要先了解下什么是JPA，因为Spring Data JPA是建立的JPA的基础之上的，那到底什么是JPA呢？

我们都知道不同的数据库厂商都有自己的实现类，后来统一规范也就有了数据库驱动，Java在操作数据库的时候，底层使用的其实是JDBC，而JDBC是一组操作不同数据库的规范。我们的Java应用程序，只需要调用JDBC提供的API就可以访问数据库了，而JPA也是类似的道理。

![img](../media/pictures/Mybatis.assets/37d3d539b6003af38427311c555ea05a1138b6ac.jpeg)

JPA全称为Java Persistence API（Java持久层API），它是Sun公司在JavaEE 5中提出的Java持久化规范。它为Java开发人员提供了一种对象/关联映射工具，来管理Java应用中的关系数据，JPA吸取了目前Java持久化技术的优点，旨在规范、简化Java对象的持久化工作。很多ORM框架都是实现了JPA的规范，如：Hibernate、EclipseLink。

需要注意的是JPA统一了Java应用程序访问ORM框架的规范

**JPA为我们提供了以下规范：**

- ORM映射元数据：JPA支持XML和注解两种元数据的形式，元数据描述对象和表之间的映射关系，框架据此将实体对象持久化到数据库表中
- JPA 的API：用来操作实体对象，执行CRUD操作，框架在后台替我们完成所有的事情，开发人员不用再写SQL了
- JPQL查询语言：通过面向对象而非面向数据库的查询语言查询数据，避免程序的SQL语句紧密耦合。

**Hibernate是啥**

Hibernate是Java中的对象关系映射解决方案。对象关系映射或ORM框架是将应用程序数据模型对象映射到关系数据库表的技术。Hibernate 不仅关注于从 Java 类到数据库表的映射，也有 Java 数据类型到 SQL 数据类型的映射。

![img](../media/pictures/Mybatis.assets/b151f8198618367a5e108e327107edd2b21ce53b.jpeg)

**Hibernate 和 JPA是什么关系呢**

上面我们介绍到JPA是Java EE 5规范中提出的Java持久化接口，而Hibernate是一个ORM框架

JPA和Hibernate的关系：

JPA是一个规范，而不是框架

Hibernate是JPA的一种实现，是一个框架

**Spring Data是啥**

Spring Data是Spring 社区的一个子项目，主要用于简化数据（关系型&非关系型）访问，其主要目标是使得数据库的访问变得方便快捷。

它提供很多模板操作

– Spring Data Elasticsearch

– Spring Data MongoDB

– Spring Data Redis

– Spring Data Solr

强大的 Repository 和定制的数据储存对象的抽象映射

对数据访问对象的支持

**Spring Data JPA又是啥**

Spring Data JPA是在实现了JPA规范的基础上封装的一套 JPA 应用框架，虽然ORM框架都实现了JPA规范，但是在不同的ORM框架之间切换仍然需要编写不同的代码，而使用Spring Data JPA能够方便大家在不同的ORM框架之间进行切换而不需要更改代码。Spring Data JPA旨在通过将统一ORM框架的访问持久层的操作，来提高开发人的效率。

![img](../media/pictures/Mybatis.assets/6d81800a19d8bc3e953df07fe2ffc018a9d345cc.jpeg)

**Spring Data JPA给我们提供的主要的类和接口**

Repository 接口：

Repository

CrudRepository

JpaRepository

Repository 实现类：

SimpleJpaRepository

QueryDslJpaRepository

以上这些类和接口就是我们以后在使用Spring Data JPA的时候需要掌握的。

**Spring Data JPA和Hibernate的关系**

Hibernate其实是JPA的一种实现，而Spring Data JPA是一个JPA数据访问抽象。也就是说Spring Data JPA不是一个实现或JPA提供的程序，它只是一个抽象层，主要用于减少为各种持久层存储实现数据访问层所需的样板代码量。但是它还是需要JPA提供实现程序，其实Spring Data JPA底层就是使用的 Hibernate实现。

**小结：**

Hibernate是JPA的一种实现，是一个框架

Spring Data JPA是一种JPA的抽象层，底层依赖Hibernate

**总结：**

这里主要给介绍了JPA、Hibernate、以及Spring Data JPA的概念以及三者的关系，让我们对这些常用的持久层规范和框架有一个清晰的认识。这样以后我们再接触到其他的同类ORM框架或者其他持久层框架的时候就能更加的游刃有余。



·