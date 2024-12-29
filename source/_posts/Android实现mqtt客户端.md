---
title: Android实现mqtt客户端
date: 2021-04-25 22:26:16
tags:
- Android
- mqtt
categories: mqtt
---
这一篇文章我们主要讲如何在Android上实现一个mqtt的客户端。
### 在gradle中添加依赖
```
implementation 'org.eclipse.paho:org.eclipse.paho.client.mqttv3:1.2.0'
implementation 'org.eclipse.paho:org.eclipse.paho.android.service:1.1.1'
```
### 添加相关的权限
```
<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.INTERNET" />
```
### 添加服务

`<service android:name="org.eclipse.paho.android.service.MqttService"></service>`

### 程序源码
* xml
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">
    <EditText
        android:id="@+id/zhuti"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="请输入要发送的主题"

        />
    <EditText
        android:id="@+id/neirong"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="请输入要发送的内容"

        />
    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="发送"
        />
    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        />

</LinearLayout>
```
* MainActivity.java
```
import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

import org.eclipse.paho.client.mqttv3.IMqttDeliveryToken;
import org.eclipse.paho.client.mqttv3.MqttCallback;
import org.eclipse.paho.client.mqttv3.MqttClient;
import org.eclipse.paho.client.mqttv3.MqttConnectOptions;
import org.eclipse.paho.client.mqttv3.MqttException;
import org.eclipse.paho.client.mqttv3.MqttMessage;
import org.eclipse.paho.client.mqttv3.persist.MemoryPersistence;

public class MainActivity extends AppCompatActivity {
    EditText zhuti;
    EditText neirong;
    Button fasong;
    TextView jieshou;

    MqttClient client;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //App lianjie=new App();

        String subTopic = "test";
        String pubTopic = "test";
        String content = "Hello World";
        int qos = 2;
        String broker = "tcp://49.235.68.28:1883";
        String clientId = "emqx_test";
        MemoryPersistence persistence = new MemoryPersistence();
        Log.i("aaa", "运行中");
        zhuti=findViewById(R.id.zhuti);
        neirong=findViewById(R.id.neirong);
        fasong=findViewById(R.id.button);
        jieshou=findViewById(R.id.textView);


        try {
             client = new MqttClient(broker, clientId, persistence);

            // MQTT 连接选项
            MqttConnectOptions connOpts = new MqttConnectOptions();
            connOpts.setUserName("emqx_test");
            connOpts.setPassword("emqx_test_password".toCharArray());
            // 保留会话
            connOpts.setCleanSession(true);

            // 设置回调
            client.setCallback(new OnMessageCallback1());

            // 建立连接
            Log.i("abc", "Connecting to broker: " + broker);
            client.connect(connOpts);

            Log.i("bcd", "Connected");
            Log.i("cde", "Publishing message: " + content);

            // 订阅
            client.subscribe(subTopic);

            // 消息发布所需参数
            MqttMessage  message = new MqttMessage(content.getBytes());
            message.setQos(qos);
            client.publish(pubTopic, message);
            Log.i("def", "Message published");

            //client.disconnect();
            // Log.i("efg","Disconnected");
            // client.close();
            //System.exit(0);
        } catch (MqttException me) {
            Log.i("1", "reason " + me.getReasonCode());
            Log.i("2", "msg " + me.getMessage());
            Log.i("3", "loc " + me.getLocalizedMessage());
            Log.i("4", "cause " + me.getCause());
            Log.i("5", "excep " + me);
            me.printStackTrace();
        }

        fasong.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
            if(zhuti.getText().toString().equals("")||neirong.getText().toString().equals("")){
                Toast.makeText(MainActivity.this, "请输入主题和内容", Toast.LENGTH_SHORT).show();
            }
            else{

                try {
                    MqttMessage  message = new MqttMessage(neirong.getText().toString().getBytes());
                    message.setQos(qos);
                    client.publish(zhuti.getText().toString(), message);
                } catch (MqttException e) {
                    e.printStackTrace();
                }}
            }
        });

    }
//回调函数
 class OnMessageCallback1 implements MqttCallback {
    public void connectionLost(Throwable cause) {
        // 连接丢失后，一般在这里面进行重连
        Log.i("duankai","连接断开，可以做重连");
    }

    public void messageArrived(String topic, MqttMessage message) throws Exception {
        // subscribe后得到的消息会执行到这里面
        Log.i("xiaoxi","接收消息主题:" + topic);
        Log.i("qos","接收消息Qos:" + message.getQos());
        Log.i("neirong","接收消息内容:" + new String(message.getPayload()));
        runOnUiThread(new Runnable()//不允许其他线程直接操作组件，用提供的此方法可以
        {
            public void run()
            {
                // TODO Auto-generated method stub
               // Toast.makeText(MainActivity.this, new String(message.getPayload()), Toast.LENGTH_SHORT).show();
               jieshou.append(new String(message.getPayload())+"\n");
            }
        });

    }

    public void deliveryComplete(IMqttDeliveryToken token) {
        Log.i("delivery","deliveryComplete---------" + token.isComplete());
    }
}
}
```
### 测试
模拟器与真机均测试通过
以下是模拟器测试的图片
app默认订阅了test主题，发送消息的主题可自己设置
* app
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210425_7.png" width="60%" height="60%">
* websocket(一个在线的mqtt客户端)
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210425_8.png" >

### 参考文章
[MQTT协议实现Android中的消息收发](https://www.cnblogs.com/jqnl/p/12660824.html)
[EMQX开发文档](https://docs.emqx.cn/broker/v4.3/development/java.html)