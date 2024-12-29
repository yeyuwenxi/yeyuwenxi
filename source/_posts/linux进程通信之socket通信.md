---
title: linux进程通信之socket通信
date: 2021-08-09 20:27:06
tags: linux
categories: linux
---
实现基于命令行的linux交互式通信
### 客户端的实现

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <pthread.h>
void *read_data(void * args){
		
	//读取客户端传回的数据
    char buffer[50];
	int sock=*(int *)args;
	while(1)
	{
		if(read(sock, buffer, sizeof(buffer)-1)!=(-1)){
   
		printf("Message form server: %s\n", buffer);
		}
		
		}	
		
		
	}
int main(){
    //创建套接字
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    //向服务器（特定的IP和端口）发起请求
    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));  //每个字节都用0填充
    serv_addr.sin_family = AF_INET;  //使用IPv4地址
    serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1");  //具体的IP地址
    serv_addr.sin_port = htons(1234);  //端口
    connect(sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr));
   
    //读取服务器传回的数据
    char buffer[50];
	pthread_t readThread;
	pthread_create(&readThread,NULL,read_data,(void *)&sock);
	char str[40] = "hello";
	 while(1){
        fgets(str,40,stdin);
		write(sock, str, sizeof(str));
		
	}
   
    //关闭套接字
    close(sock);
    return 0;
}
```
### 服务器端的实现

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <pthread.h>
	void *read_data(void * args){
		
	//读取客户端传回的数据
    char buffer[50];
	int sock=*(int *)args;
	while(1)
	{
		if(read(sock, buffer, sizeof(buffer)-1)!=(-1)){
   
		printf("Message form client: %s\n", buffer);
		}
		
		}	
		
		
	}
int main(){
    //创建套接字
    int serv_sock = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
    //将套接字和IP、端口绑定
    struct sockaddr_in serv_addr;
    memset(&serv_addr, 0, sizeof(serv_addr));  //每个字节都用0填充
    serv_addr.sin_family = AF_INET;  //使用IPv4地址
    serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1");  //具体的IP地址
    serv_addr.sin_port = htons(1234);  //端口
    bind(serv_sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr));
    //进入监听状态，等待用户发起请求
    listen(serv_sock, 20);
    //接收客户端请求
    struct sockaddr_in clnt_addr;
    socklen_t clnt_addr_size = sizeof(clnt_addr);
    int clnt_sock = accept(serv_sock, (struct sockaddr*)&clnt_addr, &clnt_addr_size);
    //向客户端发送数据
    char str[40] = "hello";
    write(clnt_sock, str, sizeof(str));
	
	pthread_t readThread;
	pthread_create(&readThread,NULL,read_data,(void *)&clnt_sock);
    while(1){
        fgets(str,40,stdin);
		write(clnt_sock, str, sizeof(str));
		
	}
    //关闭套接字
    close(clnt_sock);
    close(serv_sock);
    return 0;
}
```