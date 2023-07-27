---
title: "webRTC技术-流媒体服务器"
date: 2023-04-20
draft: false
tags: ["video/audio","webrtc"]
---

## 多对多通信模型

Google的WebRTC框架本身提供了浏览器间1对1的通信模型，那么在局域网中，浏览器可以直接实现P2P的通信。如果浏览器终端位于NAT后，那么通过STUN/TURN实现NAT穿越，同样可以进行直接的媒体数据交换（对于完全对称型NAT来说，需要借助TURN服务器进行数据转发）。

但当需要进行多对多的WebRTC通信时，就需要额外的服务来提供支持，以下是三种最为常见的方案：

### 1、2D Mesh 网络

2D Mesh网络是一个在描述 CPU/Chip 互联方式与片上网络 时使用的名词，在这里表述WebRTC终端之间的连接如同mesh网络的架构一样，即两两互联，如下图所示：

![image-20230712173236120](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307121732917.png)

Mesh互联方案一个最大的好处就是不需要流媒体服务端中转数据，节省了服务器资源。但是对于客户端的配置要求很高，当 A 想要共享媒体（比如音频、视频）时，它需要分别向 B 、C、D 发送数据，带宽开销很大。

### 2、MCU 模式

MCU模式下，需要在服务端对于各个WebRTC的媒体流进行解码-混合-编码-推流，对于服务端的要求比较高。

特别是编解码和混流的步骤，会消耗很多的CPU资源，所以一般单台服务器所提供的视频也有上限（十几路）。

其架构如下图：

![image-20230712174228042](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307121742282.png)

MCU服务器的处理流程一般包括几个步骤：

- 接受音视频流
- 解码
- 混流
- 编码
- 发送

![image-20230712174757786](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307182000547.png)

其中由于接受端所需要的媒体流不同（一般不包括自己发送出的媒体流），所有各个端推送的媒体流的混流过程中也不同，这就对服务端的性能有很高的要求。

一般MCU的方案分为硬MCU和软MCU两种，其中硬件MCU的方案需要使用相应的硬件设备，进行硬件编解码、混流等。

而软MCU方案具备更高的灵活性，如FreeSWITCH项目。

### 3、SFU 模式

以上两种方案都有明显的优劣势，对于客户端/服务端的要求很高。而SFU的方案，更像一个媒体流的路由服务，只负责将媒体流转发给需要他的客户端，而不对媒体流本身做更改（如编解码），这样就大大缓解了对于服务器CPU的压力。

其一般架构如下：

![image-20230719112238799](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307191122711.png)

服务端可以根据客户端的需求和网络质量，来选择转发不同的媒体流。

SFU的优势可以总结为：

- 开销较低，不需要编解码操作
- 转发延迟较低
- 能够自由决定转发给每个端设备的媒体流类型（如媒体质量，带宽等）

## 开源流媒体服务器

目前 WebRTC 多方通信媒体服务器都是 SFU 架构，如 Janus-gateway、MediaSoup、Medooze ，此处仅对较为熟悉的 Janus-gateway 做一些记录。

### Janus-gateway

Janus 是一个开源的 WebRTC 流媒体服务器，在 Linux 环境下使用c语言实现。

源码：https://github.com/meetecho/janus-gateway 。

部署文档：https://janus.conf.meetecho.com/docs/deploy.html 。

#### 整体架构

从整体上来，Janus 是以插件式架构进行实现的，收到的数据包会被转发给各个插件。

![image-20230714162458744](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307182000485.png)

其整体架构如下图所示：

![image-20230726204027892](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307262040261.png)

#### 通信流程：

1. 客户端发送create创建一个Janus会话，Janus回复success返回Janus会话句柄；

   浏览器、插件、janus core 的交互过程基本遵循以下流程（以 websocket plugin 为例）

![image-20230726202438406](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307262040772.png)

2. 客户端发送attach命令在Janus会话上attach指定插件，Janus回复success返回插件的句柄；
3. 客户端给指定的插件发送message进行信令控制，Janus上的插件发送 event 通知事件给客户端；
4. 客户端收集candidate并通过trickle消息发送给插件绑定的ICE通道，Janus发送webrtcup通知ICE通道建立；
5. 客户端发送媒体数据；Janus发送media消息通知媒体数据的第一次到达；Janus进行媒体数据转发。

### janus 作为 WebRTC 网关信令交互过程

![image-20230727105558542](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307271056945.png)



## Reference

[1] 从0打造音视频直播系统

[2] [janus 信令交互过程](https://www.52dianzi.com/bangong/?read-1686500485a2721.html)

[3] [基于声网的音视频SDK和FreeSWITCH开发WebRTC2SIP Gateway](https://juejin.cn/post/6844904031484133389#heading-1)
