---
title: AD之PCB画板初探
date: 2021-04-15 23:48:19
tags: 
- PCB
- 数字电路
categories: PCB
---
最近由于选修课的要求，速成了一波AD，这两天完成了自己的第一个PCB板的绘制。

### 任务目标
1.三个开关控制一盏灯
2.使用按钮实现开关的功能

### 实验思路
1.三个开关控制一盏灯，算是比较基础的数电问题了，这里我采用了74LS86（异或门）来实现这一逻辑功能。

2.按钮实现开关的功能，这一点我上网查资料发现都是采用模拟电路实现的，采用电容充放电实现开关的功能，电路较为复杂，实现原理也较为复杂，这里就不讲了。

分享一篇文章，有兴趣的可以自己看看[点按式轻触开关如何实现自锁轻触开关带锁按键开关功能？](https://baijiahao.baidu.com/s?id=1603206971622118062&wfr=spider&for=pc&qq-pf-to=pcqq.group)
3.后来想了想，还是采用数字电路实现比较简单，通过JK触发器的电平翻转来实现电路开关的功能，最后通过带JK触发器的74LS109实现了这一功能。

### PCB绘制流程
1.原理图库的绘制
由于官方封装库中的门电路芯片都是只显示部分的，所以这里我自己绘制了74LS86和74LS109芯片的原理图。
* 74LS86
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210416_1.png" >
* 74LS109
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210416_1.png" >

2.原理图的绘制
* 原理图
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210416_4.png" >

**ps:** 经老师提醒，原理图中按钮部分存在问题，TTL芯片悬空时为高电平，所以芯片的CLK端口需要接一个下拉电阻。

3.PCB封装库的绘制
这里偷了点懒，直接采用了TI官方的封装库。
* D014_N封装
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210416_3.png" >

4.PCB图的绘制

这一步才算是到了PCB绘制的核心部分。
#### 器件摆放
AD会将原理图中用到所有器件直接转移到PCB文件中，我们需要将各个器件以合适的位置摆放好。
#### 布线
AD有自动布线功能，但还是推荐手动布线。
布线拐角采用钝角，降低电磁干扰。
多层板可能需要过孔，将各个板层进行电气连接。
#### 铺铜
将没有导线的地方铺铜，提高抗干扰能力。
#### 丝印
添加需要的标记符号和个性化定制文字内容。
#### 版型设计
对PCB板进行裁剪，打定位孔等操作，推荐矩形版型。
* 最后贴上我的第一个PCB板
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210416_5.png" >
(空白处有姓名等相关信息，已打码)