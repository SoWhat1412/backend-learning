>高清思维导图已同步Git：https://github.com/SoWhat1412/xmindfile，关注公众号sowhat1412获取海量资源
# MyBatisPlus概述
需要的基础:[MyBatis](https://sowhat.blog.csdn.net/article/details/107706620)、[Spring](https://blog.csdn.net/qq_31821675/category_9600850.html)、[SpringMVC](https://blog.csdn.net/qq_31821675/category_9677561.html)就可以学习这个了! 为什么要学习它呢？**MyBatisPlus可以节省我们大量工作时间**，所有的CRUD代码它都可以自动化完成! 
JPA 、 [tk-mapper](https://www.cnblogs.com/wz2cool/p/7286377.html)、MyBatisPlus，偷懒用的!
### 简介
是什么? MyBatis 本来就是简化 JDBC 操作的! 官网:[https://baomidou.com/](https://baomidou.com/)，简化 MyBatis !
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830114130201.png?#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020083011420223.png?#pic_center)
### 特性
[官方描述](https://baomidou.com/guide/#%E7%89%B9%E6%80%A7)
- 无侵入：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- 损耗小：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- 强大的 CRUD 操作：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- 支持 Lambda 形式调用：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- 支持主键自动生成：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- 支持 ActiveRecord 模式：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- 支持自定义全局通用操作：支持全局通用方法注入（ Write once, use anywhere ）
- 内置代码生成器：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- 内置分页插件：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- 分页插件支持多种数据库：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
- 内置性能分析插件：可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- 内置全局拦截插件：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作

# 快速入门
地址:[https://baomidou.com/guide/quick-start.html](https://baomidou.com/guide/quick-start.html)  
使用第三方组件套路:
>1、导入对应的依赖
2、研究依赖如何配置
3、代码如何编写 
4、提高扩展技术能力!
### 步骤
1、创建数据库 `mybatis_plus`
2、创建user表
 ```sql
DROP TABLE IF EXISTS user;
CREATE TABLE user
(
id BIGINT(20) NOT NULL COMMENT '主键ID',
name VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名', age INT(11) NULL DEFAULT NULL COMMENT '年龄',
email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱', PRIMARY KEY (id)
);
INSERT INTO user (id, name, age, email) VALUES
(1, 'Jone', 18, 'test1@baomidou.com'),
(2, 'Jack', 20, 'test2@baomidou.com'),
(3, 'Tom', 28, 'test3@baomidou.com'),
(4, 'Sandy', 21, 'test4@baomidou.com'),
(5, 'Billie', 24, 'test5@baomidou.com');
-- 真实开发中，version(乐观锁)、deleted(逻辑删除)、gmt_create、gmt_modified
```
3、编写项目，初始化项目!使用SpringBoot初始化!
4、导入依赖
```xml
 <!-- 数据库驱动 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!-- lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <!-- mybatis-plus -->
        <!-- mybatis-plus 是自己开发，并非官方的！ -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.0.5</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```
`说明`：我们使用 mybatis-plus 可以节省我们大量的代码，**尽量不要同时导入** mybatis 和 mybatis- plus!版本的差异!
5、连接数据库!这一步和 mybatis 相同!
```properties
# 数据库连接配置
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis?useSSL=false&useUnicode=true&characterEncoding=utf-8&serverTimezone=GMT%2B8
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```
6、==传统方式pojo-dao(连接mybatis，配置mapper.xml文件)-service-controller==
6、使用了mybatis-plus 之后
- pojo
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```
- mapper接口
```java
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.kuang.pojo.User;
import org.springframework.stereotype.Repository;

// 在对应的Mapper上面继承基本的类 BaseMapper
@Repository // 代表持久层
public interface UserMapper extends BaseMapper<User> {
    // 所有的CRUD操作都已经编写完成了
    // 你不需要像以前的配置一大堆文件了！
}
```
- 测试类中测试
```java
    @Test
    void contextLoads() {
        // 参数是一个 Wrapper ，条件构造器，这里我们先不用 null
        // 查询全部用户
        List<User> users = userMapper.selectList(null);
        users.forEach(System.out::println);
    }
```
- 结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830121049441.png?#pic_center)
- xml配置都没有配置！BaseMapper提供了常见的若干方法，Wrapper是一个条件构造器。

#### 思考问题?
>1、SQL谁帮我们写的 ? MyBatis-Plus 都写好了 
2、方法哪里来的? MyBatis-Plus 都写好了

### 配置日志
我们所有的sql现在是不可见的，我们希望知道它是怎么执行的，所以我们必须要看日志!
```java
# 配置日志
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830121810870.png?#pic_center)
# CRUD扩展 
### 插入操作
Insert 插入
```java
    // 测试插入
    @Test
    public void testInsert(){
        User user = new User();
        user.setName("sowhat learn Java");
        user.setAge(18);
        user.setEmail("363421@qq.com");

        int result = userMapper.insert(user); // 帮我们自动生成id
        System.out.println(result); // 受影响的行数
        System.out.println(user); // 发现，id会自动回填
    }
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830122223459.png?#pic_center)
数据库插入的id的默认值为:全局的唯一id
##### 主键生成策略
>默认 ID_WORKER 全局唯一id

分布式系统唯一id生成:[9种分布式ID生成方式，总有一款适合你](https://sowhat.blog.csdn.net/article/details/107636591)
雪花算法:
snowflake是Twitter开源的分布式ID生成算法，结果是一个long型的ID。其核心思想是:使用41bit作为 毫秒数，10bit作为机器的ID(5个bit是数据中心，5个bit的机器ID)，12bit作为毫秒内的流水号(意味 着每个节点在每毫秒可以产生 4096 个 ID)，最后还有一个符号位，永远是0。可以保证几乎全球唯 一!

>主键自增
 
我们需要配置**主键自增**:
1、实体类字段上 
```java
      // 对应数据库中的主键 (uuid、自增id、雪花算法、redis、zookeeper！)
    @TableId(type = IdType.AUTO)
    private Long id;
```
2、数据库字段一定要是自增!
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830122809265.png?#pic_center)
再次运行：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830123218623.png?#pic_center)
##### 其他解释
```java
public enum IdType {
AUTO(0), // 数据库id自增
NONE(1), // 未设置主键
INPUT(2), // 手动输入
ID_WORKER(3), // 默认的全局唯一id
UUID(4), // 全局唯一id uuid
ID_WORKER_STR(5); //ID_WORKER 字符串表示法
}
```
### 更新操作
所有的sql都是自动帮你动态配置的!
```java
    // 测试更新
    @Test
    public void testUpdate(){
        User user = new User();
        // 通过条件自动拼接动态sql
        user.setId(2L);
        user.setName("关注公众号：sowhat");
        user.setAge(20);
        // 注意：updateById 但是参数是一个对象！
        int i = userMapper.updateById(user);
        System.out.println(i);
    }
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830124013487.png?#pic_center)
##### 自动填充
创建时间、修改时间!这些个操作一遍都是自动化完成的，我们不希望手动更新!
阿里巴巴开发手册:所有的数据库表:gmt_create、gmt_modified几乎所有的表都要配置上!而且需 要自动化!
**方式一:数据库级别(工作中不允许你修改数据库)**
1、在表中新增字段 create_time, update_time
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830125548207.png?#pic_center)
2、再次测试插入方法，我们需要先把实体类同步!
```java
private Date createTime;
private Date updateTime;
```
3、再次更新查看结果即可
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830130036344.png?#pic_center)
**方式二:代码级别**
1、删除数据库的默认值、更新操作!
2、实体类字段属性上需要增加注解
```java
    @TableField(fill = FieldFill.INSERT)
    private Date createTime;
    //
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date updateTime;
```
3、编写处理器来处理这个注解即可! 系统会自动查询那些字段添加了这样的注解然后将时间插入。
```java
@Slf4j
@Component // 一定不要忘记把处理器加到IOC容器中！
public class MyMetaObjectHandler implements MetaObjectHandler {
    // 插入时的填充策略
    @Override
    public void insertFill(MetaObject metaObject) {
        log.info("start insert fill.....");
        // setFieldValByName(String fieldName, Object fieldVal, MetaObject metaObject
        this.setFieldValByName("createTime",new Date(),metaObject);
        this.setFieldValByName("updateTime",new Date(),metaObject);
    }

    // 更新时的填充策略
    @Override
    public void updateFill(MetaObject metaObject) {
        log.info("start update fill.....");
        this.setFieldValByName("updateTime",new Date(),metaObject);
    }
}
```
4、测试插入
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830131716261.png?#pic_center)

5、测试更新、观察时间即可!![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830131808850.png?#pic_center)
**PS** : Mybatis-plus 会自动将Java中的驼峰命名变更为下划线命名比如 createTime ==> create_time，如果像关闭需要在yml中添加
```java
mybatis-plus.configuration.map-underscore-to-camel-case=false
```
6、只更新若干字段的时候指定即可。比如我把name屏蔽掉
```java
    // 测试更新
    @Test
    public void testUpdate(){
        User user = new User();
        // 通过条件自动拼接动态sql
        user.setId(6L);
        //user.setName("sowhat");
        user.setAge(22);
        // 注意：updateById 但是参数是一个 对象！
        int i = userMapper.updateById(user);
        System.out.println(i);
    }
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830132710566.png?#pic_center)
### 乐观锁
在面试过程中，我们经常会被问道乐观锁，悲观锁!这个其实非常简单!
> 乐观锁 : 故名思意十分乐观，它总是认为不会出现问题，无论干什么不去上锁!如果出现了问题， 再次更新值测试
悲观锁:故名思意十分悲观，它总是认为总是出现问题，无论干什么都会上锁!再去操作!

乐观锁实现方式:
- 取出记录时，获取当前 version
- 更新时，带上这个version
- 执行更新时， set version = newVersion where version = oldVersion 
- 如果version不对，就更新失败
```sql
乐观锁:1、先查询，获得版本号 version = 1
-- A
update user set name = "sowhat", version = version + 1 where id = 2 and version = 1
-- B 线程抢先完成，这个时候 version = 2，会导致 A 修改失败! 
update user set name = "sowhat", version = version + 1 where id = 2 and version = 1
```
##### 测试一下MyBatis Plus 的乐观锁插件
1、给数据库中增加version字段!
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830183827626.png?#pic_center)
2、我们实体类加对应的字段
```java
 @Version //乐观锁Version注解 
 private Integer version;
```
3、注册组件
```java
// 扫描我们的 mapper 文件夹
@MapperScan("com.sowhat.mapper")
@EnableTransactionManagement
@Configuration // 配置类
public class MyBatisPlusConfig {

    // 注册乐观锁插件
    @Bean
    public OptimisticLockerInterceptor optimisticLockerInterceptor() {
        return new OptimisticLockerInterceptor();
    }
}
```
4、测试一下!
```java
    // 测试乐观锁成功！
    @Test
    public void testOptimisticLocker(){
        // 1、查询用户信息
        User user = userMapper.selectById(1L);
        // 2、修改用户信息
        user.setName("sowhat");
        user.setEmail("353421@qq.com");
        // 3、执行更新操作
        userMapper.updateById(user);
    }


    // 测试乐观锁失败！多线程下
    @Test
    public void testOptimisticLocker2(){

        // 线程 1
        User user = userMapper.selectById(1L);
        user.setName("sowhat1");
        user.setEmail("363422@qq.com");

        // 模拟另外一个线程执行了插队操作
        User user2 = userMapper.selectById(1L);
        user2.setName("sowhat2 ");
        user2.setEmail("24736743@qq.com");
        userMapper.updateById(user2);

        // 自旋锁来多次尝试提交！
        userMapper.updateById(user); // 如果没有乐观锁就会覆盖插队线程的值！
    }
```
成功场景
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830184800230.png?#pic_center)
失败场景：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830185442185.png?#pic_center)
### 查询操作
```java
    // 测试查询
    @Test
    public void testSelectById(){
        User user = userMapper.selectById(1L);
        System.out.println(user);
    }

    // 测试批量查询！ where  id in (1,2,3) 这样的操作
    @Test
    public void testSelectByBatchId(){
        List<User> users = userMapper.selectBatchIds(Arrays.asList(1, 2, 3));
        users.forEach(System.out::println);
    }
    
    // 按条件查询之一 使用map操作 name = 'sowhat' and age = 22
    @Test
    public void testSelectByBatchIds(){
        HashMap<String, Object> map = new HashMap<>();
        // 自定义要查询
        map.put("name","sowhat");
        map.put("age",22);
        List<User> users = userMapper.selectByMap(map);
        users.forEach(System.out::println);
    }
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830190133937.png#pic_center)
### 分页查询
分页在网站使用的十分之多!
>1、原始的 limit 进行分页 
2、pageHelper 第三方插件 
3、MP 其实也内置了分页插件!

**如何使用!**
1、配置拦截器组件即可 MyBatisPlusConfig
```java
   // 分页插件
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return  new PaginationInterceptor();
    }
```
**2、直接使用Page对象即可!**
```java
    @Test
    public void testPage(){
        //  参数一：当前页
        //  参数二：页面大小
        //  使用了分页插件之后，所有的分页操作也变得简单的！ 3~4
        Page<User> page = new Page<>(2,2);
        userMapper.selectPage(page,null);

        page.getRecords().forEach(System.out::println);
        System.out.println(page.getTotal()); // 等价 SELECT COUNT(1) FROM user
    }
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830190642453.png?#pic_center)
### 删除操作
1、根据 id 删除记录
```java
    // 测试删除
    @Test
    public void testDeleteById(){
        userMapper.deleteById(1L);
    }

    // 通过id批量删除 id in (2,3)
    @Test
    public void testDeleteBatchId(){
        userMapper.deleteBatchIds(Arrays.asList(2L,3L));
    }

    // 通过map删除
    @Test
    public void testDeleteMap(){
        HashMap<String, Object> map = new HashMap<>();
        map.put("name","Sandy");
        userMapper.deleteByMap(map);
    }
```
在工作中会遇到一些问题:**逻辑删除**!

### 逻辑删除
> 物理删除 :从数据库中直接移除
逻辑删除 :再数据库中没有被移除，而是通过一个变量来让他失效! deleted = 0 => deleted = 1,类似与跟记录一个flag标识位

1、在数据表中增加一个 deleted 字段![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830191505779.png?#pic_center)
2、实体类中增加属性
```java
 @TableLogic //逻辑删除 
 private Integer deleted;
```
3、配置!
```java
    // 逻辑删除组件！
    @Bean
    public ISqlInjector sqlInjector() {
        return new LogicSqlInjector();
    }
```
application.properties
```xml
# 配置逻辑删除
mybatis-plus.global-config.db-config.logic-delete-value=1
mybatis-plus.global-config.db-config.logic-not-delete-value=0
```
4、测试
```java
    // 测试删除
    @Test
    public void testDeleteById(){
        userMapper.deleteById(1L);
    }
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830191835618.png?#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830191945403.png?#pic_center)
**以上的所有CRUD操作及其扩展操作，我们都必须精通掌握!会大大提高你的工作和写项目的效率!**

# 扩展
### 性能分析插件
我们在平时的开发中，会遇到一些慢sql。如何发现可以通过手动测试!或者druid等等。MP也提供了此种类型插件。
作用:性能分析拦截器，**用于输出每条 SQL 语句及其执行时间** MP也提供性能分析插件，如果超过这个时间就停止运行!

1、运行环境设置为开发
```java
# 设置开发环境
spring.profiles.active=dev
```
2、配置开启
```java
     * SQL执行效率插件
     */
    @Bean
    @Profile({"dev","test"})// 设置 dev test 环境开启，保证我们的效率
    public PerformanceInterceptor performanceInterceptor() {
        PerformanceInterceptor performanceInterceptor = new PerformanceInterceptor();
        //ms 设置sql执行的最大时间，如果超过了则不执行
        performanceInterceptor.setMaxTime(20); 
        // 开启格式化
        performanceInterceptor.setFormat(true); 
        return performanceInterceptor;
    }
```
故意设置小一些可以看到如下输出：
```java
    // 测试查询
    @Test
    public void testSelectById(){
        User user = userMapper.selectById(1L);
        System.out.println(user);
    }
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830192936368.png?#pic_center)
### 条件构造器
[条件构造器](https://baomidou.com/guide/wrapper.html#%E6%96%B9%E6%A1%88%E4%BA%8C-xml%E5%BD%A2%E5%BC%8F-mapper-xml) **十分重要**：Wrapper 我们写一些复杂的sql就可以使用它来替代!
说白了就是练习使用`QueryWrapper`下面到各种函数使用。
```java
package com.sowhat;

import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.sowhat.mapper.UserMapper;
import com.sowhat.pojo.User;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;
import java.util.Map;

@SpringBootTest
public class WrapperTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    void contextLoads() {
        // 查询name不为空的用户，并且邮箱不为空的用户，年龄 >= 12
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper
                .isNotNull("name")
                .isNotNull("email")
                .ge("age", 12);
        userMapper.selectList(wrapper).forEach(System.out::println); // 和我们刚才学习的map对比一下
    }

    @Test
    void test2() {
        // 查询名字 sowhat
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.eq("name", "sowhat");
        //User user = userMapper.selectOne(wrapper); // 查询一个数据，出现多个结果使用List 或者 Map
        //System.out.println(user);
        userMapper.selectList(wrapper).forEach(System.out::println);
    }

    @Test
    void test3() {
        // 查询年龄在 20 ~ 30 岁之间的用户
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.between("age", 20, 30); // 区间
        Integer count = userMapper.selectCount(wrapper);// 查询结果数
        System.out.println(count);
    }

    // 模糊查询
    @Test
    void test4() {
        // 名字不包含e，
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        // likeRight  t%
        // likeLeft %t
        wrapper
                .notLike("name", "e")
                .likeRight("email", "t");
        List<Map<String, Object>> maps = userMapper.selectMaps(wrapper);
        maps.forEach(System.out::println);
    }

    // 内查询
    @Test
    void test5() {

        QueryWrapper<User> wrapper = new QueryWrapper<>();
        // id 在子查询中查出来 ID in ()
        wrapper.inSql("id", "select id from user where id<3");

        List<Object> objects = userMapper.selectObjs(wrapper); // 查询到一切对象
        objects.forEach(System.out::println);
    }

    // 排序
    @Test
    void test6() {
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        // 通过id进行排序
        wrapper.orderByAsc("id");
        List<User> users = userMapper.selectList(wrapper);
        users.forEach(System.out::println);
    }
}
```
核心：根据官网的[条件构造器](https://baomidou.com/guide/wrapper.html#%E6%96%B9%E6%A1%88%E4%BA%8C-xml%E5%BD%A2%E5%BC%8F-mapper-xml)  实验尝试函数，每次尝试后注意 最终执行的SQL是什么样的！

### 代码自动生成器
参考官网[自动生成器](https://baomidou.com/guide/generator.html#%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B)开写即可。
```java
package com.sowhat;


import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.annotation.FieldFill;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.config.DataSourceConfig;
import com.baomidou.mybatisplus.generator.config.GlobalConfig;
import com.baomidou.mybatisplus.generator.config.PackageConfig;
import com.baomidou.mybatisplus.generator.config.StrategyConfig;
import com.baomidou.mybatisplus.generator.config.po.TableFill;
import com.baomidou.mybatisplus.generator.config.rules.DateType;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;

import java.util.ArrayList;

/**
 * 代码自动生成器
 */
public class SowhatCode {
    public static void main(String[] args) {
        // 需要构建一个 代码自动生成器 对象
        AutoGenerator mpg = new AutoGenerator();
        // 配置策略
// 1、全局配置
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir"); // 获得用户当前目录
        gc.setOutputDir(projectPath + "/src/main/java"); //代码生成目录
        gc.setAuthor("sowhat");
        gc.setOpen(false); // 是否打开文件夹 运行后
        gc.setFileOverride(false); // 是否覆盖
        gc.setServiceName("%sService"); // 去Service的I前缀
        gc.setIdType(IdType.ID_WORKER);
        gc.setDateType(DateType.ONLY_DATE);
        gc.setSwagger2(true);
        mpg.setGlobalConfig(gc);
//2、设置数据源
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/mybatis?useSSL = false&useUnicode=true&characterEncoding = utf-8 & serverTimezone=GMT%2B8");
        dsc.setDriverName(" com.mysql.cj.jdbc.Driver");
        dsc.setUsername(" root");
        dsc.setPassword("root");
        dsc.setDbType(DbType.MYSQL);
        mpg.setDataSource(dsc);
//3、包的配置
        PackageConfig pc = new PackageConfig();
        pc.setModuleName("blog");
        pc.setParent("com.kuang");
        pc.setEntity("entity");
        pc.setMapper("mapper");
        pc.setService("service");
        pc.setController("controller");
        mpg.setPackageInfo(pc);
//4、策略配置
        StrategyConfig strategy = new StrategyConfig();
        strategy.setInclude("blog_tags", "course", "links", "sys_settings", "user_record", " user_say"); // 设置要映射的表名
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        strategy.setEntityLombokModel(true); // 自动lombok;
        strategy.setLogicDeleteFieldName("deleted"); // 逻辑删除
// 自动填充配置
        TableFill gmtCreate = new TableFill("gmt_create", FieldFill.INSERT);
        TableFill gmtModified = new TableFill("gmt_modified", FieldFill.INSERT_UPDATE);
        ArrayList<TableFill> tableFills = new ArrayList<>();
        tableFills.add(gmtCreate);
        tableFills.add(gmtModified);
        strategy.setTableFillList(tableFills);
// 乐观锁
        strategy.setVersionFieldName("version");
        strategy.setRestControllerStyle(true); // 驼峰命名
        strategy.setControllerMappingHyphenStyle(true); // localhost:8080/hello_id_2
        mpg.setStrategy(strategy);

        mpg.execute(); //执行
    }
}
```

# 参考
[狂神说](https://www.bilibili.com/video/BV17E411N7KN?p=1)




