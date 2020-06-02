

# Java基础

#### 1.面向的对象和面向过程的区别

|      | **面向对象**           | **面向过程**             |
| ---- | ---------------------- | ------------------------ |
| 优点 | 易维护，易复用，易扩展 | 性能高，类调用需要实例化 |

---

#### 2.Java 特点

- 简单
- 面向对象（封装继承多态）
- 平台无关性（JVM）
- 可靠性
- 安全性
- 支持多线程（ C++ 不支持）
- 支持网络编程
- 编译与解释并存

---

#### 3.字节码

​	JVM 可以理解的代码就叫做字节码（即扩展名为 .class 的文件），只面向虚拟机。

- .class->机器码

  jvm 类加载器首先加载字节码文件，然后通过解释器逐行解释执行，这种方式的执行速度会相对比较慢。有些方法和代码块是经常需要被调用的，也就是所谓的热点代码，所以后面引进了<font color=ff0000> JIT 编译器</font>，JIT 属于运行时编译。当 JIT 编译器完成第一次编译后，其会将字节码对应的机器码保存下来，下次可以直接使用。（Java 是编译与解释共存的语言）

- HotSpot 热点代码

  采用了惰性评估(Lazy Evaluation)的做法，JVM 会根据代码每次被执行的情况收集信息并相应地做出一些优化，因此执行的次数越多，它的速度就越快。

- AOT(Ahead of Time Compilation)编译模式

  直接将字节码编译成机器码，这样就避免了JIT预热等各方面的开销。质量不如 JIT 编译器。

---

#### 4.Java 和 C++ 的区别

- 相同

  都面向对象，支持封装继承多态

- 不同
  - Java 不提供指针直接访问内存，相对安全
  - Java 类单继承，C++ 多继承；Java 接口可以多继承
  - Java 有自动内存管理机制，不用手动释放资源

---

#### 5.字符型常量和字符串常量的区别

|        | 字符型                 | 字符串               |
| ------ | ---------------------- | -------------------- |
| 形式上 | 单引号引起的一个字符   | 双引号引起的多个字符 |
| 含以上 | 一个整型值（ASCII 值） | 一个地址值           |
| 内存   | 2个字节                | 若干字节             |

![img](E:\Java\new_Java_Study\daily_notes\pictures\类型所占存储空间.jpg)

---

#### 6.重载(overload)和重写(override)

> Java允许重载任何方法，要完整描述一个方法，需要指出方法名以及参数类型，叫做方法的标签。返回类型不是方法签名的一部分。也就是说不能有两个名字相同，参数类型相同返回值却不同的方法。

- **重写注意事项**

  子类对父类允许访问的对象进行重新编写。

  - 方法名、参数列表必须相同
  - 返回值范围小于等于父类
  - 抛出异常范围小于等于父类
  - 访问修饰符范围大于等于父类

#### 7.三大特性

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

---

#### 8.String，StringBuffer，StringBuilder

- **可变性**

  - String类使用final关键字修饰字符数组来保存字符串，`private final char value[]`，所以String不可变

  - StringBuffer 和 StringBuilder 都继承自`AbstractStringBuilder`,也是使用 char[] value 保存字符串，但是可以修改

    ```java
    abstract class AbstractStringBuilder implements Appendable, CharSequence {
        char[] value;
        int count;
        AbstractStringBuilder() {
        }
        AbstractStringBuilder(int capacity) {
            value = new char[capacity];
        }	
    ```

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

---

#### 9.在Java中定义一个不做事且没有参数的构造方法的作用

​	构造方法主要是完成对类对象的初始化工作。Java在执行<font color=ff00ff>子类构造方法</font><font color=ff00>之前</font>，如果没有`super()`调用父类特定的方法，则会调用<font color=ff00ff>父类没有参数的构造方法</font>帮助子类做初始化工作,如果不定义而子类构造方法有没有 super() 来调用父类特定的构造方法，则编译发生错误。

> 构造方法特征：
>
> ​	名字和类相同
>
> ​	没有返回值，不能用void申明
>
> ​	生成类对象时自动执行，无需调用 

---

#### 10.接口和抽象类

