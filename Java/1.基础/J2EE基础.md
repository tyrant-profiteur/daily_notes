### 学习/复习笔记

#### Java 基础语法

##### 计算机基础

- **二进制**

  二进制系统中，每个0或1就是1位，叫做bit(比特)

- **字节**

  8个bit 0000-0000 表示1个字节，写成 *1 byte* 或者 *1 B*

  8 bit = 1 B

  1024 B = 1 KB

##### [JVM](#JVM,JDK,JRE的解释)

##### DOS命令运行

- javac Java 文件名 .java
- java 类名

##### 编译和运行

- **编译**

  javac 编译器将Java源文件翻译成 JVM 认识的 class 文件

- **运行**

  JVM 运行编译通过的 class 文件

##### main方法

​	称为主方法，写法固定格式不可更改，是程序的入口点，无论多少程序，JVM 都会从 main 方法开始执行。

##### 标识符

​	自己定义的内容，如类名，方法名，变量名等

- **命名规则(硬性)**

  - 可以包含26个字母(区分按大小写),0-9,$,_

  - 不能以数字开头
  - 不能是关键字

- **命名规范(软性)**

  - 类名：大驼峰
  - 方法名：小驼峰
  - 变量名：全部小写

##### 常量和变量

- **常量**：固定不变的量

  - 整数：0，1
  - 小数：0.0，-0.1
  - 字符：'A','好'
  - 字符串："你好"
  - 布尔：true，false
  - 空：null

- **变量**[^变量]：可以变化的量

  - 基本数据类型：

    - 整型

      - 字节型 byte 1字节

      - 短整型 short 2字节

      - <font color=ff0000>整型 int</font> 4字节

      - 长整型 long 8字节

        > 建议数据后面加L

    - 浮点型

      - 单精度 float 4字节

        > 建议数据后面加F

      - <font color=ff0000>双精度 double</font>  8字节

    - 字符型

      - char 2字节

    - 布尔型

      - boolean 1字节

  - 引用数据类型：

    - 类
    - 数组
    - 接口

##### 数据类型转换

- **自动类型转换**

  将取值范围小的类型自动提升为取值范围大的类型

  ![1579093176306](.\..\..\pictures\自动类型转换示意图.png)

  **转换规则：**

  ​	byte,short,char --> int --> long --> float -->double

- **强制类型转换**

  将取值范围大的类型强制转换为取值范围小的类型

  ![1579437664316](.\..\..\pictures\Java\J2EE\强制类型转换示意图.png)
    <font color='ff0000'>注：</font>

  - 浮点转为整数，直接取消小数点，会造成数据损失精度
  - int 强制转为 short，砍掉两个字节，会造成数据丢失

- **ASCII编码表**

  ~~~Java
  System.out.println('a'+1);  //98
  ~~~

  | 字符 | 数值 |
  | :--: | :--: |
  |  0   |  48  |
  |  9   |  57  |
  |  A   |  65  |
  |  Z   |  90  |
  |  a   |  97  |
  |  z   | 122  |

- **运算符**

  +，-，*，/，%，++，--

  <font color='ff0000'>注：</font>

  ​	整数使用以上运算符，无论怎么计算都不会得到小数

  ~~~java
  int i = 1234;
  System.out.println(i/1000);//1
  //1
  System.out.println((double)(i/1000) + getType((double)(i/1000)));//1.0java.lang.Double
  //2
  System.out.println((double)i/1000 + getType((double)i/1000));//1.234 java.lang.Double
  System.out.println((double)i/(double) 1000 +getType((double)i/(double)1000) );//1.234 class java.lang.Double
  double d = 1234;
  //3
  System.out.println(i/1000 +getType(i/1000));//1.234 java.lang.Double
  System.out.println(i/(double)1000 + getType(i/(double)1000));//1.234 java.lang.Double
  
  DecimalFormat df = new DecimalFormat("0.00");
  String num = df.format((float)i/1000);
  System.out.println(num);//1.23
  
  float f = new BigDecimal((float)i/1000).setScale(2,BigDecimal.ROUND_HALF_UP).floatValue();
  System.out.println(f);//1.23
  float f1 = new BigDecimal((float)i/1000).floatValue();
  System.out.println(f1);//1.234
  ~~~

  - 1: i/1000时，i是int类型，之后对(i/1000)的结果 1进行double强制转换，1转化为1.0
  - 2: i/1000前，i强制转化为double类型，在进行计算时，1000提升为double类型，最后结果为double
  - 3: i 与整数1000 相除时，1000提升为double类型，最后结果为double

