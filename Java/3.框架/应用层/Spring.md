

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

      ![1580714303429](.\..\..\..\pictures\Java\Spring\set方法注入的细节.png)

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

![1580712844714](.\..\..\..\pictures\Java\Spring\Spring对Date数据类型注入的支持.png)

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
      
        ![1581139663879](.\..\..\..\pictures\Java\Spring\Autowired唯一匹配自动注入.png)
      
        ②如果没有匹配类型，则报错；
      
        ③如果有多个匹配时，先匹配类型，再根据变量名称进行匹配
      
        ![1581142000546](.\..\..\..\pictures\Java\Spring\Autowired多个类型自动注入.png)
      
      - 出现位置：①变量上，②方法上
      
      - 细节：在使用注解注入时，set方法就不是必须的了
      
    - @Qualifier
  
      - 作用：在按照类注入的基础上，再按照名称注入。在给类成员注入时不能单独使用，但是给方法参数注入时可以
  
      - 属性：value：指定注入bean的id
  
      - 如果一个对象有多个实现类，且在方法参数要使用时，可以使用该注解定义对应的Bean对象
  
        ![1581253094794](.\..\..\..\pictures\Java\Spring\@Qualifier单独使用方法.png)
  
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

    ![1581231890818](.\..\..\..\pictures\Java\Spring\@Configuration和@ComponentScan注解的作用.png)

- @Bean

  - 作用：把当前方法的返回值作为Bean存入Spring容器中

  - 属性：name：用于指定bean的id，不写时，默认值是当前方法的名称

  - 细节：当我们使用注解配置方法时，如果方法有参数，Spring框架会去容器中查找有没有可用的bean对象。查找方式和@Autowired一样

    ![1581233406069](.\..\..\..\pictures\Java\Spring\@Bean注解的使用.png)

- @Import
  - 作用：用于导入其他的配置类
  - 属性：value：用于指定其他配置类的字节码。当使用该注解之后，有@Import注解的类就是父配置类，导入的就是子配置类，子配置类不需要再用@Configuration注解配置
  
- @PropertySource

  - 作用：用于指定properties文件的位置

  - 属性：value：指定文件的名称和路径

    ​						关键字：classpath：表示类路径下

  - 相当于 xml 的`<context:property-placeholder location=""/>`标签

  **配置类注解的细节**

  ![1581250718407](.\..\..\..\pictures\Java\Spring\Spring新注解详解.png)

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

#### Spring的事务控制

​	如果没有事务，则无法执行增删改的操作

<font color = ff00ff>	事务控制应该都是在<font color=ff0000>业务层</font>而不是持久层</font>

![1581306054741](.\..\..\..\pictures\Java\Spring\Spring事务控制原因.png)

- 无事务控制时，每个连接都有一个自己的事务，每个成功执行了就提交事务，如果报错了，下面的代码就不执行。
- 解决方法：<font color=ff0000>使用`ThreadLocal`对象把`Connection`和当前`线程`绑定,</font>从而使一个线程中只有一个能控制事务的对象

#### 动态代理

