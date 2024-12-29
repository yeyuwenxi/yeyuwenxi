---
title: Android数据库应用之单词本
date: 2021-06-28 17:38:56
tags:
- Android
- 数据库
- SQLite
categories: Android
---
整理一下以前做过的一个小应用，主要是基于SQLite实现了一个简单的记录单词的app.

### 数据库的创建
```
public class DatabaseHelper extends SQLiteOpenHelper {
    private static final int VERSION = 1;
    // private static final String SWORD="SWORD";
    //三个不同参数的构造函数
    //带全部参数的构造函数，此构造函数必不可少
    public DatabaseHelper(Context context, String name, SQLiteDatabase.CursorFactory factory,
                          int version) {
        super(context, name, factory, version);

    }


    public DatabaseHelper(Context context,String name,int version){
        this(context, name,null,version);
    }
    //创建数据库
    public void onCreate(SQLiteDatabase db) {

        //创建数据库sql语句
        String sql = "create table danciben(id int,danci text,zhushi text,xiangqing text,xingbiao int,beihui int)";
        //执行创建数据库操作
        db.execSQL(sql);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        //创建成功，日志输出提示

    }

}
```
### 添加
```
db = new DatabaseHelper(tianjia.this,"danciben", null, 1);
 //数据库实际上是没有被创建或者打开的，直到getWritableDatabase() 或者 getReadableDatabase() 方法中的一个被调用时才会进行创建或者打开
  db.getWritableDatabase();
  insertData(db.getReadableDatabase(),ciyu,zhushi,xiangqing);
```
### 删除
```
db.getWritableDatabase().delete("danciben","danci=?", new String[]{danci});
```
### 查找
使用游标进行查找
like关键词可支持模糊查找
```
 Cursor cursor=db.getReadableDatabase().query("danciben",null,"danci like?", new String[]{key+"%"},null,null,null);
   while(cursor.moveToNext()){
    String danci=cursor.getString(cursor.getColumnIndex("danci"));
     String zhushi=cursor.getString(cursor.getColumnIndex("zhushi"));
     }
```
### 部分功能测试结果

<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210628_3.png" >
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210628_4.png" >
<img src="https://cdn.jsdelivr.net/gh/yeyuwenxi/images.github.io/20210628_5.png" >

### 程序源码
[github下载链接](https://github.com/yeyuwenxi/toolcase/blob/main/datalibrary2.zip)
### 参考文章
[【Android】SQLite数据库基本用法详解（极简洁）](https://blog.csdn.net/midnight_time/article/details/80834198?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160522951719725222401033%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=160522951719725222401033&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-6-80834198.first_rank_ecpm_v3_pc_rank_v2&utm_term=androidSQLite&spm=1018.2118.3001.4449)
