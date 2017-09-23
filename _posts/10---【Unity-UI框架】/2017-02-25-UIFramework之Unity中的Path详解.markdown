---
layout: post
title:  "[UIFramework]UIFramework之Unity中的Path详解"
date:   2017-02-25 01:15:10
categories: 10---【Unity-UI框架】
comments: true
---

iOS

Application.dataPath :  Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/xxx.app/Data 
（dataPath是app程序包安装路径，app本身就在这里，此目录是只读的）

Application.streamingAssetsPath :   Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/xxx.app/Data/Raw   （只读）

Application.persistentDataPath :      Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Documents               （沙盒路径，可以将热更新的AssetBundle，配置文件，数据表等文件放在该路径下）

Application.temporaryCachePath :   Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Library/Caches         （相对临时的目录，适合存放下载缓存的临时文件，空间不足时可能会被系统清除）

Android

apk程序包目录: apk的安装路径，/data/app/package name-n/base.apk，dataPath就是返回此目录。

内部存储目录: /data/data/package name-n/，用户自己或其它app都不能访问该目录。打开会发现里面有4个目录（需要root） 

1、cache缓存目录，类似于iOS的Cache目录 

2、databases数据库文件目录 

3、files类似于iOS的Documents目录 （android平台上直接使用该路径当做沙盒路径）

4、shared_prefs 类似于iOS的Preferences目录，用于存放常用设置，比如Unity3D的PlayerPrefs就存放于此

外部存储目录: 在内置或外插的sd上，用户或其它app都可以访问，外部存储目录又分私有和公有目录。 

公有目录是像DCIM、Music、Movies、Download这样系统创建的公共目录，当然你也可以像微信那样直接在sd卡根目录创建一个文件夹。好处嘛，就是卸载app数据依旧存在。

私有目录在/storage/emulated/n/Android/data/package name/，打开可以看到里面有两个文件夹cache和files。为什么跟内部存储目录重复了？这是为了更大的存储空间，以防内存存储空间较小。推荐把不需要隐私的、较大的数据存在这里，而需要隐私的或较小的数据存在内部存储空间。

Application.dataPath : /data/app/xxx.xxx.xxx.apk   （apk程序包目录）

Application.streamingAssetsPath : jar: file:///data/app/xxx.xxx.xxx.apk/!/assets    （只读）

Application.persistentDataPath :         /data/data/xxx.xxx.xxx/files                                             
（沙盒路径，可以将热更新的AssetBundle，配置文件，数据表等文件放在该路径下）

Application.temporaryCachePath :      /data/data/xxx.xxx.xxx/cache                                     

Windows

Application.dataPath :                         /Assets              （工程路径）                                                                   

Application.streamingAssetsPath :      /Assets/StreamingAssets

Application.persistentDataPath :         C:/Users/xxxx/AppData/LocalLow/CompanyName/ProductName

Application.temporaryCachePath :      C:/Users/xxxx/AppData/Local/Temp/CompanyName/ProductName

Mac

Application.dataPath :                         /Assets

Application.streamingAssetsPath :      /Assets/StreamingAssets

Application.persistentDataPath :         /Users/xxxx/Library/Caches/CompanyName/Product Name

Application.temporaryCachePath :     /var/folders/57/6b4_9w8113x2fsmzx_yhrhvh0000gn/T/CompanyName/Product Name

PS

1、StreamingAssets 只能用过www类来读取。

安卓上跟其他平台不一样，安装后，这些文件实际上是在一个Jar压缩包里，所以不能直接用读取文件的函数去读，而要用WWW方式。具体做法如
下：
```java
byte[] InBytes;                                                                                                         
 //用来存储读入的数据   
if (Application.platform == RuntimePlatform.Android)                                             
 //判断当前程序是否运行在安卓下   
{   
        string FileName = "jar:file://" +   
Application.dataPath + "!/assets/" + "文件.txt";   
        WWW www = new WWW(FileName);                                                             
 //WWW会自动开始读取文件   
        while(!www.isDone){}                                                                         
//WWW是异步读取，所以要用循环来等待   
        InBytes = www.bytes;                                                                         
//存到字节数组里   
}   
else   
{   
       //其他平台的读取代码   
}  
```

2、根目录：StreamingAssets文件夹
```java
#if UNITY_EDITOR  
string filepath = Application.dataPath   
+"/StreamingAssets"+"/my.xml";  
#elif UNITY_IPHONE  
 string filepath = Application.dataPath   
+"/Raw"+"/my.xml";  
#elif UNITY_ANDROID  
 string filepath = "jar:file://"   
+ Application.dataPath + "!/assets/"+"/my.xml;  
#endif  
```