​	**[代理模式](https://blog.csdn.net/briblue/article/details/73928350)可以在不修改被代理对象的基础上，通过扩展代理类，进行一些功能的附加与增强。值得注意的是，代理类和被代理类应该共同实现一个接口，或者是共同继承某个类。**

- 特点：字节码随用随创建，随用随加载（装饰者模式是上来一定会有一个类）

- 作用：不修改源码的基础上对方法增强

- 主要功能：
  日志记录，性能统计，安全控制，事务处理，异常处理等等

- 分类：

  - 基于接口的动态代理
  - 基于子类的动态代理

- 基于接口的：通用的接口是该代理方法实现的基础

  - 涉及的类：Proxy

  - 提供者：JDK官方

  - 如何创建代理对象

    ​	使用Proxy类中的newProxyInstance方法

  - 创建代理对象的要求

    ​	被代理类<font color=ff00ff>至少实现一个接口</font>，如果没有则不能使用

  - newProxyInstance方法的参数

    - ClassLoader：类加载器

      用于加载代理对象字节码，和被代理对象使用同一个类加载器。

      固定写法：被代理对象.getClass().getClassLoader()

    - Class数组：字节码数组

      用于让代理对象和被代理对象有相同方法

      固定写法：被代理对象.getClass().getInterfaces()

    - InvocationHandler：提供增强的代码

      写如何代理。一般都是写接口的实现类，通常情况都是匿名内部类，但不必须

      > 匿名内部类访问外部对象时，要求外部对象必须是最终的，也就是使用final修饰

- 基于子类的：

  - 涉及的类：Enhancer

  - 提供者：第三方cglib库

  - 如何创建代理对象

    ​	使用Enhancer类中的create方法

  - 创建代理对象的要求

    ​	被代理类不能是最终类

  - create方法的参数

    - Class：

      用于指定被代理对象的字节码

    - Callback：提供增强的代码

      写如何代理。一般都是写接口的实现类，通常情况都是匿名内部类，但不必须，我们一般写该接口的子接口实现类：MethodInterceptor

#### 代理模式图解

![1581589529356](.\..\..\..\pictures\Java\Spring\代理模式图解.png)

1. 用户只关心接口功能，而不在乎谁提供了功能。上图中接口是 Subject。
2. 接口真正实现者是上图的 RealSubject，但是它不与用户直接接触，而是通过代理。
3. 代理就是上图中的 Proxy，由于它实现了 Subject 接口，所以它能够直接与用户接触。
4. 用户调用 Proxy 的时候，Proxy 内部调用了 RealSubject。所以，Proxy 是中介者，它可以增强 RealSubject 操作。

#### AOP

![1581665535568](.\..\..\..\pictures\Java\Spring\AOP.png)

- 作用： 在程序运行期间，不修改源码对已有方法进行增强。 
- 优势： 
  - 减少重复代码 
  - 提高开发效率 
  - 维护方便
- 实现：动态代理技术

##### AOP术语

- Joinpoint(连接点):

  所谓连接点是指那些被拦截到的点。也就是被代理类里所有的方法

- Pointcut(切入点):

  指我们要对哪些Joinpoint进行拦截的定义，就是那些被增强的方法

  ~~~java
  if ("test".equals(method.getName())) {
     return method.invoke(accountService, args);
  }
  ~~~

  如果是 test 方法，就不会增强，test 方法也就不是切入点，但它是连接点

  > 所有的切入点都是连接点，但不是所有的连接点都是切入点

- Advice(通知/增强): 

  拦截到Joinpoint之后所要做的事情就是通知。 也就是增强的方法

  - 通知的类型：前置通知,后置通知,异常通知,最终通知,环绕通知。

![](.\..\..\..\pictures\Java\Spring\通知的类型.jpg)

- Introduction(引介): 

  引介是一种特殊的通知在不修改类代码的前提下, Introduction可以在运行期为类动态地添加一些方法或Field

- Target(目标对象):被代理对象

- Weaving(织入): 是指把增强应用到目标对象来创建新的代理对象的过程。 Spring采用动态代理织入

-  Proxy（代理）: 一个类被AOP织入增强后，就产生一个结果代理类。

-  Aspect(切面): 是切入点和通知（引介）的结合。就是配置切入点方法和通知方法的执行顺序/关系而不是手写，配出来的就叫切面

##### AOP配置

- Spring基于xml的AOP配置步骤

  1. 把通知bean也交给Spring管理

  2. 使用`<aop:config></aop:config>`表明开始AOP配置

  3. 使用`<aop:aspect></aop:aspect>`表明配置切面

     - id:给前面提供一个唯一标识
     - ref:通知类bean的id

  4. 在`<aop:aspect></aop:aspect>`标签内部使用对应标签来配置通知的类型（前置通知,后置通知,异常通知,最终通知,环绕通知）

     - `<aop:before method="" pointcut=""></aop:before>`:表示前置通知,表示在切入点方法执行之前执行

       method：指定方法

       pointcut属性：指定切入点表达式，含义是对业务层中哪些方法增强

     - `<aop:after-returning method="" pointcut=""></aop:after-returning>`后置通知，表示在切入点方法执行之后执行，与异常通知只会执行一个

     - `<aop:after-throwing method="" pointcut=""></aop:after-throwing>`异常通知，表示在切入点方法执行产生异常之后执行，与后置通知只会执行一个

     - `<aop:after method="" pointcut=""></aop:after>`最终通知，表示无论在切入点方法是否正常执行他都会在最后执行

     

**配置切入点表达式**
     
`<aop:pointcut id="" expression=""></aop:pointcut>`
     
     - id：唯一标志
     - expression：切入点表达式
  - 注意：此标签写在`<aop:aspect></aop:aspect>`内部，只能当前标签使用；他还可以写在`<aop:aspect></aop:aspect>`标签外面，表示所有标签可用，但是要写在`<aop:aspect>`前面
    
       > 写入点表达式写法：
       >
       > 关键字：execution（表达式）
       >
       > 表达式：访问修饰符 返回值 包名.包名.包名...... 类名.方法名（参数列表）
       >
       > 标准写法：
       >
       > `public void project.service.AccountServiceImpl.saveAccount()`
       >
       > 访问修饰符可以省略：
       >
       > `void project.service.AccountServiceImpl.saveAccount()`
       >
       > 返回值可以使用通配符，表示任意返回值
       >
       > `* project.service.AccountServiceImpl.saveAccount() `
       
       >
         > 包名可以使用通配符，表示任意包，但是有几级包，就要写几个*
       >
         > `* *.*.AccountServiceImpl.saveAccount()`
         >
         > 包名可以用..，表示当前包及其子类
         >
         > `* *..AccountServiceImpl.saveAccount()`
         >
         > 类名和方法名都可以使用*来实现通配
         >
         > `* *..*.*()`
         >
         > 方法参数列表可以直接写数据类型
         >
         > ​	基本类型直接写名称 int
         >
         > ​	引用类型要写 包名.类名 java.lang.String
         >
         > ​	可以用通配符表示任意类型，但必须有参数
         >
         > ​	`* *..*.*(*)`
         >
         > ​	用..进行统配，表示有无参数都可
         >
         > 全通配学法：
         >
         > `* *..*.*(..)`
         >
         > 通常写法：
         >
         > 切到业务层实现类下的所有方法
         >
         > `* project.service.Impl.*.*(..)`
  
- Spring基于注解的AOP配置步骤

  1. 导入Spring对注解的依赖

     ~~~xml
     xmlns:context="http://www.springframework.org/schema/context"
     http://www.springframework.org/schema/context
     http://www.springframework.org/schema/context/spring-context.xsd
     ~~~

  2. 配置要扫描的包，并开启AOP的支持

     ~~~xml
     <context:component-scan base-package="project"></context:component-scan>
     <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
     ~~~

  3. 在切面上添加标签，<font color=ff0000>一点要配置IoC容器</font>

     ~~~java
     @Component("pointCut")
     @Aspect//表示当前类是个切面
     public class pointCut {}
     ~~~

  4. 方法中配置切面表达式

     ~~~java
     @Pointcut("execution(* project.service.Impl.*.*(..))")
         private void pt1(){}
     ~~~

  5. 在方法上添加注解，需要添加切面表达式
     - @Before("pt1()")：前置通知
     
     - @AfterReturning("pt1()")：后置通知
     
     - @AfterThrowing("pt1()")：异常通知
  
     - @After("pt1()")：最终通知
  
     - @Around("pt1()")：环绕通知
  
       - **环绕通知**
     
         `<aop:around method="" pointcut=""></aop:after>`
     
         - 问题：当我们配置环绕通知之后，切入点方法没有执行，而环绕通知方法执行了。
     
         - 分析：通过对比动态代理中的通知代码，动态代理的环绕通知有明确的切入点调用
     
         - 解决：Spring框架为我们提供一个接口：ProceedingJoinPoint，该接口有一个方法proceed(),此方法相当于明确调用切入点方法，该接口可以作为环绕通知的方法参数。
     
         - Spring的环绕通知：
     
           它是Spring框架提供给我们的一种可以自己手动控制增强方法合适执行的方式
     
         **类中的环绕通知方法**：
     
         ```java
         public Object aroundPrintLog(ProceedingJoinPoint pjp){
                 Object rtValue = null;
                 try {
                     Object[] args = pjp.getArgs();//得到方法执行所需要的参数
                     System.out.println("前置通知");
                     rtValue = pjp.proceed(args);//明确调用业务层方法（切入点方法）
         			System.out.println("后置通知");
                     return rtValue;
                 } catch (Throwable throwable) {
                     System.out.println("异常通知");
                     throw new RuntimeException(throwable);
                 }finally {
                     System.out.println("最终通知");
                 }
             }
         ```
     
       
  
  **注意：**
  
  ​	基于注解的AOP配置，在调用后置/异常通知和最终通知的顺序时，是会有异常的，会先条用最终通知，再调用后置/异常通知，选择时需要慎重。<font color=ffooff>建议只用注解时，使用环绕通知</font>
  
  ​	环绕通知和前置/后置/异常通知的执行顺序为：环绕通知的前置通知-》前置通知-》执行方法-》环绕通知的后置/异常通知-》环绕通知的最终通知-》最终通知-》后置/异常通知
  
  - 不适用注解的方法：在配置类中，加上`@EnableAspectJautoProxy`注解
  
  <font color=ff0000>注解配置AOP控制事务注意：</font>
  
  ![](.\..\..\..\pictures\Java\Spring\注解AOP控制事务的问题分析.png)
  
  先执行最终通知，再执行后置/异常通知，导致线程和连接先解绑后，再提交事务，此时的事务时提交不上去的。

#### JdbcTemplate

![](.\..\..\..\pictures\Java\Spring\持久层总图.jpg)

- 作用：用户和数据库交互的，实现对表的CRUD操作

- 如何创建对象：

- 常用方法：

  - query:查询所有，查询一个

    `query(sql,Object[] args,RowMapper<T> rowMapper)`

    RowMapper<T>:结果集封装策略

  - update：增删改

    `update(sql,Object[] args)`

  - queryForObject：返回一行一列

    `Long count = jt.queryForObject("select count(*) from account where money > ?",Long.class,1000f)`

- 和queryRunner的区别：

  ![](.\..\..\..\pictures\Java\Spring\jdbctemplate和queryrunner的区别.png)

  JdbcTemplate中的封装策略：new AccountRowMapper(),常用其实现子类：BeanPropertyRowMapper<>()

  QueryRunner中的封装策略：BeanListHandler<>()

  自定义JdbcTemplate中的封装策略：

  ~~~java
  /**
   * 定义Account的封装策略
   */
  class AccountRowMapper implements RowMapper<Account>{
      /**
       * 把结果集中的数据封装到Account中，然后由spring把每个Account加到集合中
       * @param rs
       * @param rowNum
       * @return
       * @throws SQLException
       */
      @Override
      public Account mapRow(ResultSet rs, int rowNum) throws SQLException {
          Account account = new Account();
          account.setId(rs.getInt("id"));
          account.setName(rs.getString("name"));
          account.setMoney(rs.getFloat("money"));
          return account;
      }
  }
  ~~~

  

