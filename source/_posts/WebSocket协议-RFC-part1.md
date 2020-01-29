---
title: WebSocket协议 RFC part1
date: 2020-01-29 18:51:55
comments: true
tags: 
- websocket
categories: 
- web 
keywords: 
- websocket
- rfc
excerpt: WebSocket协议的官方文档 第一部分
---
# The WebSocket Protocol




## Abstract

   The WebSocket Protocol enables two-way communication between a client running untrusted code in a controlled environment to a remote host that has opted-in to communications from that code.  The security model used for this is the origin-based security model commonly used by web browsers.  The protocol consists of an opening handshake followed by basic message framing, layered over TCP.  The goal of  this technology is to provide a mechanism for browser-based applications that need two-way communication with servers that does not rely on opening multiple HTTP connections (e.g., using   XMLHttpRequest or &lt iframe&gt s and long polling).

摘要
WebSocket协议提供了一种全双工通信给处于可控环境中运行着未被信任的代码的客户端和选择与这部分代码相通信的远程主机。用于这种通信的安全模型是源于通常被web浏览器所使用的的安全模型。这种协议包括一种开放的握手协议以及随后的基本消息帧，构建在TCP层上。这项技术的目的是为需要与服务器双向通信基于浏览器的应用提供一种机制，使其不在依赖于打开多个HTTP连接（例如使用XMLHttpRequest,或者&ltiframe&bt以及long polling机制）。

Status of This Memo

   This is an Internet Standards Track document.

   This document is a product of the Internet Engineering Task Force
   (IETF).  It represents the consensus of the IETF community.  It has
   received public review and has been approved for publication by the
   Internet Engineering Steering Group (IESG).  Further information on
   Internet Standards is available in Section 2 of RFC 5741.

   Information about the current status of this document, any errata,
   and how to provide feedback on it may be obtained at
   http://www.rfc-editor.org/info/rfc6455.

Copyright Notice

   Copyright (c) 2011 IETF Trust and the persons identified as the
   document authors.  All rights reserved.

   This document is subject to BCP 78 and the IETF Trust's Legal Provisions Relating to IETF Documents (http://trustee.ietf.org/license-info) in effect on the date of publication of this document.  Please review these documents carefully, as they describe your rights and restrictions with respect  to this document.  Code Components extracted from this document must include Simplified BSD License text as described in Section 4.e of the Trust Legal Provisions and are provided without warranty as described in the Simplified BSD License.
从该文档中摘录出的代码组件必须要包含根据the Trust Legal Provisions的4.e节描述的Simplified BSD License文本，且提供是需根据Simplified BSD License中所阐述中的不带任何授权。


## 1.  Introduction

### 1.1.  Background

   _This section is non-normative._

   Historically, creating web applications that need bidirectional communication between a client and a server (e.g., instant messaging and gaming applications) has required an abuse of HTTP to poll the   server for updates while sending upstream notifications as distinct HTTP calls [RFC6202].
历史上，创建需要在一个客户端和一个服务端双向通信的web应用（例如，即时通信和游戏应用）时，不得不大量使用以至于泛滥的HTTP去轮询服务器只为了获得更新，同时以HTTP调用的方式发送上行的通知消息。
   This results in a variety of problems:  
这导致了各种各样的问题：
+ The server is forced to use a number of different underlying TCP connections for each client: one for sending information to the client and a new one for each incoming message.  
服务器被迫针对于每个客户端使用一定数量不同的底层TCP连接：一个用来发送信息给客户端而另一个新的则处理每条接受的消息。 
+ The wire protocol has a high overhead, with each client-to-server message having an HTTP header.  
连线协议有一个很高的开销，~~因为~~每个client到server的消息都有一个HTTP头。
+ The client-side script is forced to maintain a mapping from the outgoing connections to the incoming connection to track replies.  
客户端侧的脚本被迫去维护~~发送链接到接受链接~~传出连接和传入连接之间的映射关系来跟踪回复消息。   

A simpler solution would be to use a single TCP connection for traffic in both directions.  This is what the WebSocket Protocol provides.  Combined with the WebSocket API [WSAPI], it provides an
   alternative to HTTP polling for two-way communication from a web page to a remote server.    
一个更简单的解决方案~~可能~~是使用一个TCP连接来同时实现两个方向的数据流通。这就是WebSocket协议提供的。结合WebSocket API[WSAPI], ~~它提供了一种实现HTTP轮询从一个网页到一个远程服务器的双向通信的替代方案~~它为从web页面到远程服务器的双向通信提供了HTTP轮询的替代方法。  
The same technique can be used for a variety of web applications: games, stock tickers, multiuser applications with simultaneous editing, user interfaces exposing server-side services in real time, etc.  
相同的技术也可以用于各种各样的web应用：游戏，股价行情指示器，~~多用户应用同时编辑~~具有同步编辑功能的多用户应用程序，实时显示~~服务器侧~~服务器端服务的用户界面，等等。

   The WebSocket Protocol is designed to supersede existing bidirectional communication technologies that use HTTP as a transport layer to benefit from existing infrastructure (proxies, filtering, authentication).  Such technologies were implemented as trade-offs between efficiency and reliability because HTTP was not initially meant to be used for bidirectional communication (see [RFC6202] for
   further discussion).  The WebSocket Protocol attempts to address the goals of existing bidirectional HTTP technologies in the context of the existing HTTP infrastructure; as such, it is designed to work
   over HTTP ports 80 and 443 as well as to support HTTP proxies and intermediaries, even if this implies some complexity specific to the current environment.  However, the design does not limit WebSocket to
   HTTP, and future implementations could use a simpler handshake over a dedicated port without reinventing the entire protocol.  This last point is important because the traffic patterns of interactive
   messaging do not closely match standard HTTP traffic and can induce unusual loads on some components.
