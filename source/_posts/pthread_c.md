---
title: C中的多线程
categories:
- 聊天室
- 多线程
tags: [pthread,聊天室]
---
**一、什么是线程**
通常，线程被定义为一个独立的指令流，可以被操作系统调度运行。线程是一个 *半进程* ，有自己的栈，执行给定的一段代码。因为线程具有进程的一些属性，所以也被成为*轻量级进程*
<!--more-->
>引用 
https://www.geeksforgeeks.org/multithreading-c-2/?ref=lbp

**二、进程和线程的区别**
线程之间不是相互独立的，线程是为了互相帮助而设计的。而进程可能来自不同的用户，所以进程不一定会互相帮助。

线程之间共享*代码部分*、*数据部分*和*操作系统资源*，与进程一样，线程也有自己的程序计数器（PC）、寄存器集和堆栈空间。

与进程相比，线程的开销要小的多，所有线程共享一个公共地址空间，从而避免了大量的的无效操作。


**三、为什么使用多线程**
线程是通过并行性改进应用程序性能的方式。例如，在浏览器中，多个选项卡可以是不同的线程,MS word使用一个线程格式化文本，另一个线程处理输入。

相比与进程，线程的运行速度快的多，这主要是由于以下原因

- 内核不需要为线程生成新的独立副本空间、文件描述等，这节约了大量的CPU时间，使得创建10-100次线程比创建新进程更快。也正因为如此，我们可以使用大量的线程数，而不必担心产生的CPU和内存开销。这表示只要在程序中有意义，我们通常就可以创建线程。
- 线程之间的上下文切换（Context Switching，从一个线程切换到另一个线程）比多进程之间的上下文切换快的多。
- 终止线程的时间比终止进程的时间短。
- 线程间的通信很快，因为线程间共享地址空间，一个线程产生的数据可以立即供所有其他线程使用。

**四、可以用C语言编写多线程吗**
[POSIX线程（或Pthreads）](https://www.geeksforgeeks.org/multithreading-c-2/?ref=lbp)是线程的POSIX标准。想用C语言实现`pthread`的实现需要通过gcc编译器，或者带有`pthread`库的C编译器。

如下是一个简单的C程序，用于演示pthread的基本功能。

	#include <stdio.h>
	#include <stdlib.h>
	#include <unistd.h> //Header file for sleep(). man 3 sleep for details.
	#include <pthread.h>

	// A normal C function that is executed as a thread
	// when its name is specified in pthread_create()
	void *myThreadFun(void *vargp)
	{
		sleep(1);
		printf("Printing GeeksQuiz from Thread \n");
		return NULL;
	}

	int main()
	{
		pthread_t thread_id;
		printf("Before Thread\n");
		pthread_create(&thread_id, NULL, myThreadFun, NULL);
		pthread_join(thread_id, NULL);
		printf("After Thread\n");
		exit(0);
	}
在main()中，我们声明了一个名为`thread_id`的变量，它的类型为`pthread_t`，是一个整数，作用是标识系统中的线程。在创建`thread_id`后，我们调用`pthread_create`函数创建线程。
pthread_create()接收四个参数。
第一个参数指向此函数设置的`thread_id`指针
第二个参数指定属性。值为NULL时使用默认属性。
第三个参数是要创建的线程执行的函数名称。
第四个参数用于将参数传递给待执行函数（`myThreadFun`）。
线程的`pthread_join()`函数等效于进程的`wait()`。对pthread_join的调用会阻塞主线程，直到标识符等于第一个参数的线程终止。

**五、如何编译上面的程序**
想要用gcc编译上述多线程程序（这里假定上面的C程序文件名为`test.c`），我们需要将其与pthreads库连接，具体操作如下。
	test@ubuntu:~/$ gcc multithread.c -lpthread
	test@ubuntu:~/$ ./a.out
	Before Thread
	Printing GeeksQuiz from Thread 
	After Thread

**六、多线程访问全局变量和静态变量时的情况**
上文中提到，线程之间不是相互独立的，所有线程共享一个数据段，而全局变量和静态变量都存储在数据段中。下面是一个演示程序:

	#include <stdio.h>
	#include <stdlib.h>
	#include <unistd.h>
	#include <pthread.h>

	// Let us create a global variable to change it in threads
	int g = 0;

	// The function to be executed by all threads
	void *myThreadFun(void *vargp)
	{
		// Store the value argument passed to this thread
		int *myid = (int *)vargp;

		// Let us create a static variable to observe its changes
		static int s = 0;

		// Change static and global variables
		++s; ++g;

		// Print the argument, static and global variables
		printf("Thread ID: %d, Static: %d, Global: %d\n", *myid, ++s, ++g);
	}

	int main()
	{
		int i;
		pthread_t tid;

		// Let us create three threads
		for (i = 0; i < 3; i++)
			pthread_create(&tid, NULL, myThreadFun, (void *)&tid);

		pthread_exit(NULL);
		return 0;
	}
以下是执行结果
	
	Thread ID: 3, Static: 2, Global: 2
	Thread ID: 3, Static: 4, Global: 4
	Thread ID: 3, Static: 6, Global: 6

在实际工作中，一般不建议在线程中访问全局变量，因为不清楚线程之间的优先级。如果需要工作中多线程访问全局变量，应该通过互斥锁进行访问。