```java
/* 
* Name: GameFileUtil.cs 
* Function: N/A  
*  
* Ver     变更日期               负责人                            变更内容 
* ────────────────────────────────────────────────────────────────────── 
* V1.0.0  $time$    http://blog.csdn.net/husheng0      
*  
* Copyright (c). All rights reserved. 
* 
* 游戏中的路径相关管理类  
*  
*/  
  
using UnityEngine;  
using System.Collections;  
using System.IO;  
using System;  
  
public class GameFileUtil : SingletonManager<GameFileUtil>  
{  
 
#if UNITY_STANDALONE_WIN || UNITY_EDITOR_WIN  
    public const string FileProto = "file:///";  
#else  
    public const string FileProto = "file://";  
#endif  
    /// <summary>  
    /// 返回沙盒路径  
    /// Android：假设文件都存在手机上，不存在SD卡，返回路径  
    /// Application.persistentDataPath :         /data/data/xxx.xxx.xxx/files   
    /// iOS：返回路径Application.persistentDataPath :  
    /// Application/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Documents  
    /// PC：返回路径Application.dataPath的上一级路径        /Assets/../   
    /// </summary>  
    /// <param name="type">文件类型，对应沙盒路径下的文件名字</param>  
    /// <param name="fileName">文件名</param>  
    /// <param name="useType">使用类型</param>  
    /// <returns></returns>  
    public string GetFilePath(GameFileType type, string fileName)  
    {  
        string filePath = Application.dataPath + "/../"   
            +type.ToString()+ fileName.ToString();  
#if UNITY_ANDROID||UNITY_IPHONE  
        filePath = Application.persistentDataPath    
        + "/" + type.ToString()+ fileName.ToString();  
#endif  
        switch (type)  
        {  
            case(GameFileType.AssetBundles):  
                filePath = filePath + ".assetbundle";  
                break;  
            case(GameFileType.Config):  
                filePath = filePath + ".ini";  
                break;  
            case(GameFileType.CapturePicture):  
                filePath = filePath + ".png";  
                break;  
            case(GameFileType.Datas):  
                filePath = filePath + ".csv";  
                break;  
            case(GameFileType.NetDatas):  
                filePath = filePath + ".xml";  
                break;  
            default:  
                break;  
        }  
        return filePath;  
    }  
  
    public string GetStreamingAssetsPath()  
    {  
        string filePath = Application.dataPath + "/StreamingAssets/";  
#if UNITY_ANDROID||UNITY_IPHONE  
        filePath = Application.streamingAssetsPath+"/";  
#endif  
        return filePath;  
    }  
  
    public string GetAssetFilePath(GameFileType fileType,  
        string fileName, GameFileUseType useType)  
    {  
        string filePath = null;  
        switch (useType)  
        {  
            case(GameFileUseType.Read):  
            case(GameFileUseType.Write):  
                if (File.Exists(GetFilePath(fileType, fileName)))  
                {  
                    filePath = GetFilePath(fileType, fileName);  
                }  
                break;  
            case(GameFileUseType.CreatePathRead):  
            case(GameFileUseType.CreatePathWrite):  
                if (File.Exists(GetFilePath(fileType, fileName)))  
                {  
                    filePath = GetFilePath(fileType, fileName);  
                }  
                else  
                {  
                    try  
                    {  
                        FileStream stream = new FileStream(filePath, FileMode.CreateNew);  
                        filePath = GetFilePath(fileType, fileName);  
                    }  
                    catch (Exception e)  
                    {  
                        Debug.LogError(e.Message);  
                    }  
                }  
                break;  
            default:  
                break;  
        }  
        return filePath;  
    }  
  
    /// <summary>  
    /// 使用WWW加载文件的路径  
    /// </summary>  
    /// <param name="fileType">文件类型</param>  
    /// <param name="subFilePath">在其文件类型下的相对目录，定位到文件名，不使用/开头</param>  
    public string GetWWWLoadPath(GameFileType fileType, string fileName,  
        GameFileUseType useType = GameFileUseType.Read)  
    {  
        string filePath = GetAssetFilePath(fileType, fileName, useType);  
        if (string.IsNullOrEmpty(filePath))  
        {  
            return null;  
        }  
        return FileProto + filePath;  
    }  
}  
/// <summary>  
/// 文件类型枚举  
/// </summary>  
public enum GameFileType  
{  
    AssetBundles,       //AssetBundle文件  
    Datas,              //数据表文件  
    Config,             //配置文件  
    CapturePicture,     //截屏文件  
    NetDatas,           //网络数据缓存文件  
}  
/// <summary>  
/// 文件使用类型  
/// </summary>  
public enum GameFileUseType  
{  
    Read,           // 读文件  
    CreatePathRead, // 读文件，如果文件所在的目录不存在，会自动创建目录  
    Write,          // 写文件  
    CreatePathWrite,// 写文件，如果文件所在的目录不存在，会自动创建目录  
}  
```

