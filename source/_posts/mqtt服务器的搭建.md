---
title: mqtt服务器的搭建
date: 2021-04-25 12:39:23
tags:
- mqtt
categories: mqtt
---
### 简介
MQTT（Message Queuing Telemetry Transport，消息队列遥测传输协议），是一种基于发布/订阅（publish/subscribe）模式的"轻量级"通讯协议，该协议构建于TCP/IP协议上，由IBM在1999年发布。MQTT最大优点在于，可以以极少的代码和有限的带宽，为连接远程设备提供实时可靠的消息服务。作为一种低开销、低带宽占用的即时通讯协议，使其在物联网、小型设备、移动应用等方面有较广泛的应用。

MQTT是一个基于客户端-服务器的消息发布/订阅传输协议。MQTT协议是轻量、简单、开放和易于实现的，这些特点使它适用范围非常广泛。在很多情况下，包括受限的环境中，如：机器与机器（M2M）通信和物联网（IoT）。其在，通过卫星链路通信传感器、偶尔拨号的医疗设备、智能家居、及一些小型化设备中已广泛使用。
*(以上内容来自百度百科)*

mqtt是一种基于发布/订阅的消息传输协议，客户端可以发送或接受某个主题的消息，而服务器则是用来为各个客户端推送消息的中间件，通常称为代理。

通俗地说，如果把我们每个人的qq账号看做是客户端，那么腾讯的服务器就类似于mqtt的代理，只有有了这个代理，各个客户端之间才能正常通信，所以说代理（mqtt服务器）是mqtt通信的重要组成部分。

### 安装
下面直接进入正题，开始我们mqtt服务器的搭建。
常见的mqtt服务器软件有很多种，这里我们采用EMQX来进行我们mqtt服务器的搭建。
EMQX的使用文档如下:
[EMQX](https://docs.emqx.cn/broker/v4.3/)

首先登陆自己购买的服务器的管理界面（我这里使用的是腾讯云的服务器），打开webshell连接到服务器。
在webshell中输入以下命令:
`curl https://repos.emqx.io/install_emqx.sh | bash`
如果显示权限不够，请切换到root用户执行该命令
* 显示以下结果，则说明安装成功

<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210425_1.png" >

* 最后，在shell中输入`emqx start`启动mqtt服务器

### 测试
EMQ X 提供了 Dashboard 以方便用户管理设备与监控相关指标。通过 Dashboard，你可以查看服务器基本信息、负载情况和统计数据，可以查看某个客户端的连接状态等信息甚至断开其连接，也可以动态加载和卸载指定插件。除此之外，EMQ X Dashboard 还提供了规则引擎的可视化操作界面，同时集成了一个简易的 MQTT 客户端工具供用户测试使用。

我们可以通过ip:18083来访问dashboard.
如果访问失败，检查服务器的安全组规则是否放行该端口。
Dashboard的默认用户名是 admin，密码是 public。
在设置中可以将语言改成中文。
dashboard提供了一个websocket,可以作为一个在线的mqtt客户端来进行测试。
* websocket
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210425_2.png" >

我们可以使用一个本地的mqtt客户端，来和websocket进行通信。
这里我们使用mqttx来进行测试，下载链接如下。
[mqttx客户端](https://mqttx.app/cn/)

在websocket中订阅一个test主题，并在该主题下发布消息，可以看到，在下方的列表中显示了该主题下的消息。
* websocket
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210425_3.png" >
使用本地的mqtt客户端连接到mqtt服务器
* 本地客户端连接到服务器
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210425_4.png" >
* 在本地mqtt客户端订阅test主题，并在test主题下发布消息“hello”
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210425_5.png" >
* 在websocket订阅test主题，并在test主题下发布消息“hi”
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210425_6.png">
可以看到，websocket和本地的mqtt客户端都收到了他们互相发布的消息，测试成功。同时，我们也可以看到，当客户端订阅一个主题的时候，即使主题下有一个消息是自己发布的，客户端也会收到这个消息。

### 拓展知识
未完待续。。。