- 接口方法默认 public，所有方法不能实现(<font color=ff00ff>Java 8开始，接口可以有默认实现</font>)，抽象类可以有非抽象的方法。
- 接口中除了 static，final 变量，不能有其他变量，而抽象类则不一定
- 一个类可以实现多个接口，但是只能实现一个抽象类。一个接口可以继承多个接口
- 接口方法默认修饰符是 public，抽象方法可以有 public、protected 和 default 这些修饰符（抽象方法就是为了被重写所以不能使用 private 关键字修饰）
- 接口是对行为的抽象，是一种行为的规范；而抽象是对类的抽象，是一种模板设计

---

#### 11.成员变量和局部变量

- 成员变量属于类，局部变量在方法中定义或者是方法的参数。
- 成员变量可以被 public，private，static 等修饰，局部变量不能被访问控制修饰符和 static 修饰，但都能为final修饰
- 如果成员变量是使用`static`修饰的，那么这个成员变量是属于类的，否则属于实例。而对象存在于堆内存，局部变量则存在于栈内存
- 成员变量是对象的一部分，它随着对象的创建而存在，而局部变量随着方法的调用而自动消失
- 成员变量如果没有被赋初值:则会自动以类型的默认值而赋值（一种情况例外:<font color=ff00ff>被 final 修饰的成员变量也必须显式地赋值</font>），而局部变量则不会自动赋值

---

#### 12.对象实例和对象引用

- new创建对象实例（<font color=ff00ff>对象实例在堆内存中</font>），对象引用指向对象实例（<font color=ff00ff>对象引用存放在栈内存中</font>）
- 一个对象引用可以指向0个或1个对象（一根绳子可以不系气球，也可以系一个气球）;一个对象可以有 n 个引用指向它（可以用 n 条绳子系住一个气球）

---

#### 13.静态方法和实例方法

- 外部调用静态方法，可以`类名.方法名`调用，也可以`对象名.方法名`，而实例方法只可以`对象名.方法名`调用
- 静态方法在访问本类的成员时，只允许访问静态成员（即静态成员变量和静态方法），实例方法则无此限制

---

#### 14.<font color=ff0000>==，equals，hashCode(*重要)</font>

- **==**

  判断两个对象是否为同一个对象

  - 基本数据类型：比较值
  - 引用数据类型：比较内存地址

- **equals**

  - 没有重写 equals() 方法：等价于`==`比较两个对象
  - 重写 equals() 方法：覆盖 equals() 方法来比较两个对象的<font color=ff00ff>内容</font>是否相等；若它们的内容相等，则返回 true 

  ```Java
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
  ```

  > - String 中的 equals 方法是被重写过的，因为 object 的 equals 方法是比较的对象的内存地址，而 String 的 equals 方法比较的是对象的值。
  > - 当创建 String 类型的对象时，虚拟机会在常量池中查找有没有已经存在的值和要创建的值相同的对象，如果有就把它赋给当前引用。如果没有就在常量池中重新创建一个 String 对象

  ```java
  @Override
      public boolean equals(Object obj) {
          if (obj == null) {
              return false;
          }
  
  		//如果是同一个对象返回true 
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
  ```

- **hashCode**

  获取哈希码，也称为散列码；它实际上是返回一个int整数,是确定该对象在哈希表中的索引位置

  1. 如果两个对象相等，则 hashcode 一定也是相同的
  2. 两个对象相等,对两个对象分别调用 equals 方法都返回 true
  3. 两个对象有相同的 hashcode 值，它们也不一定是相等的
  4. **equals 方法被覆盖过，则 hashCode 方法也必须被覆盖**
  5. hashCode() 的默认行为是对堆上的对象产生独特值。如果没有重写 hashCode()，则该 class 的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）

  > 其实简单的说就是为了保证同一个对象，保证在equals相同的情况下hashcode值必定相同

---

#### 15.线程，程序、进程的基本概念

​	**进程**是**程序**的一次执行过程,一个**进程**包含大于等于一个**线程**

---

#### 16.线程基本状态

![img](E:\Java\new_Java_Study\daily_notes\pictures\面试\基础\线程状态作用.png)

![1584889948805](E:\Java\new_Java_Study\daily_notes\pictures\面试\基础\线程流程.png)

![img](E:\Java\new_Java_Study\daily_notes\pictures\面试\基础\线程状态.png)

