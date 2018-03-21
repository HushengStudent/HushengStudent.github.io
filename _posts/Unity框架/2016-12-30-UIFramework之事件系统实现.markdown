---
layout: post
title:  "[UIFramework]UIFramework之事件系统实现"
date:   2016-12-30 01:38:10
categories: Unity框架
comments: true
---

在Unity开发中，自定义消息系统经常会被用到，事件系统的实现主要是基于C#委托来完成的。下面介绍的两种实现，一种是利用C#自带的Action委托来实现；第二种是通过自定义一个信息类来作为传递信息的载体，再通过自定义委托来实现的。

实现一：

首先在MessageSystem.cs中实现事件系统的基本功能：
```java
/* 
* Name: MessageSystem.cs 
* Function: N/A  
*  
* Ver     变更日期               负责人                            变更内容 
* ────────────────────────────────────────────────────────────────────── 
* V1.0.0  $time$    http://blog.csdn.net/husheng0      
*  
* Copyright (c). All rights reserved. 
*/  
  
using System;  
using System.Collections.Generic;  
using UnityEngine;  
  
public static class MessageSystem<TKey> where TKey : struct  
{  
    //MessageKey对应的注册的委托  
    private static Dictionary<TKey, List<Delegate>> RegisterMessageDic   
        = new Dictionary<TKey, List<Delegate>>();  
    //移出对应的委托  
    public static void UnRegisterDelegate(TKey MessageKey)  
    {  
        RegisterMessageDic.Remove(MessageKey);  
    }  
    //清空事件系统已注册事件  
    public static void CleanEvents()  
    {  
        RegisterMessageDic.Clear();  
  
    }  
    //能不能注册该委托  
    private static bool IsCanRegister(TKey MessageKey, Delegate registerDelegate)  
    {  
        List<Delegate> registerDelegateList = null;  
        if (RegisterMessageDic.TryGetValue(MessageKey, out registerDelegateList))  
        {  
            if (null == registerDelegateList || registerDelegateList.Count == 0)   
                return true;  
            foreach (Delegate dele in registerDelegateList)  
            {  
                if (dele == registerDelegate) return false;  
            }  
        }  
        else  
        {  
            RegisterMessageDic[MessageKey] = new List<Delegate>();  
        }  
        return true;  
    }  
 
    #region Action委托  
    /// <summary>  
    /// 注册无参Action委托  
    /// </summary>  
    /// <param name="MessageKey"></param>  
    /// <param name="handler"></param>  
    public static void AddListener(TKey MessageKey, Action handler)  
    {  
        if (IsCanRegister(MessageKey, handler))  
        {  
            RegisterMessageDic[MessageKey].Add(handler);  
            Debug.Log("Register" + MessageKey.ToString() + "success");  
        }  
        else  
        {  
            Debug.LogError("Register" + MessageKey.ToString() + "failure");  
        }  
    }  
    /// <summary>  
    /// 移除无参Action委托  
    /// </summary>  
    /// <param name="MessageKey"></param>  
    /// <param name="handler"></param>  
    public static void RemoveListenerer(TKey MessageKey, Action handler)  
    {  
        if (!IsCanRegister(MessageKey, handler))  
        {  
            RegisterMessageDic[MessageKey].Remove(handler);  
            Debug.Log("Remove" + MessageKey.ToString() + "success");  
        }  
        else  
        {  
            Debug.LogError("Remove" + MessageKey.ToString() + "failure");  
        }  
    }  
    /// <summary>  
    /// 广播无参Action委托  
    /// </summary>  
    /// <param name="MessageKey"></param>  
    public static void Broadcast(TKey MessageKey)  
    {  
        List<Delegate> registerDelegateList;  
        if (RegisterMessageDic.TryGetValue(MessageKey, out registerDelegateList))  
        {  
            Action callback;  
            for (int i = 0; i < registerDelegateList.Count; ++i)  
            {  
                try  
                {  
                    callback = (registerDelegateList[i]) as Action;  
                    if (callback != null)  
                    {  
                        callback();  
                    }  
                }  
                catch (Exception exp)  
                {  
                    Debug.LogException(exp);  
                }  
            }  
        }  
    }  
 
    #endregion  
 
 
    #region Action<T>委托  
    /// <summary>  
    /// 注册含一个参数的委托  
    /// </summary>  
    /// <typeparam name="T"></typeparam>  
    /// <param name="MessageKey"></param>  
    /// <param name="handler"></param>  
    public static void AddListener<T>(TKey MessageKey, Action<T> handler)  
    {  
        if (IsCanRegister(MessageKey, handler))  
        {  
            RegisterMessageDic[MessageKey].Add(handler);  
            Debug.Log("Register" + MessageKey.ToString() + "success");  
        }  
        else  
        {  
            Debug.LogError("Register" + MessageKey.ToString() + "failure");  
        }  
    }  
    /// <summary>  
    /// 移除含一个参数的Action委托  
    /// </summary>  
    /// <typeparam name="T"></typeparam>  
    /// <param name="MessageKey"></param>  
    /// <param name="handler"></param>  
    public static void RemoveListenerer<T>(TKey MessageKey, Action<T> handler)  
    {  
        if (!IsCanRegister(MessageKey, handler))  
        {  
            RegisterMessageDic[MessageKey].Remove(handler);  
            Debug.Log("Remove" + MessageKey.ToString() + "success");  
        }  
        else  
        {  
            Debug.LogError("Remove" + MessageKey.ToString() + "failure");  
        }  
    }  
    /// <summary>  
    /// 广播含一个参数的Action委托  
    /// </summary>  
    /// <typeparam name="T"></typeparam>  
    /// <param name="MessageKey"></param>  
    /// <param name="arg1"></param>  
    public static void Broadcast<T>(TKey MessageKey, T arg1)  
    {  
        List<Delegate> registerDelegateList;  
        if (RegisterMessageDic.TryGetValue(MessageKey, out registerDelegateList))  
        {  
            Action<T> callback;  
            for (int i = 0; i < registerDelegateList.Count; ++i)  
            {  
                try  
                {  
                    callback = (registerDelegateList[i]) as Action<T>;  
                    if (callback != null)  
                    {  
                        callback(arg1);  
                    }  
                }  
                catch (Exception exp)  
                {  
                    Debug.LogException(exp);  
                }  
            }  
        }  
    }  
 
    #endregion  
 
 
    #region Action<T1,T2>委托  
    /// <summary>  
    /// 注册含两个参数的Action委托  
    /// </summary>  
    /// <typeparam name="T1"></typeparam>  
    /// <typeparam name="T2"></typeparam>  
    /// <param name="MessageKey"></param>  
    /// <param name="handler"></param>  
    public static void AddListener<T1, T2>(TKey MessageKey, Action<T1, T2> handler)  
    {  
        if (IsCanRegister(MessageKey, handler))  
        {  
            RegisterMessageDic[MessageKey].Add(handler);  
            Debug.Log("Register" + MessageKey.ToString() + "success");  
        }  
        else  
        {  
            Debug.LogError("Register" + MessageKey.ToString() + "failure");  
        }  
    }  
    /// <summary>  
    /// 移除含两个参数的Action委托  
    /// </summary>  
    /// <typeparam name="T1"></typeparam>  
    /// <typeparam name="T2"></typeparam>  
    /// <param name="MessageKey"></param>  
    /// <param name="handler"></param>  
    public static void RemoveListenerer<T1, T2>(TKey MessageKey, Action<T1, T2> handler)  
    {  
        if (!IsCanRegister(MessageKey, handler))  
        {  
            RegisterMessageDic[MessageKey].Remove(handler);  
            Debug.Log("Remove" + MessageKey.ToString() + "success");  
        }  
        else  
        {  
            Debug.LogError("Remove" + MessageKey.ToString() + "failure");  
        }  
    }  
    /// <summary>  
    /// 广播含两个参数的Action委托  
    /// </summary>  
    /// <typeparam name="T1"></typeparam>  
    /// <typeparam name="T2"></typeparam>  
    /// <param name="MessageKey"></param>  
    /// <param name="arg1"></param>  
    /// <param name="arg2"></param>  
    public static void Broadcast<T1, T2>(TKey MessageKey, T1 arg1, T2 arg2)  
    {  
        List<Delegate> registerDelegateList;  
        if (RegisterMessageDic.TryGetValue(MessageKey, out registerDelegateList))  
        {  
            Action<T1, T2> callback;  
            for (int i = 0; i < registerDelegateList.Count; ++i)  
            {  
                try  
                {  
                    callback = (registerDelegateList[i]) as Action<T1, T2>;  
                    if (callback != null)  
                    {  
                        callback(arg1, arg2);  
                    }  
                }  
                catch (Exception exp)  
                {  
                    Debug.LogException(exp);  
                }  
            }  
        }  
    }  
 
    #endregion  
}  
```

