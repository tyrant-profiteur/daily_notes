#### Spring整合Junit单元测试注意事项

​	Spring——牛逼框架，Junit——单元测试。用了Spring要做测试——Spring+Junit，开整。

1. 导入spring整合junit的jar(坐标)

   ```xml
   <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
   </dependency>
   ```

2. 使用Junit提供的一个注解把原有的main方法替换了，替换成spring提供的@Runwith

   `@RunWith(SpringJUnit4ClassRunner.class)`

3. 告知spring的运行器，spring和ioc创建是基于xml还是注解的，并且说明位置@ContextConfiguration

   - locations：指定xml文件的位置，加上classpath关键字，表示在类路径下

     `@ContextConfiguration(locations = "classpath:bean.xml")`

   - classes：指定注解类所在地位置

     `@ContextConfiguration(classes = SpringConfiguration.class)`

   整完，开始测，走~

   ```java
   @RunWith(SpringJUnit4ClassRunner.class)
   @ContextConfiguration(locations = "classpath:bean.xml")
   public class Test {
       private IAccountService accountService = new AccountServiceImpl();
       @Test
       public void test() {
           accountService.test();
       }
   }
   ```

   1. 创建对象
   2. 调用方法
   3. 来~跑起来，嘟驾~

   ![1581656701280](.\..\pictures\articles\Spring整合Junit报错.png)😱为什么。。`java.lang.NullPointerException`。。。看下报错信息，嗯，看不懂，再瞅瞅，有认识的了，`SpringJunit4ClassRunner`，这不就是我整合时候修改的 main 方法嘛，Spring提供的。

   再想想Spring核心——IoC,AOP。还没有AOP的事儿啊，那就是容器了。难道我调用了Spring，就不能new对象了？再试试。。

   ```java
   @RunWith(SpringJUnit4ClassRunner.class)
   @ContextConfiguration(locations = "classpath:bean.xml")
   public class Test {
       @Resource(name = "accountService")
       private IAccountService accountService;
   
       @Test
       public void test() {
           accountService.test();
       }
   }
   ```

   IAccountService我用注解注入总行了吧。走~然后。。真的就可以了🤔真的是。。

   至此，只能说，Spring牛逼，我用了你，我连想要有个增加耦合的机会都没有。

   总结：Spring整合了Junit<font color=ff0000>做单元测试</font>(其他时候还是可以new创建的，全都不让new也不现实)，在对象创建时，只能使用注解注入，无法使用new创建。用BeanFactory工厂直接get也创建不了的呦~

   ~~~java
   @Resource
   private BeanFactory beanFactory;
   IAccountService accountService = beanFactory.getAccountService();
   ~~~

   当然了，你都学了Spring了，还new啥new，new啥new，这不是违背Spring降低耦合的原则嘛，简直没事做（说给自己听的）

   ps：如果有小伙伴能够在test文件中new出来对象或者大佬觉得是错的，可以纠正我，万分感谢~我是小白一个，嗯。至此，溜啦