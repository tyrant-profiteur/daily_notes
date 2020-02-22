# 程序设计三层架构

![三层架构](C:\Users\tyrant\Desktop\框架\mybatis\day01\01三层架构.png)

# mybatis

## 1.jdbc回顾

1.加载驱动	class.forname

2.获取连接	connection conn = DriverManager.getConnection

3.定义sql	 string sql = "?,?"

4.获取预处理statement		PreparedStatement pst = conn.preparedStatement(sql);

5.处理预处理参数			pst.setString(1,"");

6.获取结果集  ResultSet rst = pst.executeQuery()

7.释放资源

## 2.mybaits入门案例配置

### 2.1.maven添加坐标

mybatis使用,junit单元测试,mysql-connector-java数据库连接,log4j日志

```xml
<dependencies> 
    <!-- mybaits-->
    <dependency> 
        <groupId>org.mybatis</groupId> 
        <artifactId>mybatis</artifactId> 
        <version>3.4.5</version> 
    </dependency> 
    <!-- junit单元测试-->
    <dependency> 
    <groupId>junit</groupId> 
    <artifactId>junit</artifactId> 
    <version>4.10</version> 
    <scope>test</scope> 
    </dependency> 
    <!-- 数据库连接-->
    <dependency> 
        <groupId>mysql</groupId> 
        <artifactId>mysql-connector-java</artifactId> 
        <version>5.1.6</version> 
        <scope>runtime</scope> 
    </dependency> 
    <!-- 日志-->
    <dependency> 
        <groupId>log4j</groupId> 
        <artifactId>log4j</artifactId> 
        <version>1.2.12</version> 
    </dependency> 
</dependencies>
```

### 2.2.实体类编写

```java
public class User implements Serializable{}
```

所有实体类都需要实现serializable序列化接口,setter/getter方法,toString方法

### 2.3.持久层接口编写

```java
public interface IUserDao {
    返回值 方法名();
    List<User> findAll();
}
```

### 2.4.持久层接口xml映射文件

![1565836511241](C:\Users\tyrant\AppData\Roaming\Typora\typora-user-images\1565836511241.png)

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd"> 
<mapper namespace="com.itheima.dao.IUserDao"> 
    <!-- 配置查询所有操作 --> 
    <select id="findAll" resultType="com.itheima.domain.User"> 
        select * from user 	  		
    </select>
</mapper>
```

namespace:所需要映射的持久层接口全限定类名

id:接口中的方法名

resultType:返回值的类型,自定义的引用类型写全限定类名,可以使用[typeAliases](#8.2.typeAliases(类别名))更改类别名

### 2.5.SqlMapConfig.xml配置

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd"> 
<configuration> 
    <!-- 配置mybatis的环境 --> 
    <environments default="mysql"> 
        <!-- 配置mysql的环境 --> 
        <environment id="mysql"> 
            <!-- 配置事务的类型 --> 
            <transactionManager type="JDBC"></transactionManager> 
            <!-- 配置连接数据库的信息：用的是数据源(连接池) --> 
            <dataSource type="POOLED"> 
                <property name="driver" value="com.mysql.jdbc.Driver"/> 
                <property name="url" value="jdbc:mysql://localhost:3306/ee50"/> 
                <property name="username" value="root"/> 
                <property name="password" value="1234"/> 
            </dataSource> 
        </environment> 
    </environments> 
    <!-- 告知mybatis映射配置的位置 --> 
    <mappers> 
        <mapper resource="com/itheima/dao/IUserDao.xml"/> 
    	<mapper class="com.itheima.dao.IUserDao"/>
    </mappers> 
</configuration>
```

映射文件配置两种方式:resource写xml路径,class写接口类路径

注解配置只能用class配置接口类路径

### 2.6.测试类

```java
//1.读取配置文件
InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
//2.创建sqlSessionFactory工厂对象
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
//3.使用工厂创建SqlSession对象
SqlSession session = factory.openSession();
//4.使用Session创建dao接口的代理对象
IUserDao userDao = session.getMapper(IUserDao.class);
//5.使用代理对象执行方法
List<User> users = userDao.findAll();
//6.释放资源
session.close();
in.close();
```

- 1.流对象读取配置文件2.使用工厂创建SqlSession对象3.从session域中获取mapper(dao)接口的代理对象4.执行方法

![](C:\Users\tyrant\Desktop\框架\mybatis\day01\入门案例的分析.png)