### 问题总结

##### 环境变量 path 和 classpath 的区别

- **path**：操作系统查找可执行文件的目录

- **classpath**： java.exe查找class文件的目录，现在一般不设置

##### 面向对象和面向过程的区别

- **面向对象：**易维护，易复用，易扩展，可以设计出低耦合系统，但是性能比面向过程低
- **面向过程：**比面向对象性能好，在单片机，嵌入式，Linux/Unix 一般采用面向过程

> **为什么说面向过程性能比面向对象高？**
>
> 类调用时需要实例化，消耗资源。但是面向后成也需要分配资源，计算内存偏移量，主要原因是 Java 是半编译语言，执行代码不是能直接被 CPU 执行的二进制机械码。而面向过程大都是直接编译成机械码在电脑执行。<font color=ff00ff>*编译的复杂度不同导致执行时间不同*</font>

##### Java特点

* **面向对象：**封装，继承，多态
* **平台无关性：**JVM实现平台无关性
* **可靠性和安全性**
* **支持网络编程：**Java 本就是为了简化网络编程设计的语言
* **编译与解释并存**

##### JVM,JDK,JRE的解释

- **JVM**

  Java 虚拟机，运行 Java 字节码的虚拟机。目的是使用相同的字节码，在不同的系统实现，会给出相同的结果。<font color=ff00ff>*一次编译，随处实现*</font>

> **采用字节码的好处？**
>
> JVM 可以理解的代码叫做字节码（.class 文件），只面向虚拟机。Java 通过字节码方式，在解决了传统解释型语言执行效率低问题的同时，又保留了解释性语言的可移植性。

![Java程序运行过程](.\..\..\pictures\Java\J2EE\Java 程序运行过程.png)

.class 文件->机器码这步，JVM类加载器加载字节码文件，解释器进行逐行解释执行，但是有些代码会被经常性调用(热点代码[^hotSpot])，后来引进了JIT编译器(运行时编译)。当JIT完成第一次编译，会将字节码对应的机器码保存下来，下次可以直接使用。Java是编译与解释共存的语言的原因。

- **JDK**

  Java Development Kit,Java 开发工具，拥有 JRE 所有用的一切，还有编译器(javac)和工具(javadoc 和 jdb)，能够创建和编译程序。

- **JRE**

  Java Runtime Environment,Java 运行时环境，运行已编译 Java 程序的所有内容，包括 JVM,Java 类库，Java 命令。

JDK>JRE>JVM

> 在使用 JSP 部署 Web 应用时，虽然只是在应用程序服务器中运行 Java 程序，但是程序服务器会将 JSP 转换为 Java Servlet，并且需要使用 JDK 来编译 Servlet。

##### Oracle JDK 和 OpenJDK

- OpenJDK 是一个参考模型并且是完全开源的，而 Oracle JDK 是 OpenJDK 的一个实现，并不是完全开源的；
- Oracle JDK 比 OpenJDK 更稳定。OpenJDK 和 Oracle JDK 的代码几乎相同，但 Oracle JDK 有更多的类和一些错误修复。
- 在响应性和JVM性能方面，Oracle JDK 与 OpenJDK 相比提供了更好的性能；
- Oracle JDK 根据二进制代码许可协议获得许可，而 OpenJDK 根据 GPL v2 许可获得许可。

##### Java和C++的区别

- 都是面向对象的语言，都支持封装、继承、多态
- Java 不支持指针直接访问内存，程序内存更安全
- Java 的类是单继承的，C++ 支持多重继承，但 Java 接口可以多继承
- Java 有内存管理机制，不需要手动所释放无用内存

##### 应用程序和小程序

- 一个程序可以有多个类，但是只能有一个主类
  - 应用程序的主类，指包含 main() 方法的类
  - 小程序主类，是继承系统类 JApplet 或者 Applet 的子类
- 应用程序主类不一定要是 public，小程序主类必须是 public
- 应用程序从主线程启动(main 方法)，applet小程序没有 main 方法，主要潜在浏览器页面调用 init() 或者 run() 来启动

