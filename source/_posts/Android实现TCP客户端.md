---
title: Android实现TCP客户端
date: 2021-03-30 22:27:02
categories: Android
---


## 1.添加相关权限，使得android app可以访问网络

在AndroidManiffest.xml中添加

```
<uses-permission android:name="android.permission.INTERNET"/>
```

## 2.socket的连接

由于android平台的限制，与网络相关的操作只能在子线程中进行，所以这里我们单独建立一个线程用于socket的连接

```
 //子线程中进行网络相关操作
    class connectthread extends Thread {

        OutputStream outputStream=null;
        InputStream inputStream=null;
        @Override
        public void run() {

            //连接
            try {
                socket=new Socket(a, b);
                runOnUiThread(new Runnable()//不允许其他线程直接操作组件，用提供的此方法可以
                {
                    public void run()
                    {
                        // TODO Auto-generated method stub
                        Toast.makeText(MainActivity.this,"连接成功",Toast.LENGTH_SHORT).show();

                    }
                });
            } catch (UnknownHostException e) {
                // TODO Auto-generated catch block
                runOnUiThread(new Runnable()//不允许其他线程直接操作组件，用提供的此方法可以
                {
                    public void run()
                    {
                        // TODO Auto-generated method stub
                        Toast.makeText(MainActivity.this,"连接失败",Toast.LENGTH_SHORT).show();
                    }
                });
                e.printStackTrace();
            }catch (IOException e) {
                e.printStackTrace();
                runOnUiThread(new Runnable()//不允许其他线程直接操作组件，用提供的此方法可以
                {
                    public void run()
                    {
                        // TODO Auto-generated method stub
                        Toast.makeText(MainActivity.this,"连接失败",Toast.LENGTH_SHORT).show();
                    }
                });
            }
}
```
socket=new socket(a,b)这一方法可以建立一个tcp连接,其中，a为ip地址，b为端口号
如果连接成功，我们及通过Toast在屏幕中显示“连接成功”，若连接失败，则会转到异常中，我们通过Toast显示“连接失败”。

## 3.通过输出流发送消息

在子线程中添加以下代码，获取socket的输出流对象
并通过输出流对象的write()方法向服务器发送“123”
```
 try {
                outputStream=socket.getOutputStream();
                outputStream.write(123);
            } catch (IOException e) {
                e.printStackTrace();
            }
```
以上就是最简单的消息发送，下面我们通过edittext获取输入的内容，并将输入的内容发送给服务器

```
  //发送
        send.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //子线程中进行网络操作
    new Thread(new Runnable() {
        @Override
        public void run() {
            if(socket!=null){
            try {
                String text=out.getText().toString();
                lianjie.outputStream.write(text.getBytes());
            } catch (UnknownHostException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();

            }catch (IOException e) {
                e.printStackTrace();
            }}else{
                runOnUiThread(new Runnable()//不允许其他线程直接操作组件，用提供的此方法可以
                {
                    public void run()
                    {
                        // TODO Auto-generated method stub
                        Toast.makeText(MainActivity.this,"请先建立连接",Toast.LENGTH_SHORT).show();
                    }
                });
            }
        }
    }).start();

            }
        });
```
需要注意的是，发送的操作需要在子线程中进行，所以这里我又建立了一个线程来进行发送的操作，在线程中获取socket的输出流对象即可进行内容的发送。
在这里我加了一个提示信息，如果socket为空的话，则提示“请先建立连接”。

## 4.通过输入流获取消息

在子线程中建立一个死循环，时刻监听输入流，读取服务器发送来的消息

```
 try{
                while (true)
                {
                    final byte[] buffer = new byte[1024];//创建接收缓冲区
                    inputStream = socket.getInputStream();
                    final int len = inputStream.read(buffer);//数据读出来，并且返回数据的长度
                    runOnUiThread(new Runnable()//不允许其他线程直接操作组件，用提供的此方法可以
                    {
                        public void run()
                        {
                            // TODO Auto-generated method stub
                            receive.append(new String(buffer,0,len)+"\r\n");
                        }
                    });
                }
            }
            catch (IOException e) {

        }
```

## 5.测试结果

