# Spring

#### 程序的耦合

​	**耦合**:程序间的依赖关系

- 包括：

​				类之间的依赖

​				方法间的依赖

- 解耦：

​				降低程序间的依赖

- 实际开发中：

​				做到编译期不依赖，运行时才依赖

- 解耦思路：

  1. 使用反射来创建对象，避免使用 new 关键字

     ​	不会报错，能够编译成功，运行时会出异常Exception

  2. 通过读取配置文件来获取要创建的对象全限定类名

     ​	使用反射写死全限定类名后修改不便捷

-----

----

## 1.容器的理解

为生产对象（OBJECT）的地方，在这里容器不只是帮我们创建了对象那么简单，它负责了对象的整个生命周期--创建、装配、销毁。

简单理解为是一个map集合，存储键值对，键为id，值为创建的类的全限定类名。

## 2.IOC控制反转

把创建对象的权利交给框架

```java
private IAccountDao accountDao = new AccountDaoImpl();   
```

直接new创建对象，拥有自主的控制权，创建谁，自己说的算

```java
 private IAccountDao accountDao = (IAccountDao)BeanFactory.getBean("accountDao");
```

使用工厂创建，将自己创建的权利交给BeanFactory来创建bean对象，通过名称来找到想要的对象，是不是自己能用的已经无法得知。因为得到的对象是工厂的配置文件来决定的。无法自主控制

[^]: 这种控制权发生的转移就叫控制反转！实际的意义就是降低耦合，降低程序间的依赖关系

### 2.1.获取核心容器对象ApplicationContext三个实现类

ClassPathXmlApplicationContext:

​		加载类路径下的配置文件，要求配置文件必须在类路径下

FileSystemXmlApplicationContext：

​		可以加载磁盘任意路径下的配置文件（必须有访问权限）

AnnotationConfigApplicationContext

​		用于读写注解创建容器的

### 2.2.核心容器两个接口

ApplicationContext：单例对象适合

​	在构建核心容器时，创建对象采用的策略是立即加载，只要一读配置文件，立马加载对象

```java
//加载对象
ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
//获取对象
IAccountService as  = (IAccountService)ac.getBean("accountService");
```

BeanFactory：多例对象适合

​	在构建核心容器时，创建对象采用的策略是延迟加载，也就是说什么时候根据id获取对象，什么时候才真正的创建对象。

```java
//加载配置
Resource resource = new ClassPathResource("bean.xml");     
//创建工厂
BeanFactory factory = new XmlBeanFactory(resource);       
//获取对象
IAccountService as  = (IAccountService)factory.getBean("accountService");
```

BeanFactory 才是Spring 容器中的顶层接口。
		ApplicationContext 是它的子接口。
BeanFactory 和ApplicationContext 的区别：
		创建对象的时间点不一样。
		ApplicationContext：只要一读取配置文件，默认情况下就会创建对象。
		BeanFactory：什么使用什么时候创建对象。

![1566279289280](C:\Users\tyrant\AppData\Roaming\Typora\typora-user-images\1566279289280.png)

### 2.3.bean的三种构造方式

- 第一种方式：使用默认构造函数创建。

​		在spring的配置文件中使用bean标签，配以id和class属性之后，且没有其他属性和标签时。
​		采用的就是默认构造函数创建bean对象，此时如果类中没有默认构造函数，则对象无法创建。

```xml
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"></bean>
```

- 第二种方式： 使用普通工厂中的方法创建对象（使用某个类中的方法创建对象，并存入spring容器）

```java
public class InstanceFactory { 
    public IAccountService createAccountService(){ 
        return new AccountServiceImpl(); 
    } 
}
```

```xml
<bean id="instanceFactory" class="com.itheima.factory.InstanceFactory"></bean>
<bean id="accountService" factory-bean="instanceFactory" factory-method="getAccountService"></bean>
```

- 第三种方式：使用工厂中的静态方法创建对象（使用某个类中的静态方法创建对象，并存入spring容器)

```java
public class StaticFactory { 
    public static IAccountService createAccountService(){ 
        return new AccountServiceImpl(); 
    }
}
```

```xml
<bean id="accountService" class="com.itheima.factory.StaticFactory" factory-method="createAccountService"></bean>
```

### 2.4.bean对象

#### 2.4.1.作用范围调整

 bean标签的scope属性：
