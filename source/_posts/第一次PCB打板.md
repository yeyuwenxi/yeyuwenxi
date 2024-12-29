---
title: 第一次PCB打板
date: 2021-05-28 16:41:34
tags: 
- PCB
- 矩阵键盘
categories: PCB
---
记录一下第一次打PCB板。
虽然之前用AD画过原理图和PCB版图，但还一直没有真正打过板，碰巧捷配在搞活动，每月领券免费打板，就试着打了一次PCB板。
第一次打板，没搞什么特别复杂的电路，一是怕出错，二是手上没有特别多的器件可以焊在PCB板上验证。
这次板子主要就是一个简单的流水灯和4x4的矩阵键盘，自己手动画了画四角按键的封装。
画完之后就向捷配上传了工程文件和生产文件，几天后收到了板子，没想到的是居然发了六块。
下面直接放图
* PCB板
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210528_1.jpg" width="60%" height="60%" style="transform:rotate(270deg)"   >
* 焊接成品
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210528_2.jpg" width="60%" height="60%" style="transform:rotate(270deg)">

总的来说，这次打板还是比较成功的，当然也有一些小细节做的不是很好
* 流水灯和矩阵按键的接口排针放在了两侧，相对来说还是放在一侧比较好。
* 矩阵键盘行与行之间间隙略大
* 布线不是特别好看
* 流水灯其实可以放8个的，比6个更好写程序

另外本次打板没有铺铜，主要是没有确定的地，也没有哪个网络比其他网络要大很多。问了一些专业人士，简单的二层板不铺铜的话，也没有特别大的影响。
