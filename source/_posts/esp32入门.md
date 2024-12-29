---
title: esp32入门
date: 2021-05-24 16:13:02
tags: 
- 单片机
- esp32
categories: 单片机
---
最近打算做一个物联网的应用，买了块ESP32的开发板。
### ESP32的简单介绍

### 开发环境的搭建
ESP32支持使用arduino,espif等进行开发，可供使用的开发编辑器也有很多。
我们由简到难，先从最简单的开始，这里使用arduino IDE加上ESP32的arduino包进行开发。
开发环境搭建详见下文
[ESP32 开发之旅① 走进ESP32的世界 安装开发环境](https://blog.csdn.net/dpjcn1990/article/details/94414983)

### 烧录遇到的问题
安装完CP2102的驱动后，连到电脑上居然无法下载，串口时断时续的，后来采用自己的USB转TTL的烧录器成功地烧录了程序。
跟客服聊了聊，认为是CP2102的芯片坏了，客服直接发了块新的给我，而且说旧的不用退了。
这里强推一波优信电子！物美价廉，客服也很周到！
### arduino开发环境介绍
arduino既有自己的硬件平台，又有一个基于arduino IDE的软件框架，arduino IDE属实不好用，连个最基本的函数查找都做不到，但arduino本身的框架封装了很多简单易用的函数，而且网上基于这一框架有很多有趣的开发实例，这里我们主要使用arduino的框架进行开发。
### 开发板原理图
最离谱的是，淘宝商家居然没有提供原理图，自己在网上搜索才找到了一张跟手上开发板一样的原理图
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210604_1.png" >

### helloworld程序
编写程序如下，编译完成后点击上传烧录到单片机中。
**注意:** 烧录时屏幕下方出现connecting时按住boot按钮不放，直到烧录完成后松开boot按钮，此时，按一下复位按钮，程序就可以在单片机上正常运行了。
打开arduino IDE中的串口监视器，可以看到每秒都会收到一次单片机发送的helloworld.
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210524_1.png" >


### 点灯程序
这块开发板上自带两个LED，其中红色的应该时电源指示灯，不受我们控制。
网上查询发现蓝色的LED，应该时接在GPIO2上，下面我们写一个驱动程序控制蓝色LED闪烁。
编写程序如下，并烧录到单片机中，可以看到LED正常闪烁。
```
#include <WiFi.h>
#define LED       2
void setup() {
   pinMode(LED, OUTPUT);
  
}

void loop() {
  digitalWrite(LED, LOW);
  delay(1000);
  digitalWrite(LED, HIGH);
  delay(1000);
}

```


