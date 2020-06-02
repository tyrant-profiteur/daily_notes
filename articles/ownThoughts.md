

#### 事务控制小结

​	操作数据库，就涉及到事务控制。如何确保事务的原子性是这次重点讨论的话题。

​	在 web 应用中，我们如果不对事务进行控制，那么事务的控制其实是在 Dao 层也就是持久层中，也就是我们每次在 Service 中每成功调用一次 Dao 层中的方法，事务就会提交一次，这其实是很不安全的。如果不报错还好，一旦报错，不好意思，你数据库的数据就完蛋啦！所以我们进行事务控制，都在 Service 层进行，一个服务一个事务，服务结束，整个数据提交，否则全部回滚。为了突出文章重点——事务控制，这里只在单线程的环境下做对比（其实多线程我也不熟啊现在🙃）

​	在 Service 中控制事务的核心为：<font color=ff00ff>线程和事务进行绑定</font>，也就是一个方法执行完后事务才提交(表达欠缺，)

​	事务的大体操作流程是：①开始事务-->②执行操作-->③提交事务-->⑤释放资源，如果②执行操作报错，则执行④事务回滚，Service 基础代码如下：

~~~java
try{
    //1.开启事务
    transactionManager.begin();
    //2.执行操作
    accountDao.updateAccount(account);
    //3.提交事务
    transactionManager.commit();
}catch (Exception e){
    //4.回滚事务
    transactionManager.rollback();
}finally {
    //5.释放资源
    transactionManager.release();
}
~~~

​	`TransactionManager.class`为控制事务的类，下面会有详细代码