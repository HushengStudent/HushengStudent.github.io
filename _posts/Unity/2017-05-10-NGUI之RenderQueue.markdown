---
layout: post
title:  "[Unity 工具技巧]NGUI之RenderQueue"
date:   2017-05-10 10:10:10
categories: Unity
comments: true
---

之前在做特效被UI遮挡时，我们是使用了修改UI和特效的RenderQueue的方法来避免这一问题的。

现在我们通过扩展UIPanel来将我们要绘制的物体插入RenderQueue队列中，来达到之前的目的。

在我们自定义的要显示的物体上挂载如下脚本：
```java
using UnityEngine;  
using System.Collections;  
  
public class RenderQueueUtil : MonoBehaviour  
{  
    public UIPanel targetPanel;  
    public UIWidget targetGameObject;  
    private Renderer renderer;  
    private int lasetQueue = int.MinValue;  
    void Awake()  
    {  
        renderer = GetComponent<Renderer>();  
    }  
    void OnEnable()  
    {  
        AddToPanel();  
    }  
    void AddToPanel()  
    {  
        if (targetPanel != null) targetPanel.renderQueueUtils.Add(this);  
    }  
    void OnDisable()  
    {  
        targetPanel.renderQueueUtils.Remove(this);  
    }  
    public void setQueue(int queue)  
    {  
        if (this.lasetQueue != queue)  
        {  
            this.lasetQueue = queue;  
            renderer.material.renderQueue = this.lasetQueue;  
        }  
    }  
}  
```

然后对UIPanel.cs进行下扩展：
```java
public BetterList<RenderQueueUtil> renderQueueUtils   
                  = new BetterList<RenderQueueUtil>();  
  
void UpdateDrawCalls ()  
{  
    ......  
  
    int renderQueueValue = -1;  
  
    for (int i = 0; i < drawCalls.size; ++i)  
    {  
        renderQueueValue += 1;  
  
        ......  
  
           BetterList<RenderQueueUtil> queueList = FindModifier(dc);  
        if (queueList.size > 0)   
           {  
               for (int j = 0; j < this.renderQueueUtils.size; j++)  
               {  
                renderQueueValue += 1;  
                   RenderQueueUtil util = queueList.buffer[j];  
                   util.setQueue(startingRenderQueue + renderQueueValue);  
            }  
        }  
      
    }  
}  
  
   public BetterList<RenderQueueUtil> FindModifier(UIDrawCall dc)   
{  
       BetterList<RenderQueueUtil> queueList = new BetterList<RenderQueueUtil>();  
  
       for (int i = 0; i < this.renderQueueUtils.size; ++i)  
    {  
           RenderQueueUtil util = this.renderQueueUtils.buffer[i];  
           if (util.targetPanel != null && util.targetGameObject.drawCall !=  
                                null && util.targetGameObject.drawCall == dc)  
           {  
               queueList.Add(util);  
        }  
    }  
    return queueList;  
}  
```

指定所挂载脚本所在的UIPanel和对照的目标，就能实现插入到RenderQueue了。
