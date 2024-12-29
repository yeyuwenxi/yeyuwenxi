---
title: Android实现TCP服务器
date: 2021-03-30 22:26:35
categories: Android
---


## 1.获取本机ip地址
建立socket连接之前，我们首先要获取本地的ip地址，这样，才能让客户端通过ip地址连接到服务器

```
  /**获得IP地址，分为两种情况，一是wifi下，二是移动网络下，得到的ip地址是不一样的*/
     String getIPAddress() {
        Context context=MainActivity.this;
        NetworkInfo info = ((ConnectivityManager) context
                .getSystemService(Context.CONNECTIVITY_SERVICE)).getActiveNetworkInfo();
        if (info != null && info.isConnected()) {
            if (info.getType() == ConnectivityManager.TYPE_MOBILE) {//当前使用2G/3G/4G网络
                try {
                    //Enumeration<NetworkInterface> en=NetworkInterface.getNetworkInterfaces();
                    for (Enumeration<NetworkInterface> en = NetworkInterface.getNetworkInterfaces(); en.hasMoreElements(); ) {
                        NetworkInterface intf = en.nextElement();
                        for (Enumeration<InetAddress> enumIpAddr = intf.getInetAddresses(); enumIpAddr.hasMoreElements(); ) {
                            InetAddress inetAddress = enumIpAddr.nextElement();
                            if (!inetAddress.isLoopbackAddress() && inetAddress instanceof Inet4Address) {
                                return inetAddress.getHostAddress();
                            }
                        }
                    }
                } catch (SocketException e) {
                    e.printStackTrace();
                }
            } else if (info.getType() == ConnectivityManager.TYPE_WIFI) {//当前使用无线网络
                WifiManager wifiManager = (WifiManager) context.getSystemService(Context.WIFI_SERVICE);
                WifiInfo wifiInfo = wifiManager.getConnectionInfo();
                //调用方法将int转换为地址字符串
                String ipAddress = intIP2StringIP(wifiInfo.getIpAddress());//得到IPV4地址
                return ipAddress;
            }
        } else {
            //当前无网络连接,请在设置中打开网络
        }
        return null;
    }
    /**
     * 将得到的int类型的IP转换为String类型
     * @param ip
     * @return
     */
     String intIP2StringIP(int ip) {
        return (ip & 0xFF) + "." +
                ((ip >> 8) & 0xFF) + "." +
                ((ip >> 16) & 0xFF) + "." +
                (ip >> 24 & 0xFF);
    }
```
这里需要注意的是，我们要添加以下几个权限
使得app可以访问网络状态
```
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
```

## 2.建立socket连接
建立一个子线程，在子线程中建立socket连接
 

```
class ConnectThread extends Thread{
        OutputStream os;
        Socket socket;
        @Override
        public void run() {
            try {
               
                // 1.新建ServerSocket对象，创建指定端口的连接
                ServerSocket serverSocket = new ServerSocket(10000);

                // 2.进行监听
                socket = serverSocket.accept();// 开始监听10000端口，并接收到此套接字的连接。}}}
```

## 3.通过输入流接收消息
建立一个死循环，监听输入流来自客户端的消息
 

```
while (true)
                {
                    final byte[] buffer = new byte[1024];//创建接收缓冲区
                   InputStream inputStream = socket.getInputStream();
                    final int len = inputStream.read(buffer);//数据读出来，并且返回数据的长度
                    runOnUiThread(new Runnable()//不允许其他线程直接操作组件，用提供的此方法可以
                    {
                        public void run()
                        {
                            // TODO Auto-generated method stub
                            text1.append(new String(buffer,0,len)+"\r\n");
                        }
                    });	}
```

## 4.通过输出流发送消息
  通过输出流的write()方法将消息发送到客户端
```
os = socket.getOutputStream();
                String text="我是服务器";
                os.write(text.getBytes());
```
下面实现通过Edittext发送输入的内容
 

```
 fasong.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if(connectThread.socket!=null){
                String a=fasong_text.getText().toString();

                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        try {
                            connectThread.os.write(a.getBytes());
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                }).start();

            }else{
                    Toast.makeText(MainActivity.this,"请先建立连接",Toast.LENGTH_SHORT).show();
                }}

        });
```

## 5.测试

**注意，由于模拟器的原因，模拟器ip无法作为tcp服务器被访问，所以这一部分测试只能在真机上进行。**
经过测试，服务器可以正常地和客户端进行通信。
这里偷点懒，放一张界面图，懒得再截真机测试的图片了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210321165721149.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NoZW54aWhhbmh1aQ==,size_16,color_FFFFFF,t_70#pic_center)

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
       android:id="@+id/fasong_text"
       android:hint="请输入要发送的内容"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"/>
    <Button
        android:text="发送"
        android:id="@+id/fasong"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
    <TextView
        android:id="@+id/text1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="\n"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</LinearLayout>
