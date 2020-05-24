---
layout: post
title:  "[UIFramework]UIFramework之Unity4.x 场景管理策略"
date:   2017-05-20 15:06:10
categories: 框架
comments: true
---

基本策略：一个初始场景+一个空场景。所有资源动态从Resource目录下加载或者从AssetBundle加载。

每个场景加载和销毁都有相似的处理过程，所以每个场景的主模块可以提取父类。

```java
using UnityEngine;  
using System.Collections;  
  
public abstract class AbsSceneBaseUI : MonoBehaviour  
{  
    //when come in this Scene，this fuction must to execute;  
    public virtual void InitSceneSnyc(object[] args) { }  
  
    //when come in this Scene，this fuction must to execute;  
    public virtual IEnumerator InitSceneAsnyc(object[] args)  
    {  
        yield return null;  
    }  
  
    //when come out this Scene，this fuction must to execute;  
    public virtual void FinallizSceneSync() { }  
  
    //when come out this Scene，this fuction must to execute;  
    public virtual IEnumerator FinallizSceneAsync()  
    {  
        yield return null;  
    }  
  
    //Save system infomation  
    public virtual void SaveSystemInfo() { }  
}  
```

场景切换的过程：

①判断目标场景是否与当前场景是否一样，一样则return。

②开启Loading界面。

③执行加载前的回调函数。

④保存上一个场景中的信息。

⑤清理上一个场景。

⑥加载空场景x2。

⑦加载目标场景主模块。

⑧初始化主模块。

⑨执行加载完成后的回调。

⑩关闭Loading界面。

`tips：Unity4.x里面加载一个新的场景会执行一些清理工作，但是会隔一个场景才会卸载内存。即加载下下个场景才会清理本场景的相关内存，所以在加载空场景的时候加载两次。`
`场景加载后也可执行Resources.UnloadUnusedAssets();    Unloads assets that are not used。`
`关于GC，GC会造成卡顿，所以一般可以使用定时器，每隔一段时间GC一次。`
