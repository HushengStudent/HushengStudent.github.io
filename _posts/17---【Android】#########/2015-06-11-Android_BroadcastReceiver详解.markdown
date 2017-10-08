---
layout: post
title:  "[Android]Android_BroadcastReceiver详解"
date:   2015-06-11 10:11:10
categories: 17---【Android】#########
comments: true
---

### 一、概述：
Android系统的四大组件之一BroadcastReceiver本质上就是一种全局监听器，用于监听系统全局的广播。

![图片](http://owk5gjdrg.bkt.clouddn.com/0014Android_BroadcastReceiver%E8%AF%A6%E8%A7%A3.png)

### 二、简介：
BroadcastReceiver用于接收程序所发出的Broadcast Intent，过程如下：

1.创建启动BroadcastReceiver的Intent。
2.调用Context的sendBroadcast()或者sendOrdereBroadcast()方法来启动指定的BroadcastReceiver。

#### ①发送自定义广播：

```java

public class MainActivity extends Activity {  
  
    private Button button;  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        button = (Button) findViewById(R.id.button1);  
        button.setOnClickListener(new OnClickListener(){  
  
            @Override  
            public void onClick(View v) {  
                // TODO 自动生成的方法存根  
                Intent intent = new Intent();  
                intent.setAction("com.example.blogbroadcastreceiver.husheng");  
                intent.putExtra("husheng", "hello BroadcastReceiver !");  
                sendBroadcast(intent);  
            }  
              
        });  
    }  
```

```java

public class MyBroadcastReceiver extends BroadcastReceiver {  
  
    @Override  
    public void onReceive(Context context, Intent intent) {  
        // TODO 自动生成的方法存根  
Toast.makeText(context, intent.getAction()+"\n"+intent.getStringExtra("husheng"), Toast.LENGTH_LONG).show();  
    }  
  
}  
```

```xml

<receiver android:name=".MyBroadcastReceiver">  
    <intent-filter >  
        <action android:name="com.example.blogbroadcastreceiver.husheng"/>  
    </intent-filter>              
</receiver>
```

#### ②发送有序广播：
```java

public class MainActivity extends Activity {  
  
    private Button button;  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        button = (Button) findViewById(R.id.button1);  
        button.setOnClickListener(new OnClickListener(){  
  
            @Override  
            public void onClick(View v) {  
                // TODO 自动生成的方法存根  
                Intent intent = new Intent();  
                intent.setAction("com.example.blogbroadcastreceiver.husheng");  
                intent.putExtra("husheng", "hello BroadcastReceiver !");  
                sendOrderedBroadcast(intent,null);  
            }  
              
        });  
    }  
}  
```

```java

public class MyBroadcastReceiver extends BroadcastReceiver {  
  
    @Override  
    public void onReceive(Context context, Intent intent) {  
        // TODO 自动生成的方法存根  
Toast.makeText(context, "first\n"+intent.getStringExtra("husheng"), Toast.LENGTH_LONG).show();  
    abortBroadcast();  
    }  
}  
```

```java

public class anotherBroadcastReceiver extends BroadcastReceiver {  
  
    @Override  
    public void onReceive(Context context, Intent intent) {  
        // TODO 自动生成的方法存根  
        Toast.makeText(context, "second\n"+intent.getStringExtra("husheng"), Toast.LENGTH_LONG).show();  
    }  
  
}  

```

```java

<receiver android:name=".MyBroadcastReceiver">  
            <intent-filter android:priority="100">  
                <action android:name="com.example.blogbroadcastreceiver.husheng"/>  
            </intent-filter>  
              
        </receiver>  
        <receiver android:name=".anotherBroadcastReceiver">  
            <intent-filter >  
                <action android:name="com.example.blogbroadcastreceiver.husheng"/>  
            </intent-filter>  
              
        </receiver>  
```

配置文件中，前一接收者中  android:priority="100" 保证它的优先级较高，先接收到广播。
没有加abortBroadcast()之前，两者都可以接收到广播，加了abortBroadcast()之后，阻断了广播，第二个接收者没有接收到广播。

### 三、深入：
使用本地广播：
```java

public class MainActivity extends Activity {  
  
    private IntentFilter intentFilter;  
    private LocalReceiver localReceiver;  
    private LocalBroadcastManager localBroadcastManager;  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        localBroadcastManager = LocalBroadcastManager.getInstance(this);  
        Button button = (Button) findViewById(R.id.button);  
        button.setOnClickListener(new OnClickListener() {  
            @Override  
            public void onClick(View v) {  
                Intent intent = new Intent(  
                        "com.example.broadcasttest.LOCAL_BROADCAST");  
                localBroadcastManager.sendBroadcast(intent);  
            }  
        });  
        intentFilter = new IntentFilter();  
        intentFilter.addAction("com.example.broadcasttest.LOCAL_BROADCAST");  
        localReceiver = new LocalReceiver();  
        localBroadcastManager.registerReceiver(localReceiver, intentFilter);  
    }  
  
    @Override  
    protected void onDestroy() {  
        super.onDestroy();  
        localBroadcastManager.unregisterReceiver(localReceiver);  
    }  
  
    class LocalReceiver extends BroadcastReceiver {  
  
        @Override  
        public void onReceive(Context context, Intent intent) {  
            Toast.makeText(context, "received local broadcast",  
                    Toast.LENGTH_SHORT).show();  
        }  
    }  
}  
```

本地广播的优势：

1.安全，不会泄露数据。

2.高效。

### 四、总结：

![图片](http://owk5gjdrg.bkt.clouddn.com/0015Android_BroadcastReceiver%E8%AF%A6%E8%A7%A3.png)
![图片](http://owk5gjdrg.bkt.clouddn.com/0016Android_BroadcastReceiver%E8%AF%A6%E8%A7%A3.png)
![图片](http://owk5gjdrg.bkt.clouddn.com/0018Android_BroadcastReceiver%E8%AF%A6%E8%A7%A3.png)