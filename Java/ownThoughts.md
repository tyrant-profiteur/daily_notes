#### 对定义接口接受实现类的思考

~~~Java
private IAccountDao accountDao = new AccountDaoImpl();
~~~

​	这行代码，相信大家很容易理解，`IAccountDao`,持久层的接口，`AccountDaoImpl`，接口的实现类，定义一个接口接受该接口的实现类。为什么大家都按照这个思路写(学完Spring的，用依赖注入一个意思，不做描述)，而不是写成`AccountDaoImpl accountDao = new AccountDaoImpl()`。

