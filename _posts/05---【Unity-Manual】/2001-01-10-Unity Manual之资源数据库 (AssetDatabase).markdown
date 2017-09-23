---
layout: post
title:  "[Unity Manual]Unity Manual之资源数据库 (AssetDatabase)"
date:   2001-01-10 10:10:10
categories: 05---【Unity-Manual】
comments: true
---

资源数据库 (AssetDatabase) 是允许您访问工程中的资源的 API。此外，其提供 方法供您查找和加载资源，还可创建、删除和修改资源。

Unity 编辑器 (Editor) 在内部使用资源数据库 (AssetDatabase) 追踪资源文件，并维护资源和引用资源的对象之间的关联。Unity 需要追踪工程文件夹发生的所有变化，如需访问或修改资源数据，您应始终使用资源数据库 (AssetDatabase) API，而非文件系统。 资源数据库 (AssetDatabase) 接口仅适用于编辑器，不可用于内置播放器。和所有其他编辑器类一样，其只适用于置于编辑器 (Editor) 文件夹中的脚本（只在主要的资源 (Assets) 文件夹中创建名为“编辑器”的文件夹（不存在该文件夹的情况下））。

#### 1、导入资源
通常，Unity 只在需要时自动导入已拖放至该工程的资源，但也可能在脚本控制下导入这些资源。为此，您可以使用以下示例中的 AssetDatabase.ImportAsset 类函数。

```java
using UnityEngine;  
using UnityEditor;  
  
public class ImportAsset {  
    [MenuItem ("AssetDatabase/ImportExample")]  
    static void ImportExample ()  
    {  
        AssetDatabase.ImportAsset("Assets/Textures/texture.jpg",   
                       ImportAssetOptions.Default);  
    }  
}  
```

您也可将额外的 AssetDatabase.ImportAssetOptions 类型参数传递至资源数据库 (AssetDatabase) 。脚本组件手册页面记录了不同的选项及其对函数行为的影响。

#### 2、加载资源
如果将资源添加至场景或在检视 (Inspector) 面板中编辑这些资源，则编辑器仅在需要时加载资源。但是，您可以使用以下脚本加载和访问资源：AssetDatabase.LoadAssetAtPath、AssetDatabase.LoadMainAssetAtPath、AssetDatabase.LoadAllAssetRepresentationsAtPath 和 AssetDatabase.LoadAllAssetsAtPath。有关更多详细信息，请参阅脚本文档。

```java
using UnityEngine;  
using UnityEditor;  
  
public class ImportAsset {  
    [MenuItem ("AssetDatabase/LoadAssetExample")]  
    static void ImportExample ()  
    {  
        Texture2D t = AssetDatabase.LoadAssetAtPath("Assets/Textures/texture.jpg",  
                               typeof(Texture2D)) as Texture2D;  
    }  
}  
```

#### 3、使用 AssetDatabase 操作文件
Unity 将保留资源文件的元数据，您决不可使用文件系统创建、移动或删除它们。相反，您应使用 AssetDatabase.Contains、AssetDatabase.CreateAsset、AssetDatabase.CreateFolder、AssetDatabase.RenameAsset、AssetDatabase.CopyAsset、AssetDatabase.MoveAsset、AssetDatabase.MoveAssetToTrash 和 AssetDatabase.DeleteAsset 进行上述操作。

```java
public class AssetDatabaseIOExample {  
    [MenuItem ("AssetDatabase/FileOperationsExample")]  
    static void Example ()  
    {  
        string ret;  
  
        // Create  
        Material material = new Material (Shader.Find("Specular"));  
        AssetDatabase.CreateAsset(material, "Assets/MyMaterial.mat");  
        if(AssetDatabase.Contains(material))  
            Debug.Log("Material asset created");  
  
        // Rename  
        ret = AssetDatabase.RenameAsset("Assets/MyMaterial.mat", "MyMaterialNew");  
        if(ret == "")  
            Debug.Log("Material asset renamed to MyMaterialNew");  
        else  
            Debug.Log(ret);  
  
        // Create a Folder  
        ret = AssetDatabase.CreateFolder("Assets", "NewFolder");  
        if(AssetDatabase.GUIDToAssetPath(ret) != "")  
            Debug.Log("Folder asset created");  
        else  
            Debug.Log("Couldn't find the GUID for the path");  
  
        // Move  
        ret = AssetDatabase.MoveAsset(AssetDatabase.GetAssetPath(material),  
                                   "Assets/NewFolder/MyMaterialNew.mat");  
        if(ret == "")  
            Debug.Log("Material asset moved to NewFolder/MyMaterialNew.mat");  
        else  
            Debug.Log(ret);  
  
        // Copy  
        if(AssetDatabase.CopyAsset(AssetDatabase.GetAssetPath(material),  
                        "Assets/MyMaterialNew.mat"))  
            Debug.Log("Material asset copied as Assets/MyMaterialNew.mat");  
        else  
            Debug.Log("Couldn't copy the material");  
        // Manually refresh the Database to inform of a change  
        AssetDatabase.Refresh();  
        Material MaterialCopy = AssetDatabase.LoadAssetAtPath("Assets/MyMaterialNew.mat",  
                        typeof(Material)) as Material;  
  
        // Move to Trash  
        if(AssetDatabase.MoveAssetToTrash(AssetDatabase.GetAssetPath(MaterialCopy)))  
            Debug.Log("MaterialCopy asset moved to trash");  
  
        // Delete  
        if(AssetDatabase.DeleteAsset(AssetDatabase.GetAssetPath(material)))  
            Debug.Log("Material asset deleted");  
        if(AssetDatabase.DeleteAsset("Assets/NewFolder"))  
            Debug.Log("NewFolder deleted");  
  
        // Refresh the AssetDatabase after all the changes  
        AssetDatabase.Refresh();  
    }  
}  
```

#### 4、使用 AssetDatabase.Refresh
修改完资源后，您应调用 AssetDatabase.Refresh 将更改提交至数据库，并使其显示在工程中。