##### 字符型常量(Char)和字符串常量(String)的区别
|              |                     Char                     |                  String                  |
| :----------: | :------------------------------------------: | :--------------------------------------: |
|  **形式上**  |             单引号引起的一个字符             |          双引号引起的若干个字符          |
|  **含义上**  | 相当于一个整型值(ASCII值),可以参加表达式运算 | 代表一个地址值(字符串在内存中的存放位置) |
| **内存大小** |                   2个字节                    |                若干个字节                |


![img](.\..\..\pictures\Java\J2EE\类型所占存储空间.jpg)

##### 重载(overload)和重写(override)

> Java允许重载任何方法，要完整描述一个方法，需要指出方法名以及参数类型，叫做方法的标签。返回类型不是方法签名的一部分。也就是说不能有两个名字相同，参数类型相同返回值却不同的方法。

- **重写注意事项**

  子类对父类允许访问的对象进行重新编写。

  - 方法名、参数列表必须相同
  - 返回值范围小于等于父类
  - 抛出异常范围小于等于父类
  - 访问修饰符范围大于等于父类

- **三大特性**

  - 封装

    对象属性私有化，同时提供可以被外界访问的属性方法

  - 继承

    方便地复用以前的代码

    - 子类拥有父类所有属性和方法(包括私有属性和方法，**只是拥有，无法访问**)
    - 字类可以有自己的属性和方法
    - 字类可以重写父类方法

  - 多态

    程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编程时并不确定，而是在程序运行期间才确定，实现多态的**两种形式**：

    - 继承：多个子类对同一方法的重写
    - 接口：实现接口并覆盖接口中同一方法

##### String，StringBuffer，StringBuilder

