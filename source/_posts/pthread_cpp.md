---
title: C++中的多线程
categories:
- 聊天室
- 多线程
tags: [pthread,聊天室,c++]
date: 2022-05-05 18:54:18
---
c++11中引入了多线程支持，在C++11之前，我们必须在C中使用POSIX线程或者P=p线程库。而C++11给了我们`std::thread`，线程类及相关函数在`thread`头文件中定义。
<!--more-->
`std::thread`是C++中表示单个线程的线程类。要启动一个线程，我们只需要创建一个新的线程对象并将要调用的执行代码（即可调用对象）传递给对象的构造函数。创建对象后，将启动一个新线程，并执行`callable`中指定的代码。

可调用对象可以是下列三个中任意一个

- 函数指针
```
	定义可调用对象后，将其传递给构造函数
	#include<thread>
	std::thread thread_object(callable)
```
- 一个函数对象
```
	使用函数指针启动线程
	void foo(param)
	{
		// Do something
	}
	std::thread thread_obj(foo, params);
```
- lambda表达式
```
// Define a lamda expression
auto f = [](params) {
	// Do Something
};

// Pass f and its parameters to thread
// object constructor as
std::thread thread_object(f, params);

```

我们也可以将`lambda`函数直接传递给构造函数。
```
std::thread thread_object([](params;,params))
```
**等待线程完成**
想要等待线程，我们应该使用`std::thread::join()`函数，该函数会让当前线程等待直至`*this`执行完毕。

例如，我们想要阻塞主线程直至线程`t1`完成，可以这样做
```
int main()
{
	// Start thread t1
	std::thread t1(callable);

	// Wait for t1 to finish
	t1.join();

	// t1 has finished do other stuff

	...
}
```

**完整的线程调用**
```
// CPP program to demonstrate multithreading
// using three different callables.
#include <iostream>
#include <thread>
using namespace std;

// A dummy function
void foo(int Z)
{
	for (int i = 0; i < Z; i++) {
		cout << "Thread using function"
			" pointer as callable\n";
	}
}

// A callable object
class thread_obj {
public:
	void operator()(int x)
	{
		for (int i = 0; i < x; i++)
			cout << "Thread using function"
				" object as callable\n";
	}
};

int main()
{
	cout << "Threads 1 and 2 and 3 "
		"operating independently" << endl;

	// This thread is launched by using
	// function pointer as callable
	thread th1(foo, 3);

	// This thread is launched by using
	// function object as callable
	thread th2(thread_obj(), 3);

	// Define a Lambda Expression
	auto f = [](int x) {
		for (int i = 0; i < x; i++)
			cout << "Thread using lambda"
			" expression as callable\n";
	};

	// This thread is launched by using
	// lamda expression as callable
	thread th3(f, 3);

	// Wait for the threads to finish
	// Wait for thread t1 to finish
	th1.join();

	// Wait for thread t2 to finish
	th2.join();

	// Wait for thread t3 to finish
	th3.join();

	return 0;
}

```

输出结果：（与机器有关）
```
Threads 1 and 2 and 3 operating independently                                                       
Thread using function pointer as callable                                                           
Thread using lambda expression as callable                                                          
Thread using function pointer as callable                                                           
Thread using lambda expression as callable                                                          
Thread using function object as  callable                                                          
Thread using lambda expression as callable                                                          
Thread using function pointer as callable                                                          
Thread using function object as  callable                                                           
Thread using function object as  callable
```