下面使用两个脚本进行简单的测试：
```java
/// <summary>  
/// 全局事件  
/// </summary>  
public enum MsgTest  
{  
    Test1,  
    Test2,  
    Test3  
}  
```

```java
using UnityEngine;  
using System.Collections;  
  
public class MessageSystemTest : MonoBehaviour  
{  
    void OnGUI()  
    {  
        if (GUI.Button(new Rect(50, 50, 200, 30), "Register Action"))  
            MessageSystem<MsgTest>.AddListener(MsgTest.Test1, Test1Func);  
        if (GUI.Button(new Rect(50, 100, 200, 30), "Broadcast Action"))  
            MessageSystem<MsgTest>.Broadcast(MsgTest.Test1);  
        if (GUI.Button(new Rect(50, 150, 200, 30), "Remove Action"))  
            MessageSystem<MsgTest>.RemoveListenerer(MsgTest.Test1, Test1Func);  
        if (GUI.Button(new Rect(50, 200, 200, 30), "Register Action<T>"))  
            MessageSystem<MsgTest>.AddListener<int>(MsgTest.Test2, Test2Func);  
        if (GUI.Button(new Rect(50, 250, 200, 30), "Broadcast Action<T>"))  
            MessageSystem<MsgTest>.Broadcast<int>(MsgTest.Test2,100);  
        if (GUI.Button(new Rect(50, 300, 200, 30), "Remove Action<T>"))  
            MessageSystem<MsgTest>.RemoveListenerer<int>(MsgTest.Test2, Test2Func);  
        if (GUI.Button(new Rect(50, 350, 200, 30), "Register Action<T1,T2>"))  
            MessageSystem<MsgTest>.AddListener<int,string>(MsgTest.Test3, Test3Func);  
        if (GUI.Button(new Rect(50, 400, 200, 30), "Broadcast Action<T1,T2>"))  
            MessageSystem<MsgTest>.Broadcast<int,string>(MsgTest.Test3,100,"test");  
        if (GUI.Button(new Rect(50, 450, 200, 30), "Remove Action<T1,T2>"))  
            MessageSystem<MsgTest>.RemoveListenerer<int,string>(MsgTest.Test3, Test3Func);  
    }  
    private void Test1Func()  
    {  
        Debug.LogError("--->Test1Func");  
    }  
  
    private void Test2Func(int i)  
    {  
        Debug.LogError("--->Test1Func"+i.ToString());  
    }  
  
    private void Test3Func(int i,string str)  
    {  
        Debug.LogError("--->Test1Func" + i.ToString()+"  "+str);  
    }  
}  
```

