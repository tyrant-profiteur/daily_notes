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

