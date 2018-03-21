---
layout: post
title:  "[UIFramework]UIFramework之Unity5.x AssetBundle打包策略"
date:   2017-09-09 11:29:10
categories: Unity UI
comments: true
---

Unity5.x在AssetBundle打包和AssetBundle加载方面做了一些更改，AssetBundle打包主要是通过设置AssetBundle Name，然后Unity会自动分析依赖关系进行打包，AssetBundle加载主要是可以通过加载AssetBundleManifest文件来获得依赖关系进行加载，此外，AssetBundle加载Asset时，不再支持直接获得Component。`Unity5.x在AssetBundle打包时，依然需要程序员自己分析依赖关系设置AssetBundle Name。`

#### 1、BuildAssetBundles(string outputPath, BuildAssetBundleOptions assetBundleOptions, BuildTarget targetPlatform)
所有设置了AssetBundle Name的资源会打成AssetBundle，未设置AssetBundle Name的资源会根据依赖关系(包括直接依赖和间接依赖)打包到被依赖的资源的AssetBundle中，所以要注意不需要打包的资源的冗余问题。

如：

①APrefab设置了AssetBundle Name为a.assetbundle，APrefab依赖于BPrefab(未设置AssetBundle Name)，BPrefab依赖于CPrefab(未设置AssetBundle Name)，使用该函数打包，则APrefab、BPrefab与CPrefab都会被打包到a.assetbundle中。

②APrefab设置了AssetBundle Name为a.assetbundle，APrefab依赖于BPrefab(设置AssetBundle Name为a.assetbundle)，BPrefab依赖于CPrefab(未设置AssetBundle Name)，使用该函数打包，则APrefab会被打包到a.assetbundle中，BPrefab与CPrefab都会被打包到b.assetbundle中。

#### 2、BuildAssetBundles(string outputPath, AssetBundleBuild[] builds, BuildAssetBundleOptions assetBundleOptions, BuildTarget targetPlatform)
该打包方法与AssetBundle Name无关。

①APrefab依赖于BPrefab，当AssetBundleBuild[]只打打包APrefab时，BPrefab也会被打到该AssetBundle中，与BPrefab是否设置了AssetBundle Name无关。当BPrefab需要单独打包时，AssetBundleBuild[]应同时包含APrefab与BPrefab。

#### 3、GetAllDependencies(string assetBundleName)和GetDirectDependencies(string assetBundleName)
会根据依赖关系获取信息，与资源存在于哪个AssetBundle文件无关。

#### 4、AssetBundle.LoadFromFile(string path):AssetBundle同步加载
AssetBundle.LoadFromFileAsync(string path):AssetBundle异步加载

`The function supports bundles of any compression type.`

#### 5、AssetBundle.LoadAsset(string assetName):同步加载AssetBundle资源
AssetBundle.LoadAllAssetsAsync(string assetName):异步加载AssetBundle资源

Prior to version 5.0, users could fetch individual components directly using LoadAsync.This is not supported anymore. Instead, please use LoadAssetAsync to load the game object first and then look up the component on the object.

#### 6、AssetBundle.Unload(true)卸载AssetBundle加载的Asset Memory与AssetBundle序列化头文件
AssetBundle.Unload(false)仅卸载AssetBundle序列化头文件

依赖关系分析可参开之前Unity4.x打包的分析策略：`UIFramework之Unity4.x AssetBundle打包策略`（见本博客）

#### Summary
①不再支持脚本打包了；

②AssetBundle资源加载不再支持直接获取组件了，需先加载Object然后再获得Components；

`参考代码：`[HushengStudent/myAssetBundleTools](https://github.com/HushengStudent/myAssetBundleTools)

③下载工程，点击AssetBundleTools/BuildAllAssetBundle/OK，打开场景GameController，运行。