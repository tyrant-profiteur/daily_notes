redis，mq,dubbo,分布式,elasticsearch

java并发，mysql,网络，JDK集合，jvm，Spring源码，tomcat，lunix，系统设计，生产实践

**hashMap对hash算法和寻址算法的优化**

hash算法：对key的hash值和hash值右移16位的值进行异或运算(^,两个数不同则为1)，对数组长度取模，得到index，高十六位和第十六位进行异或：为了低十六位有高低十六位的特征，尽量避免一些hash值后续冲突

寻址算法：得到的异或运算的值，对**数组取模**的过程进行优化

hash&(n -1) 跟对hash对n取模的效果是一样的，但是运算性能比hash取模高很多（数组的长度是2的n次方，hash对n取模 -> hash&(n-1)）

与（n-1）进行与运算，因为n-1比较小，前十六位可能都不参加与运算，所以为了保证数据的可靠性 ，需要将hash值的高十六位和低十六位进行异或 

hash值的优化是为了寻址算法的后续优化做准备，避免两个数的hash值差不多

例：

1111 1111 1111 1111 1111 1010 0111 1100 —高低异或运算—》1111 1111 1111 1111 0000 0101 1000 0011

1111 1111 1111 1110 1111 1010 0111 1100 —高低异或运算—》1111 1111 1111 1110 0000 0101 1000 0010

和（n-1）进行与运算，算出来的值就会不一样

**hashMap冲突**

会在这个位置挂一个链表，这个链表里放入多个元素

get，如果发现这个位置挂了一个链表，遍历链表，找到自己要的key-value

如果链表长度太大，遍历花费时间会很长O(n)

优化：如果链表长度达到一定的长度（拉链度：8），就会将链表转化为红黑树，时间复杂度O(logn)

**hashMap扩容**

当数组长度满了之后，hashMap每次扩容到原来的两倍，会有一个reHash的过程，原来数组里放的数，是跟 n - 1 进行与运算的，现在要重新跟 2*n - 1进行与运算

数组长度 = 16

n-1	     0000 0000 0000 0000 0000 0000 0000 1111

hash1	1111 1111 1111 1111 0000 1111 0000 0101

&		0000 0000 0000 0000 0000 0000 0000 0101	index=5



n-1	     0000 0000 0000 0000 0000 0000 0000 1111

hash2	1111 1111 1111 1111 0000 1111 0001 0101

&		0000 0000 0000 0000 0000 0000 0000 0101	index=5

扩容到32之后，重新对每个hash值进行寻址

数组长度 = 32

n-1	     0000 0000 0000 0000 0000 0000 0001 1111

hash1	1111 1111 1111 1111 0000 1111 0000 0101

&		0000 0000 0000 0000 0000 0000 0000 0101	index=5



n-1	     0000 0000 0000 0000 0000 0000 0001 1111

hash2	1111 1111 1111 1111 0000 1111 0001 0101

&		0000 0000 0000 0000 0000 0000 0001 0101	index=21

 判断二进制结果是否多出一个bit的1，如果没有，就是原来的index，如果有，则位置位index+oldCap，避免rehash时候 ，用每个hash对新数组长度取模的性能低问题



## 并发

synchronized实现原理，CAS无锁化原理，AQS是什么，lock锁，ConcurrentHashMap的分段加锁原理，线程池原理，java内存模型，volatile是什么，对java并发包的理解

**synchronized**

synchronized 关键字底层原理跟 jvm 和 monitor 有关

用到 synchronized 关键字，底层编译后虎仔jvm指令中有monitorenter 和monitorexit 两个指令

~~~
monitorenter
//代码对应的指令
monitorexit
~~~

每个对象都有一个对应的monitor，一个类的Class也有一个monitor，如果要对对象加锁，必须获取这个对象关联的monitor的lock锁

monitor有个计数器，线程获取monitor锁，计数器加1

支持可重用加锁

~~~
synchronized(myObject){
    //代码
    synchronized(myObject){
    //代码
	}
}
~~~

**CAS的理解和原理**

synchronized对方法加锁

~~~
public Class MyObject{
    int i = 0;
    //对myObject对象
    //同一时间，只有一个线程可以进入此方法
    public synchronized void increment(){
        i++;
    }
}
MyObject myObject = new Myobject();
myObject.increment();
~~~

用此方法，线程执行是串行的，效率不高，需要排队 执行

~~~
public Class MyObject{
    AtomicInteger i = new AtomicInteger(0);//底层基于CAS实现的
    //不需要synchronized，线程 也是安全的
    public void increment(){
        i.incrementAndGet();
    }
}
MyObject myObject = new Myobject();
myObject.increment();
~~~

CAS：compare and set

CAS的操作是原子的，针对硬件级别,主要用于累加操作 

