---
layout: post
title:  "[UIFramework]UIFramework之Unity4.x 资源加载策略"
date:   2017-05-07 21:01:10
categories: 基础框架
comments: true
---

Unity4.x资源加载主要有从Resources文件夹下加载资源和从AssetBundle文件中加载资源这两种方式。

1、Resource资源加载

加载过程：`源文件->Asset->Asset(Clone)`

源文件到 Asset 的过程，主要的实现方式有：

①Resources.Load()同步加载Resource目录下的资源；

```java
T ctrl = Resources.Load<T>(Path);  
        if (ctrl != null)  
        {  
            return Object.Instantiate(ctrl) as T;  
        }  
```

②Resources.LoadAsync()异步加载Resource目录下的资源；

```java
IEnumerator itor =LoadResourceAsyncIEnumerator<T>(Path,  
            //Action回调参数,匿名函数  
            go=>   
            {  
                if (null != go)  
                {  
                    ctrl = Object.Instantiate(go) as T;  
                }  
            },  
            progress);  
        while (itor.MoveNext())  
        {  
            yield return null;  
        }  
```

```java
//Load Resource Async base function.  
    private IEnumerator LoadResourceAsyncIEnumerator<T>(string Path, Action<T> action, Action<float> progress)where T:Object  
    {  
        ResourceRequest req = Resources.LoadAsync<T>(Path);  
        while (req.progress < 0.99)  
        {  
            if (null != progress)  
                progress(req.progress);  
            yield return null;  
        }  
        while (!req.isDone)  
        {  
            if (null != progress)  
                progress(req.progress);  
            yield return null;  
        }  
        T ctrl = req.asset as T;  
        if (null != action)  
        {  
            action(ctrl);  
        }  
    }  
```

Asset 到 Asset(Clone)的过程主要是有 Instantiate()来实现的。

Asset(Clone)的卸载主要是通过 Destroy()来实现的。

Asset 的卸载一般是Reources.UnloadAsset(Object)和Resources.UnloadUnusedAssets()来卸载，Resources.UnloadUnusedAssets()一般这个函数我们不会手动调用，Unity 在切换场景时，会调用该函数，完成一些清理工作。

2、AssetBundle资源加载：`AssetBundle 源文件->AssetBundle 镜像->Asset->Asset(Clone)`

AssetBundle 源文件到 AssetBundle 镜像：[由于AssetBundle打包存在依赖关系，故AssetBundle的加载需要根据依赖关系，加载被依赖的AssetBundle]

①AssetBundle.CreateFromFile：加载AssetBundle最快的方式。CreateFromFile只能适用于未压缩的AssetBundle，而Android系统下StreamingAssets是在压缩目录(.jar)中，因此需要先将未压缩的AssetBundle放到SD卡中才能对其使用CreateFromFile。iOS系统有256个开启文件的上限，通过CreateFromFile加载的AssetBundle对象也会低于该值。

②WWW.assetBundle：官方推荐使用 WWW + 协程的方式来异步加载 AssetBundle。

当 yield return www的时候，会将 www 对象抛给 unity3d 内核进行处理，虽然文档里没有太多内容，但是从接口提供的 WWW.isDone 和 WWW.progress 可以猜测 www 本质上是一个异步操作对象（AsyncOperation）。

即使是本地文件，即以 file:/// 协议开头的资源，也是可以使用上述的异步方式进行加载。

WWW.assetBundle 在内存中会有较大的WebStream，而AssetBundle.CreateFromFile在内存中只有通常较小的SerializedFile。`（对于序列化信息较多的Prefab，很可能出现SerializedFile比WebStream更大的情况）。`
对于AssetBundle 镜像还未卸载的，如果重复加载，就会报错。

AssetBundle 镜像到 Asset：

①AssetBundle.Load()：`从AssetBundle读取一个指定名称的Asset并生成Asset内存对象，如果多次Load同名对象，除第一次外都只会返回已经生成的Asset 对象，也就是说多次Load一个Asset并不会生成多个副本（singleton）。`