```
Mainactivity

```
public class MainActivity extends AppCompatActivity {
    TextView text1;
    EditText fasong_text;
    ConnectThread connectThread;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //Toast.makeText(MainActivity.this,"123",Toast.LENGTH_SHORT).show();
        text1=findViewById(R.id.text1);
        fasong_text=findViewById(R.id.fasong_text);
        Button fasong=findViewById(R.id.fasong);

         connectThread=new ConnectThread();
        connectThread.start();
   //获取本地ip地址
        text1.append("本地ip地址为："+getIPAddress()+" 端口号为10000");
        text1.append("\n");


        fasong.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if(connectThread.socket!=null){
                String a=fasong_text.getText().toString();

                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        try {
                            connectThread.os.write(a.getBytes());
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                }).start();

            }else{
                    Toast.makeText(MainActivity.this,"请先建立连接",Toast.LENGTH_SHORT).show();
                }}

        });
    }

    /**获得IP地址，分为两种情况，一是wifi下，二是移动网络下，得到的ip地址是不一样的*/
     String getIPAddress() {
        Context context=MainActivity.this;
        NetworkInfo info = ((ConnectivityManager) context
                .getSystemService(Context.CONNECTIVITY_SERVICE)).getActiveNetworkInfo();
        if (info != null && info.isConnected()) {
            if (info.getType() == ConnectivityManager.TYPE_MOBILE) {//当前使用2G/3G/4G网络
                try {
                    //Enumeration<NetworkInterface> en=NetworkInterface.getNetworkInterfaces();
                    for (Enumeration<NetworkInterface> en = NetworkInterface.getNetworkInterfaces(); en.hasMoreElements(); ) {
                        NetworkInterface intf = en.nextElement();
                        for (Enumeration<InetAddress> enumIpAddr = intf.getInetAddresses(); enumIpAddr.hasMoreElements(); ) {
                            InetAddress inetAddress = enumIpAddr.nextElement();
                            if (!inetAddress.isLoopbackAddress() && inetAddress instanceof Inet4Address) {
                                return inetAddress.getHostAddress();
                            }
                        }
                    }
                } catch (SocketException e) {
                    e.printStackTrace();
                }
            } else if (info.getType() == ConnectivityManager.TYPE_WIFI) {//当前使用无线网络
                WifiManager wifiManager = (WifiManager) context.getSystemService(Context.WIFI_SERVICE);
                WifiInfo wifiInfo = wifiManager.getConnectionInfo();
                //调用方法将int转换为地址字符串
                String ipAddress = intIP2StringIP(wifiInfo.getIpAddress());//得到IPV4地址
                return ipAddress;
            }
        } else {
            //当前无网络连接,请在设置中打开网络
        }
        return null;
    }
    /**
     * 将得到的int类型的IP转换为String类型
     * @param ip
     * @return
     */
     String intIP2StringIP(int ip) {
        return (ip & 0xFF) + "." +
                ((ip >> 8) & 0xFF) + "." +
                ((ip >> 16) & 0xFF) + "." +
                (ip >> 24 & 0xFF);
    }
    class ConnectThread extends Thread{
        OutputStream os;
        Socket socket;
        @Override
        public void run() {
            try {

                // 1.新建ServerSocket对象，创建指定端口的连接
                ServerSocket serverSocket = new ServerSocket(10000);

                // 2.进行监听
                socket = serverSocket.accept();// 开始监听10000端口，并接收到此套接字的连接。
                // 3.拿到输入流（客户端发送的信息就在这里）

                Log.i("a","连接成功");
                 os = socket.getOutputStream();
                String text="我是服务器";
                os.write(text.getBytes());

                while (true)
                {
                    final byte[] buffer = new byte[1024];//创建接收缓冲区
                   InputStream inputStream = socket.getInputStream();
                    final int len = inputStream.read(buffer);//数据读出来，并且返回数据的长度
                    runOnUiThread(new Runnable()//不允许其他线程直接操作组件，用提供的此方法可以
                    {
                        public void run()
                        {
                            // TODO Auto-generated method stub
                            text1.append(new String(buffer,0,len)+"\r\n");
                        }
                    });	}


                // 关闭输入流
               // socket.shutdownInput();



                /*os.flush();
                // 关闭输出流
                socket.shutdownOutput();
                os.close();

                // 关闭IO资源
                bufReader.close();
                reader.close();
                is.close();

                socket.close();// 关闭socket
                serverSocket.close();// 关闭ServerSocket*/

            } catch (IOException e) {
                e.printStackTrace();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

}
```

## 7.参考文章
[Android实现TCP客户端](https://blog.csdn.net/chenxihanhui/article/details/115050849)
[Android网络编程之--Socket编程](https://blog.csdn.net/pingping_010/article/details/86527609)
[Android获得设备状态信息、Mac地址、IP地址的方法](https://www.jb51.net/article/153245.htm)
[Android 一步步实现TCP客户端](https://blog.csdn.net/lyndon_li/article/details/82263172?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160611526819725225053660%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=160611526819725225053660&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-2-82263172.first_rank_ecpm_v3_pc_rank_v2&utm_term=android%20tcp%E5%AE%A2%E6%88%B7%E7%AB%AF&spm=1018.2118.3001.4449)