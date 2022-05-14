---
title: socket编程中使用多线程处理多个客户端
categories:
- 聊天室
- socket
tags: [socket,聊天室,pthread]
date: 2022-05-14 10:05:00
---

在基本的客户端/服务器模型中，服务器一次只处理一个客户端，为了让服务器能够处理多个客户端，我们可以简单的为每一个客户端生成一个新线程，在客户端数量大于两个时，我们通常采用*信号量*作为一种多线程解决方案。
<!--more-->
### 信号量（Samaphore）
**信号量**是一种线程同步工具，是主要用作多个线程对共享资源进行并行操作的工具类。它代表一种是否允许多个线程对同一资源进行操作的许可，使用信号量可以控制并发访问资源的线程个数，用于在多线程环境下解决临界区问题，实现进程同步。

## 原理
**原理**：在进入一个临界区之前，线程必须获取一个信号量，一旦该关键代码执行完成了，那么该线程必须释放信号量。其它想进入该关键代码段的线程必须等待直到地一个线程释放信号量。

### 两种操作 

- Wait（等待）：当一个线程调用Wait操作时，它要么得到资源并将信号量减一，要么进入阻塞队列，直到信号量大于一。
- Release（释放）：实际上是对信号量执行加一操作，也即为释放了由信号量守护的资源。
### 两个函数

- sem_post函数
```
#include <semaphore.h>
int sem_post(sem_t *sem)
```
作用是给信号量的值加一。当有线程阻塞在这个i信号量上时，调用这个函数会使其中一个线程不再阻塞，选择机制由线程调度策略决定。
- sem_wait函数
```
#include <semaphore.h>
int sem_wait(sem_t * sem)
```
作用是给信号量的值减一，但它永远会先等待该信号量为一个非零值才开始做减法。

### 实现方法
#### 创建套接字
对于服务器端，创建两个不同的线程，一个读线程和一个写线程。首先，声明一个整型变量`serverSocket`用于保存`socket`函数的返回值

	int serverSocket = socket(domain, type ,protocol);

- **serverSocket**:套接字描述符。
- **domain**: 通信域，如`AF_INET`(IPV4协议)和`AF_INET6`(ipv6协议)
- **SOCK_STREAM**:TCP协议，可靠，面向连接。
- **SOCK_DGRAM**:UDP协议，不可靠，面向无连接。
- **protocol**: 表示传输协议，常用的有`IPPRPOTO_TCP`和`IPPTOTO_UDP`，分别表示TCP传输协议和UDP传输协议，设置为0时表示程序自行判断选择哪种传输协议。

#### 绑定套接字
创建`socket`后，使用`bind`函数将`socket`绑定到`addr`（自定义struct）中指定的地址和端口号。在实例代码中，我们将服务器绑定到本地主机，因此`INADDR_ANY`用于指定IP地址。

	int bind(int sockfd, const struct sockaddr * addr, socklen_t addrlen);

#### 监听套接字

	int listen(int sockfd, int backlog);

用于将服务器置于被动模式，等待客户端接近服务器以建立连接。`baklog`定义了`sockfd`的挂起连接队列可能增长到的最大长度。如果连接请求到达时队列已满，客户端可能会收到带有`ECONNREFUSED`指示的错误。

#### 方法

- 接收到所需端口的连接后，从客户端接收一个整数，这个整数为1时代表读取，为2时代表写入。
- 成功接收数据后，调用`pthread_creat`创建读进程和写进程。
- 成功连接到服务器客户端后，会要求用户输入选择变量。
- 在从用户那里得到选择之后，客户端将这个段则发送到服务器，服务器为客户端请求创建一个客户端线程来调用读取器线程或写入器线程。