- **可变性**

  - String类使用final关键字修饰字符数组来保存字符串，`private final char value[]`，所以String不可变

  - StringBuffer 和 StringBuilder 都继承自`AbstractStringBuilder`,也是使用 char[] value 保存字符串，但是可以修改

    ~~~java
    abstract class AbstractStringBuilder implements Appendable, CharSequence {
        char[] value;
        int count;
        AbstractStringBuilder() {
        }
        AbstractStringBuilder(int capacity) {
            value = new char[capacity];
        }	
    ~~~

- **线程安全性**

  - String 对象不可变，可以理解为常量，线程安全
  - StringBuffer 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的
  - StringBuilder 并没有加锁，所以是非线程安全的

- **性能**

  - 每次对 String 操作都会生成一个新的 String 对象，指针指向新的 String 对象
  - StringBuffer 和 StringBuilder 都是对对象本身进行操作，不改变对象引用
  - 相同情况下，StringBuilder 只比 StringBuffer 性能提高10%~15%，但是要冒多线程不安全的风险

- **总结**
  - 操作少量数据，适用 String
  - 单线程字符串缓冲区下操作大量数据，适用 StringBuilder
  - 多线程字符串缓冲区下操作大量数据，适用 StringBuffer

##### 自动拆箱装箱

##### 在一个静态方法中调用非静态成员为什么是非法的？

​	静态方法随类的加载而加载，可以不通过对象进行调用，所以静态方法不能调用非静态变量，也不可以访问非静态变量成员。

##### 在Java中定义一个不做事且没有参数的构造方法的作用

​	构造方法主要是完成对类对象的初始化工作。Java在执行<font color=ff00ff>子类构造方法</font><font color=ff00>之前</font>，如果没有`super()`调用父类特定的方法，则会调用<font color=ff00ff>父类没有参数的构造方法</font>帮助子类做初始化工作,如果不定义而子类构造方法有没有 super() 来调用父类特定的构造方法，则编译发生错误。

![](.\..\..\pictures\Java\J2EE\父类中无参构造的作用.jpg)

> 构造方法特征：
>
> ​	名字和类相同
>
> ​	没有返回值，不能用void申明
>
> ​	生成类对象时自动执行，无需调用 

##### import java和javax有什么区别

​	一开始的 Java API 所必需的包都是 java 开头的，javax 只是扩展 API 包开始用。后来逐渐成为 Java API 一部分，只因为合并包太繁琐，最终决定 javax 成为标准 API 的一部分。

##### 接口和抽象类

- 接口方法默认 public，所有方法不能实现(<font color=ff00ff>Java 8开始，接口可以有默认实现</font>)，抽象类可以有非抽象的方法。

- 接口中除了 static，final 变量，不能有其他变量，而抽象类则不一定
- 一个类可以实现多个接口，但是只能实现一个抽象类。一个接口可以继承多个接口
- 接口方法默认修饰符是 public，抽象方法可以有 public、protected 和 default 这些修饰符（抽象方法就是为了被重写所以不能使用 private 关键字修饰）
- 接口是对行为的抽象，是一种行为的规范；而抽象是对类的抽象，是一种模板设计

##### 成员变量和局部变量

- 成员变量属于类，局部变量在方法中定义或者是方法的参数。
- 成员变量可以被 public，private，static 等修饰，局部变量不能被访问控制修饰符和 static 修饰，但都能为final修饰
- 如果成员变量是使用`static`修饰的，那么这个成员变量是属于类的，否则属于实例。而对象存在于堆内存，局部变量则存在于栈内存
- 成员变量是对象的一部分，它随着对象的创建而存在，而局部变量随着方法的调用而自动消失
- 成员变量如果没有被赋初值:则会自动以类型的默认值而赋值（一种情况例外:<font color=ff00ff>被 final 修饰的成员变量也必须显式地赋值</font>），而局部变量则不会自动赋值

##### 对象实例和对象引用

- new创建对象实例（<font color=ff00ff>对象实例在堆内存中</font>），对象引用指向对象实例（<font color=ff00ff>对象引用存放在栈内存中</font>）
- 一个对象引用可以指向0个或1个对象（一根绳子可以不系气球，也可以系一个气球）;一个对象可以有n个引用指向它（可以用n条绳子系住一个气球）

##### 静态方法和实例方法

- 外部调用静态方法，可以`类名.方法名`调用，也可以`对象名.方法名`，而实例方法只可以`对象名.方法名`调用
- 静态方法在访问本类的成员时，只允许访问静态成员（即静态成员变量和静态方法），实例方法则无此限制

##### <font color=ff0000>==，equals，hashCode(*重要)</font>

- **==**

  判断两个对象是否为同一个对象

  - 基本数据类型：比较值
  - 引用数据类型：比较内存地址

- **equals**

  - 没有重写 equals() 方法：等价于`==`比较两个对象
  - 重写 equals() 方法：覆盖 equals() 方法来比较两个对象的<font color=ff00ff>内容</font>是否相等；若它们的内容相等，则返回 true 

  ~~~Java
  public class test1 {
      public static void main(String[] args) {
          String a = new String("ab"); // a 为一个引用
          String b = new String("ab"); // b为另一个引用,对象的内容一样
          String aa = "ab"; // 放在常量池中
          String bb = "ab"; // 从常量池中查找
          if (aa == bb) // true
              System.out.println("aa==bb");
          if (a == b) // false，非同一对象
              System.out.println("a==b");
          if (a.equals(b)) // true
              System.out.println("aEQb");
          if (42 == 42.0) { // true
              System.out.println("true");
          }
      }
  }
  ~~~

  > - String 中的 equals 方法是被重写过的，因为 object 的 equals 方法是比较的对象的内存地址，而 String 的 equals 方法比较的是对象的值。
  > - 当创建 String 类型的对象时，虚拟机会在常量池中查找有没有已经存在的值和要创建的值相同的对象，如果有就把它赋给当前引用。如果没有就在常量池中重新创建一个 String 对象

  ~~~java
  @Override
      public boolean equals(Object obj) {
          if (obj == null) {
              return false;
          }
  
  //如果是同一个对象返回true，反之返回false  
          if (this == obj) {
              return true;
          }
  
          //判断是否类型相同  
          if (this.getClass() != obj.getClass()) {
              return false;
          }
  
          Person person = (Person) obj;
          return name.equals(person.name) && age == person.age;
      }
  ~~~

- **hashCode**

  > 面试官可能会问你：“你重写过 hashCode 和 equals 么，为什么重写 equals 时必须重写 hashCode 方法？”






























































[^hotSpot]:采用惰性评估(Lazy Evaluation)的做法，根据二八定律，消耗大部分资源的小部分代码，也就是JIT需要编译的部分。JVM会根据每次执行情况做出一些优化，因此执行次数越多速度越快。JDK 9引入了AOT(Ahead of Time Compilation)编译模式，直接将字节码编译成机器码，避免了JIT预热等开销。JDK支持分层编译和AOT协作使用。编译质量上，AOT不如JIT。

[^变量]: 每次只能保存一个数据，必须明确保存的数据类型