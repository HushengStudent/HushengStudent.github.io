---
layout: post
title:  "[Android]Android_Activity详解"
date:   2015-06-08 17:57:10
categories: 17---【Android】#########
comments: true
---

#### 一、概述：
Activity是android四大组件之一。

#### 二、简介：
①一个最简单的Activity实例：

（覆写父类onCreate方法，实现main.xml文件布局）
```java

public class HelloActivity extends Activity {  
  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
    }  
}  
```
②Activity的配置：

android要求所有组件（Activity、Service、ContentProvider、BroadcastReceive）都必须显示进行配置：
```xml

<?xml version="1.0" encoding="utf-8"?>  
<manifest xmlns:android="http://schemas.android.com/apk/res/android"  
    package="com.husheng.activity"  
    android:versionCode="1"  
    android:versionName="1.0" >  
    <uses-sdk  
        android:minSdkVersion="14"  
        android:targetSdkVersion="21" />  
    <application  
        android:allowBackup="true"  
        android:icon="@drawable/ic_launcher"  
        android:label="@string/app_name"  
        android:theme="@style/AppTheme" >  
        <activity  
            android:name=".MainActivity"  
            android:label="@string/app_name" >  
            <intent-filter>  
                <action android:name="android.intent.action.MAIN" />  
                <category android:name="android.intent.category.LAUNCHER" />  
            </intent-filter>  
        </activity>  
    </application>  
</manifest>  
```
③启动、关闭Activity：

启动：

1.startActivity(Intent intent):启动其他Activity

2.startActivityForResult(Intent intent,int requestCode):以指定的请求码(requestCode)启动Activity。

关闭：

1.finish()：结束当前Activity。

2.finishActivity(int requestCode)：结束以startActivityForResult(Intent intent,int requestCode)启动的Activity。

下面创建两个Activity来演示：
```java

/** 
*StartActivity 
*/  
public class StartActivity extends Activity{  
    @Override  
    public void onCreate(Bundle savedInstanceState){  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
        Button button1 = (Button) findViewById(R.id.button1);  
        <span style="font-family: Arial, Helvetica, sans-serif;">button1</span><span style="font-family: Arial, Helvetica, sans-serif;">.setOnClickListener(new OnClickListener(){</span>  
            @Override  
            public void onClick(View source){  
                Intent intent = new Intent(StartActivity.this,SecondActivity.class);  
                startActivity(intent);  
            }  
        });  
    }  
}  
  
/** 
*SecondActivity 
*/  
  
public class SecondActivity extends Activity{  
    @Override  
    public void onCreate(Bundle savedInstanceState){  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.second);  
        Button back = (Button) findViewById(R.id.back);  
        Button close = (Button) findViewById(R.id.close);  
        back.setOnClickListener(new OnClickListener(){  
            @Override  
            public void onClick(View source){  
                Intent intent = new Intent(SecondActivity.this,StartActivity.class);  
                startActivity(intent);  
            }  
        });  
        close.setOnClickListener(new OnClickListener(){  
            @Override  
            public void onClick(View source){  
                Intent intent = new Intent(SecondActivity.this,StartActivity.class);  
                startActivity(intent);  
                finish();  
            }  
        });  
    }  
}  
```
④使用Bundle在Activity之间传递数据：

当一个Activiy跳转到另外一个Activiy时，常常伴随着数据的传递。

而数据的交换通常使用Intent。

Intent提供了多个重载的方法来携带数据：

下面通过一个实例进行说明：
```java

public class Person implements Serializable  
{  
    private static final long serialVersionUID = 1L;  
  
    private Integer id;  
    private String name;  
    private String pass;  
    private String gender;  
  
    public Person()  
    {  
    }  
  
    public Person(String name, String pass, String gender)  
    {  
        this.name = name;  
        this.pass = pass;  
        this.gender = gender;  
    }  
    public Integer getId()  
    {  
        return id;  
    }  
    public void setId(Integer id)  
    {  
        this.id = id;  
    }  
    public String getName()  
    {  
        return name;  
    }  
    public void setName(String name)  
    {  
        this.name = name;  
    }  
    public String getPass()  
    {  
        return pass;  
    }  
    public void setPass(String pass)  
    {  
        this.pass = pass;  
    }  
    public String getGender()  
    {  
        return gender;  
    }  
    public void setGender(String gender)  
    {  
        this.gender = gender;  
    }  
  
}  
```
```java

public class BundleTest extends Activity  
{  
    @Override  
    public void onCreate(Bundle savedInstanceState)  
    {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
        Button button1 = (Button) findViewById(R.id.button1);  
        button1.setOnClickListener(new OnClickListener()  
        {  
            public void onClick(View v)  
            {  
                EditText name = (EditText)findViewById(R.id.name);  
                EditText passwd = (EditText)findViewById(R.id.passwd);  
                RadioButton male = (RadioButton) findViewById(R.id.male);  
                String gender = male.isChecked() ? "男 " : "女";  
                Person p = new Person(name.getText().toString(), passwd.getText().toString(), gender);  
                Bundle data = new Bundle();  
                data.putSerializable("person", p);  
                Intent intent = new Intent(BundleTest.this,ResultActivity.class);  
                intent.putExtras(data);  
                startActivity(intent);  
            }  
        });  
    }  
}  
```
```java

public class ResultActivity extends Activity  
{  
    @Override  
    public void onCreate(Bundle savedInstanceState)  
    {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.result);  
        TextView name = (TextView) findViewById(R.id.name);  
        TextView passwd = (TextView) findViewById(R.id.passwd);  
        TextView gender = (TextView) findViewById(R.id.gender);  
        Intent intent = getIntent();  
        Person p = (Person) intent.getSerializableExtra("person");  
        name.setText("您的用户名为：" + p.getName());  
        passwd.setText("您的密码为：" + p.getPass());  
        gender.setText("您的性别为：" + p.getGender());  
    }  
}  
```

这样就实现了在两个Activity之间传递数据了。

⑤启动其他Activity并返回一个结果：

1.当前Activity需要重写onActivityResult(int requestCode,int resultCode,Intent intent).

2.被启动的Activity需要调用setResult()方法设置处理结果。

#### 三、深入：
①Activity的生命周期：

从官方文档可以看出，Activity的生命周期分为七步。

②Activity的4种加载模式：

1.standard模式：标准模式。

创建一个新的Activity，并将该Activity添加到当前的Task栈中，这种模式不会启动新的Task。

2.singleTop模式：与standard类似，唯一区别是，当目标Activity已经存在且位于Task栈顶时，不会再新建一个Activity，而会复用当前Activity。

3.singleTask模式：与singleTop的区别是，当目标Activity存在，但不位于Task栈顶时，会将目标Activity移到Task栈顶，而不会新建。

4.singleInstance模式：与singleTask的模式的区别是，会重新建一个Task栈。

③Fragment：

Fragment可片以理解为Activity的段：

关于Fragment的知识点，将在下一篇博客中详细讲解。

#### 四、总结
掌握Activity的常见用法。

掌握Activity与Intent的使用。

掌握Activity在AndroidManifest.xml文件中的配置。