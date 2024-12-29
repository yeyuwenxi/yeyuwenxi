---
title: linux多进程
date: 2021-09-12 13:57:36
tags: linux
categories: linux
---
使用fork创建子进程

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
int main(){

pid_t pid;
pid=fork();

if(pid==0){
	while(1){
		printf("this is the child process\n");
		sleep(1);
	}
}
else{
	while(1){
		printf("this is the father process\n");
		sleep(1);
	}
	
}

}
```
运行效果如下
```c
this is the father process
this is the child process
this is the father process
this is the child process
this is the father process
this is the child process
this is the father process
this is the child process

```

### vfork
vfork函数用于创建一个新进程，而该新进程的目的是exec一个新进程。
vfork与fork都创建一个子进程，但是它并不会将父进程的地址空间完全复制到子进程中，因为子进程会立即调用exec或exit,于是也就不会引用该地址空间。不过在子进程中调用exec或exit之前，它在父进程的空间中运行。
vfork与fork的另一个区别是：vfork保证子进程先运行，在它调用exec或exit之后父进程才开始运行。

### wait和waitpid
* wait函数等待子进程的结束信号。
它是阻塞函数,只有任意一个子进程结束,它才能继续往下执行,否则卡住那里等。

* waitpid等待指定pid对应的进程结束，可以选择阻塞或者不阻塞的方式。
### exec
exec系列的函数有很多个，主要用来执行一个新的程序，即用一个全新的程序来替换子进程的内容。


### 参考文章
[Linux多进程编程(典藏、含代码)](https://blog.csdn.net/weixin_40519315/article/details/104156838?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162976969216780255264774%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162976969216780255264774&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-2-104156838.first_rank_v2_pc_rank_v29&utm_term=linux%E5%A4%9A%E8%BF%9B%E7%A8%8B&spm=1018.2226.3001.4187)
《UNIX环境高级编程》