作用：用于指定bean的作用范围
取值： 常用的就是单例的和多例的
singleton：单例的（默认值）
prototype：多例的
request：作用于web应用的请求范围
session：作用于web应用的会话范围
global-session：作用于集群环境的会话范围（全局会话范围），当不是集群环境时，它就是session

```
 <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl" scope="prototype"></bean>
```

#### 2.4.2.生命周期

 单例对象
                出生：当容器创建时对象出生
                活着：只要容器还在，对象一直活着
                死亡：容器销毁，对象消亡
                总结：单例对象的生命周期和容器相同
多例对象
                出生：当我们使用对象时spring框架为我们创建
                活着：对象只要是在使用过程中就一直活着。
                死亡：当对象长时间不用，且没有别的对象引用时，由Java的垃圾回收器回收

## 3.DI依赖注入

Dependency Injection

### 3.1.依赖关系的管理

​		以后都交给Spring来维护
​        在当前类需要用到其他类的对象，由Spring为我们提供，我们只需要在配置文件中说明

### 3.2.依赖关系的维护

​		就称之为依赖注入。

###  3.3.依赖注入

 能注入的数据：有三类
		基本类型和String
		其他bean类型（在配置文件中或者注解配置过的bean）
		复杂类型/集合类型

#### 3.3.1.注入的方式：有三种

​		 第一种：使用构造函数提供
​		 第二种：使用set方法提供
​		第三种：使用注解提供

##### 3.3.1.1.构造函数注入

 使用的标签:constructor-arg
标签出现的位置：bean标签的内部
标签中的属性
	type：用于指定要注入的数据的数据类型，该数据类型也是构造函数中某个或某些参数的类型
	index：用于指定要注入的数据给构造函数中指定索引位置的参数赋值。索引的位置是从0开始
	name：用于指定给构造函数中指定名称的参数赋值           常用的		         	

​	value：用于提供基本类型和String类型的数据
​    ref：用于指定其他的bean类型数据。它指的就是在spring的IOC核心容器中出现过的bean对象

```java
public AccountServiceImpl(String name, Integer age, Date birthday) 
{ 
    this.name = name; 
    this.age = age; 
    this.birthday = birthday; 
}
```

```java
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
<constructor-arg name="name" value="泰斯特"></constructor-arg>
<constructor-arg name="age" value="18"></constructor-arg>
<constructor-arg name="birthday" ref="now"></constructor-arg>
</bean>
<!-- 配置一个日期对象 -->
<bean id="now" class="java.util.Date"></bean>
```

优势

在获取bean对象时，注入数据是必须的操作，否则对象无法创建成功。

弊端

改变了bean对象的实例化方式，使我们在创建对象时，如果用不到这些数据，也必须提供。

##### 3.3.1.2.set方法注入

 涉及的标签：property          更常用的方式
出现的位置：bean标签的内部
标签的属性
	name：用于指定注入时所调用的set方法名称
	value：用于提供基本类型和String类型的数据
	ref：用于指定其他的bean类型数据。它指的就是在spring的Ioc核心容器中出现过的bean对象

```java
<bean id="accountService2" class="com.itheima.service.impl.AccountServiceImpl2">
<property name="name" value="TEST" ></property>
<property name="age" value="21"></property>
<property name="birthday" ref="now"></property>
</bean>
<!-- 配置一个日期对象 -->
<bean id="now" class="java.util.Date"></bean>
```

优势

​		创建对象时没有明确的限制，可以直接使用默认构造函数

弊端

​		如果有某个成员必须有值，则获取对象时有可能set方法没有执行。

##### 3.3.1.3.复杂类型的注入/集合类型的注入

用于给List结构集合注入的标签：list array set

用于个Map结构集合注入的标签: map  props
 结构相同，标签可以互换

```java
<bean id="accountService3" class="com.itheima.service.impl.AccountServiceImpl3">
        <property name="myStrs">
            <set>
                <value>AAA</value>
                <value>BBB</value>
                <value>CCC</value>
            </set>
        </property>
         <property name="myList">
        <array>
            <value>AAA</value>
            <value>BBB</value>
            <value>CCC</value>
        </array>
    </property>

    <property name="mySet">
        <list>
            <value>AAA</value>
            <value>BBB</value>
            <value>CCC</value>
        </list>
    </property>

    <property name="myMap">
        <props>
            <prop key="testC">ccc</prop>
            <prop key="testD">ddd</prop>
        </props>
    </property>

    <property name="myProps">
        <map>
            <entry key="testA" value="aaa"></entry>
            <entry key="testB">
                <value>BBB</value>
            </entry>
        </map>
    </property>
</bean>
```

