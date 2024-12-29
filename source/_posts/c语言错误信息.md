---
title: c语言错误信息
date: 2021-09-12 14:00:55
tags: c语言
categories: c语言
---
### errno
c语言中存在一个error，用来保存最后的错误代码.
errno定义在<errno.h>中，是一个宏定义，用来储存错误代码。
当程序发生错误时，就会将错误代码写入errno.
程序启动时，errno为0，当发生错误时，程序就会将错误代码写入errno,注意，errno是不会自动清零的，而且错误代码的写入是可覆盖的。
所以我们必须在错误发生后立即读取errno的值，进行相关处理。

错误代码只是一个数字，想要获得具体的错误提示信息有两种办法，一是使用perror直接输出错误信息，二是使用strerror 将错误代码转换成对应的文本信息。

### perror
`void perror(const char *s)`
* 函数说明
perror ( )用 来 将 上 一 个 函 数 发 生 错 误 的 原 因 输 出 到 标 准 设备 (stderr) 。参数 s 所指的字符串会先打印出,后面再加上错误原因字符串。
```c
#include <stdio.h>
#include <stdlib.h>

int main ()
{
    
    FILE *fp = fopen("D:/demo.txt", "r");
    if(fp){
        printf("Congratulations, the file opens successfully!\n");
    }else{
        perror("file error");
    }
    return 0;
}
```

运行结果：file error: No such file or directory
### strerror

`char *strerror(int errno);`
* 函数说明
strerror()用来依参数errno 的错误代码来查询其错误原因的描述字符串，然后将该字符串指针返回。

```c
#include <stdio.h>
#include <string.h>
#include <errno.h>
int main ()
{
    errno = 0;  //将 errno 设置回 0 值
    FILE *fp = fopen("D:/demo.txt", "r");
    if(fp){
        printf("Congratulations, the file opens successfully!\n");
    }else{
        printf("Error no.%d: %s\n", errno, strerror(errno));
    }
    return 0;
}
```
运行结果为
Error no.2: No such file or directory
### 参考文章
[errno 记录最后的错误代码](http://c.biancheng.net/ref/errno.html)
[C语言系统错误信息](https://blog.csdn.net/wucz122140729/article/details/98437350)