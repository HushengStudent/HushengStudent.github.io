---
layout: post
title:  "[Android]Android_ContentProvider详解"
date:   2015-06-12 15:17:10
categories: 17---【Android】#########
comments: true
---

### 一、概述：
Contentprovider是不同应用程序之间进行数据交换的标准API，ContentProvider以某种Uri的形式对外提供数据，允许其他应用访问或修改数据，其他应用程序通过使用ContentResolver根据Uri去访问操作指定数据。
![图片](http://owk5gjdrg.bkt.clouddn.com/0021Android_ContentProvider%E8%AF%A6%E8%A7%A3.png)
![图片](http://owk5gjdrg.bkt.clouddn.com/0022Android_ContentProvider%E8%AF%A6%E8%A7%A3.png)

### 二、简介：
#### ①开发ContentProvider的步骤：
1、定义ContentProvider类。

2、在AndroidManifest.xml文件中注册ContentProvider，并为它绑定一个Uri。

3、重写如下方法：

public boolean onCreate()

初始化内容提供器的时候调用。

public Cursor query()从内容提供器中查询数据。

public Uri insert()向内容提供器中添加一条数据。

public int update()更新内容提供器中已有的数据。

public int delete()从内容提供器中删除数据。

public String getType()根据传入的内容URI来返回相应的MIME类型。

#### ②ContentResolver的使用：

Context提供如下方法获取ContentResolver对象：

getContentResolver():

1.insert(Uri url,ContentValues values):插入values对应的数据。

2.delete(Uri uri,ContentValues values,String where,String[] selectionArgs):删除Uri对应的ContentProvider中where提交
匹配的数据。

3.update(Uri uri,ContentValues values,String where,String[] selectionArgs):更新Uri对应的ContentProvider中where提交匹配的数据。

4.query(Uri uri,String[] projection,String[] selectionArgs,String sortOrder):查询Uri对应的ContentProvider中where提交匹配的数据。

#### ③开发ContentProvider：
```java
public class FirstProvider extends ContentProvider  
{  
    // 第一次创建该ContentProvider时调用该方法  
    @Override  
    public boolean onCreate()  
    {  
        System.out.println("onCreate方法被调用");  
        return true;  
    }  
  
    // 该方法的返回值代表了该ContentProvider所提供数据的MIME类型  
    @Override  
    public String getType(Uri uri)  
    {  
        System.out.println("getType方法被调用");  
        return null;  
    }  
  
    // 实现查询方法，该方法应该返回查询得到的Cursor  
    @Override  
    public Cursor query(Uri uri, String[] projection, String where,  
        String[] whereArgs, String sortOrder)  
    {  
        System.out.println(uri + "query方法被调用");  
        System.out.println("where参数为：" + where);  
        return null;  
    }  
  
    // 实现插入的方法，该方法应该新插入的记录的Uri  
    @Override  
    public Uri insert(Uri uri, ContentValues values)  
    {  
        System.out.println(uri + "insert方法被调用");  
        System.out.println("values参数为：" + values);  
        return null;  
    }  
  
    // 实现删除方法，该方法应该返回被删除的记录条数  
    @Override  
    public int delete(Uri uri, String where, String[] whereArgs)  
    {  
        System.out.println(uri + "delete方法被调用");  
        System.out.println("where参数为：" + where);  
        return 0;  
    }  
  
    // 实现删除方法，该方法应该返回被更新的记录条数  
    @Override  
    public int update(Uri uri, ContentValues values, String where,  
        String[] whereArgs)  
    {  
        System.out.println(uri + "update方法被调用");  
        System.out.println("where参数为："  
            + where + ",values参数为：" + values);  
        return 0;  
    }  
}  
```
配置ContentProvider：
```java
<application  
    android:icon = "@drawable/ic_launcher">  
    <provider  
        android:name=".FirstProvider"  
        android:authorities = "com.contentprovider.husheng"  
        android:exported = "ture"/>  
</application>  
```
使用ContentResolver调用方法：
```java
public class FirstResolver extends Activity  
{  
    ContentResolver contentResolver;  
    Uri uri = Uri.parse("content://com.contentprovider.husheng.firstprovider/");  
  
    @Override  
    public void onCreate(Bundle savedInstanceState)  
    {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
        // 获取系统的ContentResolver对象  
        contentResolver = getContentResolver();  
    }  
  
    public void query(View source)  
    {  
        // 调用ContentResolver的query()方法。  
        // 实际返回的是该Uri对应的ContentProvider的query()的返回值  
        Cursor c = contentResolver.query(uri, null  
            , "query_where", null, null);  
        Toast.makeText(this, "远程ContentProvide返回的Cursor为：" + c,  
            Toast.LENGTH_LONG).show();  
    }  
  
    public void insert(View source)  
    {  
        ContentValues values = new ContentValues();  
        values.put("name", "fkjava");  
        // 调用ContentResolver的insert()方法。  
        // 实际返回的是该Uri对应的ContentProvider的insert()的返回值  
        Uri newUri = contentResolver.insert(uri, values);  
        Toast.makeText(this, "远程ContentProvide新插入记录的Uri为："  
            + newUri, Toast.LENGTH_LONG).show();  
    }  
  
    public void update(View source)  
    {  
        ContentValues values = new ContentValues();  
        values.put("name", "fkjava");  
        // 调用ContentResolver的update()方法。  
        // 实际返回的是该Uri对应的ContentProvider的update()的返回值  
        int count = contentResolver.update(uri, values  
            , "update_where", null);  
        Toast.makeText(this, "远程ContentProvide更新记录数为："  
            + count, Toast.LENGTH_LONG).show();  
    }  
  
    public void delete(View source)  
    {  
        // 调用ContentResolver的delete()方法。  
        // 实际返回的是该Uri对应的ContentProvider的delete()的返回值  
        int count = contentResolver.delete(uri  
            , "delete_where", null);  
        Toast.makeText(this, "远程ContentProvide删除记录数为："  
            + count, Toast.LENGTH_LONG).show();  
    }  
}  
```
### 三、深入：
#### ①操作系统的ContentProvider：
#### ②监听ContentProvider的数据变化：
监听用户发出的短信：
```java
public class MonitorSms extends Activity  
{  
    @Override  
    public void onCreate(Bundle savedInstanceState)  
    {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
        // 为content://sms的数据改变注册监听器  
        getContentResolver().registerContentObserver(  
            Uri.parse("content://sms"), true,  
            new SmsObserver(new Handler()));  
    }  
  
    // 提供自定义的ContentObserver监听器类  
    private final class SmsObserver extends ContentObserver  
    {  
        public SmsObserver(Handler handler)  
        {  
            super(handler);  
        }  
  
        public void onChange(boolean selfChange)  
        {  
            // 查询发送箱中的短信(处于正在发送状态的短信放在发送箱)  
            Cursor cursor = getContentResolver().query(  
                Uri.parse("content://sms/outbox")  
                , null, null, null, null);  
            // 遍历查询得到的结果集，即可获取用户正在发送的短信  
            while (cursor.moveToNext())  
            {  
                StringBuilder sb = new StringBuilder();  
                // 获取短信的发送地址  
                sb.append("address=").append(cursor  
                    .getString(cursor.getColumnIndex("address")));  
                // 获取短信的标题  
                sb.append(";subject=").append(cursor  
                    .getString(cursor.getColumnIndex("subject")));  
                // 获取短信的内容  
                sb.append(";body=").append(cursor  
                    .getString(cursor.getColumnIndex("body")));  
                // 获取短信的发送时间  
                sb.append(";time=").append(cursor  
                    .getLong(cursor.getColumnIndex("date")));  
                System.out.println("Has Sent SMS::" + sb.toString());  
            }  
        }  
    }  
}  
```
### 四、总结：
掌握ContentResolver、ContentProvider和ContentObserver的使用。