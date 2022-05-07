---
title: C/C++中的socket编程
categories:
- 聊天室
- socket
tags: [socket,聊天室]
date: 2022-05-06 22:05:00
---
**一、什么是socket编程**
socket编程可以通过网络让两个节点间实现通信。一个socket监听某个特定IP的特定端口，另一个socket发起连接申请。服务器（Server）形成一个监听socket，客户端负责向服务器发起连接请求。
<!--more-->
>引用 
https://www.geeksforgeeks.org/socket-programming-cc/?ref=lbp
http://c.biancheng.net/cpp/html/3032.html
https://www.cnblogs.com/skynet/archive/2010/12/12/1903949.html
https://www.cnblogs.com/eeexu123/p/5275783.html

![客户端-服务器模型](/img/cs-model.png)

**二、socket套接字创建**
	
	int sockfd = socket(domain,type,protocol);

- **sockfd**:socket描述符，它唯一标识一个socket，可以把它作为参数来进行读写操作。
- **domain**: 用于指定连接域的整型。用`POSIX`标准中定义的`AF_LOCAL`来进行同一主机上进程之间的通信。对于不同主机间通过`IPV4`和`IPV6`连接的进程，我们分别使用`AF_INET`和`AF_INET6`。
- **type**: 通讯类型
		1. SOCK_STREAM: TCP (可靠的，面向连接)
		2. SOCK_DGRAM: UDP(不可靠的，无连接)
- **protocol**:表示传输协议，常用的有`IPPROTO_TCP`和`IPPTOTO_UDP`，分别表示TCP传输协议和UDP传输协议，设置为0时会自动推演出要使用什么协议。

**三、Setsockopt**
用于获取或者设置与某个套接字关联的选项。（没看懂，后续用到再学）

	int setsockopt(int sockfd,int leverl,int optname,const void* optval,socklen_t optlen);
- **sockfd**:将要被设置或者获取选项的套接字。
- **level**:选项所在的协议层。
- **optname**: 需要访问的选项名。
- **optval**:对于getsockopt(),指向返回选项值的缓冲。对于setsockopt(),指向包含新选项值的缓冲。
- **optlen**:对于getsockopt()，作为入口值时是选项值的最大长度。作为出口参数时是选项值的实际长度。对于setsockopt()，是现选项的长度。<br>

**四、bind**
创建socket后，bind函数将socket绑定到addr中指定的地址和端口号。

	int bind(int sockfd,const struct sockaddr * addr,socklen_t addrlen);

- **sockfd**: socket描述字，是socket的唯一标识符。
- **adr**: 一个`const struct sockaddr*`指针，指向要绑定给`sockfd`的协议地址，这个地址结构根据地址创建socket时的地址协议族不同而不同。
比如，IPV4对应的是
```
struct sockaddr_in {
    sa_family_t    sin_family; /* address family: AF_INET */
    in_port_t      sin_port;   /* port in network byte order */
    struct in_addr sin_addr;   /* internet address */
};

/* Internet address. */
struct in_addr {
    uint32_t       s_addr;     /* address in network byte order */
};
```
IPV6对应的是
```
struct sockaddr_in6 { 
    sa_family_t     sin6_family;   /* AF_INET6 */ 
    in_port_t       sin6_port;     /* port number */ 
    uint32_t        sin6_flowinfo; /* IPv6 flow information */ 
    struct in6_addr sin6_addr;     /* IPv6 address */ 
    uint32_t        sin6_scope_id; /* Scope ID (new in 2.4) */ 
};

struct in6_addr { 
    unsigned char   s6_addr[16];   /* IPv6 address */ 
};
```
- **addrlen**：对应的是地址的长度。

**五、listen**
listen将服务器socket置于被动模式，等待客户端发送connect请求以建立连接。

	int listen(int sockfd, int backlog);
- sockfd为要监听的socket的描述字
- backlog定义了sockfd可以排队的最大连接个数，如果连接已满时有新的连接请求到达，客户端就可能会收到ECONNREFUSED指示的错误。<br>
**六、connect**
客户端通过调用connect()函数来建立与TCP服务器的连接。使用之前，客户端需要先执行socket操作创建客户端自身的套接字。

	int connect(int sockfd,const struct sockaddr * addr,socklen_t addrlen);
- **sockfd**：为客户端自身的socket描述字。
- **addr**： 为服务器的socket地址
- **addrlen**：服务器socket地址的长度。
**七、accept与close**
TCP服务器依次调用`socket()`、`bind()`、`listen()`后，就会监听指定的socket地址了。

TCP客户端依次调用`socket()`、`connect()`后就向TCP服务器发送了一个连接请求。TCP服务器监听到这个请求之后，就会通过`accept()`函数接收请求，这样连接就建立好了。之后就可以开始网络I/O操作了，类似于普通文件的I/O操作。
	
	int accept(int sockfd,const struct sockaddr * addr,socklen_t * addrlen);
如果accept成功，那么其返回值是由内核自动生成的全新的描述字，代表服务器与客户的TCP连接。

- **sockfd**:服务器的描述字，是服务器调用socket函数生成的，称为`监听socket描述字`；而accept()函数返回的是`已连接的socket描述字`。内核为每个由服务器进程接受的客户连接创建一个 `已连接socket描述字`，当服务器完成了对某个客户的服务时，相应的`已连接socket描述字`就会被关闭。
- **addr**:用于返回客户端的协议地址。
- **addrlen**: 协议地址的长度
```
#include <unistd.h>
int close(int fd);
```
close一个socket时会默认把该socket标记为已关闭，然后立即返回到调用线程，且该描述字不能由调用线程使用。

