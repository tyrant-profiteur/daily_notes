## 网络编程快速入门

### 1. 什么是网络编程

> 通过操作相应 API 调度计算机硬件资源，并利用传输管道（网线）进行数据交互的过程

具体涉及到：网络模型，套接字，数据包

### 2. 7 层网络模型 - OSI

![1593610059144](.\..\..\pictures\Java\socket\7层网络模型-OSI.png)

- 基础层：物理层（Physical），数据链路层（Datalink），网络层（Network）
- 传输层（Transport）：TCP-UDP协议层，Socket
- 高级层：会话层（Session），表示层（Presentation），应用层（Application）

### 3. 网络模型-对应关系

![1593611185722](./../..\pictures\Java\socket\网络模型-对应关系.png)

### 4. 什么 Socket

> Ip 地址与端口的结合协议（RFC 793）
> TCP/IP 协议的相关 API 总称
> 网络 API 的集合实现
> 涵盖了 Stream Socket/Datagram Socket

### 5. Socket 的作用与组成

- 作用：

  在网络传输中用于唯一标示两个端点之间的连接

- 端点：IP + Port

  四个要素：客户端地址/客户端端口；服务端地址/服务端端口

  ![1593612522416](./../..\pictures\Java\socket\Socket 传输原理.png)

### 6. Socket 之 TCP

> TCP 是面向连接的通信协议

通过三次握手建立连接，通讯完成时要拆除连接

由于 TCP 是面向连接的，所以只能用于端到端的通讯

### 7. Socket 之 UDP

> UDP 是面向无连接的通讯协议

UDP 数据包括目的端口号和源端口号信息

由于通讯不需要连接，所以可以实现广播发送，并不局限于端到端

### 8. Client-Server Application

​	TCP/IP 协议中，两个进程间通信的主要模式为：CS模型

- 目的：协同网络中的计算机资源、服务模式、进程间数据共享
- 常见：FTP、SMTP、HTTP

### 9. 代码演示