实现二：

第二种是实现方式与第一种实现方式其实是大同小异：
```java
using System;  
using System.Collections.Generic;  
using UnityEngine;  
  
public static class MessageCenter  
{  
  
    public delegate void MessageEvent(Message message);  
  
    private static Dictionary<string, List<MessageEvent>> MessageEventsDic =  
        new Dictionary<string, List<MessageEvent>>();  
 
    #region Add & Remove Listener  
  
    public static void AddListener(string messageName, MessageEvent messageEvent)  
    {  
        Debug.Log("AddListener Name : " + messageName);  
        List<MessageEvent> list = null;  
        if (MessageEventsDic.ContainsKey(messageName))  
        {  
            list = MessageEventsDic[messageName];  
        }  
        else  
        {  
            list = new List<MessageEvent>();  
            MessageEventsDic.Add(messageName, list);  
        }  
        if (!list.Contains(messageEvent))  
        {  
            list.Add(messageEvent);  
        }  
    }  
  
    public static void RemoveListener(string messageName, MessageEvent messageEvent)  
    {  
        Debug.Log("RemoveListener Name : " + messageName);  
        if (MessageEventsDic.ContainsKey(messageName))  
        {  
            List<MessageEvent> list = MessageEventsDic[messageName];  
            if (list.Contains(messageEvent))  
            {  
                list.Remove(messageEvent);  
            }  
            if (list.Count <= 0)  
            {  
                MessageEventsDic.Remove(messageName);  
            }  
        }  
    }  
  
    public static void RemoveAllListener()  
    {  
        MessageEventsDic.Clear();  
    }  
 
    #endregion  
 
    #region Send Message  
  
    public static void SendMessage(Message message)  
    {  
        DoMessageDispatcher(message);  
    }  
  
    public static void SendMessage(string name, object sender)  
    {  
        SendMessage(new Message(name, sender));  
    }  
  
    public static void SendMessage(string name, object sender, object content)  
    {  
        SendMessage(new Message(name, sender, content));  
    }  
  
    public static void SendMessage(string name, object sender, object content, params object[] dicParams)  
    {  
        SendMessage(new Message(name, sender, content, dicParams));  
    }  
  
    private static void DoMessageDispatcher(Message message)  
    {  
        Debug.Log("DoMessageDispatcher Name : " + message.Name);  
        if (MessageEventsDic == null || !MessageEventsDic.ContainsKey(message.Name))  
            return;  
        List<MessageEvent> list = MessageEventsDic[message.Name];  
        for (int i = 0; i < list.Count; i++)  
        {  
            MessageEvent messageEvent = list[i];  
            if (null != messageEvent)  
            {  
                messageEvent(message);  
            }  
        }  
    }  
    #endregion  
}  
```