[^]: 在使用基于注解的Mybatis配置时，请移除xml的映射配置（IUserDao.xml）。

## 3.自定义mybatis

### 3.1.涉及的内容

工厂模式（Factory工厂模式）、构造者模式（Builder模式）、代理模式，反射，自定义注解，注解的反射，xml解析，数据库元数据，元数据的反射等

![](C:\Users\tyrant\Desktop\框架\mybatis\day01\查询所有的分析.png)

### 3.2.自定义时的jar包

```xml
<!-- 解析xml的dom4j --> 
<dependency> 
    <groupId>dom4j</groupId> 
    <artifactId>dom4j</artifactId> 
    <version>1.6.1</version> 
</dependency>
<!-- dom4j的依赖包jaxen --> 
<dependency> 
    <groupId>jaxen</groupId> 
    <artifactId>jaxen</artifactId> 
    <version>1.1.6</version> 
</dependency>
```

### 3.3.自定义的实现

[12-30]: C:\Users\tyrant\Desktop\框架\mybatis\Mybatis第一天讲义.pdf

## 4.mybatis执行流程

![](C:\Users\tyrant\Desktop\框架\mybatis\day02\自定义mybatis开发流程图.png)

## 5.代理Dao实现CRUD操作

- session.commit();

### 5.1.根据Id查询

1.持久层添加方法

```java
User findById(Integer userId);
```

2.映射配置文件配置

```xml
<!-- 根据id查询 --> 
<select id="findById" resultType="com.itheima.domain.User" parameterType="int"> 
	select * from user where id = #{uid} 
</select>
```

resultType属性：用于指定结果集的类型。 

parameterType属性：用于指定传入参数的类型。 

sql语句中使用#{}字符： 它代表占位符，相当于原来jdbc部分所学的?，都是用于执行语句时替换实际的数据。 具体的数据是由#{}里面的内容决定的。 

#{}中内容的写法： 由于数据类型是基本类型，所以此处可以随意写。

### 5.2.保存操作

1.添加用户

```java
int saveUser(User user)
```

2.映射配置文件配置

```xml
<insert insert id="saveUser" parameterType="com.itheima.domain.User">
	insert into user(username,birthday,sex,address)
	values(#{username},#{birthday},#{sex},#{address})
</insert>
```

parameterType属性： 代表参数的类型，因为我们要传入的是一个类的对象，所以类型就写类的全名称。 

sql语句中使用#{}字符： 它代表占位符，相当于原来jdbc部分所学的?，都是用于执行语句时替换实际的数据。 具体的数据是由#{}里面的内容决定的。 

#{}中内容的写法： 由于我们保存方法的参数是 一个User对象，此处要写User对象中的属性名称。 它用的是ognl表达式。 

- ognl表达式： 它是apache提供的一种表达式语言，全称是：

   Object Graphic Navigation Language 对象图导航语言

   它是按照一定的语法格式来获取数据的。 

  语法格式就是使用 #{对象.对象}的方式

- #{user.username}它会先去找user对象，然后在user对象中找到username属性，并调用getUsername()方法把值取出来。但是我们在parameterType属性上指定了实体类名称，所以可以省略user.而直接写username。

### 5.3.用户更新

1.更新方法

```java
int updateUser(User user);
```

2.配置文件中的配置

```xml
<!-- 更新用户 --> 
<update id="updateUser" parameterType="com.itheima.domain.User"> 
    update user set username=#{username},birthday=#{birthday},sex=#{sex}, address=#{address} where id=#{id} 
</update>
```

### 5.4.用户删除

1.删除方法

```java
int deleteUser(Integer userId);
```

2.配置文件中的配置

```
<!-- 删除用户 --> 
<delete id="deleteUser" parameterType="java.lang.Integer"> delete from user where id = #{uid} 
</delete>
```

### 5.5.用户模糊查询

1.模糊查询

```java
List<User> findByName(String username);
```

#### 5.5.1.方法一

2.配置文件中的配置

```xml
<!-- 根据名称模糊查询 --> 
<select id="findByName" resultType="com.itheima.domain.User" parameterType="String"> select * from user where username like #{username} 
</select>
```

3.查询语句

```java
List<User> users = userDao.findByName("%王%");
```

- 底层执行的sql语句:

select * from user where username like ?

parameters:"%王%

#### 5.5.2.方法二

2.配置文件中的配置

```xml
<!-- 根据名称模糊查询 --> 
<select id="findByName" parameterType="string" resultType="com.itheima.domain.User"> 
    select * from user where username like '%${value}%' 
</select>
```