客户端
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210321163105573.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NoZW54aWhhbmh1aQ==,size_10,color_FFFFFF,t_70#pic_center)
服务器
![在这里插入图片描述](https://img-blog.csdnimg.cn/202103211631546.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NoZW54aWhhbmh1aQ==,size_16,color_FFFFFF,t_70#pic_center)
经过测试，服务器和客户端之间可以正常的发送和接收信息。

## 6.源代码

xml

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
        android:id="@+id/ip"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="ip"
        />
    <EditText
        android:id="@+id/port"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="port"
        />
    <EditText
        android:id="@+id/out"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="请输入要发送的内容"
        />
    <Button
        android:id="@+id/connect"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="连接"/>
    <Button
        android:id="@+id/send"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="发送"/>
    <TextView
        android:id="@+id/receive"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

</LinearLayout>
```
Mainactivity

```
public class MainActivity extends AppCompatActivity {
    String a;
    int b;
    connectthread lianjie;
    TextView receive;
    Socket socket=null;
    Button connect;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        EditText ip=findViewById(R.id.ip);
        EditText port=findViewById(R.id.port);
        EditText out=findViewById(R.id.out);
         receive=findViewById(R.id.receive);
         connect=findViewById(R.id.connect);
        Button send=findViewById(R.id.send);


        connect.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                a=ip.getText().toString();
                String c=port.getText().toString();
                if("".equals(a)||"".equals(c)){
                    Toast.makeText(MainActivity.this,"请输入ip和端口号",Toast.LENGTH_SHORT).show();
                }
                else{b=Integer.valueOf(c);
                 lianjie=new connectthread();
                lianjie.start();}

            }
        });



        //发送
        send.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //子线程中进行网络操作
    new Thread(new Runnable() {
        @Override
        public void run() {
            if(socket!=null){
            try {
                String text=out.getText().toString();
                lianjie.outputStream.write(text.getBytes());
            } catch (UnknownHostException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();

            }catch (IOException e) {
                e.printStackTrace();
            }}else{
                runOnUiThread(new Runnable()//不允许其他线程直接操作组件，用提供的此方法可以
                {
                    public void run()
                    {
                        // TODO Auto-generated method stub
                        Toast.makeText(MainActivity.this,"请先建立连接",Toast.LENGTH_SHORT).show();
                    }
                });
            }
        }
    }).start();

            }
        });






    }
    //子线程中进行网络相关操作
    class connectthread extends Thread {

        OutputStream outputStream=null;
        InputStream inputStream=null;
        @Override
        public void run() {

            //连接
            try {
                socket=new Socket(a, b);
                runOnUiThread(new Runnable()//不允许其他线程直接操作组件，用提供的此方法可以
                {
                    public void run()
                    {
                        // TODO Auto-generated method stub
                        Toast.makeText(MainActivity.this,"连接成功",Toast.LENGTH_SHORT).show();

                    }
                });
            } catch (UnknownHostException e) {
                // TODO Auto-generated catch block
                runOnUiThread(new Runnable()//不允许其他线程直接操作组件，用提供的此方法可以
                {
                    public void run()
                    {
                        // TODO Auto-generated method stub
                        Toast.makeText(MainActivity.this,"连接失败",Toast.LENGTH_SHORT).show();
                    }
                });
                e.printStackTrace();
            }catch (IOException e) {
                e.printStackTrace();
                runOnUiThread(new Runnable()//不允许其他线程直接操作组件，用提供的此方法可以
                {
                    public void run()
                    {
                        // TODO Auto-generated method stub
                        Toast.makeText(MainActivity.this,"连接失败",Toast.LENGTH_SHORT).show();
                    }
                });
            }
           if(socket!=null){
            //获取输出流对象
            try {
                outputStream=socket.getOutputStream();
                outputStream.write(123);
            } catch (IOException e) {
                e.printStackTrace();
            }

            try{
                while (true)
                {
                    final byte[] buffer = new byte[1024];//创建接收缓冲区
                    inputStream = socket.getInputStream();
                    final int len = inputStream.read(buffer);//数据读出来，并且返回数据的长度
                    runOnUiThread(new Runnable()//不允许其他线程直接操作组件，用提供的此方法可以
                    {
                        public void run()
                        {
                            // TODO Auto-generated method stub
                            receive.append(new String(buffer,0,len)+"\r\n");
                        }
                    });
                }
            }
            catch (IOException e) {

        }}
    };
}}
```

## 7.参考文章

[Android 一步步实现TCP客户端](https://blog.csdn.net/lyndon_li/article/details/82263172?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160611526819725225053660%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=160611526819725225053660&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-2-82263172.first_rank_ecpm_v3_pc_rank_v2&utm_term=android%20tcp%E5%AE%A2%E6%88%B7%E7%AB%AF&spm=1018.2118.3001.4449)
[Android网络编程之--Socket编程](https://blog.csdn.net/pingping_010/article/details/86527609)
[android 之TCP客户端编程](https://blog.csdn.net/qq_39400113/article/details/108183449?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161579882416780271565422%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=161579882416780271565422&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-108183449.first_rank_v2_pc_rank_v29&utm_term=android%20tcp%E5%AE%A2%E6%88%B7%E7%AB%AF)
[Android Studio TCP客户端实现](https://blog.csdn.net/weixin_48848716/article/details/107429683?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161579882416780271525479%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=161579882416780271525479&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_v2~rank_v29-16-107429683.first_rank_v2_pc_rank_v29&utm_term=android%20tcp%E5%AE%A2%E6%88%B7%E7%AB%AF)
[Android的SocketTCP客户端发送信息](https://blog.csdn.net/ASWaterbenben/article/details/90172103?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&dist_request_id=1328642.52964.16157988951892531&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control)