②AssetBundle.LoadAsync()：异步读取；


③AssetBundle.LoadAll()：全部读取；

AssetBundle.Unload(false)：释放AssetBundle文件内存镜像(AssetBundle)。

AssetBundle.Unload(true)：释放AssetBundle文件内存镜像(AssetBundle)同时销毁所有已经Load的Assets内存对象(Asset)。

Reources.UnloadAsset(Object)：显式的释放已加载的Asset对象，只能卸载磁盘文件加载的Asset对象，CPU开销小。

Resources.UnloadUnusedAssets()：用于释放所有没有引用的Asset对象，CPU开销大。

GC.Collect()：强制垃圾收集器立即释放内存 Unity的GC功能不算好，没把握的时候就强制调用一下。

WWW对象：调用对象的Dispose函数或将其置为null即可；

WebStream：在卸载WWW对象以及对应的AssetBundle对象后，这部分内存即会被引擎自动卸载；

SerializedFile：卸载AssetBundle后，这部分内存会被引擎自动卸载；

`在通过AssetBundle.Unload(false)卸载AssetBundle对象后，如果重新创建该对象并加载之前加载过的资源到内存时，会出现冗余，即两份相同的资源。`

从AssetBundle读取一个指定名称的Asset并生成Asset内存对象，如果多次Load同名对象，`除第一次外都只会返回已经生成的Asset 对象，也就是说多次Load一个Asset并不会生成多个副本（singleton）。多次Resource下Load，也只会返回第一次Load的Asset对象`，也就是说，什么时候卸载AssetBundle镜像很关键，如果每次Instantiate()后，立即卸载AssetBundle镜像，当你再次读取AssetBundle后，就会Load出多余的Asset，从而出现资源冗余。

被脚本的静态变量引用的资源，在调用Resources.UnloadUnusedAssets时，并不会被卸载，在Profiler中能够看到其引用情况。

`对于Prefab，目前仅能通过DestroyImmediate来卸载Asset，且卸载后，必须重新加载AssetBundle才能重新加载该Prefab。由于内存开销较小，通常不建议进行针对性地卸载。`

尽量避免在游戏进行中调用Resources.UnloadUnusedAssets()，因为该接口开销较大，容易引起卡顿，可尝试使用Resources.Unload(obj)来逐个进行卸载，以保证游戏的流畅度。

`比如需要从AssetBundle中加载A，然而A依赖于B，正确的顺序应该是先把A，B的AssetBundle加载进来，然后将A先Load为Asset，再实例化，在A实例化之前，不能Unload(false)。`

Unity内存分析：

Unity游戏在运行时的内存占用

①库代码占用：Unity库，第三方库（减少打包时的引用库）；

②本机堆（Native Heap）：资源、Unity逻辑和第三方逻辑，Unity使用了自己的一套内存管理机制来使这块内存具有和托管堆类似的功能。尽量减少在Hierarchy对资源的直接引用，而是使用Resource.Load的方法，在需要的时候从硬盘中读取资源，在使用后用

Resource.UnloadAsset()和Resources.UnloadUnusedAssets()尽快将其卸载掉。另外，static的单例（singleton）在场景切换时也不会被摧毁，同样地，如果这种单例含有大量的对资源的引用，也会成为大问题。

③托管堆（Managed Heap）：C#代码（及时回收内存；使用对象池等，避免内存还未回收又申请新的内存）。

内存管理主要是针对托管堆（Managed Heap）内存：

目前绝大部分Unity游戏逻辑代码所使用的语言为C#，C#代码所占用的内存又称为mono内存，这是因为Unity是通过mono来跨平台解析并运行C#代码的，在Android系统上，游戏的lib目录下存在的libmono.so文件，就是mono在Android系统上的实现。C#代码通过mono解析执行，所需要的内存自然也是由mono来进行分配管理，下面就介绍一下mono的内存管理策略以及内存泄漏分析。