3.查询语句

```java
List<User> users = userDao.findByName("王");
```

- 底层执行的sql语句:

select * from user where username like '%王%'

### 5.6.#{}和${}的区别

#### #{}表示一个占位符号 

​	通过#{}可以实现preparedStatement向占位符中设置值，自动进行java类型和jdbc类型转换，#{}可以有效防止sql注入。 #{}可以接收简单类型值或pojo属性值。 如果parameterType传输单个简单类型值，#{}括号中可以是value或其它名称。 

#### ${}表示拼接sql串 

​	通过${}可以将parameterType 传入的内容拼接在sql中且不进行jdbc类型转换， ${}可以接收简单类型值或pojo属性值，如果parameterType传输单个简单类型值，${}括号中只能是value。

## 6.参数深入

### 6.1.parameterType

基本类型,引用类型,实力类类型(pojo类)

- 基本类型和String我们可以直接写类型名称，也可以使用包名.类名的方式

- 实体类类型，目前我们只能使用全限定类名。

6.1.传递pojo包装对象

如果A中有a,通过A,查询出在A中的a

1.QueryVo中有user对象(set/get)

2.IUserDao中有List<User> findUserByVo(QueryVo vo)方法

3.映射文件  

```xml
<!-- 根据用户名称模糊查询，参数变成一个QueryVo对象了 --> 
<select id="findByVo" resultType="com.itheima.domain.User" parameterType="com.itheima.domain.QueryVo"> select * from user where username like #{user.username}; 
</select>
```

- 传入的参数为QueryVo,参数应该为vo.getUser.getUsername,mybatis底层反射,占位符为#{user.username}

4.执行方法

```java
 List<User> users = userDao.findByVo(vo);
```

### 6.2.resultType

resultType属性可以指定结果集的类

#### 6.2.1.特殊情况

- 实体类属性和数据库表的列名不一致,数据库的结果集封装不到实体类中

- 解决方案

1. sql语句修改,起别名,使得数据库列名和实体类属性名一样

2. resultMap(推荐)

   type属性：指定实体类的全限定类名 

   id属性：给定一个唯一标识，是给查询select标签引用用的。

   - ```xml
     <resultMap type="com.itheima.domain.User" id="userMap"> 
         <id column="id" property="userId"/> 
         <result column="username" property="userName"/> 
         <result column="sex" property="userSex"/> 
         <result column="address" property="userAddress"/> 
         <result column="birthday" property="userBirthday"/> 
     </resultMap>
     ```

     id标签：用于指定主键字段 

     result标签：用于指定非主键字段 

     column属性：用于指定数据库列名 

     property属性：用于指定实体类属性名称

   - 此时的结果集不再是resultType,而实resultType

     ```xml
     <!-- 配置查询所有操作 --> 
     <select id="findAll" resultMap="userMap"> 
         select * from user 
     </select>
     ```

## 7.传统Dao层开发

[19-25]: C:\Users\tyrant\Desktop\框架\mybatis\Mybatis第二天讲义.pdf

## 8.SqlSessionConfig.xml配置详解

### 8.1.properties(属性)

1. 方法一

   ```xml
   <properties> 
   	<property name="jdbc.driver" value="com.mysql.jdbc.Driver"/> 
   	<property name="jdbc.url" value="jdbc:mysql://localhost:3306/eesy"/>
   	<property name="jdbc.username" value="root"/> 
       <property name="jdbc.password" value="1234"/> 
   </properties>
   ```

2. 方法二

   - 在classpath下定义db.properties文件

   - properties标签配置

     常用:

     ```xml
     <properties resource="jdbcConfig.properties"> </properties>
     ```

     了解:

     ```xml
     <properties url= file:///D:/IdeaProjects/day02_eesy_01mybatisCRUD/src/main/resources/jdbcConfig.properties"> </properties>
     ```

   - dataSource标签引用配置

     ```xml
     <dataSource type="POOLED"> 
     	<property name="driver" value="${jdbc.driver}"/> 
         <property name="url" value="${jdbc.url}"/> 
         <property name="username" value="${jdbc.username}"/> 
         <property name="password" value="${jdbc.password}"/> 
     </dataSource>
     ```

### 8.2.typeAliases(类别名)

在SqlMapConfig.xml中配置：

