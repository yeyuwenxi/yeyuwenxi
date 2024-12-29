---
title: esp8266通过串口发送http请求
date: 2021-04-26 09:31:57
tags:
- esp8266
categories: esp8266
---
这两天在想如何使用esp8266发送一个http请求，于是上网查找有没有相关的库，后来查资料发现，在tcp透传模式下，可以直接自己模拟一个http的请求。
仔细想想也确实是这样，http协议本身就是基于tcp协议实现的，通过tcp手动模拟http是完全可行的。
不得不说，计算机网络的一些知识好久不用都忘的差不多了，有些东西果然还是要在实践中加强认识。

### http协议
超文本传输协议（Hypertext Transfer Protocol，HTTP）是一个简单的请求-响应协议，它通常运行在TCP之上。它指定了客户端可能发送给服务器什么样的消息以及得到什么样的响应。
#### http请求头
请求头包括四个部分：
请求行
请求首部
空行
请求正文


#### http返回头
返回头同样包括四个部分：
状态行
消息报头
空行
响应正文

### 通过网络调试助手测试
我们测试使用的api如下
http://hn216.api.yesapi.cn/?s=App.Common_Weather.LiveWeather&return_data=0&city=%E9%95%BF%E6%B2%99&app_key=7DD22AAA0953B916BA785C889640AA62&sign=91C852984E53DF0E2DC87968E9EE32B8
这是果创云提供的一个天气查询的接口，通过发送get请求，可以得到对应城市的天气，返回的数据类型为json格式。
我们先使用浏览器访问一下这个接口
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210502_1.png" >
由于网络调试助手的限制，我们必须要知道api的服务器地址和端口号
在浏览器上点击检查网页，在network标签下的header标签中可以找到我们想要的信息

通过网络调试助手，输入刚刚查到的ip和端口号，建立一个tcp连接
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210502_2.png" >
发送相应格式的请求，可以看到，我们已经收到了api接口返回的数据
（这里由于调试助手编码格式的问题，中文会显示乱码）
### esp8266通过TCP透传发送http请求

调试时遇到了挺大的bug,模块的硬件电路出了点莫名其妙的问题，串口连接模块后，发送任何指令都没有反应，后来拿电压表测了一下，模块上电后CH_PD居然是低电平，于是又自己把CH_PD拉到高电平，解决了这一问题。
esp8266连接串口之后，依次发送如下指令：
`AT+RST`
//复位模块
`AT+CWMODE=1`
//进入STA模式
`AT+RST`
//复位生效上一条命令
`AT+CWJAP=”note”,”123456789” `
//连接到wifi
`AT+CIPSTART="TCP","192.168.1.115",8080`
//连接到tcp服务器
`AT+CIPMODE=1`
//开启透传模式
`AT+CIPSEND`
//进入透传

进入透传后，发送以下数据
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210502_3.png" >
可以看到，我们成功地收到了服务器返回的接口数据
（由于电脑编码格式问题，此处存在乱码）

