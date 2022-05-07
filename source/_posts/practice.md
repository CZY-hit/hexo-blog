---
title: 练习1-客户端和服务器的socket通信
categories:
- 练习
- socket
tags: [socket,聊天室]
date: 2022-05-06 20:59:33
---
小练习，服务器依次执行socket()、bind()、listen()函数，客户端依次执行socket()、connect()函数，服务器在收到connect()请求后发送accept()请求，完成本地的客户端-服务器连接。
<!--more-->
服务器代码
```
#include<netinet/in.h>
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<sys/socket.h>
#include<unistd.h>
#define PORT 28233
int main(){
    int server_fd; //服务器套接字文件描述符
    struct sockaddr_in address; //地址族，内部存储协议族、端口号、IP地址
    server_fd = socket(AF_INET,SOCK_STREAM,0);//初始化socket并存储描述字
    int addrlen = sizeof(address);
    char *message ="a message from server"; //服务器向客户读端传送的消息内容
    char buffer[1024]={0}; //每次接受消息的最大长度为1024,且存储在bufer中
    address.sin_addr.s_addr=INADDR_ANY;//初始化
    address.sin_family=AF_INET;
    address.sin_port=htons(PORT);

    bind(server_fd,(struct sockaddr*)&address,addrlen);//把地址和socket绑定起来
    listen(server_fd,3); //监听server_fd这个socket，且可以排队的最大连接数为3
    int newsocket=accept(server_fd,(struct sockaddr*)&address,(socklen_t *)&addrlen);//服务器监听到客户端的connect()请求后，发送accept()函数接收请求
    int valread=read(newsocket,buffer,1024);
    send(newsocket,message,strlen(message),0);
    //read()和send()都是以 已连接的socket描述字 为对象的。 
    printf("%s \n",buffer);
    return 0;

}

```
客户端代码
```
#include<netinet/in.h>
#include<arpa/inet.h>
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<string.h>
#include<sys/socket.h>
#define PORT 28233
int main(){
    struct sockaddr_in address;
    address.sin_port=htons(PORT);
    address.sin_family=AF_INET;
   inet_pton(AF_INET,"127.0.0.1",&address.sin_addr);//把IP地址转换为用于网络传输的二进制数值
   
   
    int sockfd = socket(AF_INET,SOCK_STREAM,0);
    char* message = "client say hello";
    int sock = socket(AF_INET,SOCK_STREAM,0);
    char buffer[1024]={0};
    
    int addrlen = sizeof(address);

    connect(sockfd,(struct sockaddr*)&address,sizeof(address));
    send(sockfd,message,strlen(message),0);
    read(sockfd,buffer,1024);
    printf("%s\n",buffer);



}

```