[^]: property标签的name属性即所属bean的字段名

## 4.注解配置IOC

##### 曾经XML的配置：

```xml
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl" scope=""  init-method="" destroy-method="">
	<property name=""  value="" | ref=""></property>
</bean>
```

##### 使用注解时的头文件

spring使用官网上ctrl+f 搜索xmlns：context

```
<!--告知spring在创建容器时要扫描的包，配置所需要的标签不是在beans的约束中，而是一个名称为context名称空间和约束中-->
<context:component-scan base-package="com.itheima"></context:component-scan>
```

写在头文件bean里面

##### 用于创建对象的

他们的作用就和在XML配置文件中编写一个<bean>标签实现的功能是一样的

###### Component

​	作用：用于把当前类对象存入spring容器中

​	属性value：用于指定bean的id。当我们不写时，它的默认值是当前类名，且首字母改小写。

###### Controller

一般用在表现层

###### Service

一般用在业务层

###### Repository

一般用在持久层

以上三个注解他们的作用和属性与Component是一模一样。

他们三个是spring框架为我们提供明确的三层使用的注解，使我们的三层对象更加清晰

#####  用于注入数据的

他们的作用就和在xml配置文件中的bean标签中写一个<property>标签的作用是一样的

###### Autowired

​	作用：自动按照类型注入。

​	只要容器中有唯一的一个bean对象类型和要注入的变量类型匹配，就可以注入成功；

​	如果ioc容器中没有任何bean的类型和要注入的变量类型匹配，则报错。

​	如果Ioc容器中有多个类型匹配时：先用数据类型圈定作用范围，再通过变量名称匹配，有一样的就注入成空，没有就报错。

​	出现位置：可以是变量上，也可以是方法上

​	细节：在使用注解注入时，set方法就不是必须的了。

###### Qualifier

给类成员注入时，必须和autowired一起使用

给方法参数注入时，可以单独使用

​	作用：在按照类中注入的基础之上再按照名称注入。它在给类成员注入时不能单独使用。但是在给方法参数注入时可以（稍后我们讲）

​	属性：value：用于指定注入bean的id。

###### Resource

​	作用：直接按照bean的id注入。它可以独立使用

​	属性：name：用于指定bean的id。

​	以上三个注入都只能注入其他bean类型的数据，而基本类型和String类型无法使用上述注解实现。另外，集合类型的注入只能通过XML来实现。

###### Value

作用：用于注入基本类型和String类型的数据

