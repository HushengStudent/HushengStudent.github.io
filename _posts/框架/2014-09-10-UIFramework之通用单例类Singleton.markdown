---
layout: post
title:  "[UIFramework]UIFramework之通用单例类Singleton"
date:   2014-09-10 10:10:10
categories: 框架
comments: true
---

在Unity的开发中，单例模式是最经常使用的设计模式之一，在每个模块开发中，如果都单独去实现单例模式，会很繁琐，所以实现一个Singleton类，每个需要实现单例模式的模块只需要集成自该类，即可。

```java
/* 
* Name: SingletonManager.cs 
* Function: N/A  
*  
* Ver     变更日期               负责人                            变更内容 
* ────────────────────────────────────────────────────────────────────── 
* V1.0.0  $time$    http://blog.csdn.net/husheng0      
*  
* Copyright (c). All rights reserved. 
* 
* 单例类 
*  
*/  
  
using UnityEngine;  
using System.Collections;  
/// <summary>  
/// 单例管理类的实现，不继承Monobehaviour  
/// </summary>  
/// <typeparam name="T"></typeparam>  
public class SingletonManager<T> where T : class,new()  
{  
  
    protected static T instance = null;  
  
    public static T Instance  
    {  
        get  
        {  
            if (null == instance)  
            {  
                instance = new T(); //调用构造函数    
            }  
            return instance;  
        }  
    }  
    /// <summary>  
    /// SingletonManager构造函数  
    /// </summary>  
    protected SingletonManager()  
    {  
        if (null != instance)  
            Debug.Log("This " + (typeof(T)).ToString()   
+ " Singleton Instance is not null！");  
        Init();  
    }  
    public virtual void Init() { }  
}  
```

```java
/* 
* Name: MonoSingletonManager.cs 
* Function: N/A  
*  
* Ver     变更日期               负责人                            变更内容 
* ────────────────────────────────────────────────────────────────────── 
* V1.0.0  $time$    http://blog.csdn.net/husheng0      
*  
* Copyright (c). All rights reserved. 
* 
* 单例类  
*  
*/  
  
using UnityEngine;  
using System.Collections;  
/// <summary>  
/// 单例管理类的实现，继承Monobehaviour  
/// 该类会在Hierarchy下创建"___MonoSingleton"并把所有的集成MonoSingletonManager的脚本添加到"___MonoSingleton"上  
/// </summary>  
/// <typeparam name="T"></typeparam>  
public class MonoSingletonManager<T> : MonoBehaviour where T : MonoSingletonManager<T>  
{  
  
    private static T instance = null;  
  
    public static T Instance  
    {  
        get  
        {  
            if (null == instance)  
            {  
                GameObject go = GameObject.Find("___MonoSingleton");  
                if (null == go)  
                {  
                    go = new GameObject("___MonoSingleton");  
                    DontDestroyOnLoad(go);  
                }  
                instance = go.AddComponent<T>();  
  
            }  
            return instance;  
        }  
    }  
  
    /// <summary>  
    /// Raises the application quit event.  
    /// </summary>  
    private void OnApplicationQuit()  
    {  
        instance = null;  
    }  
}  
```