close操作只是使相应描述字的引用次数-1,只有当引用次数为0的时候，才会触发TCP客户端向服务器发送终止连接请求。

**八、sockaddr_in结构体**
![sockaddr_in结构体](/img/sockaddr_in_struct.png)
- **sin_family**：指代协议族，在socket编程中只能用AF_INET
- **sin_port**：存储端口号（使用网络字节顺序）
- **sin_addr**：存储IP地址，使用`in_addr`这个结构体。
- **sin_zero**：空字节，为了占位。

`sockaddr_in`结构体变量的基本配置
```
struct sockaddr_in address;
address.sin_family=AF_INET;
address.sin_port=htons(23);
address.sin_addr.s_addr=inet_addr("127.0.0.1");
//inet_addr作用是将一个IP点分十进制IP地址转换为一个无符号长整型。
//inet_addr函数的头文件是netinet/in.h
```
**九、socket中的TCP三次握手建立连接**
tcp建立连接需要进行三次握手，这三次握手的大致流程如下：

- 第一次握手，客户端向服务器发送一个`SYN J`
- 第二次握手，服务器对客户端的`SYN J`进行确认`ACK J+1`，并向客户端发送一个`SYN K`
- 第三次握手，客户端确认接收到服务器的`SYN K `后再向服务器发送一个`确认ACK K+1`

那么，这三次握手过程中，socket中的函数在何时起何种作用呢
![socket中函数在三次握手中的作用](/img/handshake3_to_connect.png)
从图片中可以看到，客户端调用`connect`时，触发连接请求，向服务器发送了`SYN J`包，此时`connect`进入阻塞状态；

服务器监听到连接请求，接收到`SYN J`包，调用`accept`函数接收请求，并向客户端发送`SYN K`、`ACK J+ 1`，此时`accept`进入阻塞状态；

客户端接收到服务器的`SYN K`、`ACK J+1`之后，`connect`返回，并对`SYN K`进行确认，向服务器发送`ACK K+1`; 

服务器接收到`ACK K+1`时`accept`返回，至此三次握手建立完毕，连接建立。

**十、socket中的TCP四次握手释放连接**
介绍一下socket中四次握手释放连接的过程，先看下图。
![socket中四次握手释放连接的过程](/img/handshake4_to_disconnect.png)
图示过程如下

- 第一次握手，应用进程先调用`close`主动关闭连接，向服务器发送连接释放请求`FIN M`，并在发送连接释放请求的同时启动一个计时器。
- 第二次握手，服务器接收到`FIN M`后，会对这个`FIN M`回发一个确认`ACK`。这个`ACK`到达应用进程后，应用进程的连接就释放了
- 第三次握手，一段时间后，服务器会向应用进程发送一个连接释放的请求`FIM N`，并同样启动定时器。
- 第四次握手，最后这个`FIN N`到达应用进程的时候，应用进程会再回发一个`ACK`，当这个`ACK`到达服务器的时候，服务器的连接也就释放了。

这样每个方向上都有一个`FIN`和`ACK`

**十一、示例**
在这里，我们在客户端与服务器之间交换一条hello消息来演示client/server模型。

服务器代码
```
#include <netinet/in.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>
#define PORT 8080
int main(int argc, char const* argv[])
{
    int server_fd, new_socket, valread;
    struct sockaddr_in address;
    int opt = 1;
    int addrlen = sizeof(address);
    char buffer[1024] = { 0 };
    char* hello = "Hello from server";
 
    // Creating socket file descriptor
    if ((server_fd = socket(AF_INET, SOCK_STREAM, 0))
        == 0) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }
 
    // Forcefully attaching socket to the port 8080
    if (setsockopt(server_fd, SOL_SOCKET,
                   SO_REUSEADDR | SO_REUSEPORT, &opt,
                   sizeof(opt))) {
        perror("setsockopt");
        exit(EXIT_FAILURE);
    }
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);
 
    // Forcefully attaching socket to the port 8080
    if (bind(server_fd, (struct sockaddr*)&address,
             sizeof(address))
        < 0) {
        perror("bind failed");
        exit(EXIT_FAILURE);
    }
    if (listen(server_fd, 3) < 0) {
        perror("listen");
        exit(EXIT_FAILURE);
    }
    if ((new_socket
         = accept(server_fd, (struct sockaddr*)&address,
                  (socklen_t*)&addrlen))
        < 0) {
        perror("accept");
        exit(EXIT_FAILURE);
    }
    valread = read(new_socket, buffer, 1024);
    printf("%s\n", buffer);
    send(new_socket, hello, strlen(hello), 0);
    printf("Hello message sent\n");
    return 0;
}
```

客户端代码
```
#include <arpa/inet.h>
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>
#define PORT 8080

int main(int argc, char const* argv[])
{
	int sock = 0, valread;
	struct sockaddr_in serv_addr;
	char* hello = "Hello from client";
	char buffer[1024] = { 0 };
	if ((sock = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
		printf("\n Socket creation error \n");
		return -1;
	}

	serv_addr.sin_family = AF_INET;
	serv_addr.sin_port = htons(PORT);

	// Convert IPv4 and IPv6 addresses from text to binary
	// form
	if (inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr)
		<= 0) {
		printf(
			"\nInvalid address/ Address not supported \n");
		return -1;
	}

	if (connect(sock, (struct sockaddr*)&serv_addr,
				sizeof(serv_addr))
		< 0) {
		printf("\nConnection Failed \n");
		return -1;
	}
	send(sock, hello, strlen(hello), 0);
	printf("Hello message sent\n");
	valread = read(sock, buffer, 1024);
	printf("%s\n", buffer);
	return 0;
}

```