---

#### 17. final 的总结

1. 变量

   - 基本数据类型

     数值一旦在初始化之后便不能更改

   - 引用类型的变量

     对其初始化之后便不能再让其指向另一个对象

2. 类：不能被继承

3. 方法

   - 方法锁定，以防任何继承类修改它的含义

   - 效率

     ~~final方法转为内嵌调用。但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升，~~类中所有的private方法都隐式地指定为final

---

#### 18. Java 异常处理

![1584888577010](E:\Java\new_Java_Study\daily_notes\pictures\面试\基础\异常处理.png)

**Throwable**： 

1. 有两个重要的子类

- **Exception（异常）**:是程序本身可以处理的异常 Exception 类有一个重要的子类 RuntimeException。RuntimeException 异常由Java虚拟机抛出。
  - NullPointerException（要访问的变量没有引用任何对象时，抛出该异常）
  - ArithmeticException（算术运算异常，一个整数除以0时，抛出该异常）
  - ArrayIndexOutOfBoundsException （下标越界异常）
- **Error（错误）** :是程序无法处理的错误

2. 常用方法
   - public string getMessage():返回异常发生时的详细信息
   - public string toString():返回异常发生时的简要描述
   - public string getLocalizedMessage():返回异常对象的本地化信息。使用Throwable的子类覆盖这个方法，可以声称本地化信息。如果子类没有覆盖该方法，则该方法返回的信息与getMessage（）返回的结果相同
   - public void printStackTrace():在控制台上打印 Throwable 对象封装的异常信息

**异常处理**

- try 块

  用于捕获异常。其后可接零个或多个 catch 块，如果没有catch 块，则必须跟一个 finally 块

- catch 块

  用于处理try捕获到的异常

- finally 块

  无论是否捕获或处理异常，finally 块里的语句都会被执行。当在 try 块或 catch 块中遇到 return 语句时，finally 语句块将在方法返回之前被执行

~~~Java
try{
	int a = 30;
	return a;
}finally{
	a = 40;
    System.out.print(a);
}
~~~

​	返回结果为30，但是控制台打印出来是40

- finally 块不会被执行
  - 在 finally 语句块中发生了异常
  - 在前面的代码中用了 System.exit() 退出程序
  - 程序所在的线程死亡
  - 关闭 CPU

---

#### 19. Java序列化中如果有些字段不想进行序列化 怎么办

​	对于不想进行序列化的变量，使用 transient 关键字修饰。

> 阻止实例中那些用此关键字修饰的的变量序列化；当对象被反序列化时，被transient修饰的变量值不会被持久化和恢复。transi ent 只能修饰变量，不能修饰类和方法。

---

#### 20.获取键盘输入的方法

- 通过 Scanner

  ~~~Java
  Scanner input = new Scanner(System.in);
  String s = input.nextLine();
  input.close();
  ~~~

- 通过 BufferedReader

  ~~~java
  BufferedReader input = new BufferedReader(new InputStreamReader(System.in));
  String s = input.readLine();
  ~~~

---

#### 21. List 和 Set 的区别

都继承自 Collection 接口

- List
  - 存放有序，可重复
  - 和数组类似，List可以动态增长，查找元素效率高，插入删除元素效率低，因为会引起其他元素位置改变
- Set
  - 元素无放入顺序，元素不可重复
  - 元素在 set 中的位置是有该元素的 HashCode 决定的
  - 只能用迭代器遍历
  - 检索元素效率低下，删除和插入效率高，插入和删除不会引起元素位置改变

---

#### 22. HashSet 是如何保证不重复的

向 HashSet 中 add ()元素时，判断元素是否存在的依据，不仅要比较hash值，同时还要结合 equles 方法比较。
HashSet 中的 add ()方法会使用 HashMap 的 add ()方法。

~~~java
private static final Object PRESENT = new Object();
private transient HashMap<E,Object> map;
public HashSet() {
map = new HashMap<>();
}
public boolean add(E e) {
return map.put(e, PRESENT)==null;
}
~~~

HashSet 添加进去的值就是作为 HashMap 的 key。所以不会
重复（ HashMap 比较key是否相等是先比较 hashcode 在比较 equals ）