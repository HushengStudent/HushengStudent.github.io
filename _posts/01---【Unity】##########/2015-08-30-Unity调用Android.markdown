---
layout: post
title:  "[Unity]Unity调用Android"
date:   2015-08-30 22:19:10
categories: 01---【Unity】##########
comments: true
---

新建Android工程,其中要引用Unity安装目录下的.Jar包。

```java
package com.hust.husheng;  
  
import android.app.Activity;  
  
import android.os.Bundle;  
import android.view.Menu;  
import android.view.MenuItem;  
  
import com.unity3d.player.UnityPlayerActivity;  
import com.unity3d.player.UnityPlayer;  
  
public class MainActivity extends Activity {  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
    }  
      
    public void Test1() {  
        UnityPlayer.UnitySendMessage("cube", "BackCall", "android");  
    }  
      
    public String Test2() {  
        return "husheng";  
    }  
  
}  
```

导出src文件夹为JAR文件。

拷贝JAR文件及下列文件到Unity。
```java
using UnityEngine;  
using System.Collections;  
  
  
public class Test : MonoBehaviour {  
  
    public string info;  
    // Use this for initialization  
    void Start () {  
        UnityTest1();  
        UnityTest2();  
    }  
      
    // Update is called once per frame  
    void Update () {  
          
    }  
  
    public void UnityTest1()  
    {  
        AndroidJavaClass jc = new AndroidJavaClass("com.unity3d.player.UnityPlayer");  
        AndroidJavaObject activity = jc.GetStatic<AndroidJavaObject>("currentActivity");  
        activity.Call("Test1");  
    }  
    public void UnityTest2()  
    {  
        AndroidJavaClass jc = new AndroidJavaClass("com.unity3d.player.UnityPlayer");  
        AndroidJavaObject activity = jc.GetStatic<AndroidJavaObject>("currentActivity");  
        info=activity.Call<string>("Test2");  
        Debug.Log(info);  
    }  
  
    public void BackCall(string str)  
    {  
        info = str;  
        Debug.Log(info);  
    }  
}  
```