### 代码实现
#### 服务器代码
```
// C program for the Server Side

// inet_addr
#include <arpa/inet.h>

// For threading, link with lpthread
#include <pthread.h>
#include <semaphore.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>

// Semaphore variables
sem_t x, y;
pthread_t tid;
pthread_t writerthreads[100];
pthread_t readerthreads[100];
int readercount = 0;

// Reader Function
void* reader(void* param)
{
	// Lock the semaphore
	sem_wait(&x);
	readercount++;

	if (readercount == 1)
		sem_wait(&y);

	// Unlock the semaphore
	sem_post(&x);

	printf("\n%d reader is inside",
		readercount);

	sleep(5);

	// Lock the semaphore
	sem_wait(&x);
	readercount--;

	if (readercount == 0) {
		sem_post(&y);
	}

	// Lock the semaphore
	sem_post(&x);

	printf("\n%d Reader is leaving",
		readercount + 1);
	pthread_exit(NULL);
}

// Writer Function
void* writer(void* param)
{
	printf("\nWriter is trying to enter");

	// Lock the semaphore
	sem_wait(&y);

	printf("\nWriter has entered");

	// Unlock the semaphore
	sem_post(&y);

	printf("\nWriter is leaving");
	pthread_exit(NULL);
}

// Driver Code
int main()
{
	// Initialize variables
	int serverSocket, newSocket;
	struct sockaddr_in serverAddr;
	struct sockaddr_storage serverStorage;

	socklen_t addr_size;
	sem_init(&x, 0, 1);
	sem_init(&y, 0, 1);

	serverSocket = socket(AF_INET, SOCK_STREAM, 0);
	serverAddr.sin_addr.s_addr = INADDR_ANY;
	serverAddr.sin_family = AF_INET;
	serverAddr.sin_port = htons(8989);

	// Bind the socket to the
	// address and port number.
	bind(serverSocket,
		(struct sockaddr*)&serverAddr,
		sizeof(serverAddr));

	// Listen on the socket,
	// with 40 max connection
	// requests queued
	if (listen(serverSocket, 50) == 0)
		printf("Listening\n");
	else
		printf("Error\n");

	// Array for thread
	pthread_t tid[60];

	int i = 0;

	while (1) {
		addr_size = sizeof(serverStorage);

		// Extract the first
		// connection in the queue
		newSocket = accept(serverSocket,
						(struct sockaddr*)&serverStorage,
						&addr_size);
		int choice = 0;
		recv(newSocket,
			&choice, sizeof(choice), 0);

		if (choice == 1) {
			// Creater readers thread
			if (pthread_create(&readerthreads[i++], NULL,
							reader, &newSocket)
				!= 0)

				// Error in creating thread
				printf("Failed to create thread\n");
		}
		else if (choice == 2) {
			// Create writers thread
			if (pthread_create(&writerthreads[i++], NULL,
							writer, &newSocket)
				!= 0)

				// Error in creating thread
				printf("Failed to create thread\n");
		}

		if (i >= 50) {
			// Update i
			i = 0;

			while (i < 50) {
				// Suspend execution of
				// the calling thread
				// until the target
				// thread terminates
				pthread_join(writerthreads[i++],
							NULL);
				pthread_join(readerthreads[i++],
							NULL);
			}

			// Update i
			i = 0;
		}
	}

	return 0;
}

```
#### 客户端代码
```
// C program for the Client Side
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>

// inet_addr
#include <arpa/inet.h>
#include <unistd.h>

// For threading, link with lpthread
#include <pthread.h>
#include <semaphore.h>

// Function to send data to
// server socket.
void* clienthread(void* args)
{

	int client_request = *((int*)args);
	int network_socket;

	// Create a stream socket
	network_socket = socket(AF_INET,
							SOCK_STREAM, 0);

	// Initialise port number and address
	struct sockaddr_in server_address;
	server_address.sin_family = AF_INET;
	server_address.sin_addr.s_addr = INADDR_ANY;
	server_address.sin_port = htons(8989);

	// Initiate a socket connection
	int connection_status = connect(network_socket,
									(struct sockaddr*)&server_address,
									sizeof(server_address));

	// Check for connection error
	if (connection_status < 0) {
		puts("Error\n");
		return 0;
	}

	printf("Connection established\n");

	// Send data to the socket
	send(network_socket, &client_request,
		sizeof(client_request), 0);

	// Close the connection
	close(network_socket);
	pthread_exit(NULL);

	return 0;
}

// Driver Code
int main()
{
	printf("1. Read\n");
	printf("2. Write\n");

	// Input
	int choice;
	scanf("%d", &choice);
	pthread_t tid;

	// Create connection
	// depending on the input
	switch (choice) {
	case 1: {
		int client_request = 1;

		// Create thread
		pthread_create(&tid, NULL,
					clienthread,
					&client_request);
		sleep(20);
		break;
	}
	case 2: {
		int client_request = 2;

		// Create thread
		pthread_create(&tid, NULL,
					clienthread,
					&client_request);
		sleep(20);
		break;
	}
	default:
		printf("Invalid Input\n");
		break;
	}

	// Suspend execution of
	// calling thread
	pthread_join(tid, NULL);
}

```


>引用
https://blog.csdn.net/megayangyang/article/details/55662170
https://www.geeksforgeeks.org/handling-multiple-clients-on-server-with-multithreading-using-socket-programming-in-c-cpp/?ref=gcse
