---
title: Android实例之必应每日一图
date: 2021-03-30 22:31:24
categories: Android
---
## Android实现每天更新一张图片

第一次写博客，记录一下自己学习android过程中做过的一些实例。

必应官网每天都会更新一张图片，我们可以通过这张图片的链接来获取这张图片，并让其显示在android app中。

### 一、图片的获取

必应提供了一个图片的接口，我们可以通过访问这个接口来获取每日更新的图片，接口链接如下所示：`https://cn.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1`
通过浏览器访问该链接可以得到以下所示数据

> {"images":[{"startdate":"20210124","fullstartdate":"202101241600","enddate":"20210125",**"url":"/th?id=OHR.ChurchRock_ZH-CN6926315999_1920x1080.jpg&rf=LaDigue_1920x1080.jpg&pid=hp"**,"urlbase":"/th?id=OHR.ChurchRock_ZH-CN6926315999","copyright":"红石公园教堂岩附近的雷击，新墨西哥州 (© Tim Fitzharris/Minden Pictures)","copyrightlink":"https://www.bing.com/search?q=%E7%BA%A2%E7%9F%B3%E5%85%AC%E5%9B%AD%E6%95%99%E5%A0%82&form=hpcapt&mkt=zh-cn","title":"","quiz":"/search?q=Bing+homepage+quiz&filters=WQOskey:%22HPQuiz_20210124_ChurchRock%22&FORM=HPQUIZ","wp":true,"hsh":"cdae6c57dbb4fe473dfd4f93b7870b8b","drk":1,"top":1,"bot":1,"hs":[]}],"tooltips":{"loading":"正在加载...","previous":"上一个图像","next":"下一个图像","walle":"此图片不能下载用作壁纸。","walls":"下载今日美图。仅限用作桌面壁纸。"}}
> 
其中，url标签后的内容即为我们所需图片的地址，通过访问`http://cn.bing.com+该地址`就可以得到我们想要的图片，例如，上面得到的url为
`
/th?id=OHR.ChurchRock_ZH-CN6926315999_1920x1080.jpg&rf=LaDigue_1920x1080.jpg&pid=hp`
那我们访问

```
http://cn.bing.com/th?id=OHR.ChurchRock_ZH-CN6926315999_1920x1080.jpg&rf=LaDigue_1920x1080.jpg&pid=hp
```
就可以得到我们想要的图片。

### 二、在android上获取并显示图片

由于android自身的限制，网络操作只能在子线程中完成，这一点是我们写代码中特别需要注意的。因此我们要把访问api接口的代码写在子线程中。此外，在android中，与UI有关的操作只能在主线程或UI线程中完成，所以我们加载图片上要记得更换线程。

#### 1.访问api接口

这里我们通过android自带的httpurlconnection来访问，并将访问得到的json返回文件存储到字符串response中。
```
private void sendRequestWithHttpURLConnection() {
        // 开启线程来发起网络请求
        new Thread(new Runnable() {
            @Override
            public void run() {
                HttpURLConnection connection = null;
                BufferedReader reader = null;
                try {
                    URL url = new URL("https://cn.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1");
                    connection = (HttpURLConnection) url.openConnection();
                    connection.setRequestMethod("GET");
                    connection.setConnectTimeout(8000);
                    connection.setReadTimeout(8000);
                    InputStream in = connection.getInputStream();
                    // 下面对获取到的输入流进行读取
                    reader = new BufferedReader(new InputStreamReader(in));
                    StringBuilder response = new StringBuilder();
                    String line;
                    while ((line = reader.readLine()) != null) {
                        response.append(line);
                    }
                    parseJSONWithJSONObject(response.toString());
                   // showResponse(response.toString());
                    //Ui线程
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    if (reader != null) {
                        try {
                            reader.close();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                    if (connection != null) {
                        connection.disconnect();
                    }
                }
            }
        }).start();
    }
```
#### 2.json的解析
这里我们直接通过android自带的JSONArray和JSONObject来进行json的解析，并将解析到的结果和`http://cn.bing.com`拼接起来，得到我们想要的图片的链接，并将其存储在字符串url1中。
```
private void parseJSONWithJSONObject(String jsonData) {
        try {
           // JSONArray jsonArray = new JSONArray(jsonData);

            JSONArray jsonArray = new JSONObject(jsonData).getJSONArray("images");

            for (int i = 0; i < jsonArray.length(); i++) {
                JSONObject jsonObject = jsonArray.getJSONObject(i);
                String url = jsonObject.getString("url");

                Log.d("MainActivity", "url is " + url);
String url1="http://cn.bing.com"+url;
                showResponse(url1);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

#### 3.图片的加载
利用开源框架Glide进行图片的加载。
注意，图片的加载要写在UI线程中。
```
private void showResponse(final String response) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                // 在这里进行UI操作，将结果显示到界面上
                Glide.with(MainActivity.this).load(response).into(img);
              //  text.setText(response);
                Log.i("123",response);
            }
        });
    }
```


### 三、演示效果
![演示效果](https://img-blog.csdnimg.cn/20210125151637243.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NoZW54aWhhbmh1aQ==,color_FFFFFF,t_70 )

### 四、源代码

**1.xml**

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">


<ImageView
    android:id="@+id/img"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>

</LinearLayout>
```
这部分代码比较简单，就是线性布局中加了一个imageview.

**2.activity**

```
public class MainActivity extends AppCompatActivity {
    TextView text;
    ImageView img;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // text=findViewById(R.id.text);
       img=findViewById(R.id.img);
        sendRequestWithHttpURLConnection();


    }
    private void sendRequestWithHttpURLConnection() {
        // 开启线程来发起网络请求
        new Thread(new Runnable() {
            @Override
            public void run() {
                HttpURLConnection connection = null;
                BufferedReader reader = null;
                try {
                    URL url = new URL("https://cn.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1");
                    connection = (HttpURLConnection) url.openConnection();
                    connection.setRequestMethod("GET");
                    connection.setConnectTimeout(8000);
                    connection.setReadTimeout(8000);
                    InputStream in = connection.getInputStream();
                    // 下面对获取到的输入流进行读取
                    reader = new BufferedReader(new InputStreamReader(in));
                    StringBuilder response = new StringBuilder();
                    String line;
                    while ((line = reader.readLine()) != null) {
                        response.append(line);
                    }
                    parseJSONWithJSONObject(response.toString());
                   // showResponse(response.toString());
                    //Ui线程
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    if (reader != null) {
                        try {
                            reader.close();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                    if (connection != null) {
                        connection.disconnect();
                    }
                }
            }
        }).start();
    }
    private void showResponse(final String response) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                // 在这里进行UI操作，将结果显示到界面上
                Glide.with(MainActivity.this).load(response).into(img);
              //  text.setText(response);
                Log.i("123",response);
            }
        });
    }
    private void parseJSONWithJSONObject(String jsonData) {
        try {
           // JSONArray jsonArray = new JSONArray(jsonData);

            JSONArray jsonArray = new JSONObject(jsonData).getJSONArray("images");

            for (int i = 0; i < jsonArray.length(); i++) {
                JSONObject jsonObject = jsonArray.getJSONObject(i);
                String url = jsonObject.getString("url");

                Log.d("MainActivity", "url is " + url);
String url1="http://cn.bing.com"+url;
                showResponse(url1);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    }
```
