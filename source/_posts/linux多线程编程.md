---
title: linux多线程编程
date: 2021-09-13 21:31:15
tags: linux
categories: linux
---
# linux多线程编程
这个帖子是一边改代码一边写的，可能会有点乱，有空再好好整理一下


linux多线程编程使用的接口主要包含在头文件<pthread.h>中
## 常用接口
### 创建线程

### 多线程入门
```C
//多线程
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
void* print1(){
	while(1){
	sleep(1);	
	printf("This is thread1\n");
	
	}
}
void* print2(){
	while(1){
	sleep(1);
	printf("This is thread2\n");}
	
}
int main(){
	
pthread_t th1;
pthread_t th2;	
pthread_create(&th1,NULL,print1,NULL);
pthread_create(&th2,NULL,print2,NULL);
while(1){
	sleep(1);
	printf("This is function main\n");
	
}

return 0;
}

```

运行效果
```C
This is thread2
This is thread1
This is function main
This is thread2
This is thread1
This is function main
This is thread2
This is thread1
This is function main
This is thread1
This is thread2
This is function main
This is thread1
This is thread2
This is function main
This is thread2
This is function main
This is thread1
```
这里有一个问题一直没有想明白，sleep和printf如果交换顺序的话，代码就不能正常执行了

### 给创建的线程传递参数
这里要注意的是，我们传递的参数一定要是void *类型的指针
下面通过修改一下上面的例程，通过参数化的方法来判断线程1和线程2

```c
//多线程
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
void* print(void *arg){
	while(1){
	sleep(1);
 int n=*(int*)arg;	
	printf("This is thread%d\n",n);
	
	}
}

int main(){
	
pthread_t th1;
pthread_t th2;
int a=1;
int b=2;	
pthread_create(&th1,NULL,print,(void*)&a);
pthread_create(&th2,NULL,print,(void*)&b);
while(1){
	sleep(1);
	printf("This is function main\n");
	
}

return 0;
}
```
最终运行效果与之前是一致的
### 线程的互斥问题
先来看一个经典的卖票问题

```c
int n=100;
void* sub(void *arg){
	 
	
	while(1){
		
		
	if(n>0){
		usleep(1);
		n--;
		printf("n=%d\n",n);
	}
	
	
	
	}
	
}

int main(){

pthread_t th1;
pthread_t th2;
	
pthread_create(&th1,NULL,sub,(void*)&m_mutex);
pthread_create(&th2,NULL,sub,(void*)&m_mutex);
while(1);

return 0;
}
```

运行效果
```c
n=10
n=9
n=8
n=7
n=6
n=5
n=4
n=3
n=2
n=1
n=0
n=-1

```
什么？居然出现了-1的情况
这时候就需要考虑线程的互斥问题了

添加互斥锁之后

```c
//多线程
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
int n=100;
void* sub(void *arg){
	 pthread_mutex_t* p_mutex = (pthread_mutex_t*)arg;
	
	while(1){
		
	pthread_mutex_lock(p_mutex);	
	if(n>0){
		usleep(1);
		n--;
		printf("n=%d\n",n);
	}
	pthread_mutex_unlock(p_mutex);
	
	
 	
	
	
	
	}
	
}

int main(){
pthread_mutex_t m_mutex;
    pthread_mutex_init(&m_mutex, NULL);	
pthread_t th1;
pthread_t th2;
	
pthread_create(&th1,NULL,sub,(void*)&m_mutex);
pthread_create(&th2,NULL,sub,(void*)&m_mutex);
while(1);

return 0;
}

```
这时查看运行效果，发现到0之后正常停止了
```c
n=10
n=9
n=8
n=7
n=6
n=5
n=4
n=3
n=2
n=1
n=0

```
### 线程同步
三个线程按顺序输出ABC
```
//多线程
//多线程
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <fcntl.h>
#include <pthread.h>


pthread_mutex_t m_mutex;
pthread_cond_t condA;
pthread_cond_t condB;
pthread_cond_t condC;
void* printA(){
	while(1){
		
	
	pthread_mutex_lock(&m_mutex);	
	pthread_cond_wait(&condA,&m_mutex);
	printf("A");
	
	pthread_mutex_unlock(&m_mutex);pthread_cond_signal(&condB);
	}
}
void* printB(){
	while(1){
		
	
	pthread_mutex_lock(&m_mutex);
	pthread_cond_wait(&condB,&m_mutex);	
	printf("B");
	
	pthread_mutex_unlock(&m_mutex);pthread_cond_signal(&condC);
	}
	
}
void* printC(){
	while(1){
		
	
	pthread_mutex_lock(&m_mutex);
	pthread_cond_wait(&condC,&m_mutex);	
	printf("C");
	
	pthread_mutex_unlock(&m_mutex);pthread_cond_signal(&condA);
	}
	
}

int main(){
pthread_cond_init(&condA, NULL);
pthread_cond_init(&condB, NULL);
pthread_cond_init(&condC, NULL);
pthread_mutex_init(&m_mutex, NULL);
	
pthread_t th1;
pthread_t th2;
pthread_t th3;	
pthread_create(&th1,NULL,printA,NULL);
pthread_create(&th2,NULL,printB,NULL);
pthread_create(&th3,NULL,printC,NULL);

pthread_cond_signal(&condA);

while(1);

return 0;
}
```
执行以上代码，可以看到程序按顺序输出ABC
这里存在一个问题，程序有时候会莫名卡死，暂时没有想明白是为什么

### 生产者消费者问题 一对一

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <fcntl.h>
#include <pthread.h>
#include <semaphore.h>

pthread_mutex_t m_mutex;

sem_t sem;
int product_num=10;

void* producter(){
	while(1){
		
	sleep(1);
	sem_post(&sem);
	pthread_mutex_lock(&m_mutex);	
	
	
	product_num++;
	printf("This is producter. product a product. product_num=%d\n",product_num);
	
	pthread_mutex_unlock(&m_mutex);
	}
}
void* consumer(){
	while(1){
		
	sem_wait(&sem);
	pthread_mutex_lock(&m_mutex);
	product_num--;
	
	printf("This is consumer. consume a product. product_num=%d\n",product_num);
	
	pthread_mutex_unlock(&m_mutex);
	}
	
}


int main(){

pthread_mutex_init(&m_mutex, NULL);
	
pthread_t th1;
pthread_t th2;
sem_init(&sem, 0, 10);	
pthread_create(&th1,NULL,producter,NULL);
pthread_create(&th2,NULL,consumer,NULL);




while(1);

return 0;
}
```
运行效果如下

```c
This is consumer. consume a product. product_num=9
This is consumer. consume a product. product_num=8
This is consumer. consume a product. product_num=7
This is consumer. consume a product. product_num=6
This is consumer. consume a product. product_num=5
This is consumer. consume a product. product_num=4
This is consumer. consume a product. product_num=3
This is consumer. consume a product. product_num=2
This is consumer. consume a product. product_num=1
This is consumer. consume a product. product_num=0
This is producter. product a product. product_num=1
This is consumer. consume a product. product_num=0
This is producter. product a product. product_num=1
This is consumer. consume a product. product_num=0
This is producter. product a product. product_num=1
This is consumer. consume a product. product_num=0
This is producter. product a product. product_num=1
This is consumer. consume a product. product_num=0
This is producter. product a product. product_num=1
This is consumer. consume a product. product_num=0
This is producter. product a product. product_num=1
This is consumer. consume a product. product_num=0
This is producter. product a product. product_num=1
This is consumer. consume a product. product_num=0
This is producter. product a product. product_num=1
This is consumer. consume a product. product_num=0

```

