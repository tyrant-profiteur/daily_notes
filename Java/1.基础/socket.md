## 快速入门

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