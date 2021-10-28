---
layout: post
title:  "[UIFramework]UIFramework之UnityEngine.Debug封装"
date:   2016-11-19 05:34:10
categories: 基础框架
comments: true
---

1、真机输出Log到文件

把Log信息写在手机的客户端里。把脚本挂在任意游戏对象上。

```java
using UnityEngine;  
using System.Collections;  
using System.Collections.Generic;  
using System.IO;  
using System.Text;  
  
public class LogOut : MonoBehaviour   
{  
    static List<string> mLines = new List<string>();  
    static List<string> mWriteTxt = new List<string>();  
    private string outpath;  
    void Start()  
    {  
        //Application.persistentDataPath 安卓沙盒路径  
        outpath = Application.persistentDataPath + "/Log.txt";  
        //每次启动客户端删除之前保存的Log  
        if (System.IO.File.Exists(outpath))  
        {  
            File.Delete(outpath);  
        }  
        //在这里做一个Log的监听  
        //Application.RegisterLogCallback(HandleLog);  
        Application.logMessageReceived += HandleLog;  
    }  
  
    void Update()  
    {  
        //因为写入文件的操作必须在主线程中完成，所以在Update中哦给你写入文件。  
        if (mWriteTxt.Count > 0)  
        {  
            string[] temp = mWriteTxt.ToArray();  
            foreach (string t in temp)  
            {  
   using (StreamWriter writer = new StreamWriter(outpath, true, Encoding.UTF8))  
                {  
                    writer.WriteLine(t);  
                }  
                mWriteTxt.Remove(t);  
            }  
        }  
    }  
  
    void HandleLog(string logString, string stackTrace, LogType type)  
    {  
        mWriteTxt.Add(logString);  
        if (type == LogType.Error || type == LogType.Exception)  
        {  
            Log(logString);  
            Log(stackTrace);  
        }  
    }  
  
    //把错误的信息保存起来，用来输出在手机屏幕上  
    static public void Log(params object[] objs)  
    {  
        string text = "";  
        for (int i = 0; i < objs.Length; ++i)  
        {  
            if (i == 0)  
            {  
                text += objs[i].ToString();  
            }  
            else  
            {  
                text += ", " + objs[i].ToString();  
            }  
        }  
        if (Application.isPlaying)  
        {  
            if (mLines.Count > 20)  
            {  
                mLines.RemoveAt(0);  
            }  
            mLines.Add(text);  
  
        }  
    }  
      
    void OnGUI()  
    {  
        GUI.color = Color.red;  
        for (int i = 0, imax = mLines.Count; i < imax; ++i)  
        {  
            GUILayout.Label(mLines[i]);  
        }  
    }  
}  
```

打包之前在Android的Player Setting里面选择WriteAccess (写入访问)。

Internal Only：表示Application.persistentDataPath的路径是应用程序沙盒，（需要root不然访问不了写入的文件）。

文件路径：data/data/包名/Files/OutLog.txt

External(SDCard)：表示Application.persistentDataPath的路径是SDCard的路径。（不需要root就可以访问文件）。

文件路径：SDCard/Android/包名/Files/OutLog.txt

总结 Application.persistentDataPath 会根据你的WriteAccess选项而产生对应的路径。

2、Log信息的封装

我们的Log信息的输出，需要再次进行封装。

我们新建C#类库工程：需要引用UnityEngine.dll。

```java
using System;  
using System.Collections.Generic;  
using System.Text;  
using UnityEngine;  
public class Logger  
{  
    public static bool EnableLogger = true;  
  
    public static void Log(object message)  
    {  
        if (EnableLogger) Debug.Log(message);  
    }  
  
    public static void Log(object message, UnityEngine.Object context)  
    {  
        if (EnableLogger) Debug.Log(message, context);  
    }  
  
    public static void LogError(object message)  
    {  
        if (EnableLogger) Debug.LogError(message);  
    }  
  
    public static void LogError(object message, UnityEngine.Object context)  
    {  
        if (EnableLogger) Debug.LogError(message, context);  
    }  
  
    public static void LogErrorFormat(string format, params object[] args)  
    {  
        if (EnableLogger) Debug.LogErrorFormat(format, args);  
    }  
  
    public static void LogErrorFormat(UnityEngine.Object context,  
                        string format, params object[] args)  
    {  
        if (EnableLogger) Debug.LogErrorFormat(context, format, args);  
    }  
  
    public static void LogException(Exception exception)  
    {  
        if (EnableLogger) Debug.LogException(exception);  
    }  
  
    public static void LogException(Exception exception,  
                                        UnityEngine.Object context)  
    {  
        if (EnableLogger) Debug.LogException(exception, context);  
    }  
  
    public static void LogFormat(string format, params object[] args)  
    {  
        if (EnableLogger) Debug.LogFormat(format, args);  
    }  
  
    public static void LogFormat(UnityEngine.Object context,  
                                    string format, params object[] args)  
    {  
        if (EnableLogger) Debug.LogFormat(context, format, args);  
    }  
  
    public static void LogWarning(object message)  
    {  
        if (EnableLogger) Debug.LogWarning(message);  
    }  
  
    public static void LogWarning(object message,  
                                    UnityEngine.Object context)  
    {  
        if (EnableLogger) Debug.LogWarning(message, context);  
    }  
  
    public static void LogWarningFormat(string format,  
                                            params object[] args)  
    {  
        if (EnableLogger) Debug.LogWarningFormat(format, args);  
    }  
  
    public static void LogWarningFormat(UnityEngine.Object context,  
                                        string format, params object[] args)  
    {  
        if (EnableLogger) Debug.LogWarningFormat(context, format, args);  
    }  
}  
```

设置工程属性。

我们可以在工程目录里面找到Logger.dll文件，这样我们就完成了对Unity的Log的封装。

我们将Logger.dll文件放入Unity的Plugins文件目录下。

这样就可以在我们的Unity工程中直接使用了。

设置Logger.EnableLogger = false;

就可以关闭Log的输出了。