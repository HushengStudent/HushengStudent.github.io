---
layout: post
title:  "[UIFramework]UIFramework之单例滥用解决策略"
date:   2017-04-29 14:52:10
categories: Unity_框架
comments: true
---

由于单例的静态特性使得其生命周期跟应用的生命周期一样长，GC无法释放单例内存，所以在代码中，为了避免过度滥用单例，可使用以下策略。
```java
/* 
* Name: DataFactoryManager.cs 
* Function: DataFactory Manager Class.  
*  
* Ver     Date                Name                            Change Content 
* ──────────────────────────────────────────────────── 
* V1.0.0  20170427    http://blog.csdn.net/husheng0      Creat DataFactoryManager.cs 
*  
* Copyright (c). All rights reserved. 
*  
*/  
using System;  
using System.Collections;  
using System.Collections.Generic;  
/// <summary>  
/// 单一对象静态管理类  
/// </summary>  
public static class DataFactoryManager  
{  
    private static Dictionary<Type, object> datasDic = new Dictionary<Type, object>();  
  
    public static T GetFromDataFactory<T>() where T : new()  
    {  
        Type type = typeof(T);  
        object obj = null;  
        if (!datasDic.TryGetValue(type, out obj))  
        {  
            obj = new T();  
            datasDic.Add(type, obj);  
        }  
        return (T)obj;  
    }  
  
    public static void ClearDataFactory()  
    {  
        datasDic.Clear();  
    }  
  
    public static void RemoveFromDataFactory<T>() where T : new()  
    {  
        Type type = typeof(T);  
        if (datasDic.ContainsKey(type))  
        {  
            datasDic.Remove(type);  
        }  
    }  
}  
```

其中T必须有无参构造函数。