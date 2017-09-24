---
layout: post
title:  "[OpenGL]Visual Studio搭建OpenGL开发环境"
date:   2015-01-10 10:10:10
categories: 33---【OpenGL】
comments: true
---

首先我们需要关于Opengl的一系列文件

我们需要把这些不同类型的文件放在不同的地方。

把.H结尾的文件全部放在VS安装目录下的\VC\include\GL中，如果没有GL文件夹，则自己新建一个。

把.LIB结尾的文件全部放在VS安装目录下的\VC\lib中。

把.dll结尾的文件全部放在系统目录下System32文件夹中（最好在SysWOW64文件夹中也放一份）。
