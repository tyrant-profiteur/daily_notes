

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

#### 获取对象

~~~Java
//1.获取核心容器对象
	ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
//根据id获取bean对象,两种方法
	IAccountService as = (IAccountService) ac.getBean("accountService");
	IAccountDao adao = ac.getBean("accountDao",IAccountDao.class);
~~~

其中第一步获取核心容器对象的过程和工厂模式创建一样，只是读取配置文件，创建对象，存入Map的过程，Spring都帮我们完成了。工厂模式如下：

~~~java
public class BeanFactory {
    //定义properties对象
    private static Properties props;

    //4.定义一个Map，用于存放我们要创建的对象，我们称之为容器,来实现单例而不是多例
    private static Map<String,Object> beans;

    //使用静态代码块为properties对象赋值
    static {
        //实例化对象
        props = new Properties();
        //获取properties流对象
        try {
            /**注意此时不能使用 InputStream i = new FileInputStream("")来创建流对象
             * 写绝对路径，不能保证工程部署环境都有c盘D盘
             * 写相对路径src，web工程部署后src就没了，用不了
            */
            InputStream in = BeanFactory.class.getClassLoader().getResourceAsStream("bean.properties");
            props.load(in);
            //实例化容器
            beans = new HashMap<String, Object>();
            //去除配置文件所有的key
            Enumeration keys= props.keys();
            //遍历枚举
            while (keys.hasMoreElements()){
                //取出每个key
                String key = keys.nextElement().toString();
                //获取value
                String beanPath = props.getProperty(key);
                //反射创建对象
                Object value = Class.forName(beanPath).newInstance();
                //存入容器
                beans.put(key,value);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**根据Bean名称，获取Bean对象
    public static Object getBean(String beanName) {
        Object bean = null;
        String beanPath = props.getProperty(beanName);
        //01jdbc代码里的class反射
        try {
            //通过反射的方式来创建
            bean = Class.forName(beanPath).newInstance();//每次都调用默认构造函数创建对象
        } catch (Exception e) {
            e.printStackTrace();
        }
        return bean;
    }*/
    //在工厂加载的时候，static方法中，所有的对象都已经准备就绪，不需要每次都创建一个实例对象
    //根据名称获取bean对象
    public static Object getBean(String beanName) {
        return beans.get(beanName);
    }
}
~~~

##### ApplicationContext的三个实现类：

- ClassPathXmlApplicationContext:

  可以下载类路径下的配置文件（更常用）

- FileSystemXmlApplicationContext:

  可以加载磁盘任意路径下的配置文件（拥有访问权限）

- AnnotationConfigApplicationContext:

  读取注解创建容器

##### BeanFactory和ApplicationContext的区别

- ApplicationContext:单例对象适用

  构建核心容器的时候，<font color=ff0000>采用立即加载的方式</font>，一读配置文件，对象就被创建出来

  > <font color=ff00ff>在使用注解创建对象时，`ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml")`读取xml文件就创建对象，但是基于注解配置，bean.xml里是没有对象的，虽然在类上使用了`@Component`注解，但是Spring并不知道注解在哪里，需要有行配置告知Spring在创建容器时要扫描的包，配置所需要的标签不是在`<beans></beans>`下</font>

- BeanFactory:多例对象使用  

  创建对象采取的策略是延迟加载的方式，什么时候根据id获取对象，什么时候才真正的创建对象

##### Spring对bean的管理细节

- **创建bean的三种方式**

  - ~~~xml
    <bean id="accountService" class="service.Impl.AccountServiceImpl"></bean>
    ~~~

    使用*默认构造函数*：在配置文件中使用bean对象，配以id和class属性，且没有其他属性和标签。<font color=ff00ff>如果没有默认构造函数，则对象无法创建</font>

  - ~~~xml
    <bean id="instanceFactory" class="factory.InstanceFactory"></bean>
    <bean id="accountService" factory-bean="instanceFactory" factory-method="getAccountService"></bean>	
    ~~~
    
    使用某个类中的方法创建对象（工厂类），并存入Spring中
    
  - ~~~xml
    <bean id="accountService" class="factory.StaticFactory" factory-method="getAccountService"></bean>
    ~~~
  
    使用某个类中的静态方法创建对象（工厂类），并存入Spring中
  
- **bean对象的作用范围**

  bean标签的scope属性：用于指定bean的作用范围

  - singleton：单例的，默认值
  - prototype：多例的
  - request：作用于web应用的请求范围
  - session：作用于web应用的会话范围
  - global-session：作用于集群环境的会话范围（全局会话范围）  

- **bean对象的生命周期**

  - 单例

    - 出生：当容器创建时对象出生
    - 活着：只要容器还在，对象一直活着
    - 死亡：容器销毁，对象消亡

    单例对象生命周期和容器相同

  - 多例

    - 出生：当我们使用对象时创建
    - 活着：对象只要在使用过程中就一直活着
    - 死亡：当对象长时间不用且没有别的对象引用时，由Java垃圾回收机制处理

##### 依赖注入

​	Dependency Injection

​	**IOC的作用：**

​		降低程序间的耦合（依赖关系）

​	**依赖关系的管理：**

​		交给Spring来维护：在当前类中需要用到其他类的对象，由Spring为我们提供，我们只需要在配置文件中说明

​	**依赖注入：**

​		*三类数据类型*

- 基本类型和String

- 其他bean类型（在配置文件中或者注解配置的bean）

- 复杂类型/集合类型

  *三种注入方式*

- 构造函数提供

  - 标签：constructor-arg

  - 位置：bean内部

  - 标签中的属性

    - type：用于指定要注入的数据的数据类型，该类型也是构造函数中某个参数的数据类型<font color=ff00ff>如果构造函数有相同的数据类型，则无法单独完成注入</font>

    - index：用于指定注入的数据在构造函数的索引位置，从0开始，<font color=ff00ff>可以独立完成,但是要知道类型</font>

    - <font color=ff0000>name</font>：用于指定给构造函数中指定名称的参数赋值

      ---

    - value：提供基本类型和String类型的数据

    - ref：指定其他的bean类型数据。指的是在Spring的IoC核心容器中出现过的bean对象

  - 优势：

    在获取bean对象时，注入数据是必须的操作，否则无法创建成功

  - 弊端：

    改变了bean对象的实例化方式，使我们在创建时，用不到这些数据也必须提供

- <font color=ff0000>set方式提供</font>

  - 标签：property

  - 位置：bean内部

  - 标签中的属性

    - <font color=ff0000>name</font>：用于指定注入时所调用的<font color=ff00ff>set方法名称</font>

      ![1580714303429](E:\Java\new_Java_Study\daily_notes\pictures\set方法注入的细节.png)

    - value：提供基本类型和String类型的数据

    - ref：指定其他的bean类型数据。指的是在Spring的IoC核心容器中出现过的bean对象

  - 优势：

    创建对象时，没有明确的限制，可以直接使用默认构造函数

  - 弊端：

    如果有某个成员必须有值，则set方法无法保证一定有值

  - **复杂类型注入**（使用set方式）

    - 数组/list集合/set集合：`property`标签内加入`array/list/set`标签

    ~~~xml
    <bean id="accountServiceComplex" class="service.Impl.AccountServiceImplComplex">
            <property name="myStrs">
                <array>
                    <value>AAA</value>
                    <value>BBB</value>
                    <value>CCC</value>
                </array>
            </property>
        </bean>
    ~~~

    - Map集合/properties：`property`标签内加入`map/props`标签

    ~~~xml
    <property name="myMap">
                <map>
                    <entry key="testA" value="AAA"></entry>
                    <entry key="testB">
                        <value>BBB</value>
                    </entry>
                </map>
            </property>
    ~~~

    ~~~xml
    <property name="myMap">
                <props>
                    <prop key="testC">CCC</prop>
                    <prop key="testD">DDD</prop>
                </props>
            </property>
    ~~~

    > <font color=ff0000>结构相同，标签可以互换</font>

- 使用注解提供

> ```
> 如果是经常变化的数据，并不适合注入的方式
> ```

**Spring对Date数据类型的注入的支持**

![1580712844714](E:\Java\new_Java_Study\daily_notes\pictures\Spring对Date数据类型注入的支持.png)

##### 基于注解的 Spring

- 曾今的 xml 配置

  ~~~xml
  <bean id="accountService" class="service.Impl.AccountServiceImpl" scope="" init-method="" destory-method="">
  <property name="" value="" ref=""></property>
  </bean>
  ~~~

- 基于注解的

  <font color=ff0000>需要告知Spring在创建容器时要扫描的包，配置所需要的标签不是在`<beans></beans>`约束中，而是一个叫`context`的约束中</font>

  1. 导入注解所需要的bean.xml配置([Spring的core](E:\Java\books\黑马JAVA\00.讲义+笔记+资料\3.主流框架\32.会员版(2.0)-就业课(2.0)-Spring\spring_day01\资料\spring-framework-5.0.2.RELEASE-dist\spring-framework-5.0.2.RELEASE\docs\spring-framework-reference\core.html))
  2. <beans></beans>中使用context标签
  
  ~~~xml
  <context:component-scan base-package="java"></context:component-scan>
  ~~~
  
  - 创建对象的
  
    |      xml      |    注解     |
    | :-----------: | :---------: |
    | <bean></bean> | @Component  |
    |               | @Controller |
    |               |  @Service   |
    |               | @Repository |
    
    - @Component作用
    
      - 作用：把当前类存入Spring容器中
    
      - 属性：value：用于指定bean的id，当我们不写时，默认值为当前类型，首字母小写
    
        ~~~xml
        @Component(value=""),如果只有一个value属性时，可以省略写成@Component("")
        ~~~
    
    - @Controller    @Service    @Repository作用
    
      - 作用：和@Component一样，Spring提供的明确的三层使用的注解，使三层对象更加清晰
      - @Controller：表现层
      - @Service：应用层
      - @Repository：持久层
    
  - 注入数据的
  
    |                xml                 |    注解    |
    | :--------------------------------: | :--------: |
    | <bean><property></property></bean> | @Autowired |
    |                                    | @Qualifier |
    |                                    | @Resource  |
    |                                    |   @Value   |
  
    - @Autowired
      - 作用：自动按照类型注入。
      
        ①只要容器中有<font color=ff00ff>唯一一个</font>bean对象类型和要注入的变量类型匹配，就可以注入成功；
      
        ![1581139663879](E:\Java\new_Java_Study\daily_notes\pictures\Autowired唯一匹配自动注入.png)
      
        ②如果没有匹配类型，则报错；
      
        ③如果有多个匹配时，先匹配类型，再根据变量名称进行匹配
      
        ![1581142000546](E:\Java\new_Java_Study\daily_notes\pictures\Autowired多个类型自动注入.png)
      
      - 出现位置：①变量上，②方法上
      
      - 细节：在使用注解注入时，set方法就不是必须的了
      
    - @Qualifier
  
      - 作用：在按照类注入的基础上，再按照名称注入。在给类成员注入时不能单独使用，但是给方法参数注入时可以
  
      - 属性：value：指定注入bean的id
  
      - 如果一个对象有多个实现类，且在方法参数要使用时，可以使用该注解定义对应的Bean对象
  
        ![1581253094794](E:\Java\new_Java_Study\daily_notes\pictures\@Qualifier单独使用方法.png)
  
    - @Resource
  
      - 作用：直接按照bean的id注入，可以单独使用
      - 属性：name：用于指定dean的id
  
    <font color=ff0000>注：</font>以上三种注入只能注入其他bean类型数据，基本类型和String类型无法使用上述注解实现。
  
    ​		另外，集合类型的注入只能通过 xml 来实现
  
    - @Value
  
      - 作用：用于注入基本类型和String类型的数据
  
      - 属性：value：用于指定数据的值。可以使用 Spring 的 spEL (Spring 的EL表达式)
  
        > spEL 的写法：${表达式}
  
  - 改变作用范围的
  
    |          xml           |  注解  |
    | :--------------------: | :----: |
    | <bean scope=""></bean> | @Scope |
  
    - @Scope
      - 作用：改变bean的作用范围
      - 属性：value：指定范围的取值。常用：singleton，prototype
  
  - 生命周期相关的
  
    |               xml               |      注解      |
    | :-----------------------------: | :------------: |
    |  <bean init-method=""></bean>   | @PostConstruct |
    | <bean destory-method=""></bean> |  @PreDestroy   |

##### Spring的新注解

​	以上学习，即使基于注解，xml配置文件依旧存在，如告知要扫描的包，配置数据库，要用到jar包中的类时，一就要使用xml来配置，解决以上问题，使用Spring新注解

<font color=ff0000>**注：**</font>在不使用xml配置时，获取容器不能再使用`ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml")`,而是使用`ApplicationContext ac = new AnnotationConfigApplicationContext(SpringConfiguration.class)`

- @Configuration

  - 作用：指定当前类是一个配置类，替换 bean.xml
  - 细节：当配置类作为`AnnotationConfigurationApplicationContext`对象参数创建时，该注解可以不写

- @ComponentScan

  - 作用：用于通过注解来指定spring在创建容器时要扫描的包

  - 属性：value：它和basePackages的作用是一样的，都是用于指定创建容器时要扫描的包，使用此注解等同于在bean.xml配置了`<context:component-scan base-package="project"></context:component-scan>`

    ![1581231890818](E:\Java\new_Java_Study\daily_notes\pictures\@Configuration和@ComponentScan注解的作用.png)

- @Bean

  - 作用：把当前方法的返回值作为Bean存入Spring容器中

  - 属性：name：用于指定bean的id，不写时，默认值是当前方法的名称

  - 细节：当我们使用注解配置方法时，如果方法有参数，Spring框架会去容器中查找有没有可用的bean对象。查找方式和@Autowired一样

    ![1581233406069](E:\Java\new_Java_Study\daily_notes\pictures\@Bean注解的使用.png)

- @Import
  - 作用：用于导入其他的配置类
  - 属性：value：用于指定其他配置类的字节码。当使用该注解之后，有@Import注解的类就是父配置类，导入的就是子配置类，子配置类不需要再用@Configuration注解配置
  
- @PropertySource

  - 作用：用于指定properties文件的位置

  - 属性：value：指定文件的名称和路径

    ​						关键字：classpath：表示类路径下

  - 相当于 xml 的`<context:property-placeholder location=""/>`标签

  **配置类注解的细节**

  ![1581250718407](E:\Java\new_Java_Study\daily_notes\pictures\Spring新注解详解.png)

##### Spring整合Junit

- 分析：

> 1、应用程序的入口
> 	main方法
> 2、junit单元测试中，没有main方法也能执行
> 	junit集成了一个main方法
> 	该方法就会判断当前测试类中哪些方法有 @Test注解
> 	junit就让有Test注解的方法执行
> 3、junit不会管我们是否采用spring框架
> 	在执行测试方法时，junit根本不知道我们是不是使用了spring框架
> 	所以也就不会为我们读取配置文件/配置类创建spring核心容器
> 4、由以上三点可知
> 	当测试方法执行时，没有Ioc容器，就算写了Autowired注解，也无法实现注入

- 步骤

  1. 导入spring整合junit的jar(坐标)

  2. 使用Junit提供的一个注解把原有的main方法替换了，替换成spring提供的@Runwith

     `@RunWith(SpringJUnit4ClassRunner.class)`

  3. 告知spring的运行器，spring和ioc创建是基于xml还是注解的，并且说明位置@ContextConfiguration

     - locations：指定xml文件的位置，加上classpath关键字，表示在类路径下

       `@ContextConfiguration(locations = "classpath:bean.xml")`

     - classes：指定注解类所在地位置

       `@ContextConfiguration(classes = SpringConfiguration.class)`

  > 当我们使用spring 5.x版本的时候，要求junit的jar必须是4.12及以上

---

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