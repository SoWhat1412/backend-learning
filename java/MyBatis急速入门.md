>高清思维导图已同步Git：https://github.com/SoWhat1412/xmindfile，关注公众号sowhat1412获取海量资源
>
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803170719825.png#pic_center)


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200731094443887.png#pic_center)
# 前情概要
**环境说明**：
- jdk 8 +
- MySQL 5.5
- maven-3.6.1
- IDEA

**学习前需要掌握**：
- JDBC
- MySQL
- Java 基础
- Maven
- Junit

### 什么是MyBatis
- MyBatis 是一款优秀的`持久层`框架
- MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集的过程

- MyBatis 可以使用简单的` XML` 或`注解`来配置和映射原生信息，将接口和 Java 的实体类(Plain Old Java Objects)映射成数据库中的记录。

- MyBatis 本是apache的一个开源项目`ibatis`, 2010年这个项目由apache 迁移到了google code，并且改名为`MyBatis` 。2013年11月迁移到Github .

- Mybatis官方文档 : [http://www.mybatis.org/mybatis-3/zh/index.html](http://www.mybatis.org/mybatis-3/zh/index.html)

- GitHub : [https://github.com/mybatis/mybatis-3](https://github.com/mybatis/mybatis-3)

- 学完MyBatis后可以学 [MybatisPlus](https://sowhat.blog.csdn.net/article/details/108304599)
### 持久化

**持久化是将程序数据在持久状态和瞬时状态间转换的机制。**
即把数据（如内存中的对象）保存到可永久保存的存储设备中（如磁盘）。持久化的主要应用是将内存中的对象存储在数据库中，或者存储在磁盘文件中、XML数据文件中等等。**JDBC**就是一种持久化机制。**文件IO**也是一种持久化机制。

在生活中 : 将鲜肉冷藏，吃的时候再解冻的方法也是。将水果做成罐头的方法也是。

**为什么需要持久化服务呢？**
- 内存断电后数据会丢失，但有一些对象是无论如何都不能丢失的，比如银行账号等，遗憾的是，人们还无法保证内存永不掉电。

- 内存过于昂贵，与硬盘、光盘等外存相比，内存的价格要高2~3个数量级，而且维持成本也高，至少需要一直供电吧。所以即使对象不需要永久保存，也会因为内存的容量限制不能一直呆在内存中，需要持久化来缓存到外存。

### 持久层
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200731102140441.png#pic_center)

**什么是持久层？**
DAO (Data Access Object)层完成持久化工作的代码块，大多数情况下特别是企业级应用，数据持久化往往也就意味着将内存中的数据保存到磁盘上加以固化，而持久化的实现过程则大多通过各种关系数据库来完成。

不过这里有一个字需要特别强调，也就是所谓的`层`。对于应用系统而言，数据持久功能大多是必不可少的组成部分。也就是说，我们的系统中，已经天然的具备了`持久层`概念？也许是，但也许实际情况并非如此。之所以要独立出一个`持久层`的概念,而不是`持久模块`，`持久单元`，也就意味着，我们的系统架构中，应该有一个相对独立的逻辑层面，**专注于数据持久化逻辑的实现**，与系统中其他部分相对而言，这个层面应该具有一个较为清晰和严格的逻辑边界。说白了就是用来操作数据库存在的！
### 为什么需要Mybatis
[百度百科](https://baike.baidu.com/item/MyBatis/2824918?fr=aladdin) 说明优点如下：
- Mybatis就是帮助程序猿将数据存入数据库中 , 和从数据库中取数据 。

- 传统的jdbc操作 , 有很多重复代码块，比如 : 数据取出时的封装 , 数据库的建立连接等等， 通过框架可以**减少重复代码，提高开发效率** 。

- MyBatis 是一个**半自动化的ORM框架** (Object Relationship Mapping) -->对象关系映射

- 所有的事情，不用Mybatis依旧可以做到，只是用了它，所有实现会更加简单！只是相对来说是个轻量便捷的框架大家都在用，我们也用呗。

**MyBatis的优点**

  - 简单易学：本身就很小且简单。没有任何第三方依赖，最简单安装只要两个jar文件+配置几个sql映射文件就可以了，易于学习，易于使用，通过文档和源代码，可以比较完全的掌握它的设计思路和实现。

  - 灵活：mybatis不会对应用程序或者数据库的现有设计强加任何影响。sql写在xml里，便于统一管理和优化。通过sql语句可以满足操作数据库的所有需求。

  - 解除sql与程序代码的**耦合**：通过提供DAO层，将业务逻辑和数据访问逻辑分离，使系统的设计更清晰，更易维护，更易单元测试。sql和代码的分离，提高了可维护性。

  - 提供xml标签，支持编写动态sql。

**最重要的一点，使用的人多、公司需要！**

# 一、MyBatis第一个程序
### 1. 搭建实验数据库
```sql
create database mybatis;

use mybatis;
DROP TABLE IF EXISTS `user`;

CREATE TABLE `user` (
`id` int(20) NOT NULL,
`name` varchar(30) DEFAULT NULL,
`pwd` varchar(30) DEFAULT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert  into `user`(`id`,`name`,`pwd`) values 
(1,'sowhat','123456'),
(2,'zhangsan','abcdef'),
(3,'lisi','987654');
```
### 2. 导入MyBatis相关 jar 包
GitHub或者Maven中央仓库找即可。
```xml
     <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13</version>
        </dependency>
```
### 3. 配置文件编写
[配置文件](https://mybatis.org/mybatis-3/zh/getting-started.html) mybatis-config.xml，主要是连接数据库的配置
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=utf8"/>
                <!-- MySQL8 的话主要要设置时区问题-- >
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com/sowhat/dao/userMapper.xml"/>
    </mappers>
</configuration>
```
### 4. MySQL连接池
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200731123305372.png#pic_center)

```java
package com.sowhat.utils;

/**
 * @author sowhat
 * @create 2020-07-31 12:04
 */

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

public class MybatisUtils
{
	private static SqlSessionFactory sqlSessionFactory;
	static
	{
		try
		{
			String resource = "mybatis-config.xml";
			InputStream inputStream = Resources.getResourceAsStream(resource);
			sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
		} catch (IOException e)
		{
			e.printStackTrace();
		}
	}

	//获取SqlSession连接
	public static SqlSession getSession()
	{
		return sqlSessionFactory.openSession();
	}
}
```
### 5. POJO类
```java
package com.sowhat.pojo;

/**
 * @author sowhat
 * @create 2020-07-31 12:01
 */
public class User
{

	private int id;
	private String name;
	private String pwd;


	public int getId()
	{
		return id;
	}

	public void setId(int id)
	{
		this.id = id;
	}

	public String getName()
	{
		return name;
	}

	public void setName(String name)
	{
		this.name = name;
	}

	public String getPwd()
	{
		return pwd;
	}

	public void setPwd(String pwd)
	{
		this.pwd = pwd;
	}

	@Override
	public String toString()
	{
		return "User{" +
				"id=" + id +
				", name='" + name + '\'' +
				", pwd='" + pwd + '\'' +
				'}';
	}
}
```
### 6. Dao层设置
接口类设置如下：
```java
package com.sowhat.dao;

import com.sowhat.pojo.User;

import java.util.List;

/**
 * @author sowhat
 * @create 2020-07-31 12:03
 */
public interface UserMapper
{
	List<User> selectUser();
}
```
往常的话就是实际编写接口类的实现类从而操作MySQL，在Mybatis中不用实现接口类，设定xml文件即可。
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.sowhat.dao.UserMapper">
    <select id="selectUser" resultType="com.sowhat.pojo.User">
         select * from user
    </select>
</mapper>
```
PS : namespace 十分重要，不能写错！
### 7.问题说明
可能出现问题说明：Maven静态资源过滤问题，因此pom.xml文件配置如下
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>mybatislearn</artifactId>
        <groupId>com.sowhat</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <modelVersion>4.0.0</modelVersion>
    <artifactId>mybatis01</artifactId>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <!-- 解决文件过滤问题 -->
    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
    </build>

</project>
```
### 8. CRUD基本操作
##### 1. select
- select标签是Mybatis中最常用的标签之一

- select语句有很多属性可以详细配置每一条SQL语句

  - SQL语句返回值类型。【完整的类名或者别名】

  - 传入SQL语句的参数类型 。【万能的Map，可以多尝试使用】

  - 命名空间中唯一的标识符

  - 接口中的方法名与映射文件中的SQL语句ID 一 一 对应

   - id

  - parameterType

  -  resultType
 
**根据id查询用户**

1、在UserMapper中添加对应方法
```java
public interface UserMapper
{
	List<User> selectUser();
	User selectUserById(int id);
}
```
2、在UserMapper.xml中添加Select语句
```xml
    <select id="selectUserById" resultType="com.sowhat.pojo.User">
        select * from mybatis.user where id = #{id}
    </select>
```
3、测试类中测试
```java
	@Test
	public void tsetSelectUserById() {
		SqlSession session = MybatisUtils.getSession();  //获取SqlSession连接
		UserMapper mapper = session.getMapper(UserMapper.class);
		User user = mapper.selectUserById(1);
		System.out.println(user);
		session.close();
	}
```
**根据 密码 和 名字 查询用户**

思路一：直接在方法中传递参数

1、在接口方法的参数前加 @Param属性

2、Sql语句编写的时候，直接取@Param中设置的值即可，不需要单独设置参数类型
```java
//通过密码和名字查询用户
User selectUserByNP(@Param("username") String username,@Param("pwd") String pwd);
```
3、在UserMapper.xml中添加Select语句
```xml
   <select id="selectUserByNP" resultType="com.sowhat.pojo.User">
     select * from user where name = #{username} and pwd = #{pwd}
   </select>
```
4、测试类
```java
	@Test
	public void tsetselectUserByNP() {
		SqlSession session = MybatisUtils.getSession();  //获取SqlSession连接
		UserMapper mapper = session.getMapper(UserMapper.class);
		User user = mapper.selectUserByNP("lisi","987654");
		System.out.println(user);
		session.close();
	}
```
**思路二：使用万能的Map**

1、在接口方法中，参数直接传递Map；
```java
User selectUserByNP2(Map<String,Object> map);
```
2、编写sql语句的时候，需要传递参数类型，参数类型为map
```java
<select id="selectUserByNP2" parameterType="map" resultType="com.sowhat.pojo.User">
select * from user where name = #{username} and pwd = #{pwd}
</select>
```
3、在使用方法的时候，Map的 key 为 sql中取的值即可，没有顺序要求！
```java
	@Test
	public void testSelectUserByNP2(){
		SqlSession session = MybatisUtils.getSession();
		UserMapper mapper = session.getMapper(UserMapper.class);
		Map<String, Object> map = new HashMap<String, Object>();
		map.put("username","lisi");
		map.put("pwd","987654");
		User user = mapper.selectUserByNP2(map);
		System.out.println(user);
		session.close();
	}
```
`总结`：如果参数过多，我们可以考虑直接使用Map实现，如果参数比较少，直接传递参数即可。传入对象的话可以直接拿对象里的若干字段来使用哦。
##### 2. insert
我们一般使用insert标签进行插入操作，它的配置和select标签差不多！

**需求**：给数据库增加一个用户
1、在UserMapper接口中添加对应的方法
```java
//添加一个用户
int addUser(User user);
```
2、在UserMapper.xml中添加insert语句
```xml 
<insert id="addUser" parameterType="com.kuang.pojo.User">
    insert into user (id,name,pwd) values (#{id},#{name},#{pwd})
</insert>
```
3、测试
```java
	@Test
	public void testAddUser() {
		SqlSession session = MybatisUtils.getSession();
		UserMapper mapper = session.getMapper(UserMapper.class);
		User user = new User(4,"wangwu","zxcvbn");
		int i = mapper.addUser(user);
		System.out.println(i);
		//提交事务,重点!不写的话不会提交到数据库
		session.commit();
		session.close();
	}
```
`注意点`：增、删、改操作需要`提交事务`！
##### 3. update
我们一般使用update标签进行更新操作，它的配置和select标签差不多！

`需求`：修改用户的信息

1、同理，编写接口方法
```java
//修改一个用户
int updateUser(User user);
```
2、编写对应的配置文件SQL
```java
<update id="updateUser" parameterType="com.sowhat.pojo.User">
  update user set name=#{name},pwd=#{pwd} where id = #{id}
</update>
```
3、测试
```java
@Test
public void testUpdateUser() {
   SqlSession session = MybatisUtils.getSession();
   UserMapper mapper = session.getMapper(UserMapper.class);
   User user = mapper.selectUserById(1);
   user.setPwd("asdfgh");
   int i = mapper.updateUser(user);
   System.out.println(i);
   session.commit(); //提交事务,重点!不写的话不会提交到数据库
   session.close();
}
```
##### 4. delete
我们一般使用delete标签进行删除操作，它的配置和select标签差不多！

需求：根据id删除一个用户

1、同理，编写接口方法
```java
//根据id删除用户
int deleteUser(int id);
```
2、编写对应的配置文件SQL
```java 
<delete id="deleteUser" parameterType="int">
  delete from user where id = #{id}
</delete>
```
3、测试
```java 
@Test
public void testDeleteUser() {
   SqlSession session = MybatisUtils.getSession();
   UserMapper mapper = session.getMapper(UserMapper.class);
   int i = mapper.deleteUser(4);
   System.out.println(i);
   session.commit(); //提交事务,重点!不写的话不会提交到数据库
   session.close();
}
```
##### 5. like 
模糊查询跟MySQL类似，不过有两种实现方式，一种是传入的是%like%，一种传入的是like后台加工为%like%。
1、接口
```java
	List<User> selectlike1(String wildcardname);
	List<User> selectlike2(String wildcardname);
```
2、xml文件配置
```xml
    <select id="selectlike1" resultType="com.sowhat.pojo.User">
       select * from user where name like #{value}
    </select>

    <select id="selectlike2" resultType="com.sowhat.pojo.User">
       select * from user where name like "%"#{value}"%"
    </select>
```
3、测试类
```java
	@Test
	public void testSelectlike() {
		SqlSession session = MybatisUtils.getSession();
		UserMapper mapper = session.getMapper(UserMapper.class);
		List<User> users1 = mapper.selectlike1("%what%");
		users1.forEach(s-> System.out.println(s));
		System.out.println("-------");
		List<User> users2 = mapper.selectlike2("what");
		users2.forEach(s-> System.out.println(s));
		System.out.println("-------");
		session.commit(); //提交事务,重点!不写的话不会提交到数据库
		session.close();
	}
```
##### 6. End
- 所有的增删改操作都需要提交事务！

- 接口所有的普通参数，尽量都写上@Param参数，尤其是多个参数时，必须写上！

- 有时候根据业务的需求，可以考虑使用map传递参数！

- 为了规范操作，在SQL的配置文件中，我们尽量将Parameter参数和resultType都写上！
# 二、Mybatis核心配置
参考 [MyBatis3 PDF](https://download.csdn.net/download/q514004204/9110451) 核心配置文件

- mybatis-config.xml 系统核心配置文件
MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。

能配置的内容如下(严格注意顺序)：
```xml
configuration（配置）
properties（属性）
settings（设置）
typeAliases（类型别名）
typeHandlers（类型处理器）
objectFactory（对象工厂）
plugins（插件）
environments（环境配置）
environment（环境变量）
transactionManager（事务管理器）
dataSource（数据源）
databaseIdProvider（数据库厂商标识）
mappers（映射器）
<!-- 注意元素节点的顺序！顺序不对会报错 -->
```
我们可以阅读 mybatis-config.xml 上面的dtd的头文件！

### environments元素
```xml
<environments default="development">
 <environment id="development">
   <transactionManager type="JDBC">
     <property name="..." value="..."/>
   </transactionManager>
   <dataSource type="POOLED">
     <property name="driver" value="${driver}"/>
     <property name="url" value="${url}"/>
     <property name="username" value="${username}"/>
     <property name="password" value="${password}"/>
   </dataSource>
 </environment>
</environments>
```
- 配置MyBatis的多套运行环境，将SQL映射到多个不同的数据库上，必须指定其中一个为默认运行环境（通过default指定）
- 子元素节点：environment
  - dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象的资源。
  - 数据源是必须配置的。
  - 有三种内建的数据源类型
   type="[UNPOOLED|POOLED|JNDI]"
     - unpooled：这个数据源的实现只是每次被请求时打开和关闭连接。
      - pooled：这种数据源的实现利用`池`的概念将 JDBC 连接对象组织起来 , 这是一种使得并发 Web 应用快速响应请求的流行处理方式，默认用此。
     -  jndi：这个数据源的实现是为了能在如 Spring 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的引用。

- 数据源`transactionManager`也有很多第三方的实现，比如dbcp，c3p0，druid等等....

这两种事务管理器类型都不需要设置任何属性。

- 具体的一套环境，通过设置id进行区别，id保证唯一！
- 子元素节点：`transactionManager` - [ JDBC | MANAGED ]
### mappers元素
`映射器` : 定义映射SQL语句文件
- 既然 MyBatis 的行为其他元素已经配置完了，我们现在就要定义 SQL 映射语句了。但是首先我们需要**告诉 MyBatis 到哪里去找到这些语句**。Java 在自动查找这方面没有提供一个很好的方法，所以最佳的方式是告诉 MyBatis 到哪里去找映射文件。你可以使用相对于类路径的资源引用， 或完全限定资源定位符（包括 file:/// 的 URL），或类名和包名等。映射器是MyBatis中最核心的组件之一，在MyBatis 3之前，只支持xml映射器，即：所有的SQL语句都必须在xml文件中配置。而从MyBatis 3开始，还支持接口映射器，这种映射器方式允许以Java代码的方式注解定义SQL语句，非常简洁。

##### 引入资源方式
1. 使用相对于类路径的资源引用
```xml 
<!-- 使用相对于类路径的资源引用 -->
 <mappers>
        <mapper resource="com/sowhat/dao/userMapper.xml"/>
    </mappers>
```
2. 使用完全限定资源定位符（URL）
```xml 
<!-- 使用完全限定资源定位符（URL） -->
<mappers>
 <mapper url="file:///var/mappers/AuthorMapper.xml"/>
</mappers>
```
3. 使用映射器接口实现类的完全限定类名,需要配置文件名称和接口名称一致，并且位于同一目录下
```xml 
<!--
使用映射器接口实现类的完全限定类名
需要配置文件名称和接口名称一致，并且位于同一目录下
-->
<mappers>
 <mapper class="org.mybatis.builder.AuthorMapper"/>
</mappers>
```
4. 将包内的映射器接口实现全部注册为映射器，但是需要配置文**件名称和接口名称一致**，并且位于同一目录下
```xml 
<!--
将包内的映射器接口实现全部注册为映射器
但是需要配置文件名称和接口名称一致，并且位于同一目录下
-->
<mappers>
 <package name="org.mybatis.builder"/>
</mappers>
```
`PS` ：良好编程习惯，确保Dao层Mapper类跟xml文件`完全一样`。
### Mapper文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.sowhat.dao.UserMapper">
    <select id="selectUser" resultType="com.sowhat.pojo.User">
         select * from user
    </select>
</mapper>
```
namespace中文意思是命名空间，作用如下：
- namespace的命名必须跟某个接口同名
- 接口中的方法与映射文件中sql语句id应该 一 一 对应
-  namespace和子元素的id联合保证唯一 ，区别不同的mapper
- 绑定DAO接口
- namespace命名规则 : 包名+类名

MyBatis 的真正强大在于它的`映射语句`，这是它的魔力所在。由于它的异常强大，映射器的 XML 文件就显得相对简单。如果拿它跟具有相同功能的 JDBC 代码进行对比，你会立即发现省掉了将近 95% 的代码。`MyBatis 为聚焦于 SQL 而构建`，以尽可能地为你减少麻烦。
### Properties优化
数据库这些属性都是可外部配置且可动态替换的，既可以在典型的 Java 属性文件中配置，亦可通过 properties 元素的子元素来传递。具体的官方文档

我们来优化我们的配置文件

第一步 ; 在资源目录下新建一个db.properties
```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSL=true&useUnicode=true&characterEncoding=utf8
username=root
password=root
```
第二步 : 将文件导入properties 配置文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <properties resource="db.properties">
        <property name="password" value="123"/>
         <!--  此处的优先级小于外面 db.properties 设定参数的优先级   -->
    </properties>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="com/sowhat/dao/UserMapper.xml"/>
    </mappers>

</configuration>
```
End：db.properties参数配置优先级大于加载该文件里的优先级。
### typeAliases优化
类型别名是为 Java 类型设置一个短的名字。它只和 XML 配置有关，存在的意义仅在于用来减少类完全限定名的冗余。
#####  1. typeAlias
在`mybatis-config.xml`文件中进行如下映射
```xml 
<typeAliases>
   <typeAlias type="com.sowhat.pojo.User" alias="User"/>
</typeAliases>
```
当这样配置时，User可以用在任何使用com.sowhat.pojo.User的地方。
##### 2. package 
指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean，比如:
```xml
<typeAliases>
   <package name="com.sowhat.pojo"/>
</typeAliases>
```
##### 3. POJO类用Alias
若有注解，则别名为其注解值。见下面的例子：
```java
@Alias("user")
public class User {
  ...
}
```
### 其他配置浏览
设置（settings）相关
- 懒加载
- 日志实现
- 缓存开启关闭

一个配置完整的 settings 元素的示例如下：
```xml
<settings>
 <setting name="cacheEnabled" value="true"/>
 <setting name="lazyLoadingEnabled" value="true"/>
 <setting name="multipleResultSetsEnabled" value="true"/>
 <setting name="useColumnLabel" value="true"/>
 <setting name="useGeneratedKeys" value="false"/>
 <setting name="autoMappingBehavior" value="PARTIAL"/>
 <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
 <setting name="defaultExecutorType" value="SIMPLE"/>
 <setting name="defaultStatementTimeout" value="25"/>
 <setting name="defaultFetchSize" value="100"/>
 <setting name="safeRowBoundsEnabled" value="false"/>
 <setting name="mapUnderscoreToCamelCase" value="false"/>
 <setting name="localCacheScope" value="SESSION"/>
 <setting name="jdbcTypeForNull" value="OTHER"/>
 <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```
### 生命周期和作用域

##### 作用域（Scope）和生命周期
理解我们目前已经讨论过的不同作用域和生命周期类是至关重要的，因为错误的使用会导致非常严重的并发问题。

我们可以先画一个流程图，分析一下Mybatis的执行过程！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200731153938269.png#pic_center)
- `SqlSessionFactoryBuilder` 的作用是**创建** `SqlSessionFactory`，创建成功后，`SqlSessionFactoryBuilder` 就失去了作用，所以它只能存在于创建 `SqlSessionFactory` 的方法中，而不要让其长期存在。因此 `SqlSessionFactoryBuilder `实例的最佳作用域是**方法作用域**（也就是局部方法变量）。

- `SqlSessionFactory `可以被认为是一个**数据库连接池**，它的作用是创建` SqlSession `接口对象。因为 **MyBatis 的本质就是 Java 对数据库的操作**，所以` SqlSessionFactory` 的生命周期存在于**整个 MyBatis 的应用之中**，所以一旦创建了 `SqlSessionFactory`，就要长期保存它，直至不再使用 MyBatis 应用，所以可以认为 **SqlSessionFactory 的生命周期就等同于 MyBatis 的应用周期**。

- 由于` SqlSessionFactory` 是一个对数据库的连接池，所以它占据着数据库的连接资源。如果创建多个 `SqlSessionFactory`，那么就存在多个数据库连接池，这样不利于对数据库资源的控制，也会导致数据库连接资源被消耗光，出现系统宕机等情况，所以尽量避免发生这样的情况。因此在一般的应用中我们**往往希望 SqlSessionFactory 作为一个单例**，让它在应用中被共享。所以说 SqlSessionFactory 的最佳作用域是`应用作用域`。

- 如果说 SqlSessionFactory 相当于`数据库连接池`，那么` SqlSession` 就相当于一个数据库连接（Connection 对象），你可以在一个事务里面执行多条 SQL，然后通过它的 commit、rollback 等方法，提交或者回滚事务。所以它应该**存活在一个业务请求中**，处理完整个请求后，应该关闭这条连接，让它归还给 SqlSessionFactory，否则数据库资源就很快被耗费精光，系统就会瘫痪，所以用 try...catch...finally... 语句来保证其正确关闭。

- 所以 SqlSession 的最佳的作用域是**请求或方法**作用域。
![在这里插入图片描述](https://img-blog.csdnimg.cn/202007311553015.png#pic_center)

# 三、ResultMap及分页
### ResultMap
属性名和字段名不一致情况，我们先看下MySQL中的三个字段。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200731155602422.png#pic_center)
我们定义的User类三个字段：
```java
	private int id;
	private String name;
	private String pwd;
```
三个字段是可以 一 一对应的，所以MySQL查询到自动后是可以自动的实现适配的，转变成对象。 如果我们User类字段设定为如下三个属性则会出现null；
```java
	private int id;
	private String name;
	private String password; //  不一样了
```
1、接口
```java
//根据id查询用户
User selectUserById(int id);
```
2、mapper映射文件
```xml
<select id="selectUserById" resultType="user">
  select * from user where id = #{id}
</select>
```
3、测试
```java
@Test
public void testSelectUserById() {
   SqlSession session = MybatisUtils.getSession();  //获取SqlSession连接
   UserMapper mapper = session.getMapper(UserMapper.class);
   User user = mapper.selectUserById(1);
   System.out.println(user);
   session.close();
}
```
4、结果
```
User{id=1, name='sowhat', password='null'}
```
查询出来发现 password 为空 . 说明出现了问题！

5、分析：

`select * from user where id = #{id}` 可以看做`select  id,name,pwd  from user where id = #{id}`

mybatis会自动映射，根据这些查询的列名(会将列名转化为小写，数据库不区分大小写) , 去对应的实体类中查找相应列名的set方法设值 , 由于找不到setPwd() , 所以password返回null 。

6、解决方案
方案一：为列名指定别名 , 别名和java实体类的属性名一致 .
```xml
<select id="selectUserById" resultType="User">
  select id , name , pwd as password from user where id = #{id}
</select>
```
方案二：使用结果集映射->ResultMap 【推荐】
```xml
<resultMap id="SowhatMap" type="User">
   <!-- id为主键 -->
   <id column="id" property="id"/>
   <!-- column是数据库表的列名 , property是对应实体类的属性名 -->
   <result column="name" property="name"/>
   <result column="pwd" property="password"/>
</resultMap>

<select id="selectUserById" resultMap="SowhatMap">
  select id , name , pwd from user where id = #{id}
</select>
```

##### 自动映射
- resultMap 元素是 MyBatis 中**最重要最强大**的元素。它可以让你从 90% 的 JDBC **ResultSets** 数据提取代码中解放出来。

- 实际上在为一些比如连接的复杂语句编写映射代码的时候，一份 resultMap 能够代替实现同等功能的长达数千行的代码。

- ResultMap 的设计思想是：**对于简单的语句根本不需要配置显式的结果映射**，而对于复杂一点的语句只需要描述它们的关系就行了。

你已经见过简单映射语句的示例了，但并没有显式指定 resultMap。比如：
```xml
<select id="selectUserById" resultType="map">
    select id, name, pwd from user where id = #{id}
</select>
```
上述语句只是简单地将所有的列映射到` HashMap` 的键上，这由 resultType 属性指定。虽然在大部分情况下都够用，但是 HashMap 不是一个很好的模型。你的程序更可能会使用 JavaBean 或 POJO（Plain Old Java Objects，普通老式 Java 对象）作为模型。

ResultMap 最优秀的地方在于，虽然你已经对它相当了解了，但是根本就不需要显式地用到他们。
##### 手动映射
1、返回值类型为resultMap
```xml
<select id="selectUserById" resultMap="UserMap">
  select id , name , pwd from user where id = #{id}
</select>
```
2、编写resultMap，实现手动映射！
```xml
<resultMap id="UserMap" type="User">
   <!-- id为主键 -->
   <id column="id" property="id"/>
   <!-- column是数据库表的列名 , property是对应实体类的属性名 -->
   <result column="name" property="name"/>
   <result column="pwd" property="password"/>
</resultMap>
```
如果总是这么简单就好了。但是肯定不是的，数据库中，存在一对多，多对一的情况，之后会使用到一些高级的结果集映射，association，collection这些，将在之后讲解。现在只需要把这些知识都消化掉才是最重要的！理解`结果集映射`的这个概念！

### 分页
##### limit实现分页
思考：为什么需要分页？
- 在学习mybatis等持久层框架的时候，会经常对数据进行增删改查操作，使用最多的是对数据库进行查询操作，如果查询大量数据的时候，我们往往使用**分页**进行查询，也就是每次处理小部分数据，这样对数据库压力就在可控范围内。

使用Limit实现分页
```sql
#语法
SELECT * FROM table LIMIT stratIndex，pageSize

SELECT * FROM table LIMIT 5,10; // 检索记录行 6-15  

#为了检索从某一个偏移量到记录集的结束所有的记录行，可以指定第二个参数为 -1：   
SELECT * FROM table LIMIT 95,-1; // 检索记录行 96-last.  

#如果只给定一个参数，它表示返回最大的记录行数目：   
SELECT * FROM table LIMIT 5; //检索前 5 个记录行  
#换句话说，LIMIT n 等价于 LIMIT 0,n。 
```
`注意`：当我们用 limit n,m的时候。MySQL是先获取到[0,n+m]行数据，然后从中 [截取](https://sowhat.blog.csdn.net/article/details/71135933) [n+1,m]的数据

**步骤：**

1、修改Mapper文件
```xml
<select id="selectUser" parameterType="map" resultType="user">
  select * from user limit #{startIndex},#{pageSize}
</select>
```
2、Mapper接口，参数为map
```java
//选择全部用户实现分页
List<User> selectUser(Map<String,Integer> map);
```
3、在测试类中传入参数测试
- 推断：起始位置 =  （当前页面 - 1 ） * 页面大小
```java
@Test
public void testSelectUser() {
   SqlSession session = MybatisUtils.getSession();
   UserMapper mapper = session.getMapper(UserMapper.class);

   int currentPage = 1;  //第几页
   int pageSize = 2;  //每页显示几个
   Map<String,Integer> map = new HashMap<String,Integer>();
   map.put("startIndex",(currentPage-1)*pageSize);
   map.put("pageSize",pageSize);

   List<User> users = mapper.selectUser(map);

   for (User user: users){
       System.out.println(user);
  }

   session.close();
}
```
##### RowBounds分页

我们除了使用`Limit`在SQL层面实现分页，也可以使用`RowBounds` 在**Java代码层面**实现分页，当然此种方式作为`了解即可`。我们来看下如何实现的！

**步骤：**

1、mapper接口
```java
//选择全部用户RowBounds实现分页
List<User> getUserByRowBounds();
```
2、mapper文件
```xml
<select id="getUserByRowBounds" resultType="user">
    select * from user
</select>
```
3、测试类
  在这里，我们需要使用RowBounds类
```java
	@Test
	public void testUserByRowBounds() {
		SqlSession session = MybatisUtils.getSession();

		int currentPage = 2;  //第几页
		int pageSize = 2;  //每页显示几个
		RowBounds rowBounds = new RowBounds((currentPage-1)*pageSize,pageSize);

		//通过session.**方法进行传递rowBounds，[此种方式现在已经不推荐使用了]
		List<User> users = session.selectList("com.sowhat.dao.UserMapper.getUserByRowBounds", null, rowBounds);

		for (User user: users){
			System.out.println(user);
		}
		session.close();
	}
```
##### pagehelper分页插件
如果想要将现有的select语句改为支持分页功能的查询语句该怎么做呢？最简单的一种做法就是将所有的select语句都加上limit来实现分页，这种做法有什么问题呢？
- 要改动的地方非常多，而且每个sql改动逻辑基本上一致；
- DAO层的查询逻辑要改动，要在原来查询之后执行查询 SELECT count(1) from ….. 查询数据总条数。

庆幸的是有个开源工具包贼好用 ：pagehelper
官方文档：[https://pagehelper.github.io/](https://pagehelper.github.io/)

依赖导入：
```xml
    <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper</artifactId>
        <version>5.1.2</version>
    </dependency>
    <dependency>
        <groupId>com.github.jsqlparser</groupId>
        <artifactId>jsqlparser</artifactId>
        <version>0.9.1</version>
    </dependency>
```
mybatis-config.xml配置：
```xml
    <!--注意这里要写成PageInterceptor, 5.0之前的版本都是写PageHelper, 5.0之后要换成PageInterceptor-->
    <plugins>
        <plugin interceptor="com.github.pagehelper.PageInterceptor" >
        </plugin>
    </plugins>
```
接口类：
```java
   List<User> queryUserListLikeName(@Param("name") String name);
```
xml文件配置：
```xml
    <select id="queryUserListLikeName" parameterType="String" resultType="User">
        SELECT * FROM user WHERE name LIKE '%${name}%'
    </select>
```
测试代码：
```java
	// 第一页 显示三个
	@Test
	public void testQueryUserListLikeName() {
		SqlSession session = MybatisUtils.getSession();
		UserMapper mapper = session.getMapper(UserMapper.class);
		//设置分页条件，Parameters:pageNum 页码 pageSize 每页显示数量count 是否进行count查询
		PageHelper.startPage(1, 3, true);
		List<User> users = mapper.queryUserListLikeName("l");

		for (User user : users) {
			System.out.println(user);
		}
		session.close();
	}
```
# 四、日志工厂
思考：我们在测试SQL的时候，要是能够在控制台输出 SQL 的话，是不是就能够有更快的排错效率？

如果一个数据库相关的操作出现了问题，我们可以根据输出的SQL语句快速排查问题。对于以往的开发过程，我们会经常使用到debug模式来调节，跟踪我们的代码执行过程。但是现在使用Mybatis是基于接口，配置文件的源代码执行过程。因此，我们必须选择日志工具来作为我们开发，调节程序的工具。

Mybatis内置的日志工厂提供日志功能，具体的日志实现有以下几种工具：
- SLF4J
- Apache Commons Logging
- Log4j 2
- Log4j
- JDK logging

具体选择哪个日志实现工具由MyBatis的内置日志工厂确定。它会使用最先找到的（按上文列举的顺序查找）。如果一个都未找到，日志功能就会被禁用。

### 标准日志实现

指定 MyBatis 应该使用哪个日志记录实现。如果此设置不存在，则会自动发现日志记录实现。
```xml
<settings>
        <!-- mybatis-config.xml 插入 只打印至控制台 -->
       <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```
测试，可以看到控制台有大量的输出！我们可以通过这些输出来判断程序到底哪里出了Bug。

### Log4j
- Log4j是Apache的一个开源项目
- 通过使用Log4j，我们可以控制日志信息输送的目的地：控制台，文本，GUI组件....
- 我们也可以控制每一条日志的输出格式；
- 通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程。最令人感兴趣的就是，这些可以通过一个配置文件来灵活地进行配置，而不需要修改应用的代码。

**使用步骤**：
1、导入log4j的包
```xml
<dependency>
   <groupId>log4j</groupId>
   <artifactId>log4j</artifactId>
   <version>1.2.17</version>
</dependency>
```

```xml
<settings>
        <!-- mybatis-config.xml 插入 只打印至控制台 -->
       <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```

2、配置文件 log4j.properties 编写
```properties
#将等级为DEBUG的日志信息输出到console和file这两个目的地，console和file的定义在下面的代码
log4j.rootLogger=DEBUG,console,file

#控制台输出的相关设置
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.Target = System.out
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%c]-%m%n

#文件输出的相关设置
log4j.appender.file = org.apache.log4j.RollingFileAppender
log4j.appender.file.File=./log/sowhat.log
log4j.appender.file.MaxFileSize=10mb
log4j.appender.file.Threshold=DEBUG
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n

#日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sql=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
```
3、setting设置日志实现
```xml
<settings>
   <setting name="logImpl" value="LOG4J"/>
</settings>
```
4、在程序中使用Log4j进行输出！

```java
//注意导包：org.apache.log4j.Logger
static Logger logger = Logger.getLogger(MyTest.class);
@Test
public void selectUser() {
   logger.info("info：进入selectUser方法");
   logger.debug("debug：进入selectUser方法");
   logger.error("error: 进入selectUser方法");
   SqlSession session = MybatisUtils.getSession();
   UserMapper mapper = session.getMapper(UserMapper.class);
   List<User> users = mapper.selectUser();
   for (User user: users){
       System.out.println(user);
  }
   session.close();
}
```
5、测试，看控制台输出！
- 使用Log4j 输出日志。
-  灵活性的修改file的日志级别，可以看到还生成了一个日志的文件。

# 五、使用注解开发
### 面向接口编程

大家之前都学过面向**对象**编程，也学习过接口，但在真正的开发中，很多时候我们会选择面向接口编程

- 根本原因 :  解耦 , 可拓展 , 提高复用 , 分层开发中 , 上层不用管具体的实现 , 大家都遵守共同的标准 , 使得开发变得容易 , 规范性更好

- 在一个面向对象的系统中，系统的各种功能是由许许多多的不同对象协作完成的。在这种情况下，各个对象内部是如何实现自己的，对系统设计人员来讲就不那么重要了。

- 而各个对象之间的**协作关系**则成为系统设计的关键。小到不同类之间的通信，大到各模块之间的交互，在系统设计之初都是要着重考虑的，这也是系统设计的主要工作内容。面向接口编程就是指按照这种思想来编程。

**关于接口的理解**

- 接口从更深层次的理解，应是**定义（规范，约束）与实现（名实分离的原则）的分离**。

- 接口的本身反映了系统设计人员对**系统的抽象理解**。

- 接口应有两类：

     - 第一类是对一个个体的抽象，它可对应为一个抽象体(abstract class)；

     -  第二类是对一个个体某一方面的抽象，即形成一个抽象面（interface）；

 - 一个体有可能有多个抽象面。抽象体与抽象面是有区别的。

**三个面向区别**

 -  面向对象是指我们考虑问题时，以对象为单位，考虑它的属性及方法 .

 -  面向过程是指我们考虑问题时，以一个具体的流程（事务过程）为单位，考虑它的实现 .

 -  接口设计与非接口设计是针对复用技术而言的，与面向对象（过程）不是一个问题.更多的体现就是对系统整体的架构


### 利用注解开发
**mybatis最初配置信息是基于 XML ,映射语句(SQL)也是定义在 XML 中的。但到MyBatis 3 提供了新的基于注解的配置。不幸的是，Java 注解的的表达力和灵活性十分有限。最强大的 MyBatis 映射并不能用注解来构建**

sql 类型主要分成 :
- @select ()
- @update ()
- @Insert ()
- @delete ()

`注意`：利用注解开发就不需要mapper.xml映射文件了 .
##### demo
1、我们在我们的接口中添加注解
```java
//查询全部用户
@Select("select id,name,pwd password from user")
public List<User> getAllUser();
```
2、在mybatis的核心配置mybatis-config.xml文件中注入
```xml
<!--使用class绑定接口-->
<mappers>
   <mapper class="com.sowhat.mapper.UserMapper"/>
</mappers>
```
3、我们去进行测试
```java
@Test
public void testGetAllUser() {
   SqlSession session = MybatisUtils.getSession();
   //本质上利用了jvm的动态代理机制
   UserMapper mapper = session.getMapper(UserMapper.class);

   List<User> users = mapper.getAllUser();
   for (User user : users){
       System.out.println(user);
  }
   session.close();
}
```
4、利用Debug查看本质
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200801233635204.png#pic_center)
注解在接口上实现，通过反射获得各种配置。
5、本质上利用了jvm的动态代理机制
6、Mybatis详细的执行流程
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200802001727170.png#pic_center)
系统只是把几个关键步骤提供出来了，底层的配置都会自动实现的！跟踪源码就可看到。

`PS` ：增删改一定记得对事务的处理

改造MybatisUtils工具类的getSession( ) 方法，重载实现事务的自动提交。
```java
  //获取SqlSession连接
  public static SqlSession getSession(){
      return getSession(true); //事务自动提交
  }
 
  public static SqlSession getSession(boolean flag){
      return sqlSessionFactory.openSession(flag);
  }
```
##### 1. select
1、编写接口方法注解
```java
//根据id查询用户
@Select("select * from user where id = #{id}")
User selectUserById(@Param("id") int id);
```
2、测试
```java
@Test
public void testSelectUserById() {
   SqlSession session = MybatisUtils.getSession();
   UserMapper mapper = session.getMapper(UserMapper.class);

   User user = mapper.selectUserById(1);
   System.out.println(user);
   session.close();
}
```
##### 2. Insert
1、编写接口方法注解
```java
//添加一个用户
@Insert("insert into user (id,name,pwd) values (#{id},#{name},#{pwd})")
int addUser(User user);
```
2、测试
```java
@Test
public void testAddUser() {
   SqlSession session = MybatisUtils.getSession();
   UserMapper mapper = session.getMapper(UserMapper.class);
   User user = new User(1412, "sowhat", "123456");
   mapper.addUser(user);
   session.close();
}
```
##### 3.  update
1、编写接口方法注解
```java
//修改一个用户
@Update("update user set name=#{name},pwd=#{pwd} where id = #{id}")
int updateUser(User user);
```
2、测试
```java
@Test
public void testUpdateUser() {
   SqlSession session = MybatisUtils.getSession();
   UserMapper mapper = session.getMapper(UserMapper.class);
   User user = new User(1412, "sowhat", "654321");
   mapper.updateUser(user);
   session.close();
}
```
##### 4. delete
1、编写接口方法注解
```java
//根据id删除用
@Delete("delete from user where id = #{id}")
int deleteUser(@Param("id")int id);
```
2、测试
```java
@Test
public void testDeleteUser() {
   SqlSession session = MybatisUtils.getSession();
   UserMapper mapper = session.getMapper(UserMapper.class);
   mapper.deleteUser(1412);
   session.close();
}
```
### 关于@Param
@Param注解用于给方法参数起一个名字。以下是总结的使用原则：
- 在方法只接受一个参数的情况下，可以不使用@Param。
- 在方法接受多个参数的情况下，建议一定要使用@Param注解给参数命名。
- 如果参数是 JavaBean ， 则不能使用@Param。
- 不使用@Param注解时，参数只能有一个，并且是Javabean。

### #与$的区别
- `#{}` 的作用主要是替换预编译语句(PrepareStatement)中的占位符? ，推荐使用
INSERT INTO user (name) VALUES (#{name});
INSERT INTO user (name) VALUES (?);

-  `${}` 的作用是直接进行字符串替换
INSERT INTO user (name) VALUES ('${name}');
INSERT INTO user (name) VALUES ('sowhat');

- 总结：
   - #{}速度快，能防止sql注入，是占位符方式，先预编译，然后填充参数，字符串格式，相当于填空题 用户名=（___），参数只是下划线上的内容
   - ${}是直接拼接到语句上，执行语句，对于上面那道填空题 ，这种方式需要自己拼括号和参数，但是也可以拼接想执行的任何语句，也就是传说中的sql注入
   - 能用的#就不用$

使用注解和配置文件协同开发，才是MyBatis的最佳实践！

# 六、一对多和多对一处理
### 多对一的处理
多对一的理解：多个学生对应一个老师，如果对于学生这边，就是一个多对一的现象，即从学生这边关联一个老师！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200731195310862.png#pic_center)
数据准备：
```sql
CREATE TABLE `teacher` (
`id` INT(10) NOT NULL,
`name` VARCHAR(30) DEFAULT NULL,
PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT INTO teacher(`id`, `name`) VALUES (1, 'sowhat');

CREATE TABLE `student` (
`id` INT(10) NOT NULL,
`name` VARCHAR(30) DEFAULT NULL,
`tid` INT(10) DEFAULT NULL,
PRIMARY KEY (`id`),
KEY `fktid` (`tid`),
CONSTRAINT `fktid` FOREIGN KEY (`tid`) REFERENCES `teacher` (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8


INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('1', '小明', '1');
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('2', '小红', '1');
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('3', '小张', '1');
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('4', '小李', '1');
INSERT INTO `student` (`id`, `name`, `tid`) VALUES ('5', '小王', '1');
```
1、pom文件中加入防过滤
```xml
    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
    </build>
```

2、引入Maven依赖
```xml
<!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
<dependency>
 <groupId>org.projectlombok</groupId>
 <artifactId>lombok</artifactId>
 <version>1.16.10</version>
</dependency>
```
3、在代码中增加注解
```java
package com.sowhat.bean;
import lombok.Data;
/**
 * @author sowhat
 * @create 2020-07-31 19:56
 */
@Data
public class Teacher
{
	private int id;
	private String name;
}

```
```java
package com.sowhat.bean;
import lombok.Data;
/**
 * @author sowhat
 * @create 2020-07-31 19:58
 */
@Data
public class Student
{
	private int id;
	private String name;
	//多个学生可以是同一个老师，即多对一
	private Teacher teacher;
}
```
4、编写实体类对应的Mapper接口 【两个】无论有没有需求，都应该写上，以备后来之需！
```java
package com.sowhat.mapper;
import com.sowhat.bean.Student;
import java.util.List;
/**
 * @author sowhat
 * @create 2020-07-31 19:59
 */
public interface  StudentMapper
{
	//获取所有学生及对应老师的信息
	public List<Student> getStudents();
	public List<Student> getStudents2();
}
```
```java
package com.sowhat.mapper;

/**
 * @author sowhat
 * @create 2020-07-31 19:59
 */
public interface TeacherMapper
{
}
```

5、编写Mapper接口对应的 mapper.xml配置文件 【两个】无论有没有需求，都应该写上，以备后来之需！
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
       PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.sowhat.mapper.StudentMapper">
</mapper>
```
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
       PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.sowhat.mapper.TeacherMapper">
</mapper>
```
##### 1. 按查询嵌套处理
1、给StudentMapper接口增加方法
```java
//获取所有学生及对应老师的信息
public List<Student> getStudents();
```
2、编写对应的Mapper文件
 1. 获取所有学生的信息
 2. 根据获取的学生信息的老师ID获取该老师的信息
3. 思考问题，这样学生的结果集中应该包含老师，该如何处理呢，数据库中我们一般使用**关联查询**？
           1. 做一个结果集映射：StudentTeacher
           2. StudentTeacher结果集的类型为 Student
           3. 学生中老师的属性为teacher，对应数据库中为tid。
              多个 [1,...）学生关联一个老师=> 一对一，一对多
           4. 查看官网找到：association – 一个复杂类型的关联；使用它来处理关联查询

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.sowhat.mapper.StudentMapper">

    <select id="getStudents" resultMap="StudentTeacher">
        select * from student
    </select>

    <resultMap id="StudentTeacher" type="Student">
        <!--association 关联属性 property属性名 javaType返回属性类型 column在多的一方的表中的列名-->
       <association property="teacher"  column="tid" javaType="Teacher" select="getTeacher"/>
      <association property="teacher"  column="{id = tid}" javaType="Teacher" select="getTeacher"/>
    </resultMap>
    <!--
    这里传递过来的id，只有一个属性的时候，下面可以写任何值
    association中column多参数配置：column="{key=value,key=value}" id = tid 这样
        其实就是键值对的形式，key是传给下个sql的取值名称，value是片段一中sql查询的字段名。
    -->
    <select id="getTeacher" resultType="teacher">
      select * from teacher where id = #{id}
   </select>

</mapper>
```
3、编写完毕去mybatis-config.xml配置文件中，注册Mapper！
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <properties resource="db.properties">
        <property name="password" value="123"/>
        <!--  此处的优先级小于外面 db.properties 设定参数的优先级   -->
    </properties>

    <typeAliases>
        <package name="com.sowhat.bean"/>
    </typeAliases>


    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper class="com.sowhat.mapper.StudentMapper"/>
        <mapper class="com.sowhat.mapper.TeacherMapper"/>
    </mappers>

</configuration>
```
4、测试
```java
@Test
public void testGetStudents(){
   SqlSession session = MybatisUtils.getSession();
   StudentMapper mapper = session.getMapper(StudentMapper.class);
   List<Student> students = mapper.getStudents();
   for (Student student : students){
       System.out.println(
               "学生名:"+ student.getName()
                       +"\t老师:"+student.getTeacher().getName());
  }
}
```
##### 2. 按结果嵌套处理
除了上面这种方式，还有其他思路吗？我们还可以按照结果进行嵌套处理；
1、接口方法编写
```java
public List<Student> getStudents2();
```
2、编写对应的mapper文件
```xml
<!--
按查询结果嵌套处理
思路：
   1. 直接查询出结果，进行结果集的映射
-->
<select id="getStudents2" resultMap="StudentTeacher2" >
  select s.id as sid,s.name as sname, t.name as tname 
  from student s,teacher t where s.tid = t.id
</select>

<resultMap id="StudentTeacher2" type="Student">
   <id property="id" column="sid"/> <!-- 主键配置-->
   <result property="name" column="sname"/>
   <!--关联对象property 关联对象在Student实体类中的属性-->
   <association property="teacher" javaType="Teacher">
       <result property="name" column="tname"/>
   </association>
</resultMap>
```
3、测试
```java
@Test
public void testGetStudents2(){
   SqlSession session = MybatisUtils.getSession();
   StudentMapper mapper = session.getMapper(StudentMapper.class);

   List<Student> students = mapper.getStudents2();

   for (Student student : students){
       System.out.println(
               "学生名:"+ student.getName()
                       +"\t老师:"+student.getTeacher().getName());
  }
}
```
##### End
- 按照查询进行嵌套处理就像SQL中的**子查询**

- 按照结果进行嵌套处理就像SQL中的**联表查询**


### 一对多的处理
一对多的理解：一个老师拥有多个学生，如果对于老师这边，就是一个一对多的现象，即从一个老师下面拥有一群学生（集合）！
1、pojo类
```java
@Data
public class Student {
    private int id;
    private String name;
    private int tid;
}
```
```java
@Data
public class Teacher {
    private int id;
    private String name;
    private List<Student> students;
}
```
2、接口类
```java
public interface TeacherMapper {
    Teacher getTeacher(@Param("tid") int id);
    Teacher getTeacher2(@Param("tid") int id);
}
```
3、xml解析
```xml
    <!--按照结果查询 -->
    <select id="getTeacher" resultMap="TeacherStudent">
        select s.id as sid,s.name as sname,t.name as tname,t.id as tid
        from student s,teacher t where s.tid = t.id and t.id = #{tid}
    </select>

    <resultMap id="TeacherStudent" type="Teacher">
        <result property="id" column="tid"/>
        <result property="name" column="tname"/>
<!--         复杂的属性，对象用association，集合用collection，javaType指定属性类型，集合中的泛型用ofTypoe-->
        <collection property="students" ofType="Student">
            <result property="id" column="sid"/>
            <result property="name" column="sname"/>
            <result property="tid" column="tid"/>
        </collection>
    </resultMap>

    <!--子查询性质 -->
    <select id="getTeacher2" resultMap="TeacherStudent2">
        select * from teacher where id = #{tid}
    </select>
    <resultMap id="TeacherStudent2" type="Teacher">
        <result property="id" column="id"/>
        <result property="name" column="name"/>
        <collection property="students" javaType="ArrayList" ofType="Student" select = "getStudentByTeacherId" column="id" />
    </resultMap>

    <select id="getStudentByTeacherId" resultType="Student">
        select * from  mybatis.student where tid = #{tid}
    </select>
```
4、测试
```java
@Test
public void testGetTeacher(){
   SqlSession session = MybatisUtils.getSession();
   TeacherMapper mapper = session.getMapper(TeacherMapper.class);
   Teacher teacher = mapper.getTeacher2(1);
   //Teacher teacher = mapper.getTeacher(1);
   System.out.println(teacher);
}
```
5、总结
- 按照结果嵌套处理
- 按照查询嵌套处理
- 关联 多对一 association
- 集合 一对多 collection
- JavaType用来指定类中属性类型
- ofType 用来指定映射到List 或者集合中的pojo，泛型中的约束类型。
### 延迟加载
##### 1、什么是延迟加载
- 延迟加载的条件：resultMap可以实现高级映射（使用association、collection实现一对一及一对多映射），`association`、`collection`具备延迟加载功能。
- 延迟加载的好处：**先从单表查询、需要时再从关联表去关联查询**，大大提高 数据库性能，因为查询单表要比关联查询多张表速度要快。
- 延迟加载的实例：查询订单并且关联查询用户信息。如果先查询订单信息即可满足要求，当我们需要查询用户信息时再查询用户信息。把对用户信息的按需去查询就是延迟加载。
- 立即加载：不管用不用，只要一调用方法，马上发起查询。
- 使用场景：在对应的四种表关系中，一对多、多对多通常情况下采用延迟加载，多对一、一对一通常情况下采用立即加载。

理解了延迟加载的特性以后再看Mybatis中如何实现查询方法的延迟加载，在MyBatis 的配置文件中通过设置settings的`lazyLoadingEnabled`属性为true进行**开启**全局的延迟加载，通过`aggressiveLazyLoading`属性开启**立即加载**。看一下官网的介绍，然后通过一个实例来实现Mybatis的延迟加载，在例子中我们展现一对多表关系情况下，通过实现查询用户信息同时查询出该用户所拥有的账户信息的功能展示一下延迟加载的实现方式以及延迟加载和立即加载的结果的不同之处。

1、用户类跟账户类
```java
public class User implements Serializable{
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;
    private List<Account> accountList;
    get和set方法省略.....      
}
---
public class Account implements Serializable{
    private Integer id;
    private Integer uid;
    private Double money;
    get和set方法省略.....      
}
```
2、查询所有用户
```java
 List<User> findAll();
```
3、在UserDao.xml中配置findAll方法的映射
```xml
<resultMap id="userAccountMap" type="com.example.domain.User">
        <id property="id" column="id"/>
        <result property="username" column="username"/>
        <result property="birthday" column="birthday"/>
        <result property="sex" column="sex"/>
        <result property="address" column="address"/>
        <collection property="accountList" ofType="com.example.domain.Account" column="id"
                    select="com.example.dao.AccountDao.findAllByUid"/>
    </resultMap>
    <select id="findAll" resultMap="userAccountMap">
        SELECT * FROM USER;
    </select>
```
通过select指定集合中的每个元素如何查询，在本例中select的属性值为AccountDao.xml文件的namespace   com.example.dao.AccountDao路径以及指定该映射文件下的`findAllByUid`方法，通过这个唯一标识指定集合中元素的查找方式。因为在这里需要用到根据用户ID查找账户，所以需要同时配置一下`findAllByUid`方法的实现。
```java
AccountDao接口中添加
 /**
     * 根据用户ID查询账户信息
     * @return
     */
    List<Account> findAllByUid(Integer uid);

AccountDao.xml文件中配置
<select id="findAllByUid" resultType="com.example.domain.Account">
        SELECT * FROM account WHERE uid = #{uid};
    </select>
```
4、在Mybatis-config.xml的配置文件中开启全局延迟加载
```xml
configuration>
    <settings>
        <!--开启全局的懒加载-->
        <setting name="lazyLoadingEnabled" value="true"/>
        <!--关闭立即加载，其实不用配置，默认为false-->
        <setting name="aggressiveLazyLoading" value="false"/>
        <!--开启Mybatis的sql执行相关信息打印-->
        <setting name="logImpl" value="STDOUT_LOGGING" />
    </settings>
    <typeAliases>
        <typeAlias type="com.example.domain.Account" alias="account"/>
        <typeAlias type="com.example.domain.User" alias="user"/>
        <package name="com.example.domain"/>
    </typeAliases>
    <environments default="test">
        <environment id="test">
            <!--配置事务-->
            <transactionManager type="jdbc"></transactionManager>
            <!--配置连接池-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/test1"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <!--配置映射文件的路径-->
    <mappers>
        <mapper resource="com/example/dao/UserDao.xml"/>
        <mapper resource="com/example/dao/AccountDao.xml"/>
    </mappers>
</configuration>
```
6、测试方法
```java
private InputStream in;
    private SqlSession session;

    private UserDao userDao;
    private AccountDao accountDao;
    private SqlSessionFactory factory;
    @Before
    public void init()throws Exception{
        //获取配置文件
        in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //获取工厂
        factory = new SqlSessionFactoryBuilder().build(in);

        session = factory.openSession();

        userDao = session.getMapper(UserDao.class);
        accountDao = session.getMapper(AccountDao.class);
    }
    @After
    public void destory()throws Exception{
        session.commit();
        session.close();
        in.close();
    }
    @Test
    public void findAllTest(){
        List<User> userList = userDao.findAll();
//        for (User user: userList){
//            System.out.println("每个用户的信息");
//            System.out.println(user);
//            System.out.println(user.getAccountList());
//        }
    }
```
测试说明：当我们注释了findAllTest()方法中的for循环打印的时候，我们将**不会需要用户的账户信息**，按照延迟加载的特性程序只会查询用户的信息，而不会查询账户的信息。当我我们放开for循环打印的时候我们使用到了用户和账户的信息，程序会同时将用户以及对应的账户信息打印出来。

7、测试结果
1. 注释for循环，不使用数据，这时候不需要查询账户信息，我们能在控制台中看到sql执行结果是因为在Mybatis配置文件中添加了logImpl的配置，具体参考第5步中的配置信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803162744924.png#pic_center)
2. 通过for循环打印查询的数据，使用数据，这时候因为使用了数据所以将查询账户信息，我们发现用户和账户查询都进行了执行![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803162821446.png#pic_center)


# 七、动态SQL
什么是动态SQL：动态SQL指的是**根据不同的查询条件 , 生成不同的Sql语句**。

官网描述：
>MyBatis 的强大特性之一便是它的**动态** SQL。如果你有使用 JDBC 或其它类似框架的经验，你就能体会到根据不同条件拼接 SQL 语句的痛苦。例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL 这一特性可以彻底摆脱这种痛苦。
虽然在以前使用动态 SQL 并非一件易事，但正是 MyBatis 提供了可以被用在任意 SQL 映射语句中的强大的动态 SQL 语言得以改进这种情形。
动态 SQL 元素和 JSTL 或基于类似 XML 的文本处理器相似。在 MyBatis 之前的版本中，有很多元素需要花时间了解。MyBatis 3 大大精简了元素种类，现在只需学习原来一半的元素便可。MyBatis 采用功能强大的基于 OGNL 的表达式来淘汰其它大部分元素。

  -------------------------------
  - if
  - choose (when, otherwise)
  - trim (where, set)
  - foreach
  -------------------------------

我们之前写的 SQL 语句都比较简单，如果有比较复杂的业务，我们需要写复杂的 SQL 语句，往往需要拼接，而拼接 SQL ，稍微不注意，由于引号，空格等缺失可能都会导致错误。

那么怎么去解决这个问题呢？这就要使用 mybatis 动态SQL，通过 if、choose、when、otherwise、trim、 where、 se、oreach等标签，可组合成非常灵活的SQL语句，从而在提高 SQL 语句的准确性的同时，也大大提高了开发人员的效率。

### 搭建环境
新建一个数据库表：blog
字段：id，title，author，create_time，views
```sql
use mybatis;
CREATE TABLE `blog` (
`id` varchar(50) NOT NULL COMMENT '博客id',
`title` varchar(100) NOT NULL COMMENT '博客标题',
`author` varchar(30) NOT NULL COMMENT '博客作者',
`create_time` datetime NOT NULL COMMENT '创建时间',
`views` int(30) NOT NULL COMMENT '浏览量'
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```
1、创建Mybatis基础工程
2、IDutil工具类UUID生成主键
```java
public class IDutils {
    public static  String getId(){
        return UUID.randomUUID().toString().replaceAll("-","");
    }
}
```
3、实体类编写，用Lombok。
```java
@Data
public class Blog {
    private String id;
    private String title;
    private String author;
    private Date createTime;
    private int views;
}
```
4、编写Mapper接口及xml文件
```java
public interface BlogMapper {
    int addBlog(Blog blog);
}
```
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.sowhat.dao.BlogMapper">
    <insert id="addBlog" parameterType="Blog">
        insert into mybatis.blog (id,title,author,create_time,views) values
        (#{id},#{title},#{author},#{createTime},#{views})
    </insert>
</mapper>
```
5、mybatis核心配置文件，下划线驼峰自动转换
```xml
<settings>
   <setting name="mapUnderscoreToCamelCase" value="true"/>
   <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
<!--注册Mapper.xml-->
<mappers>
 <mapper resource="mapper/BlogMapper.xml"/>
</mappers>
```
6、初始化博客方法
```java
    @Test
    public void testGetTeacher(){
        SqlSession session = MybatisUtils.getSession();
        BlogMapper mapper = session.getMapper(BlogMapper.class);
        Blog blog = new Blog();
        blog.setId(IDutils.getId());
        blog.setTitle("Mybatis轻松学");
        blog.setAuthor("sowhat");
        blog.setCreateTime(new Date());
        blog.setViews(1412);
        mapper.addBlog(blog);

        blog.setId(IDutils.getId());
        blog.setTitle("Java入门到放弃");
        mapper.addBlog(blog);

        blog.setId(IDutils.getId());
        blog.setTitle("Spring good");
        mapper.addBlog(blog);

        blog.setId(IDutils.getId());
        blog.setTitle("python大法好");
        mapper.addBlog(blog);

        session.close();
    }
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200802185335821.png#pic_center)
### if 语句
需求：根据作者名字和博客名字来查询博客！如果作者名字为空，那么只根据博客名字查询，反之，则根据作者名来查询

1、编写接口类
```java
//需求1
List<Blog> queryBlogIf(Map map);
```
2、编写SQL语句
```xml
<!--需求1：
根据作者名字和博客名字来查询博客！
如果作者名字为空，那么只根据博客名字查询，反之，则根据作者名来查询
select * from blog where title = #{title} and author = #{author}
-->
<select id="queryBlogIf" parameterType="map" resultType="blog">
  select * from blog where
   <if test="title != null">
      title = #{title}
   </if>
   <if test="author != null">
      and author = #{author}
   </if>
</select>
```
3、测试
```java
@Test
public void testQueryBlogIf(){
   SqlSession session = MybatisUtils.getSession();
   BlogMapper mapper = session.getMapper(BlogMapper.class);

   HashMap<String, String> map = new HashMap<String, String>();
   map.put("title","Mybatis轻松学");
   map.put("author","sowhat");
   List<Blog> blogs = mapper.queryBlogIf(map);

   System.out.println(blogs);

   session.close();
}
```
这样写我们可以看到，如果 author 等于 null，那么查询语句为 select * from user where title=#{title},但是如果title为空呢？那么查询语句为 select * from user where and author=#{author}，这是错误的 SQL 语句，如何解决呢？请看下面的 **where** 语句！
### where
修改上面的SQL语句；
```xml
<select id="queryBlogIf" parameterType="map" resultType="blog">
  select * from blog
   <where>
       <if test="title != null">
          title = #{title}
       </if>
       <if test="author != null">
          and author = #{author}
       </if>
   </where>
</select>
```
这个`where`标签会知道如果它包含的标签中有返回值的话，它就插入一个`where`。此外，如果标签返回的内容是以AND 或OR 开头的，则它会剔除掉。
### set 
在我们update到时候，set可以动态删除无关逗号。
1、接口配置
```java
   int updateBlog(Map map);
```
2、xml配置
```xml
    <update id="updateBlog" parameterType="map">
        update mybatis.blog
        <set>
            <if test ="title !=null">
                title = #{title},
            </if>
            <if test = "author !=null">
                author = #{author},
            </if>
        </set>
        where id = #{id}
    </update>
```
3、测试
```java
    @Test
    public void testGetTeacher(){
        SqlSession session = MybatisUtils.getSession();
        BlogMapper mapper = session.getMapper(BlogMapper.class);
        HashMap<Object, Object> hashMap = new HashMap<>();
        hashMap.put("title","mybatis");
        hashMap.put("author","SOWHAT");
        hashMap.put("id","3ba8f24d5a5248d38aa975a20afdb416");
        mapper.updateBlog(hashMap);
    }
```
### trim
mybatis的 [trim](https://blog.csdn.net/wt_better/article/details/80992014) 标签一般用于去除sql语句中多余的and关键字，逗号，或者给sql语句前拼接 where、set以及values(等前缀，或者添加)等后缀，可用于选择性插入、更新、删除或者条件查询等操作。
|属性|	描述|
|--|--|
|prefix	|给sql语句拼接的前缀|
|suffix	|给sql语句拼接的后缀|
|prefixOverrides|	去除sql语句前面的关键字或者字符，该关键字或者字符由prefixOverrides属性指定，假设该属性指定为"AND"，当sql语句的开头为"AND"，trim标签将会去除该"AND"|
|suffixOverrides	|去除sql语句后面的关键字或者字符，该关键字或者字符由suffixOverrides属性指定|

### choose
多选一，跟Java中到switch类似。
1、接口类
```java
    List<Blog> queryBlogChoose(Map map);
```
2、xml配置
```xml
    <select id="queryBlogChoose" parameterType="map" resultType="Blog">
        select *  from mybatis.blog
        <where>
            <choose>
                <when test="title !=null">
                   and  title = #{title}
                </when>
                <when test = "author !=null">
                    and author = #{author}
                </when>
                <otherwise>
                    and view = #{views}
                </otherwise>
            </choose>
        </where>
    </select>
```
3、测试
```java
    @Test
    public void testGetTeacher(){
        SqlSession session = MybatisUtils.getSession();
        BlogMapper mapper = session.getMapper(BlogMapper.class);
        HashMap<Object, Object> hashMap = new HashMap<>();
        // choose 多选一
        hashMap.put("title","Mybatis轻松学");
        hashMap.put("author","sowhat");
        hashMap.put("views",9999);

        List<Blog> blogs = mapper.queryBlogChoose(hashMap);
        blogs.forEach(s-> System.out.println(s));
    }
```
所谓到动态SQL本质上还是SQL语句，只是我们可以在SQL层面执行一些逻辑判断嵌套。
### SQL片段
 有时候可能某个 sql 语句我们用的特别多，为了增加代码的重用性，简化代码，我们需要将这些代码抽取出来，然后使用时直接调用。
提取SQL片段：
```xml
<sql id="if-title-author">
   <if test="title != null">
      title = #{title}
   </if>
   <if test="author != null">
      and author = #{author}
   </if>
</sql>
```
引用SQL片段：
```xml
<select id="queryBlogIf" parameterType="map" resultType="blog">
  select * from blog
   <where>
       <!-- 引用 sql 片段，如果refid 指定的不在本文件中，那么需要在前面加上 namespace -->
       <include refid="if-title-author"></include>
       <!-- 在这里还可以引用其他的 sql 片段 -->
   </where>
</select>
```
注意：
- 最好基于单表来定义 sql 片段，提高片段的可重用性
- 在 sql 片段中不要包括 where 标签

### foreach
将数据库中前三个数据的id修改为1,2,3；

需求：我们需要查询 blog 表中 id 分别为1,2,3的博客信息

1、编写接口
```java
List<Blog> queryBlogForeach(Map map);
```
2、编写SQL语句
```xml
<select id="queryBlogForeach" parameterType="map" resultType="blog">
  select * from blog
   <where>
       <!--
       collection:指定输入对象中的集合属性
       item:每次遍历生成的对象
       open:开始遍历时的拼接字符串
       close:结束时拼接的字符串
       separator:遍历对象之间需要拼接的字符串
       select * from blog where (id=1 or id=2 or id=3)
     -->
       <foreach collection="ids"  item="id" open="and (" close=")" separator="or">
          id=#{id}
       </foreach>
   </where>
</select>
```
3、测试
```java
    @Test
    public void testQueryBlogForeach(){
        SqlSession session = MybatisUtils.getSession();
        BlogMapper mapper = session.getMapper(BlogMapper.class);

        HashMap map = new HashMap();
        List<Integer> ids = new ArrayList<Integer>();
        ids.add(1);
        ids.add(2);
        ids.add(3);
        map.put("ids",ids);

        List<Blog> blogs = mapper.queryBlogForeach(map);
        blogs.forEach(s-> System.out.println(s));
        session.close();
    }
```
结果：
```
Blog(id=1, title=mybatis, author=SOWHAT, createTime=Sun Aug 02 18:42:39 CST 2020, views=1412)
Blog(id=2, title=Java入门到放弃, author=sowhat, createTime=Sun Aug 02 18:42:39 CST 2020, views=1412)
Blog(id=3, title=Spring good, author=sowhat, createTime=Sun Aug 02 18:42:39 CST 2020, views=1412)
```
`小结`：其实动态 sql 语句的编写往往就是一个拼接的问题，为了保证拼接准确，我们最好首先要写原生的 sql 语句出来，然后在通过 mybatis 动态sql 对照着改，防止出错。多在实践中使用才是熟练掌握它的技巧。
# 八、缓存
### 简介
1、什么是缓存 [ Cache ]？
- 存在内存中的临时数据。
- 将用户经常查询的数据放在缓存（内存）中，用户去查询数据就不用从磁盘上(关系型数据库数据文件)查询，从缓存中查询，从而提高查询效率，解决了高并发系统的性能问题。

2、为什么使用缓存？
>减少和数据库的交互次数，减少系统开销，提高系统效率。

3、什么样的数据能使用缓存？
>经常查询并且不经常改变的数据。

### Mybatis缓存
- MyBatis包含一个非常强大的查询缓存特性，它可以非常方便地定制和配置缓存。缓存可以极大的提升查询效率。

- MyBatis系统中默认定义了两级缓存：`一级缓存`和`二级缓存`

   - 默认情况下，只有一级缓存开启。（SqlSession级别的缓存，也称为本地缓存）

    - 二级缓存需要手动开启和配置，他是基于`namespace`级别的缓存。

    - 为了提高扩展性，MyBatis定义了缓存接口Cache。我们可以通过实现`Cache`接口来**自定义**二级缓存

可选缓存策略有：LRU（最近最少使用，默认策略）、FIFO（先进先出）、LFU（最少访问次数）。
-  FIFO：first in first out，这个是大家最熟的，先进先出。
- LFU： Less Frequently Used，就是上面例子中使用的策略，直白一点就是讲一直以来最少被使用的。如上面所讲，缓存的元素有一个hit属性，hit值最小的将会被清出缓存。
 -  LRU：Least Recently Used，最近最少使用的，缓存的元素有一个时间戳，当缓存容量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳离当前时间最远的元素将被清出缓存。

#### 一级缓存
一级缓存也叫本地缓存：
- 与数据库同一次会话期间查询到的数据会放在本地缓存中。
- 以后如果需要获取相同的数据，直接从缓存中拿，没必须再去查询数据库；

**测试**

1、在mybatis中加入日志，方便测试结果
```xml
   <!--    mysql中的数据类型转换为驼峰命名-->
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!-- mybatis-config.xml 插入 只打印至控制台 -->
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
```
2、编写接口方法
```java
//根据id查询用户
User queryUserById(@Param("id") int id);
```
3、接口对应的Mapper文件
```xml
<select id="queryUserById" resultType="user">
  select * from user where id = #{id}
</select>
```
4、测试
```xml
@Test
public void testQueryUserById(){
   SqlSession session = MybatisUtils.getSession();
   UserMapper mapper = session.getMapper(UserMapper.class);

   User user = mapper.queryUserById(1);
   System.out.println(user);
   User user2 = mapper.queryUserById(1);
   System.out.println(user2);
   System.out.println(user==user2);

   session.close();
}
```
5、结果分析
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020080222083445.png#pic_center)
##### 一级缓存失效的四种情况

- 一级缓存是SqlSession级别的缓存，是一直开启的，我们关闭不了它；

- 一级缓存失效情况：没有使用到当前的一级缓存，效果就是，还需要再向数据库中发起一次查询请求！ 

1、sqlSession不同
```java
@Test
public void testQueryUserById(){
   SqlSession session = MybatisUtils.getSession();
   SqlSession session2 = MybatisUtils.getSession();
   UserMapper mapper = session.getMapper(UserMapper.class);
   UserMapper mapper2 = session2.getMapper(UserMapper.class);

   User user = mapper.queryUserById(1);
   System.out.println(user);
   User user2 = mapper2.queryUserById(1);
   System.out.println(user2);
   System.out.println(user==user2);

   session.close();
   session2.close();
}
```
观察结果：发现发送了两条SQL语句！

`结论`：每个sqlSession中的缓存相互独立

2、sqlSession相同，查询条件不同
```java
@Test
public void testQueryUserById(){
   SqlSession session = MybatisUtils.getSession();
   UserMapper mapper = session.getMapper(UserMapper.class);
   UserMapper mapper2 = session.getMapper(UserMapper.class);

   User user = mapper.queryUserById(1);
   System.out.println(user);
   User user2 = mapper2.queryUserById(2);
   System.out.println(user2);
   System.out.println(user==user2);

   session.close();
}
```
观察结果：发现发送了两条SQL语句！很正常的理解

`结论`：当前缓存中，不存在这个数据

3、sqlSession相同，两次查询之间执行了增删改操作！
增加方法
```java
//修改用户
int updateUser(Map map);
```
编写SQL
```xml
<update id="updateUser" parameterType="map">
  update user set name = #{name} where id = #{id}
</update>
```
测试
```java
@Test
public void testQueryUserById(){
   SqlSession session = MybatisUtils.getSession();
   UserMapper mapper = session.getMapper(UserMapper.class);
   User user = mapper.queryUserById(1);
   System.out.println(user);
   HashMap map = new HashMap();
   map.put("name","kuangshen");
   map.put("id",4);
   mapper.updateUser(map);

   User user2 = mapper.queryUserById(1);
   System.out.println(user2);
   System.out.println(user==user2);
   session.close();
}
```
观察结果：查询在中间执行了增删改操作后，重新执行了

`结论`：因为增删改操作可能会对当前数据产生影响

4、sqlSession相同，手动清除一级缓存
```java
@Test
public void testQueryUserById(){
   SqlSession session = MybatisUtils.getSession();
   UserMapper mapper = session.getMapper(UserMapper.class);
   User user = mapper.queryUserById(1);
   System.out.println(user);

   session.clearCache();//手动清除缓存
   
   User user2 = mapper.queryUserById(1);
   System.out.println(user2);
   System.out.println(user==user2);
   session.close();
}
```
一级缓存就是一个map

##### 二级缓存
- 二级缓存也叫**全局缓存**，一级缓存作用域太低了，所以诞生了二级缓存
- 基于`namespace`级别的缓存，一个名称空间，对应一个二级缓存；
- 工作机制
    - 一个会话查询一条数据，这个数据就会被放在当前会话的一级缓存中；
    - 如果当前会话关闭了，这个会话对应的一级缓存就没了；但是我们想要的是，会话关闭了，一级缓存中的数据被保存到二级缓存中；
   - 新的会话查询信息，就可以从二级缓存中获取内容；
   - 不同的mapper查出的数据会放在自己对应的缓存（map）中；

**使用步骤**

1、开启全局缓存 【mybatis-config.xml】
```xml
<!--默认开启-->
<setting name="cacheEnabled" value="true"/>
```
2、去每个mapper.xml中配置使用二级缓存，这个配置非常简单；【xxxMapper.xml】
```xml
<cache/>

官方示例=====>查看官方文档
<cache
 eviction="FIFO"
 flushInterval="60000"
 size="512"
 readOnly="true"/>
 ```
这个更高级的配置创建了一个 FIFO 缓存，每隔 60 秒刷新，最多可以存储结果对象或列表的 512 个引用，而且返回的对象被认为是只读的，因此对它们进行修改可能会在不同线程中的调用者产生冲突。
3、代码测试
所有的实体类先实现序列化接口
```java
@Test
public void testQueryUserById(){
   SqlSession session = MybatisUtils.getSession();
   SqlSession session2 = MybatisUtils.getSession();

   UserMapper mapper = session.getMapper(UserMapper.class);
   UserMapper mapper2 = session2.getMapper(UserMapper.class);

   User user = mapper.queryUserById(1);
   System.out.println(user);
   session.close();

   User user2 = mapper2.queryUserById(1);
   System.out.println(user2);
   System.out.println(user==user2);

   session2.close();
}
```
结论
- 只要开启了二级缓存，我们在同一个Mapper中的查询，可以在二级缓存中拿到数据
- 查出的数据都会被默认先放在一级缓存中
- 只有会话提交或者关闭以后，一级缓存中的数据才会转到二级缓存中

缓存原理图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200802224804491.png#pic_center)
# 九、逆向工程
简单的理解，MyBatis逆向工程，就是通过相应插件，**自动生成**MyBatis数据库连接的一些文件。

mybatis需要编写sql语句，mybatis官方提供逆向工程，可以针对单表自动生成mybatis执行所需要的代码（mapper.java、mapper.xml、pojo…），提高工作效率。

# 十、高频点
- MySQL存储引擎
- InnoDB底层
- 索引及优化
- 事务
- TODO
# 参考
[mybatis文档](https://download.csdn.net/download/qq_31821675/12681999)
[B站狂神说](https://me.csdn.net/qq_33369905)