Mono通过垃圾回收机制（Garbage Collect，简称GC）对内存进行管理。Mono内存分为两部分，已用内存（used）和堆内存（heap），已用内存指的是mono实际需要使用的内存，堆内存指的是mono向操作系统申请的内存，两者的差值就是mono的空闲内存。当mono需要分配内存时，会先查看空闲内存是否足够，如果足够的话，直接在空闲内存中分配，否则mono会进行一次GC以释放更多的空闲内存，如果GC之后仍然没有足够的空闲内存，则mono会向操作系统申请内存，并扩充堆内存

GC的主要作用在于从已用内存中找出那些不再需要使用的内存，并进行释放。Mono中的GC主要有以下几个步骤：

1、停止所有需要mono内存分配的线程。

2、遍历所有已用内存，找到那些不再需要使用的内存，并进行标记。

3、释放被标记的内存到空闲内存。

4、重新开始被停止的线程。

`除了空闲内存不足时mono会自动调用GC外，也可以在代码中调用GC.Collect()手动进行GC，但是GC本身是比较耗时的操作，而且由于GC会暂停那些需要mono内存分配的线程（C#代码创建的线程和主线程），因此无论是否在主线程中调用，GC都会导致游戏一定程度的卡顿，需要谨慎处理。另外，GC释放的内存只会留给mono使用，并不会交还给操作系统，因此mono堆内存是只增不减的。`

`一个Prefab从assetBundle里Load出来 里面可能包括：Gameobject transform mesh texture material shader script和各种其他Assets。`

`你Instaniate一个Prefab，是一个对Assets进行Clone(复制)+引用结合的过程，GameObject transform 是Clone是新生成的。其他mesh / texture / material / shader 等，这其中些是纯引用的关系的，包括：Texture和TerrainData，还有引用和复制同时存在的，包括：Mesh/material /PhysicMaterial。引用的Asset对象不会被复制，所以加载时不需要Instantiate()，只是一个简单的指针指向已经Load的Asset对象。这种含糊的引用加克隆的混合， 大概是搞糊涂大多数人的主要原因。`

专门要提一下的是一个特殊的东西：Script Asset，看起来很奇怪，Unity里每个Script都是一个封闭的Class定义而已，并没有写调用代码，光Class的定义脚本是不会工作的。其实Unity引擎就是那个调用代码，Clone一个script asset等于new一个class实例，实例才会完成工作。把他挂到Unity主线程的调用链里去，Class实例里的OnUpdate OnStart等才会被执行。多个物体挂同一个脚本，其实就是在多个物体上挂了那个脚本类的多个实例而已，这样就好理解了。在new class这个过程中，数据区是复制的，代码区是共享的，算是一种特殊的复制+引用关系。

你可以再Instaniate一个同样的Prefab,还是这套mesh/texture/material/shader...，这时候会有新的GameObject等，但是不会创建新的引用对象比如Texture。所以你Load出来的Assets其实就是个数据源，用于生成新对象或者被引用，生成的过程可能是复制（clone)也可能是引用（指针）。当你Destroy一个实例时，只是释放那些Clone对象，并不会释放引用对象和Clone的数据源对象，Destroy并不知道是否还有别的object在引用那些对象。等到没有任何 游戏场景物体在用这些Assets以后，这些assets就成了没有引用的游离数据块了，是UnusedAssets了，这时候就可以通过Resources.UnloadUnusedAssets来释放，Destroy不能完成这个任 务，AssetBundle.Unload(false)也不行，AssetBundle.Unload(true)可以但不安全，除非你很清楚没有任何 对象在用这些Assets了。

为什么第一次Instaniate 一个Prefab的时候都会卡一下？因为在你第一次Instaniate之前，相应的Asset对象还没有被创建，要加载系统内置的 AssetBundle并创建Assets,第一次以后你虽然Destroy了，但Prefab的Assets对象都还在内存里，所以就很快了。