- 服务端

  ~~~java
  public class Server {
      public static void main(String[] args)throws IOException {
          ServerSocket server = new ServerSocket(2000);
          System.out.println("服务器准备就绪");
          System.out.println("服务器信息：" + server.getInetAddress() + " p:"+ server.getLocalPort());
  
          //等待客户端连接
          for (;;){
              //得到客户端
              Socket client = server.accept();
              //客户端构建异步线程
              ClientHandler clientHandler = new ClientHandler(client);
              //启动线程
              clientHandler.start();
          }
      }
  }
  ~~~

  ~~~java
  class ClientHandler extends Thread {
      private Socket socket;
      private boolean flag = true;
  
      ClientHandler(Socket socket){
          this.socket = socket;
      }
  
      /**
       * 服务器异步处理数据
       */
      @Override
      public void run() {
          super.run();
          System.out.println("新客户端连接：" + socket.getInetAddress()+
                  " p:"+ socket.getPort());
          try{
              //得到打印流，用于数据输出；服务器回送数据使用
              PrintStream socketOutput = new PrintStream(socket.getOutputStream());
              BufferedReader socketInput = new BufferedReader(
                      new InputStreamReader(socket.getInputStream()));
  
              do {
                  //客户端拿到一条数据
                  String str = socketInput.readLine();
                  if ("bye".equalsIgnoreCase(str)){
                      flag = false;
                      //回送
                      socketOutput.println(str);
                  }else {
                      //打印，并回送
                      System.out.println(str);
                      socketOutput.println("回送："+str.length());
                  }
              }while (flag);
              socketInput.close();
              socketOutput.close();
          }catch (Exception e){
              System.out.println("连接异常关闭");
          }finally {
              try {
                  socket.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
          System.out.println("客户端已关闭：" + socket.getInetAddress()+
                  " p:"+ socket.getPort());
      }
  }
  ~~~

- 客戶端

  ~~~java
  public class Client {
      public static void main(String[] args) throws IOException {
          Socket socket = new Socket();
          //超时时间
          socket.setSoTimeout(3000);
          //连接本地 端口号 2000 超时时间3000ms
          socket.connect(new InetSocketAddress(InetAddress.getLocalHost(),2000),3000);
          System.out.println("已发起服务器连接，并进入后续流程");
          System.out.println("客户端信息：" + socket.getLocalAddress()+ " p:"+ socket.getLocalPort());
          System.out.println("服务器信息：" + socket.getInetAddress() + " p:"+ socket.getPort());
  
          try {
              //发送接收数据
              todo(socket);
          }catch (Exception e){
              System.out.println("异常关闭");
          }
  
          //释放资源
          socket.close();
          System.out.println("客户端已退出！");
      }
  
      /**
       * 发送数据
       * @param client
       * @throws IOException
       */
       private static void todo(Socket client) throws IOException{
           //构建键盘输入流
           BufferedReader input = new BufferedReader(new InputStreamReader(System.in));
  
           //得到 Socket 输出流，并转化为打印流
           OutputStream outputStream = client.getOutputStream();
           PrintStream printStream = new PrintStream(outputStream);
  
           //得到 Socket 输入流,并转化为 BufferedReader
           InputStream inputStream = client.getInputStream();
           BufferedReader socketIn = new BufferedReader(new InputStreamReader(inputStream));
  
           boolean flag = true;
           do{
               //键盘读取一行
               String str = input.readLine();
               //发送到服务器
               printStream.println(str);
  
               //从服务器读取一行
               String echo = socketIn.readLine();
               if ("bye".equalsIgnoreCase(echo)){
                   flag = false;
               }else {
                   System.out.println(echo);
               }
           }while (flag);
  
           socketIn.close();
           printStream.close();
       }
  }
  ~~~

  |                     | Socket                         | ServerSocket                 |
  | ------------------- | ------------------------------ | ---------------------------- |
  | getLocalAddress（） | 获取此套接字绑定到的本地地址   |                              |
  | getInetAddress（）  | 返回此套接字所连接的地址       | 返回此服务器套接字的本地地址 |
  | getLocalPort（）    | 返回此套接字绑定到的本地端口号 | 返回此套接字正在侦听的端口号 |
  | getPort（）         | 返回此套接字所连接的远程端口号 |                              |

  

<img src=".\..\..\pictures\Java\socket\socket 代码流程演示.jpg" alt="socket 代码流程演示"  />

### 10. 报文段

> TCP/IP 协议网络传输过程中，起着路由导航作用
>
> 用以查询各个网络路由网段、IP 地址、交换协议等 IP 数据包

- 报文在传输过程中会不断地封装成组、包、帧来传输

### 11. 传输协议

> 约定大于配置，在网络传输中依然适用；网络地传输流程是健壮的，稳定的，得益于基础的协议构成

### 12. Mac 地址

> Media Access Control 或者 Medium Access Control
>
> 媒体访问控制，或者物理地址、硬件地址；用来定义网络设备的地址

### 13. IP 地址

> 或联网协议地址（Internet Protocol Address，网际协议地址）
>
> 是分配给网络上使用网际协议的设备的数字标签
>
> 常见为 IPv4 和 IPv6

- IPv4
  - 由 32 位二进制数组成
  - 分为 A、B、C、D、E 五大类，E 属于特殊保留地址
  - 1.1.1.1 为直接广播地址（会被因特网拦截）；255.255.255.255 为局域网广播地址
- IPv6
  - 总长128位，采用 32 个十六进制数
  - 一个 64 位的网络前缀和一个 64 位的主机地址，主机地址通常根据物理地址自动生成，叫做 EUI-64（64 位扩展唯一标志）

### 14. 端口

- 计算机之间依照互联网传输层 TCP/IP 协议的协议通信，不同的协议对应不同的端口
- 49152 到 65525 端口属于“动态端口”范围，没有端口可以被正式注册占用

![1594641498126](./..\..\pictures\Java\socket\端口-特殊端口1.png)

### 15. 数据传输层次

![1594641762138](./../..\pictures\Java\socket\数据传输层次.png)

## UDP 快速入门

### 1. UDP 是什么

> User Datagram Protocal，一种用户数据报协议，又称用户数据报文协议，是一种非连接协议
>
> 简单面向数据报的传输层协议，正是规范为 RFC 768

### 2. 为什么不可靠

- 一旦把网络层数据发送出去，就不保留数据备份
- UDP 在 IP 数据包的头部仅仅加入了复用和数据校验字段
- 发送端生产数据，接收端从网络中抓取数据
- 结构简单，无校验，速度快，容易丢包，可以广播

### 3. UDP 能做什么

- DNS、TFTP、SNMP
- 视频、音频、普通数据（无关紧要数据）

- UDP 包最大长度为 65535 - 28 = 65507 byte
  - udp包头占8字节, ip包头占20字节

### 4. UDP 核心 API 讲解

- DatagramSocket

  >  用于接受与发送 UDP 的类
  >
  > 负责发送或者接受 UDP 包
  >
  > 不同于 TCP，UDP 并没有合并到 Socket API 中

  - DatagramSocket()

    创建简单实例，不指定端口和 IP

  - DatagramSocket(int port)

    创建监听固定端口的实例

  - DatagramSocket(int port，InetAddress localAddr)

    创建固定端口和指定 IP 的实例

  - receive(DatagramPacket d)

    接受

  - send(DatagramPacket d)

    发送

  - setSoTimeout(int timeout)

    设置超时，毫秒

  - close()

    关闭，释放资源

- DatagramPacket

  > 用于处理报文
  >
  > 将 byte 数组、目标地址、目标端口等数据包装成报文或者将报文拆解成 byte 数组

  - DatagramSocket(byte[] buf,int offset,int length,InetAddress address,int port)

    前面三个参数指 buf 的使用区间

    后面两个参数指定目标机器地址与端口

  - DatagramSocket(byte[] buf,int offset,int length,SocketAddress address)

    前面三个参数指 buf 的使用区间

    SocketAddress 相当于 InetAddress  + Port

  - setData(byte[] buf,int offset,int length)

  - setData(byte[] buf)

  - setData(int length)

  - getData()、getOffset()、getLength()

  - setAddress(InetAddress address)、setPort(int port)

    发送时有效，告诉接收方有关发送方的信息

  - setSocketAddress(SocketAddress address)

### 5. 单播、广播、多播（按组来发送）

![1594643598200](E:\Java\new_Java_Study\daily_notes\pictures\Java\socket\单播广播多播.png)

![1594644523682](./../..\pictures\Java\socket\IP地址分类.png)

- C 网广播地址一般为：XXX.XXX.XXX.255

### 6. IP 地址构成

![1594647610778](./../..\pictures\Java\socket\IP 地址构成.png)

### 7. 广播地址运算

- IP 地址：192.168.1.7
- 网络地址：192.168.1.0
- 子网掩码：255.255.255.192 --》可分为 2^2 = 4 个网段
- 广播地址为：192.168.1.63