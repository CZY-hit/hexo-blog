---
title: C/C++中的多线程头文件pthread
categories:
- 聊天室
- 多线程
tags: [pthread,聊天室]
date: 2022-05-05 12:30:15
---
### 一、pthread头文件
在Unix/Linux系统中，C/C++提供了`pthread`（POSIX线程）API。它允许我们为并发流程创建多个线程，这可以提高程序在多核处理器或上的执行速度。
<!--more-->


想要使用pthreads库的所有功能，我们必须在`.c`或`.c++`文件中包含`pthread.h`头文件，在编译文件时在命令行中使用 `-pthread` 或 `-lpthread`。

	cc -pthread file.c 或者
		cc -lpthread file.c
	
### 二、pthreads库中定义的函数
#### a.  *pthread_create:*  用于创建新线程

	int pthread_create(pthread_t  * thread,
				const pthread_attr_t * attr,
				void * (*start_routine)(void *),
				void * arg);

参数：

-  **thread**:返回创建的线程的线程ID，是一个指向无符号整数值的指针。
- **attr**: 默认值为NULL，目前没有用，不需要修改。是一个指向用于定义线程属性（如分离状态、调度策略等）的结构的指针。
- **start_routine**: 是一个指向线程执行子程序的指针。子程序的返回类型和参数类型必须是 `void * `类型。如果同时传递多个指针需要用到结构体。
- **arg**: void类型的指针，其中包含前面参数中定义的函数的参数。

#### b.*pthread_exit:* 用于终止线程

	 void pthread_exit(void * retval);

参数：

- **retval**: 它是一个指向整型变量的指针，该整数储存线程终止的返回状态。读取的整型变量必须为全局变量，以便让任何等待加入该线程的线程可以读取其返回状态。

#### c. *pthread_join:* 用于等待线程终止

将子线程并入主线程，主线程会一直阻塞直至子线程执行结束（收到目标子线程的返回值）后，才回收子线程资源，解除阻塞状态，并继续执行主线程
	int pthread_join(pthread_t th,
					void **thread_return);

参数：

- **th**：当前线程等待加入的目标线程的线程id。
- **thraad_return**：指向th中提到的线程的退出状态存储位置的指针。

#### d.*pthread_self:* 用于获取当前线程id

	pthread_t phread_self(void);
#### e.*pthread_equal:* 用于比较两个线程是否相同。如果两个线程相等则返回一个非零值，否则返回0。

	int pthread_equal(pthread_t t1,
						pthread_t t2);
						
参数：

- **t1**：第一个线程id
- **t2**：第二个线程id

#### f.*pthread_cancel:* 用于向线程发送取消请求

	int pthead_cancel(pthread_t thread);

参数：

- **thread**：用于指定发送Cancel信号的目标线程

#### g.*pthread_detach:* 用于分离线程
使用pthread_create创建的线程有两种状态：`joinable`和`ubjoinable`。默认情况下线程处于`joinable`状态，可以通过`pthread_attr_getdetachstate` 来获取线程状态。
也可以通过如下代码来设置线程状态为`joinable`或者`unjoinable`

	pthread_attr_t attr;
	pthread_attr_init(&attr);
	pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
	pthread_create(&thr, &attr, &thread_start, NULL);
pthread_detach()和pthread_join()就是控制子线程回收资源的两种不同方式。统一线程间的资源具有共享和独立的资源，其中共享的资源有堆、全局变量、静态变量、文件等公共资源。而独享的资源有栈和寄存器，这两种方式决定子线程结束时如何回收独享的资源。

- 在`joinable`状态结束的子线程不会释放线程所占用的堆栈和线程描述符（总计8K多）等资源，直至主线程调用了`pthread_join`函数后才会释放这些资源。所以`pthread_join`一般应用在主线程需要等待子线程结束后才继续执行的场景。

- 在`unjoinable`状态结束的子线程在结束后会自动释放占用资源。实现方式可以是在创建线程时指定属性，或者在线程执行体最后一行添加`pthread_detach(pthread_self());`
这样线程结束后就会自动释放所有资源。
<br>
样例
```
		#include <pthread.h>
		#include <stdio.h>
		#include <stdlib.h>

		void* func(void* arg)
		{
			// detach the current thread
			// from the calling thread
			pthread_detach(pthread_self());

			printf("Inside the thread\n");

			// exit the current thread
			pthread_exit(NULL);
		}

		void fun()
		{
			pthread_t ptid;

			// Creating a new thread
			pthread_create(&ptid, NULL, &func, NULL);
			printf("This line may be printed"
				" before thread terminates\n");

			// The following line terminates
			// the thread manually
			// pthread_cancel(ptid);

			// Compare the two threads created
			if(pthread_equal(ptid, pthread_self())
				printf("Threads are equal\n");
			else
				printf("Threads are not equal\n");

			// Waiting for the created thread to terminate
			pthread_join(ptid, NULL);

			printf("This line will be printed"
				" after thread ends\n");

			pthread_exit(NULL);
		}

		// Driver code
		int main()
		{
			fun();
			return 0;
		}
```
输出
```
	This line may be printed before thread terminates
	Threads are not equal
	Inside the thread
	This line will be printed after thread ends
```

代码中有两个执行线程，主线程等待新创建的进程退出后才会打印输出的最后一行，如果我们想手动终止线程，可以使用*`pthread_cacnel`*完成。
**注意**：如果我们使用`exit()`而非`pthread_exit()`来结束线程，那么与执行
`exit()`的线程关联的所有线程都会终止，即使某些线程可能正在运行。

引用
>https://www.geeksforgeeks.org/thread-functions-in-c-c/?ref=lbp
>https://blog.csdn.net/Set_Mode/article/details/121439463
>http://www.csc.villanova.edu/~mdamian/threads/posixthreads.html