**ConcurrentHashMap实现线程安全的底层原理**

HashMap的底层本身是一个数组+链表(红黑树)

~~~
HashMap map = new HashMap();
//多个线程过来
//线程1过来，要 put 的位置为数组[5]，线程2过来，要 put 的位置为数组[21]
synchronized(map){
    map.put(xxxxx,xxxx);
}
~~~

如果加锁，明显不好，因为两个线程要 put 的位置并没有关系，但是一定加锁

为此，推出了ConcurrentHashMap，默认实现了线程安全的，继承自AbstractMap，和HashMap是同级关系

JDK 1.7 及以前，采用的是分段加锁

[数组1],[数组2],[数组3] -》每个数组都对应一个锁

线程1过来，要 put 的位置为数组1[5]，线程2过来，要 put 的位置为数组2[21]

互相之间的锁是不一样的

JDK 1.8 以后，做了一些优化，锁粒度细化

一个大的数组，数组每个元素进行put操作，都有一个不同的锁

如果两个线程都在对数组[5]进行put，采取的是**CAS**策略，如果刚刚有人放进去值了，就需要对这个位置进行基于链表+红黑树来进行处理，synchronized(数组[5])，加锁，基于链表/红黑树，在这个位置插进去自己的数据

**AQS的理解和实现原理**

AQS：Abstract Queue Synchronizer 抽象队列同步器

多线程同时访问一个共享数据，synchronized，CAS，ConcurrnetHashMap，lock等

**synchronized 和 lock 的区别**？？（课后）

lock的底层基于AQS

Semaphore、其他一些并发包

~~~
ReentrantLock lock = new ReentrantLock();//默认的是非公平锁
//ReentrantLock lock = new ReentrantLock(true);//公平锁
//多线程过来尝试
lock.lock();
//代码
lock.unlock();
~~~

AQS里有个变量 state = 0，线程上锁的时候，CAS机制对state进行修改为1，加锁线程更新为当前线程，加锁失败的线程进入等待队列，将自己挂起。成功的线程执行完之后，将加锁线程置为 null，state = 0，唤醒等待队列中队列头的线程

**线程池底层原理**

构建一个线程池，让他们执行各种各样的任务，不要销毁自己，等待线程。

频繁的创建线程，销毁线程，影响性能

~~~
Executorservice threadPool = Executor.newFixedThreadPool(3);//3->corePoolSize
threadPool.submit(new Callable(){
    public void run(){
        
    };
});
~~~

创建一个线程池，会自带一个队列（放任务的），此时线程池里的线程数为0；然后当有任务来的时候，先判断线程池的数是否超过corePoolSize数，如果没超，就会创建一个线程。当任务结束后，会释放线程，该线程会在队列头进行阻塞，等待获取任务。**当线程数超过corePoolSize后，任务会直接进入队列。**

当不停的提交任务，会把线程池中的线程数量创建到corePoolSize数，然后再进入队列

fixed，队列， LinkedBlockingQueue，无界阻塞队列

**线程池的核心配置参数**

创建线程池，如果不用fixed之类的线程池，自己完全可以通过构造函数创建自己的线程池，参数：corePoolSize，maximumPoolSize，keepAliveTime，queue

~~~java
return new ThreadpoolExecutor(nThreads,nThreads,0L,TimeUnit.MILLISSECONDS,new LinkedBlockingQueue<Runabble>())
~~~

corePoolSize:3

maximumPoolSize:Integer.MAX_VALUE

keepAliveTime:60s

new ArrayBlockingQueue<Runnable>(200)

作用：线程池中的线程数量达到corePoolSize之后，任务直接进队列，当队列满了之后，达到 ArrayBlockingQueue的数量，会创建新的线程来处理任务，但是不能超过maximumPoolSize的数量。当新创建出的线程没有任务处理后，keepAliveTime之后自动销毁。

如果额外线程创建完了去处理任务，队列还是满了，有新的任务来了，只能reject掉，可以传入 RejectedExecutionHandler

- AbortPolicy：抛异常
- DiscardPolicy：扔掉任务
- DiscardOldestPolicy：抛弃最旧的任务
- CallerRunsPolicy
- 自定义

根据自己的情况，考虑 corePoolSize 的数量，队列类型，最大线程数量，拒绝策略，线程释放时间来自定义线程池

一般常用的：

- fixed，不创建新的线程，队列无限
- 队列有限，满了之后创建新的线程无线

**线程池中使用无界阻塞队列会怎样**

面试题：在远程服务异常的情况下，使用无界阻塞队列，会导致内存异常飙升吗？

调用服务失败，会等超时，导致队列里的任务积压，因为队列无界，会越来越大。队列越来越大，必然导致内存飙升起来，还会导致OOM内存溢出

**线程池中的队列满了之后，会发生什么**

