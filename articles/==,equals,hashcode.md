[TOC]



# 1. ==，equals，hashCode介绍

- ## ==：比较栈内存中的值

  - 基本数据类型比较的就是自己的值

  - 引用数据类型比较的就是物理地址，即下图的 a == b -->比较的是指向堆内存的地址值，及 0x01 和 0x02两个值比较
  ![==比较的值](.\..\pictures\articles\==，equals,hashcode\==比较的值.png)


- ## equals：Object 类的方法

  <a name="equals Object类源码">equals Object类源码</a>

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

- ## hashcode：系统本地方法，非java 语言实现,返回对象的 hash 值

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

  > 在实现 hashcode 方法时，尽可能保证不同的对象返回不同的 hash 值，但不是必须的；此方法主要是为了哈希表，例如 `java.util.HashMap ` 准备的

# 2 比较

## 2.1 比较的类

- <a name="User">User 类</a>

  ~~~java
  public class User implements Cloneable{
      private String name;
      private String age;
  
      public User() {}
  
      public User(String name, String age) {
          this.name = name;
          this.age = age;
      }
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public String getAge() {
          return age;
      }
  
      public void setAge(String age) {
          this.age = age;
      }
  
      @Override
      protected Object clone() throws CloneNotSupportedException {
          return super.clone();
      }
  }
  ~~~

- <a name="UserOverride">UseOverride 类</a>--重写了 equals 和 hashcode

  ~~~java
  public class UserOverride implements Cloneable{
      private String name;
      private String age;
  
      public UserOverride() {}
  
      public UserOverride(String name, String age) {
          this.name = name;
          this.age = age;
      }
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public String getAge() {
          return age;
      }
  
      public void setAge(String age) {
          this.age = age;
      }
  
      @Override
      public boolean equals(Object o) {
          if (this == o) return true;
          if (o == null || getClass() != o.getClass()) return false;
          UserOverride userOverride = (UserOverride) o;
          return Objects.equals(name, userOverride.name) &&
                  Objects.equals(age, userOverride.age);
      }
  
      @Override
      protected Object clone() throws CloneNotSupportedException {
          return super.clone();
      }
  
      @Override
    public int hashCode() {
          return Objects.hash(name, age);
    }
  }
  ~~~
  
  - Objects.equals
  
    ~~~java
    public final class Objects {
        public static boolean equals(Object a, Object b) {
            return (a == b) || (a != null && a.equals(b));
        }
    }
    ~~~

## 2.2 比较方案

### 2.2.1 ==	

- - #### a: 基本数据类型
  
    ~~~java
      public class MainFunction {
          public static void main(String[] args){
              //基本类型
              int a = 1,b = 1,c = 2;
              System.out.println(a == b);//true
              System.out.println(a == c);//false
          }
      }
    ~~~
  
  - ### b: 引用类型（[User 类](#User)）
  
    ~~~java
    public class MainFunction {
        public static void main(String[] args){
            //无参构造
            User user = new User();
            user.setName("老王");
            user.setAge("24");
            User user1 = new User();
            user1.setName("老王");
            user1.setAge("24");
            User user2 = user;
            //有参构造
            User user3 = new User("老王","24");
            //clone
            User user4 = (User)user.clone();
            System.out.println(user == user1);//false
            System.out.println(user == user2);//true
            System.out.println(user == user3);//false
            System.out.println(user == user4);//false
        }
    }
    ~~~
    
  
  ![image-20200710165443533](.\..\pictures\articles\==，equals,hashcode\User类==比较.png)
  
  不管是有参还是无参构造方法，new 出来对象总会在堆中重新创建一个对象出来，栈中存相关的地址，== 比较的为地址值，所以 user，user1，user3 使用 == 比较的结果都为 false；
  
  user4 使用 clone 方法获取，也是在堆中重新创建一个对象，栈中存新创建的地址，所以user 和 user4 使用 == 比较的结果为 false；
  
  user2 直接通过 = 赋值，即将 user 栈中存的地址值赋给 user2，user2指向 user 在堆中的对象，因为 == 比较的为栈中的值，所以 user 和 user2 使用 == 比较的结果为 ture；

### 2.2.2 equals

