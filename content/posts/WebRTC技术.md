---
title: "webRTC技术"
date: 2023-04-11
draft: false
tags: ["audio"]
---

## WebRTC

wikipedia:

**WebRTC** (**Web Real-Time Communication**) is a [free and open-source](https://en.wikipedia.org/wiki/Free_and_open-source_software) project providing [web browsers](https://en.wikipedia.org/wiki/Web_browser) and [mobile applications](https://en.wikipedia.org/wiki/Mobile_application) with [real-time communication](https://en.wikipedia.org/wiki/Real-time_communication) (RTC) via [application programming interfaces](https://en.wikipedia.org/wiki/Application_programming_interface) (APIs). It allows audio and video communication to work inside web pages by allowing direct [peer-to-peer](https://en.wikipedia.org/wiki/Peer-to-peer) communication, eliminating the need to install [plugins](https://en.wikipedia.org/wiki/Plug-in_(computing)) or download native apps.

![image-20230711110655383](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307111107172.png)



## 数据包传输

一般使用 UDP 进行传输，数据包需要进行一次封装，标识例如帧内分片号、起始结束标志等

### RTP 协议

![image-20230710194506591](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307101945753.png)

ref: https://datatracker.ietf.org/doc/html/rfc3550#section-5.1

### RTCP协议

​	在RTP数据包的传输过程中，对于出现丢包、抖动、乱序的情况时，由 RTCP 协议进行控制。

![image-20230710195212377](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307101952801.png)

ref: 

rtcp_type: https://datatracker.ietf.org/doc/html/rfc3550#section-6.4.1

SR type: https://datatracker.ietf.org/doc/html/rfc5760#autoid-5

## 媒体协商

媒体协商的作用就是让双方找到**共同支持的媒体能力**，如双方**都支持的编解码器**，从而最终实现彼此之间的音视频通信。

### SDP协议

SDP(session description protocol) 即会话控制协议，用来描述端上设备的能力，如编解码方式、传输协议、音视频格式等。在两个客户端进行通信时，首先要进行信令的交互，如SDP信息的交互。一个SDP携带的信息如下所示：

```
v=0
o=- 7017624586836067756 2 IN IP4 127.0.0.1
s=-
t=0 0
// 以上表示会话描述

m=audio 9 UDP/TLS/RTP/SAVPF 111 103 104 9 0 8 106 105 13 126
...
m=video 9 UDP/TLS/RTP/SAVPF 96 97 98 99 100 101 102 122 127 121 125 107 108 109 124 120 123 119 114 115 116
...
...
a=rtpmap:111 opus/48000/2
a=rtpmap:103 ISAC/16000
a=rtpmap:104 ISAC/32000
...
```

当B端收到A端发送的SDP消息后，会将其支持的编码方式等与自身做对比，协商出公共的策略后返回给A端SDP信息。

SDP格式：

v=0开始表示整个session的性质：

- `v=0` ，表示 SDP 的版本号
- `o=<username> <session id> <version> <network type> <address type> <address>`表示的是对会话发起者的描述；``
- `s=<session name>`，表示一个会话
- `t=<start time> <stop time>`描述了会话的开始时间和结束时间

m=audio...表示一路音频流，m=video则表示视频流

- `m=<media> <port> <transport> <fmt list>`表示一个会话

a=表示属性，用于进一步描述媒体信息

- `a=rtpmap:<payload type> <encoding name>/<sample rate>[/<encodingparameters>]`表示一种编码格式

**总结：SDP协议内容是由一个会话层和多个媒体层组成的。**

更多细节可以参考rfc : https://datatracker.ietf.org/doc/pdf/draft-nandakumar-rtcweb-sdp-08

### 媒体协商过程

1. 通信双方按照SDP格式封装各自的媒体信息，如编解码器、媒体流的 SSRC、传输协议、IP 地址和端口等。
2. 通信双方通过信令服务器交换 SDP 信息，选择共同支持的媒体能力。
3. 开始进行通信

假设通信双方分别为A和B，由A发起SDP协商，那么我们把A成为offer方，B为answer方：

![image-20230710202557995](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307102026157.png)

双方都会存储产生或收到的SDP信息，自己的叫做Local，对方的叫做remote.

## 连接建立

### NAT

为了解决IPV4地址不足与安全性两个主要问题，出现了NAT技术。

但是NAT技术也带来了P2P通信的困难。

NAT可以分为四类：

1. 锥形NAT

   当 host 主机Z 通过 NAT 访问外网的 B 主机时，就会在 NAT 上打洞（建立内外网映射），所有知道这个“洞”的主机（A、C）都可以通过它与内网主机上的侦听程序通信。

   ![image-20230710204056091](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307102040039.png)

2. IP限制性锥型NAT

   只有 host 主机访问过的外网主机才能穿越 NAT

   ![image-20230710204411606](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307102044513.png)

3. 端口限制型NAT

   不光在 NAT 上对打洞的 IP 地址做了限制，而且还对具体的端口做了限制

   ![image-20230710204615965](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307102046424.png)

4. 对称型NAT

   不同的连接使用不同的NAT端口甚至连接

   ![image-20230710204930081](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307102049077.png)

NAT类型探测有一系列的过程，如下图：

![image-20230710205238402](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307102052539.png)

网上有很多资料，此处不再详述。

在探测出NAT类型后，可以尝试通信。

注意：对称型 NAT 与对称型 NAT 之间以及对称型 NAT 与端口限制型 NAT 之间无法打洞成功。此处需要使用借助服务器进行互联通信。

### TURN/STUN 服务器

1. Session Traversal Utilities for NAT (STUN) - Used to establish a direct UDP connection between two clients.
2. Traversal Using Relay around NAT (TURN) - Used to establish a relayed UDP or TCP connection between two clients. Here, the traffic must be relayed through the TURN server to bypass restrictive firewall rules, and the preference is UDP over TCP because TCP's guaranteed ordered delivery of packets implies overhead that is undesirable for real-time communications.
3. Secure Traversal Using Relay around NAT (TURNS) - Used to establish a relayed TCP/TLS connection between two clients. Here, the traffic must be relayed through the TURN server and through a TLS socket to bypass extremely restrictive firewall rules.

简单来说：

- STUN服务器是用来获取外部地址的。
- TURN服务器是用来在直接连接（点到点）失败的情况下进行中继数据流量的

开源的TURN/STUN服务器：

- coturn: https://github.com/coturn/coturn

## 数据安全性

在数据包传输的过程中，为了保证数据的安全性和完整性，防止篡改，需要对对方的身份进行验证以及数据加密，其主要使用的协议为DTLS协议（TLS over UDP）。

### DTLS协议

TLS协议握手过程：

ref: https://datatracker.ietf.org/doc/html/rfc5246#section-7.3

![image-20230711094123459](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307111108629.png)

DTLS协议握手过程：

ref: https://datatracker.ietf.org/doc/html/rfc6347#section-4.2.4

![image-20230711094227571](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307111108770.png)



### 加密协商过程

![image-20230711093907884](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202307111108895.png)

整个身份确认的过程可以分为几个步骤：

1. 通过信令服务器交换 SDP 信息，SDP 中记录了用户名、密码和指纹，用于确认身份。

   eg.

   ```
   ...
   a=ice-ufrag:khLS
   a=ice-pwd:cxLzteJaJBou3DspNaPsJhlQ
   a=fingerprint:sha-256 FA:14:42:3B:C7:97:1B:E8:AE:0C2:71:03:05:05:16:8F:B9:C7:98:E9:60:43:4B:5B:2C:28:EE:5C:8F3
   ...
   ```

2. Client A 通过 STUN  Server 进行认证，如果 STUN 请求中的用户名和密码与 SDP 信息中一致，说明用户客户端请求合法。

3. 进行 DTLS 协商，交换公钥证书并协商密码相关的信息。

4. 开始传输 SRTP 加密数据。

## 信令服务

WebRTC只描述和规定了数据平面该如何采集和传输数据，而对于如何发现对端，如何找到我要交谈的人没有做任何规定，即信令协议和机制不是由WebRTC标准定义的。

因此，我们需要自己去实现一个信令服务来在客户端之间交换信令消息和应用程序数据：

- 作为“对端发现”的桥梁
- 保存  session 的状态

同时，WebRTC的标准化特性使得在浏览器中运行的WebRTC应用程序与另一个通信平台运行的设备（如电话或视频会议系统）之间建立通信成为可能。

针对不同的端设备，有一些现成的信令服务器可用：



webrtc-webrtc 端：

- webRTC.io
- easyRTC
- Signalmaster

webrtc-SIP 端：

- janus-gateway



## Reference

[1] [真实世界中的WebRTC](https://michaelyou.github.io/2018/08/01/%E7%9C%9F%E5%AE%9E%E4%B8%96%E7%95%8C%E4%B8%AD%E7%9A%84WebRTC/)

[2] [What Are STUN, TURN, and ICE?](https://developer.liveswitch.io/liveswitch-server/guides/what-are-stun-turn-and-ice.html)

[3] [coturn](https://github.com/coturn/coturn)

[4] [从0打造音视频直播系统](https://time.geekbang.org/column/article/107916)