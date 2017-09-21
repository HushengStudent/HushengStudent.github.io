---
layout: post
title:  "[Android]Android开发之activity"
date:   2015-04-29 12:10:11
categories: 17---【Android】#########
comments: true
---

### 什么是activity
Activity 是用户接口程序，原则上它会提供给用户一个交互式的接口功能。它是 android 应用程序的基本功能单元。Activity 本身是没有界面的。所以activity类创建了一个窗口，开发人员可以通过setContentView(View)接口把UI放到activity创建的窗口上，当activity指向全屏窗口时，也可以用其他方式实现：作为漂浮窗口（通过windowIsFloating的主题集合），或者嵌入到其他的activity（使用ActivityGroup）。activity是单独的，用于处理用户操作。几乎所有的activity都要和用户打交道。

![图片1](http://owk5gjdrg.bkt.clouddn.com/0001Android%E5%BC%80%E5%8F%91%E4%B9%8Bactivity.png)

在一个Activity正常启动过程中，这些方法调用的顺序是onCreate -> onStart -> onResume；在Activity被kill掉的时候方法顺序是onPause -> onStop -> onDestroy，此为一个完整的Lifecycle。那么对于中断处理（比如电话来了），则是onPause -> onStop，恢复时onStart -> onResume；如果当前应用程序的是一个Theme为Translucent（半透明） 或者Dialog 的Activity那么中断就是onPause ,恢复的时候onResume。

onCreate：在这里创建界面，做一些数据的初始化工作；

onStart： 到这一步变成“用户可见不可交互”的状态；

onResume：变成和用户可交互的，(在Activity栈系统通过栈的方式管理这些Activity，即当前Activity在栈的最上端，运行完弹出栈，则回到上一个Activity)；

onPause：到这一步是可见但不可交互的，系统会停止动画等消耗CPU的事情。从上文的描述已经知道，应该在这里保存你的一些数据,因为这个时候你的程序的优先级降　　　　　　　　  低，有可能被系统收回。在这里保存的数据，应该在onResume里读出来。

onStop：变得不可见 ，被下一个activity覆盖了

onDestroy：这是Activity被kill前最后一个被调用方法了，可能是其他类调用finish方法或者是系统为了节省空间将它暂时性的干掉，可以用isFinishing()来判断它，如果你有 一个Progress Dialog在线程中运行，请在onDestroy里把他cancel掉，不然等线程结束的时候，调用Dialog的cancel方法会抛异常。

onPause，onstop， onDestroy，三种状态下 activity都有可能被系统kill 掉。

### Activity之间的通信

![图片2](http://owk5gjdrg.bkt.clouddn.com/0002Android%E5%BC%80%E5%8F%91%E4%B9%8Bactivity.png)

在 Android 中，不同的 Activity 实例可能运行在一个进程中，也可能运行在不同的进程中。因此我们需要一种特别的机制帮助我们在 Activity 之间传递消息。Android 中通过 Intent 对象来表示一条消息，一个 Intent 对象不仅包含有这个消息的目的地，还可以包含消息的内容，这好比一封 Email，其中不仅应该包含收件地址，还可以包含具体的内容。对于一个 Intent 对象，消息“目的地”是必须的，而内容则是可选项。
Intent负责对操作的动作、动作涉及数据、附加数据进行描述，Android则根据此Intent的描述，负责找到对应的组件，将 Intent传递给调用的组件，并完成组件的调用。因此，Intent在这里起着一个媒体中介的作用，专门提供组件互相调用的相关信息，实现调用者与被调用者之间的解耦。
在应用中，我们可以以两种形式来使用Intent：

直接Intent：指定了component属性的Intent（调用setComponent(ComponentName)或者setClass(Context, Class)来指定）。通过指定具体的组件类，通知应用启动对应的组件。

间接Intent：没有指定comonent属性的Intent。这些Intent需要包含足够的信息，这样系统才能根据这些信息，在在所有的可用组件中，确定满足此Intent的组件。

对于直接Intent，Android不需要去做解析，因为目标组件已经很明确。

Android需要解析的是那些间接Intent，通过解析，将 Intent映射给可以处理此Intent的Activity、IntentReceiver或Service。Intent解析机制主要是通过查找已注册在AndroidManifest.xml中的所有IntentFilter及其中定义的Intent，最终找到匹配的Intent。

![图片3](http://owk5gjdrg.bkt.clouddn.com/0003Android%E5%BC%80%E5%8F%91%E4%B9%8Bactivity.png)