大纲

1.1 ==，equals，hashCode介绍

- ==：比较栈内存中的值

  - 基本数据类型比较的就是自己的值

  - 引用数据类型比较的就是物理地址，即下图的 a == b -->比较的是指向堆内存的地址值，及 0x01 和 0x02两个值比较
  ![==比较的值](.\..\pictures\articles\==，equals,hashcode\==比较的值.png)


- equals：Object 类的方法

  ~~~java
  public class Object {
      ...
      public boolean equals(Object obj) {
          return (this == obj);
      }
      ...
  }
  ~~~

  ​        通过以上源码我们可以看到 Object 类的 equals 方法的作用和 == 相等，它提供了<font color = ff00ff>最具辨别力的可能等价关系</font>，及比较地址值，<font color=00aaff>判断是否为同一个对象</font>。但是要比较具体的值也就是堆存储的值，就需要重写 equals(比如 String 类，[String-equals源码](#String-equals 源码)附在文底) 
  
  equals 也对非空对象提供了五个等价关系：
  
  1. 自反性：`x.equals(x) `-> true
  2. 对称性：`x.equals(y)` <-> `y.equals(x)`
  3. 可传递性：`x.equals(y)`,`y.equals(z)` -> `x.equals(z)`
  4. 一致性：如果比较值没有修改过，无论执行多少次操作，`x.equals(y)`的值都不会改变
  5. `x.equals(null)` -> false
  
  > 值得一提的是，无论 equals 方法是否重写，通常都需要对 hashcode 方法进行重写，这样才能够保证 equals 和 hashcode 的[通用约定](#通用约定)（该约定在下文 hashcode 中介绍），主要是保证 equals 相等的值拥有相同的 hashcode

- hashcode：系统本地方法，非java 语言实现,返回对象的 hash 值

  ~~~java
  public class Object {
      ...
      public native int hashCode();
      ...
  }
  ~~~

  对于 hashcode，有一个<a name="通用约定">通用约定</a>

  1. 只要进行 equals 比较的对象没有修改，则无论调用多少次 hashcode 方法，返回的都是一个值
  2. <font color=ff0000>如果两个值 equals 方法 返回 true，则 hashcode 一定相等</font>--> 重写 equals 必须重写 hashcode 的主要原因
  3. 如果两个值equals 方法 返回 false，则 hashcode 可以相同可以不同

  > 在实现 hashcode 方法时，尽可能保证不同的对象返回不同的 hash 值，但不是必须的

1.2 比较类型

- A: ==	

- - a: 基本数据类型
  - b: 引用类型（User 类）

- B: equals

- - a: 不重写（User 类）
  - b: 重写（UserOverride 类）

- C: hashCode

- - a: 不重写（User 类）
  - b: 重写（UserOverride 类）

1.3 比较方案

- == 自比较
- equals 自比较
- hashCode 自比较
- ==，equals（2*3 种）
- equals，hashcode（2*2*3 种）

> 注：2--重写不重写；3--对象创建：=赋值，new，clone



<a name="String-equals 源码">String-equals 源码</a>

 ~~~java
  public final class String {
      ...
  	public boolean equals(Object anObject) {
          if (this == anObject) {
              return true;
          }
          if (anObject instanceof String) {
              String anotherString = (String)anObject;
              int n = value.length;
              if (n == anotherString.value.length) {
                  char v1[] = value;
                  char v2[] = anotherString.value;
                  int i = 0;
                  while (n-- != 0) {
                      if (v1[i] != v2[i])
                          return false;
                      i++;
                  }
                  return true;
              }
          }
          return false;
      }
      ...
  }
 ~~~
