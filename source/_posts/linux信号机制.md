---
title: linux信号机制
date: 2021-09-13 21:32:22
tags: linux
categories: linux
---
信号机制主要包括信号的发送和处理。
可以类比于单片机中的中断和中断服务函数，也可以类比于Qt中的信号与槽。
### signal
signal函数用来处理接受到的信号

`sighandler_t signal(int signum, sighandler_t handler);`
第一个参数signum：指明了所要处理的信号类型，它可以取除了SIGKILL和SIGSTOP外的任何一种信号。 　
第二个参数handler：描述了与信号关联的动作，它可以取以下三种值：
* SIG_IGN 　 
忽略此信号
* SIG_DFL 　 
执行系统默认操作
* sighandler_t类型的函数指针
执行函数指针所指向的函数
### kill
kill函数用来发送信号

`int kill(pid_t pid, int signo);`
pid参数存在四种情况
*  pid大于零时，将该信号发送给进程ID为pid的进程
*  pid等于零时，信号将送往所有与调用kill()的那个进程属同一个使用组的进程。
*  pid等于-1时，信号将送往所有调用进程有权给其发送信号的进程，除了进程1(init)。
*  pid小于-1时，信号将送往以-pid为组标识的进程。

sig：准备发送的信号代码，假如其值为零则没有任何信号送出，但是系统会执行错误检查，通常会利用sig值为零来检验某个进程是否仍在执行。
### raise
kill函数将信号发送给进程和进程组，raise函数则允许进程向自身发送信号。
调用raise(signo)
等价于调用
kill(getpid(),signo)
### 例程
send.c

```c
#include<stdio.h>
#include<signal.h>
 #include <unistd.h>
int main(){
	int pid;
	pid=18411;
	while(1){
	sleep(1);
	//SIGUSR1是系统保留给用户使用的信号
	kill(pid, SIGUSR1);}
	
}
```

recv.c

```c
#include<stdio.h>
#include<signal.h>
void hello(){

	printf("hello world\n");
}
int main(){
	
	while(1){
	signal(SIGUSR1, hello);}
}
```
首先运行recv程序，使用ps -a命令查寻得pid
再运行send程序向该pid的进程发送信号
可以看到recv程序以一秒为周期向屏幕打印hello world
### 参考文章
《UNIX环境高级编程》
[【Linux函数】Signal ()函数详细介绍](https://blog.csdn.net/yockie/article/details/51729774?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162969933616780255238561%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162969933616780255238561&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-51729774.first_rank_v2_pc_rank_v29&utm_term=linux%20signal%E5%87%BD%E6%95%B0&spm=1018.2226.3001.4187)
[Linux 下的KILL函数的用法](https://blog.csdn.net/sweetfather/article/details/79462559?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162969932716780269892963%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162969932716780269892963&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-2-79462559.first_rank_v2_pc_rank_v29&utm_term=linux%20kill%E5%87%BD%E6%95%B0&spm=1018.2226.3001.4187)