- - #### a: 不重写（[User 类](#User)）
  
    因为不重写 equals 方法，使用的是 [Object 类的 equals 方法](#equals Object类源码)，预期结果和 == 比较相等
  
    ~~~java
    public class MainFunction1 {
	        public static void main(String[] args){
            //无参构造
            User user = new User();
            user.setName("老王");
            user.setAge("24");
            User user1 = new User();
            user1.setName("老王");
            user1.setAge("24");
            User user2 = user;
            //有参构造
            User user3 = new User("老王","24");
            //clone
            User user4 = (User)user.clone();
            System.out.println(user.equals(user1));//false
     System.out.println(user.equals(user2));//true
     System.out.println(user.equals(user3));//false
     System.out.println(user.equals(user4));//false
        }
    }
    ~~~
  
    实测结果和预期是一样的
  
  - #### b: 重写（[UserOverride 类](#UserOverride)）
  
    ~~~java
    public class MainFunction1 {
        public static void main(String[] args){
           //无参构造
            UserOverride userOverride = new UserOverride();
            userOverride.setName("老王");
            userOverride.setAge("24");
            UserOverride userOverride1 = new UserOverride();
            userOverride1.setName("老王");
            userOverride1.setAge("24");
            UserOverride userOverride2 = userOverride;
            //有参构造
            UserOverride userOverride3 = new UserOverride("老王","24");
            //clone
            UserOverride userOverride4 = (UserOverride)userOverride.clone();
            System.out.println(userOverride.equals(userOverride1));//true
     System.out.println(userOverride.equals(userOverride2));//true
     System.out.println(userOverride.equals(userOverride3));//true
     System.out.println(userOverride.equals(userOverride4));//true
        }
    }
    ~~~
    
    因为重写了 equals 方法，根据重写的方法，比较的为具体的值而不是地址值，所以之间的相互比较都为 true

### 2.2.3 hashCode

- - ### a: 不重写（[User 类](#User)）
  
    因为不重写 hashCode 方法，使用的是 Object 类的 hashCode 方法
  
    ```java
    public class MainFunction1 {
          public static void main(String[] args){
            //无参构造
            User user = new User();
            user.setName("老王");
            user.setAge("24");
            User user1 = new User();
            user1.setName("老王");
            user1.setAge("24");
            User user2 = user;
            //有参构造
            User user3 = new User("老王","24");
            //clone
            User user4 = (User)user.clone();
     		System.out.println(user.hashCode() == user1.hashCode());//false
            System.out.println(user.hashCode() == user2.hashCode());//true
            System.out.println(user.hashCode() == user3.hashCode());//false
            System.out.println(user.hashCode() == user4.hashCode());//false
        }
    }
    ```
  
  - ### b: 重写（[UserOverride 类](#UserOverride)）
  
    ```java
    public class MainFunction1 {
        public static void main(String[] args){
           //无参构造
            UserOverride userOverride = new UserOverride();
            userOverride.setName("老王");
            userOverride.setAge("24");
            UserOverride userOverride1 = new UserOverride();
            userOverride1.setName("老王");
            userOverride1.setAge("24");
            UserOverride userOverride2 = userOverride;
            //有参构造
            UserOverride userOverride3 = new UserOverride("老王","24");
            //clone
            UserOverride userOverride4 = (UserOverride)userOverride.clone();
            System.out.println(userOverride.hashCode() == userOverride1.hashCode());//true
     System.out.println(userOverride.hashCode() == userOverride2.hashCode());//true
     System.out.println(userOverride.hashCode() == userOverride3.hashCode());//true
     System.out.println(userOverride.hashCode() == userOverride4.hashCode());//true
        }
    }
    ```
  
    因为重写了 hashCode 方法，根据重写的方法，比较的为具体的值的 hash 值，所以之间的相互比较都为 true

## 2.3 总结

==，equals，hashcode 算是基础但是也是重要的知识，很多类，方法都会使用他们，最典型的就是 hashMap 的hashCode 应用。但是万变不离其宗的就是`1.栈和堆分清楚`、`2.方法重写之后实现的功能`。其实看源码，看 JDK 的解释，相信大家都能理解的。

# 3. 其他

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

