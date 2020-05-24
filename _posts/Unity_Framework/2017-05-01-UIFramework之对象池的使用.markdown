---
layout: post
title:  "[UIFramework]UIFramework之对象池的使用"
date:   2017-05-01 01:54:10
categories: Unity_Framework
comments: true
---

在游戏开发中，像子弹这类物体会被频繁的创建，然后被销毁，这样的操作对游戏性能也会造成严重的浪费。所以对象池的使用，可以减少需要频繁创建销毁的物体的操作，达到对性能的优化。
```java
/* 
* Name: ObjectPoolManager.cs 
* Function: Object Pool Manager Class.  
*  
* Ver     Date                Name                            Change Content 
* ──────────────────────────────────────────────────────────────────────────── 
* V1.0.0  20170430    http://blog.csdn.net/husheng0      Creat ObjectPoolManager.cs 
*  
* Copyright (c). All rights reserved. 
*  
*/  
using UnityEngine;  
using System.Collections;  
using System.Collections.Generic;  
using System;  
/// <summary>  
/// Object Pool Manager  
/// </summary>  
public class ObjectPoolManager : MonoSingletonManager<ObjectPoolManager>  
{  
    private Dictionary<Type, Stack<MonoBehaviour>> ObjectDic   
        = new Dictionary<Type, Stack<MonoBehaviour>>();  
    /// <summary>  
    /// Get or Great Object From Pool  
    /// </summary>  
    /// <typeparam name="T">Type</typeparam>  
    /// <param name="objectName">object Name</param>  
    /// <returns>Type</returns>  
    public T GetObjectFromPool<T>(string objectName)   
        where T :MonoBehaviour,IPoolObject,new()  
    {  
        Type type = typeof(T);  
        object obj = null;  
        T ctrl = null;  
        if (ObjectDic.ContainsKey(type))  
        {  
            if (ObjectDic[type].Count == 0)  
            {  
                obj = GameObject.Instantiate(Resources.Load(objectName));  
                ctrl = ((GameObject)obj).GetComponent<T>();  
            }  
            if (ObjectDic[type].Count > 0)  
            {  
                obj = ObjectDic[type].Pop();  
                ctrl = (T)obj;  
            }  
        }  
        else  
        {  
            obj = GameObject.Instantiate(Resources.Load(objectName));  
            ctrl = ((GameObject)obj).GetComponent<T>();  
        }  
        ctrl.gameObject.SetActive(true);  
        if(ctrl!=null)ctrl.Spawn();  
        return ctrl;  
    }  
    /// <summary>  
    /// Disappear Object To Pool  
    /// </summary>  
    /// <typeparam name="T">Type</typeparam>  
    /// <param name="ctrl">object</param>  
    /// <param name="objCount">count</param>  
    public void DisappearObjectToPool<T>(T ctrl,int objCount)    
        where T : MonoBehaviour,IPoolObject, new()  
    {  
        Type type = typeof(T);  
        if (!ObjectDic.ContainsKey(type))  
        {  
            ObjectDic.Add(type, new Stack<MonoBehaviour>());  
        }  
        ctrl.UnSpawn();  
        ctrl.gameObject.SetActive(false);  
        if (ObjectDic[type].Count < objCount)  
        {  
            ObjectDic[type].Push((MonoBehaviour)ctrl);  
        }  
        else  
        {  
            GameObject.Destroy(ctrl.gameObject);  
        }  
    }  
    /// <summary>  
    /// clear pool  
    /// </summary>  
    public void ClearPool()  
    {  
        foreach (KeyValuePair<Type,Stack<MonoBehaviour>> tempStack in ObjectDic)  
        {  
            while(tempStack.Value.Count > 0)  
            {  
                GameObject.Destroy(tempStack.Value.Pop().gameObject);  
            }  
        }  
    }  
}  
  
public interface IPoolObject  
{  
    /// <summary>  
    /// init  
    /// </summary>  
    void Spawn();  
    /// <summary>  
    /// Finallize  
    /// </summary>  
    void UnSpawn();  
}  
```

```java
using UnityEngine;  
using System.Collections;  
  
public class Bullet : MonoBehaviour, IPoolObject  
{  
    public Bullet() {  
    }  
    public void Spawn()  
    {  
        Debug.Log("----->Spawn");  
    }  
  
    public void UnSpawn()  
    {  
        Debug.Log("----->UnSpawn");  
    }  
}  
```

```java
using UnityEngine;  
using System.Collections;  
using System.Collections.Generic;  
  
public class Test : MonoBehaviour  
{  
    private const string PerfabName = "Bullet";  
    //限定对象池缓存数目  
    private int count = 3;  
  
    private List<Bullet> bulletList = new List<Bullet>();  
    void OnGUI()  
    {  
        if (GUILayout.Button("Great Bullet"))  
        {  
            for (int i = 0; i < 5; i++)  
            {  
                Bullet bullet = ObjectPoolManager.Instance.GetObjectFromPool<Bullet>(PerfabName);  
                bulletList.Add(bullet);  
            }  
        }  
        if (GUILayout.Button("Disapper Bullet"))  
        {  
            for (int i = 0; i < bulletList.Count; i++)  
            {  
                ObjectPoolManager.Instance.DisappearObjectToPool<Bullet>(bulletList[i], count);  
            }  
            bulletList.Clear();  
        }  
        if (GUILayout.Button("Distroy All"))  
        {  
            ObjectPoolManager.Instance.ClearPool();  
        }  
    }  
}  
```

点击不同的Button，会发现，Bullet.cs的Spawn();UnSpawn();顺利执行。