属性：value：用于指定数据的值。它可以使用spring中SpEL(也就是spring的el表达式）

SpEL的写法：${表达式}

##### 用于改变作用范围的

他们的作用就和在bean标签中使用scope属性实现的功能是一样的

###### Scope

作用：用于指定bean的作用范围

属性：value：指定范围的取值。常用取值：singleton prototype

##### 和生命周期相关 了解

他们的作用就和在bean标签中使用init-method和destroy-method的作用是一样的

PreDestroy：用于指定销毁方法

PostConstruct：用于指定初始化方法

#### Spring中的新注解

###### Configuration

作用：指定当前类是一个配置类

细节：当配置类作为AnnotationConfigApplicationContext对象创建的参数时，该注解可以不写。

```
ApplicationContext ac = new AnnotationConfigApplicationContext(SpringConfiguration.class);
```

此时，SpringConfiguration类上的@configuration注解可以不写

###### ComponentScan

作用：用于通过注解指定spring在创建容器时要扫描的包

属性：value：它和basePackages的作用是一样的，都是用于指定创建容器时要扫描的包。

我们使用此注解就等同于在xml中配置了:

```
<context:component-scan base-package="com.itheima"></context:component-scan>
```

```
@ComponentScan("com.itheima")
```

###### Bean

作用：用于把当前方法的返回值作为bean对象存入spring的ioc容器中

属性:

​	name:用于指定bean的id。当不写时，默认值是当前方法的名称

细节：

当我们使用注解配置方法时，如果方法有参数，spring框架会去容器中查找有没有可用的bean对象。

查找的方式和Autowired注解的作用是一样的

```
 <!--配置QueryRunner-->
<bean id="runner" class="org.apache.commons.dbutils.QueryRunner" scope="prototype">
<!--注入数据源-->
<constructor-arg name="ds" ref="dataSource"></constructor-arg>
</bean>
```

```
@bean(name="runner")
```

[^]: name不能省略



###### Import

作用：用于导入其他的配置类

属性：value：用于指定其他配置类的字节码。

当我们使用Import的注解之后，有Import注解的类就父配置类，而导入的都是子配置类

```
@Import(JdbcConfig.class)
```

此时JdbcConfig也不需要写@configuration注解

###### PropertySource

作用：用于指定properties文件的位置

属性：value：指定文件的名称和路径。

​			关键字：classpath，表示类路径下，有包名加上包名/..../.../

```
@PropertySource("classpath:jdbcConfig.properties")
```

#### Spring整合Junit配置

 1、导入spring整合junit的jar(坐标)

```
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-test</artifactId>
	<version>5.0.2.RELEASE</version>
</dependency>
```

2、使用Junit提供的一个注解把原有的main方法替换了，替换成spring提供的@Runwith

```
@RunWith(SpringJUnit4ClassRunner.class)
```

3、告知spring的运行器，spring和ioc创建是基于xml还是注解的，并且说明位置

@ContextConfiguration

locations：指定xml文件的位置，加上classpath关键字，表示在类路径下

```
@ContextConfiguration(locations = "classpath:bean.xml")
```

classes：指定注解类所在地位置

```
@ContextConfiguration(classes = SpringConfiguration.class)
```

当我们使用spring 5.x版本的时候，要求junit的jar必须是4.12及以上

[^]: 事务控制应该在业务层

#### 动态代理

特点：字节码随用随创建，随用随加载
作用：不修改源码的基础上对方法增强
分类：
	基于接口的动态代理
	基于子类的动态代理

##### 	基于接口的动态代理

​	涉及的类：Proxy
​	提供者：JDK官方
​	如何创建代理对象：
​		使用Proxy类中的newProxyInstance方法
​        创建代理对象的要求：
​        被代理类最少实现一个接口，如果没有则不能使用

```
IProducer proxyProducer = (IProducer)Proxy.newProxyInstance(producer.getClass().getClassLoader(),producer.getClass().getInterfaces(),new InvocationHandler() {
/**
* 作用：执行被代理对象的任何接口方法都会经过该方法
* 方法参数的含义
* @param proxy   代理对象的引用
* @param method  当前执行的方法
* @param args    当前执行方法所需的参数
* @return        和被代理对象方法有相同的返回值
* @throws Throwable
*/
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {}
}
```

​        newProxyInstance方法的参数：
​       	 ClassLoader：类加载器
​      		  它是用于加载代理对象字节码的。和被代理对象使用相同的类加载器。固定写法。
​       	 Class[]：字节码数组
​        		它是用于让代理对象和被代理对象有相同方法。固定写法。
​       	 InvocationHandler：用于提供增强的代码
​        		它是让我们写如何代理。我们一般都是些一个该接口的实现类，通常情况下都是匿名内部类，但不是必须的。
​        此接口的实现类都是谁用谁写。

##### 基于子类的动态代理

​	涉及的类：Enhancer
​	提供者：第三方cglib库
​	如何创建代理对象：
​		使用Enhancer类中的create方法
​	创建代理对象的要求：
​		被代理类不能是最终类

 

```
Producer cglibProducer = (Producer)Enhancer.create(producer.getClass(), new MethodInterceptor() {
/**
* 执行北地阿里对象的任何方法都会经过该方法
* @param proxy
* @param method
* @param args
*    以上三个参数和基于接口的动态代理中invoke方法的参数是一样的
* @param methodProxy ：当前执行方法的代理对象
* @return
* @throws Throwable
*/
@Override
	public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {}}
```

​	create方法的参数：
​		Class：字节码
​			它是用于指定被代理对象的字节码。
​		Callback：用于提供增强的代码
​			它是让我们写如何代理。我们一般都是些一个该接口的实现类，通常情况下都是匿名内部类，但不是必须的。
​	此接口的实现类都是谁用谁写。
​	 我们一般写的都是该接口的子接口实现类：MethodInterceptor

#### AOP面向切面编程

动态代理的加强，加强pointcut切入点方法

```xml
<!--配置AOP-->
    <aop:config>
        <!--配置切面 -->
        <aop:aspect id="logAdvice" ref="logger">
            <!-- 配置通知的类型，并且建立通知方法和切入点方法的关联-->
            <aop:before method="printLog" pointcut="execution(* com.itheima.service.impl.*.*(..))"></aop:before>
        </aop:aspect>
    </aop:config>
```

## 5.问题

#### 什么是申明式事务

​	通过配置的方式控制事务