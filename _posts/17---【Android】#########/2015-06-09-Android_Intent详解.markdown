---
layout: post
title:  "[Android]Android_Intent详解"
date:   2015-06-09 17:57:10
categories: 17---【Android】#########
comments: true
---

### 一、概述：
Intent封装应用程序需要启动某个组件的"意图"。Intent还是应用程序组件之间通信的重要媒介，如：两个Activity可以把需要交换的数据封装成Bundle对象，然后由Intent来携带Bundle对象，实现数据交换。

### 二、简介：
#### ①使用Intent启动系统组件：
Android应用程序中，Activity、Service、BroadcastReceive都可以使用Intent启动。

常见如下：

Activity：startActivity(Intent intent).

Service：xxx  startService(Intent service).

BroadcastReceiver：sendBroadcast(Intent intent).


#### ②Intent的属性及intent-filter配置：
Component属性：

```java

public class ComponentAttr extends Activity  
{  
    @Override  
    public void onCreate(Bundle savedInstanceState)  
    {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
        Button bn = (Button) findViewById(R.id.bn);  
        bn.setOnClickListener(new OnClickListener()  
        {  
            @Override  
            public void onClick(View arg0)  
            {  
            Intent intent = new Intent(ComponentAttr.this,SecondActivity.class);  
            startActivity(intent);  
            }  
        });  
    }  
}  
```
```java

public class SecondActivity extends Activity  
{  
    @Override  
    public void onCreate(Bundle savedInstanceState)  
    {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.second);  
        EditText show = (EditText) findViewById(R.id.show);  
        ComponentName comp = getIntent().getComponent();  
        show.setText("组件包名为：" + comp.getPackageName()+ "\n组件类名为：" + comp.getClassName());  
    }  
}  
```

该属性不需要使用<intent-filter.../>元素进行配置。

Action、Category属性与intent-filter配置：

```java

public class ActionAttr extends Activity  
{  
    public final static String ADDR_ACTION ="com.husheng.ADDR_ACTION";  
    @Override  
    public void onCreate(Bundle savedInstanceState)  
    {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
        Button bn = (Button) findViewById(R.id.bn);  
        bn.setOnClickListener(new OnClickListener()  
        {  
            @Override  
            public void onClick(View arg0)  
            {  
                Intent intent = new Intent();  
                intent.setAction(ActionAttr.ADDR_ACTION);  
                startActivity(intent);  
            }  
        });  
    }  
}  
```
```java

public class SecondActivity extends Activity  
{  
    @Override  
    public void onCreate(Bundle savedInstanceState)  
    {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.second);  
        EditText show = (EditText) findViewById(R.id.show);  
        String action = getIntent().getAction();  
        show.setText("Action为：" + action);  
    }  
}  
```
```xml

<activity android:name=".SecondActivity" android:label="@string/app_name">  
            <intent-filter>  
                <!-- 指定该Activity能响应Action为指定字符串的Intent -->  
                <action android:name="com.husheng.ADDR_ACTION" />  
                <!-- 指定该Activity能响应Action属性为helloWorld的Intent -->  
                <action android:name="helloWorld" />  
                <!-- 指定该Action能响应Category属性为指定字符串的Intent -->  
                <category android:name="android.intent.category.DEFAULT" />     
            </intent-filter>  
        </activity>  
```

#### ③指定Action、Category调用系统Activity：

#### ④向下一个活动传递数据：

```java

public class MainActivity extends Activity {  
  
    private Button button;  
    String data = "hello world";  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        button = (Button) findViewById(R.id.button1);         
        button.setOnClickListener(new OnClickListenter(){  
            @Override  
            public void onClick(View v) {  
                // TODO 自动生成的方法存根  
                Intent intent = new Intent(MainActivity.this,SecondActivity.class);  
                intent.putExtra("husheng", data);  
                startActivity(intent);  
            }             
        });  
    }  
}  
```
```java

public class SecondActivity extends Activity {  
  
    private TextView textView;  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        // TODO 自动生成的方法存根  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.second);  
        Intent intent = getIntent();  
        String data = intent.getStringExtra("husheng");  
        textView = (TextView) findViewById(R.id.textView1);  
        textView.setText(data);  
    }  
}  

```

#### ⑤返回数据给上一个活动：

```java

public class MainActivity extends Activity {  
  
    private Button button;  
    private TextView textView;  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        button = (Button) findViewById(R.id.button1);   
        textView = (TextView) findViewById(R.id.textView1);  
        button.setOnClickListener(new OnClickListenter(){  
            @Override  
            public void onClick(View v) {  
                // TODO 自动生成的方法存根  
                Intent intent = new Intent(MainActivity.this,SecondActivity.class);  
                startActivityForResult(intent,1);  
            }             
        });         
    }  
    @Override  
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {  
        // TODO 自动生成的方法存根  
        String returndate = data.getStringExtra("husheng");  
        textView.setText(returndate);  
    }  
}  
```
```java

public class SecondActivity extends Activity {  
  
    private Button button;  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        // TODO 自动生成的方法存根  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.second);  
        Intent intent = getIntent();  
        String data = intent.getStringExtra("husheng");  
        button = (Button)findViewById(R.id.button2);  
        button.setOnClickListener(new OnClickListener(){  
            @Override  
            public void onClick(View v) {  
                // TODO 自动生成的方法存根  
                Intent intent = new Intent();  
                intent.putExtra("husheng", "hello world");  
                setResult(RESULT_OK, intent);  
                finish();  
            }             
        });  
    }  
}  
```
### 三、总结：
掌握Intent的各种属性的功能和用法。

掌握AndroidManifest.xml文件中配置<intent-filter.../>元素。