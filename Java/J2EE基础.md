### 学习/复习笔记

#### Java基础语法

##### 计算机基础

- **二进制**

  二进制系统中，每个0或1就是1位，叫做bit(比特)

- **字节**

  8个bit 0000-0000 表示1个字节，写成 *1 byte* 或者 *1 B*

  8 bit = 1 B

  1024 B = 1 KB

##### [JVM](#JVM,JDK,JRE的解释)

##### DOS命令运行

- javac Java文件名.java
- java 类名

##### 编译和运行

- **编译**

  javac编译器将Java源文件翻译成JVM认识的class文件

- **运行**

  JVM运行编译通过的class文件

##### main方法

​	称为主方法，写法固定格式不可更改，是程序的入口点，无论多少程序，JVM都会从main方法开始执行。

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

  ![1579093176306](E:\Java\new_Java_Study\daily_notes\pictures\自动类型转换示意图.png)

  **转换规则：**

  ​	byte,short,char --> int --> long --> float -->double

- **强制类型转换**

  将取值范围大的类型强制转换为取值范围小的类型
  
  ![1579437664316](E:\Java\new_Java_Study\daily_notes\pictures\强制类型转换示意图.png)
  
  <font color='ff0000'>注：</font>
  
  - 浮点转为整数，直接取消小数点，会造成数据损失精度
  - int强制转为short，砍掉两个字节，会造成数据丢失
  
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
  System.out.println((double)(i/1000) + getType((double)(i/1000)));//1.0 class java.lang.Double
  //2
  System.out.println((double)i/1000 + getType((double)i/1000));//1.234 class java.lang.Double
  System.out.println((double)i/(double) 1000 +getType((double)i/(double)1000) );//1.234 class java.lang.Double
  double d = 1234;
  //3
  System.out.println(i/1000 +getType(i/1000));//1 class java.lang.Integer
  System.out.println(i/(double)1000 + getType(i/(double)1000));//1.234 class java.lang.Double
  
  DecimalFormat df = new DecimalFormat("0.00");
  String num = df.format((float)i/1000);
  System.out.println(num);//1.23
  
  float f = new BigDecimal((float)i/1000).setScale(2,BigDecimal.ROUND_HALF_UP).floatValue();
  System.out.println(f);//1.23
  float f1 = new BigDecimal((float)i/1000).floatValue();
  System.out.println(f1);//1.234
  ~~~

  - 1: i/1000时，i是int类型，之后对(i/1000)的结果 1进行double强制转换，1转化为1.0
  - 2: i/1000时，强制转化为double类型，在进行计算时，始终为double类型
  - 3: i 与整数1000 相除时，会先强制提升为int再进行计算

### 问题总结

##### 环境变量path和classpath的区别

**path**：操作系统查找可执行文件的目录

**classpath**： java.exe查找class文件的目录，现在一般不设置

##### 面向对象和面向过程的区别

- **面向对象：**易维护，易复用，易扩展，可以设计出低耦合系统，但是性能比面向过程低
- **面向过程：**比面向对象性能好，在单片机，嵌入式，Linux/Unix一般采用面向过程

> **为什么说面向过程性能比面向对象高？**
>
> 类调用时需要实例化，消耗资源。但是面向后成也需要分配资源，计算内存偏移量，主要原因是Java是半编译语言，执行代码不是能直接被CPU执行的二进制机械码。而面向过程大都是直接编译成机械码在电脑执行。<font color=ff00ff>*编译的复杂度不同导致执行时间不同*</font>

##### Java特点

* **面向对象：**封装，继承，多态
* **平台无关性：**JVM实现平台无关性
* **可靠性和安全性**
* **支持网络编程：**Java本就是为了简化网络编程设计的语言
* **编译与解释并存**

##### JVM,JDK,JRE的解释

- **JVM**

  Java虚拟机，运行Java字节码的虚拟机。目的是使用相同的字节码，在不同的系统实现，会给出相同的结果。<font color=ff00ff>*一次编译，随处实现*</font>

> **采用字节码的好处？**
>
> JVM可以理解的代码叫做字节码（.class文件），只面向虚拟机。Java通过字节码方式，在解决了传统解释型语言执行效率低问题的同时，又保留了解释性语言的可移植性。

![Java程序运行过程](E:\Java\new_Java_Study\daily_notes\pictures\Java 程序运行过程.png)

.class文件->机器码这步，JVM类加载器加载字节码文件，解释器进行逐行解释执行，但是有些代码会被经常性调用(热点代码[^hotSpot])，后来引进了JIT编译器(运行时编译)。当JIT完成第一次编译，会将字节码对应的机器码保存下来，下次可以直接使用。Java是编译与解释共存的语言的原因。

- **JDK**

  Java Development Kit,Java开发工具，拥有JRE所有用的一切，还有编译器(javac)和工具(javadoc和jdb)，能够创建和编译程序。

- **JRE**

  Java Runtime Environment,Java运行时环境，运行已编译Java程序的所有内容，包括JVM,Java类库，Java命令。

JDK>JRE>JVM

> 在使用JSP部署Web应用时，虽然只是在应用程序服务器中运行Java程序，但是程序服务器会将JSP转换为Java Servlet，并且需要使用JDK来编译Servlet。

##### Oracle JDK 和 OpenJDK

- OpenJDK 是一个参考模型并且是完全开源的，而Oracle JDK是OpenJDK的一个实现，并不是完全开源的；
- Oracle JDK 比 OpenJDK 更稳定。OpenJDK和Oracle JDK的代码几乎相同，但Oracle JDK有更多的类和一些错误修复。
- 在响应性和JVM性能方面，Oracle JDK与OpenJDK相比提供了更好的性能；
- Oracle JDK根据二进制代码许可协议获得许可，而OpenJDK根据GPL v2许可获得许可。

##### Java和C++的区别

- 都是面向对象的语言，都支持封装、继承、多态
- Java不支持指针直接访问内存，程序内存更安全
- Java的类是单继承的，C++支持多重继承，但Java接口可以多继承
- Java有内存管理机制，不需要手动所释放无用内存

##### 应用程序和小程序

- 一个程序可以有多个类，但是只能有一个主类
  - 应用程序的主类，指包含main()方法的类
  - 小程序主类，是继承系统类JApplet或者Applet的子类
- 应用程序主类不一定要是public，小程序主类必须是public
- 应用程序从主线程启动(main方法)，applet小程序没有main方法，主要潜在浏览器页面调用init()或者run()来启动

##### 字符型常量(Char)和字符串常量(String)的区别
|              |                     Char                     |                  String                  |
| :----------: | :------------------------------------------: | :--------------------------------------: |
|  **形式上**  |             单引号引起的一个字符             |          双引号引起的若干个字符          |
|  **含义上**  | 相当于一个整型值(ASCII值),可以参加表达式运算 | 代表一个地址值(字符串在内存中的存放位置) |
| **内存大小** |                   2个字节                    |                若干个字节                |


![img](E:\Java\new_Java_Study\daily_notes\pictures\类型所占存储空间.jpg)

































[^hotSpot]:采用惰性评估(Lazy Evaluation)的做法，根据二八定律，消耗大部分资源的小部分代码，也就是JIT需要编译的部分。JVM会根据每次执行情况做出一些优化，因此执行次数越多速度越快。JDK 9引入了AOT(Ahead of Time Compilation)编译模式，直接将字节码编译成机器码，避免了JIT预热等开销。JDK支持分层编译和AOT协作使用。编译质量上，AOT不如JIT。

[^变量]: 每次只能保存一个数据，必须明确保存的数据类型
