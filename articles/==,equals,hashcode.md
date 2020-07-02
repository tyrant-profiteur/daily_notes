大纲

1.1 ==，equals，hashCode介绍

1.2 比较类型

- A: ==	

- - a: 基本数据类型
  - b: 引用类型（User 类）

- B: equals

- - a: 不重写（User 类）
  - b: 重写（String 类，User 类）

- C: hashCode

- - a: 不重写（User 类）
  - b: 重写（String 类，User 类）

1.3 比较方案

- == 自比较
- equals 自比较
- hashCode 自比较
- ==，equals（2*3 种）
- equals，hashcode（2*2*3 种）

> 注：2--重写不重写；3--对象创建：=赋值，new，clone