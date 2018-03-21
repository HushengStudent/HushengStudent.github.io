---
layout: post
title:  "[UIFramework]UIFramework之Unity4.x AssetBundle打包策略"
date:   2017-04-12 22:06:10
categories: Unity UI
comments: true
---

#### 1、打包方法
打包资源
```java
BuildPipeline.BuildAssetBundle(UnityEngine.Object mainAsset, UnityEngine.Object[] assets,  
string pathName, BuildAssetBundleOptions assetBundleOptions)  
```

```java
BuildPipeline.PushAssetDependencies();  
BuildPipeline.PopAssetDependencies();  
```

进行一次Push，多次Build操作，如依次Build资源Prefab-A，Prefab-B时，可以认为Prefab-A，Prefab-B会依次 进栈，所以如果两者之间也存在共享资源，则后者会依赖前者。具体表现为，运行时先加载Prefab-B会出现共享资源丢失的情况。

4.x 中脚本也会作为“共享资源”参与依赖性打包，即当Prefab-A和Prefab-B同时挂有脚本M时，如果出现了上一点中的情况，那么后者同样会依赖前者。具体表现为，运行时先加载Prefab-B会出现脚本M丢失。

#### 2、资源打包

①将Shader/Script/字体等在游戏中占用内存较少的资源单独打包，在游戏开始时全部加载，常驻内存。

②将音频/动画控制器/动画/大的背景贴图、图集等不需要实例化的Prefab、存储配置设置/数据等相关的Prefab单独打包成AssetBundle。

③依赖打包：

分析剩余Prefab之间的依赖关系，对于只有一个父引用的Prefab直接打包到父Prefab中，对于有多个父引用的Prefab则进行单独打包。

依赖打包需要使用到上面控制依赖关系的两个函数，先打包子Prefab，再打包父Prefab。

BuildAssetBundleOptions：

CompleteAssets：用于保证资源的完备性。比如，当你仅打包一个Mesh资源并开启了该选项时，引擎会将Mesh资源和相关GameObject一起打入
AssetBundle文件中。

CollectDependencies：用于收集资源的依赖项。比如，当你打包一个Prefab并开启了该选项时，引擎会将该Prefab用到的所有资源和Component全部打入AssetBundle文件中。

DeterministicAssetBundle：用于为资源维护固定ID，以便进行资源的热更新。

依赖性打包的作用在于避免资源冗余，同时提高资源加载和卸载的灵活性。但是使用依赖打包的Prefab打成AssetBundle的粗细也会对游戏性能产生很大的影响，需要具体权衡。

对Shader的打包除了上述方法外，也可以将Shader放入GraphicsSettings->Always Included Shaders中后，打包时会将相应的Shader抽离，运行时加载时会自动加载其依赖的Shader。同时也意味着，如果修改了Always Included Shaders或在一个新建项目中使用该Bundle，会出现Shader丢失的问题。

当需要更新Bundle内容，但不改变依赖关系时，仍然需要重打其依赖的Bundle包。即如果Bundle-B依赖Bundle-A，那么在更新Bundle-A时可以不需要重打Bundle-B（前提是开启了DeterministicAssetBundle）；但要更新Bundle-B的话，则必须重打Bundle-A。