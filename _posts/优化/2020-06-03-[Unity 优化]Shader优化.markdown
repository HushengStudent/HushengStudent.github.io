---
layout: post
title:  "[Unity 优化]Shader优化"
date:   2020-06-03 22:49:00
categories: 优化
comments: true
---

#### Shader变种：
multi_compile：代码控制关键字开关，一定会编译变种；

shader_feature：一般使用材质控制是否使用，被使用才会生成变种(打AB的话，Shader要和材质球/ShaderVariantCollection打包到同一个AB里的时候才能知道需要哪些shader_feature)；

skip_variants：可以剪裁变种；

Graphics设置里面设置Always Inclueded Shaders，则所有变种一定都会被打包；

变种太多会导致ShaderLab内存占用变大，Shader.Parse(编译Shader)和Shader.CreateGpuProgram(创建CPU执行程序片段)占用CPU时间变长；


#### ShaderVariantCollection使用：
使用ShaderVariantCollection来WarmUp，而不是全部WarmUp，是为了优化Shader.CreateGpuProgram(创建CPU执行程序片段)；

使用ClearCurrentShaderVariantCollection和SaveCurrentShaderVariantCollection来计算生成多个ShaderVariantCollection，多帧进行WarmUp；


#### Shader打包AB：
把所有Shader和ShaderVariantCollection一起打包到一个AB，游戏运行时加载这个AB全部资源并缓存，并调用其中所有ShaderVariantCollection.WarmUp；

按以上方法打包，理论上之后运行游戏，不会再有Shader.Parse和Shader.CreateGpuProgram；

如果还出现了Shader.Parse这说明Shader存在冗余；

如果还出现了Shader.CreateGpuProgram这说明Shader存在冗余或者ShaderVariantCollection收集不完全；


#### Shader冗余：
按以上Shader打包AB的方式进行打包，如果还存在Shader冗余，则可能是：

①Resource目录下存在Shader并在运行时被使用了，常见第三方插件；

②Unity默认材质引用的Shader，又没有添加到Always Inclueded Shaders；


#### ShaderLab内存占用：
ShaderLab内存占用较大，则可能是：

①Shader冗余，可在Profiler界面Assets/Shader看到同一个Shader出现多个；

②变种特别多的Shader使用不当，甚至把这种Shader加到了Always Inclueded Shaders，常见如FBX的默认材质使用Standard；