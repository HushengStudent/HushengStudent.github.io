---
layout: post
title:  "[Golang]windows环境搭建"
date:   2014-01-02 10:10:10
categories: Golang
comments: true
---

解压GO压缩包到：C:\go；创建文件夹C:\gopath\src\myfirstgo\hello.go；

设置环境变量:

![图片](http://owk5gjdrg.bkt.clouddn.com/0067%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA.png)

![图片](http://owk5gjdrg.bkt.clouddn.com/0068%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA.png)

安装vs code，打开目录C:\gopath。

![图片](http://owk5gjdrg.bkt.clouddn.com/0069%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA.png)

安装插件与工具（ctrl+shift+p）。

![图片](http://owk5gjdrg.bkt.clouddn.com/0070%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA.png)

![图片](http://owk5gjdrg.bkt.clouddn.com/0071%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA.png)

安心等待全部安装完成。

有安装失败的，手动从github搜索下载，放入如下目录。

如果还有安装失败的，按照提示，下载https://github.com/golang/tools，新建目录C:\gopath\src\golang.org\x，把tools放进入。

重新打开vscode安装工具：

在hello.go输入如下，然后运行：

```java
package main  
  
import "fmt"  
  
func main(){  
    fmt.Print("hello world")  
}  
```