```java
using System;  
using System.Collections;  
using System.Collections.Generic;  
  
public class Message : IEnumerable<KeyValuePair<string, object>>  
{  
    private Dictionary<string, object> DatasDic = null;  
  
    public string Name { get; private set; }  
    public object Sender { get; private set; }  
    public object Content { get; set; }  
    /// <summary>  
    /// 索引器   
    /// </summary>  
    /// <param name="key"></param>  
    /// <returns></returns>  
    public object this[string key]  
    {  
        get  
        {  
            if (null == DatasDic || !DatasDic.ContainsKey(key))  
                return null;  
            return DatasDic[key];  
        }  
        set  
        {  
            if (null == DatasDic)  
                DatasDic = new Dictionary<string, object>();  
            if (DatasDic.ContainsKey(key))  
                DatasDic[key] = value;  
            else  
                DatasDic.Add(key, value);  
        }  
    }  
 
    #region IEnumerable implementation  
  
    public IEnumerator<KeyValuePair<string, object>> GetEnumerator()  
    {  
        if (null == DatasDic)  
            yield break;  
        foreach (KeyValuePair<string, object> kvp in DatasDic)  
        {  
            yield return kvp;  
        }  
    }  
  
    IEnumerator IEnumerable.GetEnumerator()  
    {  
        return DatasDic.GetEnumerator();  
    }  
 
    #endregion  
 
    #region Message Construction Function  
  
    public Message(string name, object sender)  
    {  
        Name = name;  
        Sender = sender;  
        Content = null;  
    }  
  
    public Message(string name, object sender, object content)  
    {  
        Name = name;  
        Sender = sender;  
        Content = content;  
    }  
  
    public Message(string name, object sender, object content, params object[] _dicParams)  
    {  
        Name = name;  
        Sender = sender;  
        Content = content;  
        if (_dicParams.GetType() == typeof(Dictionary<string, object>))  
        {  
            foreach (object _dicParam in _dicParams)  
            {  
                foreach (KeyValuePair<string, object> kvp in _dicParam as Dictionary<string, object>)  
                {  
                    //DatasDic[kvp.Key] = kvp.Value;  //error  
                    this[kvp.Key] = kvp.Value;  
                }  
            }  
        }  
    }  
  
    public Message(Message message)  
    {  
        Name = message.Name;  
        Sender = message.Sender;  
        Content = message.Content;  
        foreach (KeyValuePair<string, object> kvp in message.DatasDic)  
        {  
            this[kvp.Key] = kvp.Value;  
        }  
    }  
 
    #endregion  
 
    #region Add & Remove  
  
    public void Add(string key, object value)  
    {  
        this[key] = value;  
    }  
  
    public void Remove(string key)  
    {  
        if (null != DatasDic && DatasDic.ContainsKey(key))  
        {  
            DatasDic.Remove(key);  
        }  
    }  
    #endregion  
 
    #region Send  
  
    public void Send()  
    {  
        MessageCenter.SendMessage(this);  
    }  
 
    #endregion  
  
}  
```

该事件系统的关键字就是通过创建的Message里的Name来判断的，所以要注意Name防止重复。

测试文件如下：
```java
using UnityEngine;  
using System.Collections;  
  
public class MessageCenterTest : MonoBehaviour {  
  
    private readonly string test = "test";  
    void Awake()  
    {  
        MessageCenter.AddListener(test,OnMessageCome);  
    }  
  
    private void OnMessageCome(Message message)  
    {  
        Debug.LogError("--->" + message["gold"]);  
    }  
  
    void OnGUI()  
    {  
        if (GUI.Button(new Rect(50, 50, 150, 30), "OnMessageCome"))  
        {  
            Message message = new Message(test,this);  
            message["gold"] = 100;  
            message.Send();  
        }  
    }  
}  
```