```xml
 <typeAliases> 
     <!-- 单个别名定义 --> 
     <typeAlias alias="user" type="com.itheima.domain.User"/> 
     <!-- 批量别名定义，扫描整个包下的类，别名为类名（首字母大写或小写都可以） --> 
     <package name="com.itheima.domain"/> 
     <package name="其它包"/> 
</typeAliases>
```

### 8.3.mappers(映射器)

1. <mapper resource=" " />

   使用相对于类路径的资源 如：

   ```
   <mapper resource="com/itheima/dao/IUserDao.xml" />
   ```

2. <mapper class=" " />

   使用mapper接口类路径 如：

   ```
   <mapper class="com.itheima.dao.UserDao"/> 
   ```

   注意：此种方法要求mapper接口名称和mapper映射文件名称相同，且放在同一个目录中。

3. <package name=""/>  常用

   注册指定包下的所有mapper接口 如：

   ```
   <package name="cn.itcast.mybatis.mapper"/> 
   ```

   注意：此种方法要求mapper接口名称和mapper映射文件名称相同，且放在同一个目录中。

## 9.动态SQL

### 9.1.if标签

```xml
<select id="findByUser" resultType="user" parameterType="user"> 
    select * from user where 1=1 
    <if test="username!=null and username != '' "> and username like #{username} </if> 
    <if test="address != null"> and address like #{address} </if> 
</select> 
```

注意：<if>标签的test属性中写的是对象的属性名，如果是包装类的对象要使用OGNL表达式的写法。 另外要注意where 1=1 的作用~！

### 9.2.where标签

```xml
<select id="findByUser" resultType="user" parameterType="user"> 
    <include refid="defaultSql"></include> 
    <where> 
        <if test="username!=null and username != '' "> 
            and username like #{username} 
        </if> 
        <if test="address != null"> 
            and address like #{address} 
        </if> 
    </where> 
</select>
```

<include>:包含重复的sql语句,例如  select * from user

​						refid属性:<sql>标签的id

```xml
<!-- 抽取重复的语句代码片段 --> 
<sql id="defaultSql"> 
    select * from user 
</sql>
```

### 9.3.foreach标签

1.在QueryVo中加入一个List集合用于封装参数

2.持久层Dao接口:List<User> findInIds(QueryVo vo);

3.Dao映射配置

```xml
<select id="findInIds" resultType="user" parameterType="queryvo"> 
    <!-- select * from user where id in (1,2,3,4,5); --> 
    <include refid="defaultSql"></include> 
    <where> 
        <if test="ids != null and ids.size() > 0"> 
            <foreach collection="ids" open="id in ( " close=")" item="uid" separator=","> #{uid} </foreach> 
        </if> 
    </where> 
</select>
```

SQL语句： 

select 字段from user where id in (?) 

<foreach>标签用于遍历集合，它的属性： 

- collection:代表要遍历的集合元素，注意编写时不要写#{} 

- open:代表语句的开始部分 

- close:代表结束部分
- item:代表遍历集合的每个元素，生成的变量名 
- sperator:代表分隔符

## 10.多表查询

### 10.1.一对一

1. 方法一

   定义一个多表的实体类A

   持久层Dao接口List<A> findAll();

   xml配置文件resultType=A

2. 方法二

   主表的实体类A添加从表实体类B

   持久层Dao接口List<A> findAll();

   ```xml
   <!-- 建立对应关系 --> 
   <resultMap type="account" id="C"> 
       <id column="aid" property="id"/> 
       <result column="uid" property="uid"/>  
       <!-- 它是用于指定从表方的引用实体属性的 --> 
       <association property="user" javaType="user"> 
           <id column="id" property="id"/> 
           <result column="username" property="username"/> 
       </association> 
   </resultMap>
   ```

   <association>标签是定从表的引用实体属性,实体类型属性为javaType,不是type

   xml配置文件resultMap=C

### 10.2.一对多

主表实体类中添加从表实体类集合

xml配置

```xml
<resultMap type="user" id="userMap"> 
    <id column="id" property="id"></id> 
    <result column="username" property="username"/> 
    <!-- collection是用于建立一对多中集合属性的对应关系 ofType用于指定集合元素的数据类型 -->
    <collection property="accounts" ofType="account"> 
        <id column="aid" property="id"/> 
        <result column="uid" property="uid"/> 
    </collection> 
</resultMap>
```

collection 

​	部分定义了用户关联的账户信息。表示关联查询结果集 

​	property="accList"： 关联查询的结果集存储在User对象的上哪个属性。 

​	ofType="account"： 指定关联查询的结果集中的对象类型即List中的对象类型。此处可以使用别名，也可以使用全限定名。

