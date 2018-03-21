---
layout: post
title:  "[Unity 热更新]tolua集成"
date:   2017-11-08 00:00:10
categories: Unity
comments: true
---

LuaFramework采用了pureMVC，并集成了多个第三方库，对于项目而言，我们可以直接使用tolua集成到项目，也可以使用LuaFramework删掉pureMVC相关内容集成到项目，使用自己项目的游戏框架。下面为使用tolua集成到项目中可能遇到的问题整理：

1、lua的后缀是不被支持打包进assertbundle的，一般把 .lua后缀 变为.lua.txt 或者 .lua.bytes 进行打包。

2、LuaFramework打包Lua到AssetBundle时，是采用单个目录下的打包成一个AssetBundle，这里可以改成自己喜欢的方式。

3、Lua文件的AssetBundle加载，使用LuaFramework采用的加载方式即可。

4、Lua代码加密问题，lua本身可以使用luac将脚本编译为字节码(bytecode)从而实现加密。

5、LuaFramework实现的部分脚本可以利用起来：

①LuaBehaviour.cs       //便于调用Lua代码中的Awake();Start();Update()......

②LuaLoader.cs            //Lua的AssetBundle加载

③LuaManager.cs

④LuaHelper.cs             //给Lua提供调用C#接口

⑤Util.cs                        //工具类

6、编辑器的使用，之前有推荐IntelliJ+EmmyLua开发的方式，我们可以导出相关c#文件到Lua，便于开发，提高开发效率。

7、配置表的读取：即从配表到Lua文件的转换过程，github上有许多工具。

8、LuaFramework集成了pbc，sproto等第三方库，tolua本身只包含cjson，protoc-gen-lua等，对于需要支持协议热更新的项目，需要选择一个把proto文件转换到Lua的工具：pbc和pblua(protoc-gen-lua)都各有优缺点（lua的支持问题[有反馈不支持lua53的]，协议嵌套问题等）。

9、自己编译底层库，这个过程还是相当繁琐的......