WebSocket协议~~的设计是为了~~旨在替代现有的那些使用HTTP作为一个传输层来从现有的基础设施（代理，过滤，认证）获利的双向通信的技术。~~这些技术被实现成在效率和可靠性之间的交易~~此类技术是作为效率和可靠性之间的权衡实现的。因为HTTP最初并不是为了被用作双向通信（可以参看RFC6202来做更多深入的讨论）。WebSocket协议尝试在现有HTTP~~基础设施的背景下~~基础架构的上下文中来解决现有的双向HTTP技术的目标要求；~~正因为如此~~就其本身而言，它被设计成工作在HTTP的80和433端口上，同时支持HTTP代理以及~~intermediaries~~中介， 即使这意味着针对当前的环境会有一些复杂性。然而，这种设计并没有限制WebSocket~对~~为HTTP，而且未来实现可以用专门的端口来实现一种更简单的握手协议而不用重新发明整套协议。最后的一点非常重要，因为交互消息的流量模式并不是~~紧密地~~完全地和标准的HTTP流量匹配的，~~是可以引入不常用的负载在一些组件上的~~可能会在某些组件上引起不寻常的负载。

### 1.2.  Protocol Overview

   _This section is non-normative._

   The protocol has two parts: a handshake and the data transfer.   
   这套协议有两部分，一次握手和数据传输   
   The handshake from the client looks as follows:    
   从客户端的握手看起来如下：   
```
        GET /chat HTTP/1.1
        Host: server.example.com
        Upgrade: websocket
        Connection: Upgrade
        Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
        Origin: http://example.com
        Sec-WebSocket-Protocol: chat, superchat
        Sec-WebSocket-Version: 13
```
   The handshake from the server looks as follows:
从服务器端看到握手如下：
```
        HTTP/1.1 101 Switching Protocols
        Upgrade: websocket
        Connection: Upgrade
        Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
        Sec-WebSocket-Protocol: chat
```
   The leading line from the client follows the Request-Line format. The leading line from the server follows the Status-Line format.  The Request-Line and Status-Line productions are defined in [RFC2616].       
客户端的~~开头一行~~开始的行遵循的是Request-Line的格式。服务器的~~第一行~~首行则遵循的Status-Line格式。Request-Line和Status-Line的~~productions~~结果在RFC2616中定义。  

   An unordered set of header fields comes after the leading line in both cases.  The meaning of these header fields is specified in Section 4 of this document.  Additional header fields may also be
   present, such as cookies [RFC6265].  The format and parsing of headers is as defined in [RFC2616].   
在这两种情况下，一个无序的header字段的集合都会~~跟在~~位于第一行之后。这些header字段的含义在~~这篇~~本文档的第4节被指明阐述。其他的header字段可能也会出现，例如cookies[RFC6256]. headers的格式以及解析定义在RFC2616中。  
   Once the client and server have both sent their handshakes, and if the handshake was successful, then the data transfer part starts. This is a two-way communication channel where each side can, 
   independently from the other, send data at will.  
一旦客户端和服务端~~同时~~都发送了他们握手信息，~~而且~~如果握手~~是~~成功~~的~~，~~那么数据传输的部分就开始了~~则开始数据传输部分。~~这是一个每一边都能独立于对方按照意愿发送数据的双向通信通道。~~这是一个双向通信通道，每一方都可以随意发送数据，彼此独立。  
   After a successful handshake, clients and servers transfer data back and forth in conceptual units referred to in this specification as "messages".  On the wire, a message is composed of one or more  frames.  The WebSocket message does not necessarily correspond to aparticular network layer framing, as a fragmented message may be coalesced or split by an intermediary.  
在一次成功的握手之后，客户端和服务器以本规范中~~所说的~~称为“messages”的~~预想的~~概念单元来回的传输数据。~~同时~~在网络上，一条消息是由一个或多个帧构成的。WebSocket消息并不~~需要对应特殊~~一定对应于特定网络层帧，而作为一个片段消息可能~~coalesced合并或者被中介分拆~~被一个中间层合并或者分割。  
A frame has an associated type.  Each frame belonging to the same message contains the same type of data.  Broadly speaking, there are types for textual data (which is interpreted as UTF-8 [RFC3629]text), binary data (whose interpretation is left up to theapplication), and control frames (which are not intended to carrydata for the application but instead for protocol-level signaling,such as to signal that the connection should be closed).  Thisversion of the protocol defines six frame types and leaves tenreserved for future use.  
~~一个~~帧有~~一个相关的类型~~具有相关联的类型。每个属于同一条消息的帧包含相同数据的类型。~~宽泛的说~~广义上讲，类型有文本数据（被~~翻译~~解释为UTF-8【RFC3629】的文本），二进制数据（它的~~翻译~~解释被留给了对应的应用），以及控制帧（并不用来承载应用的数据而是用于协议~~层面的~~级信号通知，例如通知连接应该被关闭）。这个版本的协议定义了6个帧类型并~~空出~~剩下了十个保留以~~便~~备将来使用。  
~~一个~~帧有~~一个相关的类型~~具有相关联的类型。每个属于同一条消息的帧包含相同数据的类型。~~宽泛的说~~广义上讲，类型有文本数据（被~~翻译~~解释为UTF-8【RFC3629】的文本），二进制数据（它的~~翻译~~解释被留给了对应的应用），以及控制帧（并不用来承载应用的数据而是用于协议~~层面的~~级信号通知，例如通知连接应该被关闭）。这个版本的协议定义了6个帧类型并~~空出~~剩下了十个保留以~~便~~备将来使用。  