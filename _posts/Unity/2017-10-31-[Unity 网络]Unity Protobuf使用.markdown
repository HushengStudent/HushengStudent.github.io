---
layout: post
title:  "[Unity 网络]Unity Protobuf使用"
date:   2017-10-31 20:30:00
categories: Unity
comments: true
---

Unity3D中使用protobuf的方法：

1、下载protbuf-net的源码：[protobuf-net](https://github.com/mgravell/protobuf-net/releases)；

2、将该目录中protobuf-net目录下的所有C#源码拷贝到Unity3D中，直接使用源码而不是第三方dll。使用dll会在ios平台出问题；

3、此时在Unity中编译时，可能会报错说 unsafe不能使用；

4、采用如下方案可以解决： 在Assets目录下面新建 smcs.rsp文件，并在其中写入  -unsafe 字符串，前后不加空格；

5、重新启动unity，此时我们可以发现该工程能够通过编译；

6、服务器可以编译成dll；


    using (MemoryStream memoryStream = new MemoryStream(buffer))
    {
        ProtoBuf.Serializer.NonGeneric.Serialize(stream, body);
    }

    ProtoBody DeserializeProto(Stream stream, Type bodyType)
    {
        return ProtoBuf.Serializer.NonGeneric.Deserialize(bodyType, stream) as ProtoBody;
    }


	
[参考：](http://gad.qq.com/article/detail/12402)

网络协议

网络IO

消息 广播 同步

TCP/UDP IP

集群

负载均衡

分布式

多线程/线程池

开源网络通讯框架/模型

阻塞/非阻塞/同步/异步

Proactor/Reactor/Actor Select/Poll/Epoll/Iocp/Kqueue

短连接和长连接

游戏安全

缓存

消息编码协议

Socket

Nagle/粘包/截断/TCP_NODELAY

AI/场景

分线/分地图