### 10.3.多对多

两个一对多

## 11.延迟加载

[1-9]: C:\Users\tyrant\Desktop\框架\mybatis\Mybatis第四天讲义.pdf

## 12.缓存

### 12.1.一级缓存

属于SqlSession对象

sqlSession.clearCache();//此方法也可以清空缓存

### 12.2.二级缓存

1.SqlMapConfig.xml文件开启二级缓存:默认开启,可以不写

```xml
<settings> 
<!-- 开启二级缓存的支持 --> 
<setting name="cacheEnabled" value="true"/> 
</settings>
```

2.配置相关的Mapper映射文件

<mapper>中添加<!-- 开启二级缓存的支持 --> <cache></cache>

3.配置statement上面的useCache属性

```xml
<select id="findById" resultType="user" parameterType="int" useCache="true">
select * from user where id = #{uid} 
</select>
```

将UserDao.xml映射文件中的<select>标签中设置useCache=”true”代表当前这个statement要使用二级缓存

![](C:\Users\tyrant\Desktop\框架\mybatis\day04\二级缓存.png)

# Mybatis注解开发

- @Insert:实现新增 
- @Update:实现更新
- @Delete:实现删除 
- @Select:实现查询 
- @Result:实现结果集封装 
- @Results:可以与@Result一起使用，封装多个结果集 
- @ResultMap:实现引用
- @Results定义的封装 
- @One:实现一对一结果集封装 
- @Many:实现一对多结果集封装 
- @SelectProvider: 实现动态SQL映射 
- @CacheNamespace:实现注解二级缓存的使用

### 13.1.基于注解的CRUD

1.编写实体类

2.编写用户持久层接口

​	接口方法上加注解

```java
@Select("select * from user") 
@Results(id="userMap", 
value= { 
@Result(id=true,column="id",property="userId"), @Result(column="username",property="userName"), 
@Result(column="sex",property="userSex"), @Result(column="address",property="userAddress"), @Result(column="birthday",property="userBirthday") 
}) 
List<User> findAll();
```

@results注解和xml配置中的resultMap功能一致

@resultMap可以引用先前定义的@results

```java
@Select("select * from user where id = #{uid} ") 
@ResultMap("userMap") 
User findById(Integer userId);
```

### 13.2.基于注解的复杂关系映射

#### @One注解（一对一）

代替了<assocation>标签

- ​	select 指定用来多表查询的SqlMapper ,是接口方法的全限定类名,不是sql语句

- ​	fetchType会覆盖全局的三个配置参数:lazy   Loading   Enabled

使用格式:@Result(column=" ",property="",one=@One(select=""))

```java
/** * 查询所有账户，采用延迟加载的方式查询账户的所属用户 * @return */ 
@Select("select * from account") 
@Results(id="accountMap", value= { 
    @Result(id=true,column="id",property="id"), 
    @Result(column="uid",property="uid"), 
    @Result(column="money",property="money"), 
    @Result(column="uid", property="user", 
            one=@One(select="com.itheima.dao.IUserDao.findById",
                     fetchType=FetchType.LAZY) ) 
}) 
List<Account> findAll();
```

```java
@Select("select * from user where id = #{uid} ") 
@ResultMap("userMap") 
User findById(Integer userId);
```

主表实体类中添加从表实体类,主表持久层接口注解配置,column关联从表,关联的外键的property对应从表实体类

@one属性的select为从表持久层的方法全限定类名

#### @Many注解（多对一）

代替了<Collection>标签

[^注意]: ：聚集元素用来处理“一对多”的关系。需要指定映射的Java实体类的属性，属性的javaType（一般为ArrayList）但是注解中可以不定义；

使用格式： @Result(property="",column="",many=@Many(select=""))

一对多关系映射：主表方法应该包含一个从表方的集合引用

@Many: 相当于<collection>的配置 

select属性：代表将要执行的sql语句 

fetchType属性：代表加载方式，一般如果要延迟加载都设置为LAZY的值

### 13.3.二级缓存开启

在SqlMapConfig中开启二级缓存支持

```xml
<!-- 配置二级缓存 --> 
<settings> 
<!-- 开启二级缓存的支持 --> 
<setting name="cacheEnabled" value="true"/> 
</settings>
```

在持久层接口中使用注解配置二级缓存

```java
@CacheNamespace(blocking=true)//mybatis基于注解方式实现配置二级缓存 
public interface IUserDao {}
```

