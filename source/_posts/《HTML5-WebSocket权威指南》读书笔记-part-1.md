---
title: 《HTML5 WebSocket权威指南》读书笔记 part 1
date: 2020-01-30 18:29:30
tags: 
- websocket
categories: 
- web 
- booknotes
keywords: 
- websocket
excerpt: 《HTML5 WebSocket权威指南》读书笔记第一部分，主要是为什么要用websocket，以及描述websocket是什么，后面部分都是实际应用例子，读完后若有必要再整理
---
# Websocket 之前的世界：
轮询polling：因为不清楚什么时候有更新，无法掌握好确切的轮询时间间隔，结果是，在低信息率的情况下，你可能打开或者关闭许多不必要的连接。  
长轮询long polling,也称comet，或反向ajax，以及“挂起GET”和“搁置POST”:  
具体的long polling可以参考图![long polling](https://camo.githubusercontent.com/ee4304f7d599f41425937f3f322e944f5ff4412a/687474703a2f2f7777772e706c616e74756d6c2e636f6d2f706c616e74756d6c2f70726f78793f666d743d737667267372633d68747470733a2f2f676973742e67697468756275736572636f6e74656e742e636f6d2f6c7563747275646561752f65346333383962306531363536343066646464302f7261772f)  
流化技术中，客户端发送一个请求，服务器发送并维护一个持续更新和保持打开（可以是无限或者规定的时间段（涉及到超时断开））的开放响应（因为服务器不发送响应的完成信息，所以一直在响应前述的请求）。每当服务器有需要交付给客户端的信息时，它就更新响应。会有五个问题，  
第一，如果数据量大，客户端同样要不断重连，性能和轮询没差；  
第二，缺乏实现标准；  
第三，因为响应一直未完成，代理和防火墙反应缓慢会导致信息延迟增加；  
第四，每一种情况下，客户端都必须等待请求返回，才能发出后续的请求，而这显著地增加了延迟；  
第五，包含了附加额外的header中的字段以及延迟，开销也很大。  
对比polling的几种机制，如下图:![various polling](http://blog.outsider.ne.kr/attach/1/1340436424.gif.pagespeed.ce._mFuj4R9Nb.gif)


# Websocket 之后的世界：
HTML5规范的连接性部分包含了WebSocket。WebSocket是一种自然的全双工、双向、单套接字连接。WebSocket减少了延迟，因为一旦建立起WebSocket连接，服务器可以在消息可用时发送它们。

暂时性的理解websocket：借助于http实现的类似于底层socket的技术，所以既通过了http，又有底层socket的灵活性。
相关技术：WebSocket的重点是改进Web应用程序前端和服务器之间的通信，而SPDY优化的是应用程序内容和静态页面的交付。HTTP和WebSocket之间的不同是架构性的，而不是增量的。
WebSocket和SPDY是相互补充的，你可以将SPDY扩充的HTTP连接升级为WebSocket  

# Websocket API
主要描述了WebSocket中的事件，方法，和特性
WebSocket的连接通过在客户端和服务器之间第一次握手时将HTTP协议升级到WebSocket协议来完成，这已工作在相同的底层TCP连接上进行。而WebSocket完全是事件驱动的，这点我觉得可以想象socket的处理过程，epoll对读写事件的驱动的自动处理。  
WebSocket API定义了两种 URL方案（URL Scheme），WS和WSS（WebSocket Secure）,分别用于客户端和服务器之间的非加密和加密流量。  
## WebSocket 构造函数  
限定URL是ws开头。WebSocket的最大好处之一是能在WebSocket上建立广泛使用的协议层次（将在第3章～第6章介绍），使你可以完成许多出色的工作，例如将传统的桌面应用程序带到Web中。  
客户发送带有协议名称的Sec-WebSocket-Protocol首标。服务器选择0个或者1个协议，响应一个带有和客户请求相同的协议名称的Sec-WebSocket-Protocol首标；  

## WebSocket事件  
WebSocket编程遵循异步编程，即连接打开，应用程序就简单地监听事件，只要为WebSocket对象添加回调函数，或者用DOM方法addEventListener()添加事件监听器。
有四个不同的事件-open,messsage,error,close  
 + open事件  
 open事件触发时，握手已经完成，WebSocket已经准备发放数据了。  
 + message事件  
 WebSocketAPI只输出完整的消息而不是WebSocket帧，message事件在接收到消息时触发，可以处理文本消息，二进制（Blob或者ArrayBuffer）消息。
 + error事件  
 error事件在响应意外故障时触发。对应为回调函数onerror.
 + close事件
close事件有3个有用的属性(property), wasClean、close和error。
事件相关的可以参见[官网](http://www.w3.org/TR/websockets/) 


## WebSocket方法
WebSocket对象有两个方法：send()和close().
+ send方法  
在调用onopen监听器之后，调用onclose监听器之前调用send（）方法.
如果想发送消息响应另一个事件，可以检查WebSocket readyState属性，并选择只在套接字打开时发送数据.
Blob对象在与JavaScript File API结合使用以发送和接收文件时特别有用，这些文件主要有多媒体文件、图像、视频和音频。
+ close方法  
使用close方法，可以关闭WebSocket连接。

## WebSocket对象特性  
readyState特性：是描述连接状态的四个值，WebSocket.CONNECTING, WebSocket.OPEN,WebSocket.CLOSING,WebSocket.CLOSED   
bufferedAmount特性：检查已经进入队列，但是尚未发送到服务器的字节数。
protocol特性：它让服务器知道客户端理解并可在WebSocket上使用的协议。包含在打开握手期间WebSocket服务器选择的协议名。

注意，如何确定browser是否支持websocket，在JS控制台求取window.WebSocket表达式的值。如果你看到WebSocket构造函数对象，就意味着浏览器对WebSocket有原生支持。  
WebSocket是连接Web世界和互联网世界(更具体额说是TCP/IP)的桥梁。

Chome可以在开发者工具中的Network的选项卡中查看WebSocket的流量。

# Websocket协议  
## WebSocket握手  
+ 初始握手，websocket连接都始于一个HTTP请求，其中最主要的是header中的一个字段Upgrade,用来标明客户端将吧连接升级到不同的协议。除非
*服务器响应101代码*、  
*Upgrade首标*、  
*Sec-WebSocket-Accept首标*，  
否则WebSocket连接不能成功。Sec-WebSocket-Accept响应首标的值从Sec-WebSocket-Key请求首标继承而来，包含一个特殊的响应键值，必须与客户端的预期精确匹配。
+ 需要注意的是如何计算响应的键值：
响应函数从客户端发送的Sec-WebSocket-Key首标中取得键值，并在Sec-WebSocket-Accept首标中返回根据客户端预期计算的键值。代码清单3-3使用Node.js加密API计算键值和后缀组合的SHA1散列值。
+ 消息格式  
客户端和服务器可以在任何时候相互发送消息。这些消息在网络上用标记消息之间边界并包括简洁的类型信息的二进制语法表示。更准确地说，这些二进制首标标记另一个单位—帧（frame）—之间的边界。下面展示了WebSocket帧头![WebSocket帧头](https://i.niupic.com/images/2020/01/30/6maz.png)
WebSocket帧化代码负责：操作码/长度/解码/文本/屏蔽/多帧消息
+ WebSocket关闭握手  
当WebSocket关闭时，终止连接的端点可以发送一个数字代码，以及一个表示选择关闭套接字原因的字符串。
+ 对其他协议的支持  
用WebSocket API协商更高层协议的方法。在网络层上，这些协议用Sec-WebSocket-Protocol首标协商。
+ 扩展  
和协议一样，扩展也使用Sec-首标协商。连接的客户端发送一个Sec-WebSocket-Extensions首标，包含所支持的扩展名称。
