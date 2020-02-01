---
title: 《HTML5 WebSocket权威指南》读书笔记 part 3
date: 2020-02-01 09:45:00
tags: 
- websocket
categories: 
- web 
- booknotes
keywords: 
- websocket
excerpt: 《HTML5 WebSocket权威指南》读书笔记第二部分，主要是为什么要用websocket例子的搭建，最主要都是代码验证了构建远程JS控制台
---
# 用XMPP构建WebSocket上的即时消息和聊天
## TCP上的互联网应用层协议的典型分层
展示了TCP上的互联网应用层协议的典型分层方式包含HTTP有两个原因。  
![应用层协议的典型分层](https://i.niupic.com/images/2020/01/31/6mep.jpg)
其一，它说明HTTP作为TCP之上的一个应用层协议，可以直接用于Web应用程序中。AJAX应用程序中将HTTP当作所有网络交互的主要或唯一协议。  
其二，使用WebSocket的应用程序不需要完全忽略HTTP。静态资源总是通过HTTP加载。例如，即使你选择使用WebSocket进行通信，HTML、JavaScript和装饰用户界面的CSS仍然可以通过HTTP提供。因此，在应用协议栈中，你可以在TLS和TCP上使用HTTP及WebSocket。  

## XMPP：XML的流化
###  XMPP如何将XML文档语法用于实时通信呢？
实现方法之一是用单独的文档发送每条消息。然而，这种方法可能出现不必要的冗长文档，浪费资源。  
另一种方法是将对话当作一个长文档，随着时间推移和消息传输而增长，这是XMPP对文档语法的处理方式。XMPP连接期间的双向对话各方都由一个流化XML文档表示，这个文档在连接终止时结束。该流化文档的根节点是一个＜stream/＞元素。流的最高级子节点是协议的单独数据单位，称作“节”（stanza）。
###  选择连接性策略
用WebSocket连接XMPP服务器有两种方法：修改XMPP服务器以接受WebSocket连接，或者使用代理服务器。选择连接性策略时，理解WebSocket消息（通常由单个WebSocket帧组成）与XMPP节的对齐方式很重要，因为在上述两种方法中对齐的方式是不同的。
+ 在WebSocket感知的XMPP服务器中，节被一对一映射到WebSocket消息上，每条WebSocket消息包含一节，没有重叠或者碎片。  
+ 在网关方案中，节与消息的对齐不是必需的，因为它将WebSocket中转到TCP，反之亦然。TCP没有消息边界，所以TCP流可能随意地分割到WebSocket消息中。然而，在网关的情况下，客户端必须理解流化的XML，将字符重组为节。  

许多IM网络都是“围墙花园”。在特定网络上有账户的用户才能相互交谈。与此相反，Jabber（http：//www.jabber.org）是一个联盟网络，也就是说，如果服务器相互协作，在独立操作服务器上的用户可以通信。  

连接到Google Talk你可能从Gmail和Google+中熟悉了Google Talk聊天服务，它是JabberIM网络的一部分。有一个公众可以访问的XMPP服务器监听talk.google.com上的5222端口，如果有一个Google账户，你就可以将任何兼容的XMPP客户端连接到该地址并登录。为了将你自己的Web客户端连接到Google Talk，可以将WebSocket代理服务器指向该地址。该服务器需要加密，所以一定要配置代理服务器通过TLS连接。  

具体例子代码因为涉及到Strophe.js，并没有时间去深入研究，所以不赘述。日后有兴趣再来翻看，验证。

# 用STOMP通过WebSocket传递消息  
消息传递（messaging）是一种架构风格，特征是在独立的组件之间发送异步消息，实现松耦合的系统。消息为通信模式提供了一个抽象层，从而为编写网络应用程序提供了一种非常灵活而强大的手段。  
消息传递中的关键是消息代理和客户端。消息代理（broker）接受客户端的连接，处理来自客户端的消息，并向客户端发送消息。代理还能处理验证、授权、消息加密、可靠消息传递、消息流量控制和分类等工作。当客户端连接到消息代理，它们能够将消息发送给代理，也能接受代理发送给它们的消息。这种模型称为发布/订阅（publish/subscribe），消息代理发布一些消息，客户端订阅所有消息，或者消息的一个子集。  
PS:STOMP定义中的Text（文本）部分说明该协议是面向文本的。第6章（重点是RFB协议）描述了在WebSocket上使用面向二进制协议的方法。  

## 什么是发布/订阅模式
在消息的世界里，有两种常用的消息传播技术。  
+ 队列：向单个消费者传递消息的传播机制。任意数量的客户端（发布者）可以向队列发布消息，但是每条消息只能由一个客户端（消费者）接收。
![Queue模式](https://nats.io/img/documentation/nats-queue.png)  
+ 主题：向多个消费者传递消息的传播机制。任意数量的客户端（发布者）可以向一个主题发布消息，任何数量的客户端（消费者）可以接收消息。
![Topic模式](https://nats.io/img/documentation/nats-pub-sub.png)  
其实还有一种模式，书中没有讲到，请求回复模式。
+ 请求：向指定多个消费者传递消息的传播机制。客户端（发布者）可以指定向有几个客户端（消费者）可以收到。发送应答模式采用同步调用。
![Request Reply模式](https://nats.io/img/documentation/nats-req-rep.png)    

## STOMP简介
STOMP是一种开放消息传递协议，最初被开发用于Apache ActiveMQ，后来被广泛传播到其他系统上。STOMP没有主题或者队列。STOMP消息从不同目标发送和接收，STOMP服务器决定这些目标的行为。STOMP：面向简单（或者流化）文本的消息传递协议。而企业中最广泛使用的消息传递API是JMS：Java Message Service （Java消息服务）。与通过定义线路协议提升互操作性的STOMP不同，JMS只是一个API。STOMP已经用多种语言实现；因为JMS的本质是一个API，它几乎统治了Java世界。

结合使用STOMP和ActiveMQ。ActiveMQ使用目标名称输出消息传递特性，包括主题和队列、临时目标和层次化订阅。可以任何TCP级协议的相同方式在WebSocket上添加STOMP层，或者进行对齐，使每个STOMP帧正好占据一个WebSocket帧。

to be continued: 建一个应用程序，使用户能够玩一个流行的游戏—石头-布
暂时没有时间完成，但是很有兴趣通过STOMP以及ActiveMQ来实现这个游戏。

# 用远程帧缓冲协议实现VNC
使用VNC（Virtual Network Computing，虚拟网络计算）允许你在任何网络上共享桌面。它本质上允许你远程查看和控制另一台计算机的界面，可以看作和Telnet等价的GUI（图形用户界面）。你还可以将VNC看作一根很长的虚拟电缆，可以用它的鼠标、键盘和视频信号查看和控制另一个桌面。使用WebSocket和远程帧缓冲（RemoteFramebuffer，RFB）协议将VNC扩展到Web上的方法。这章主要是示例作为一个二进制协议，RFB如何以不同于前面两章中讨论过的面向文本协议的方式使用WebSocket API。  
### RFB是什么  
远程帧缓冲协议（RFB）是IETF的一个信息规范（RFC 6143）。虽然它不是正式的标准，但是得到了广泛的应用，有许多可互操作的实现。RFC6143本身已经有10年以上的历史，经过了数次修订。我们先来分解一下协议的定义。帧缓冲（framebuffer）是一个数组，包含了图形化计算机系统显示的所有像素值，是台式机的最小共同模型。因此，RFB是远程访问帧缓冲的一种手段。对于任何有键盘、鼠标和屏幕的系统，都可能有一种利用RFB访问它的方法。  
RFB是一个二进制协议，传输的是二进制图像数据。数据可以进行压缩，也可以随着高频率的更新流入或者流出服务器。  
 Wireshark支持RFB协议会话的分析，这在调试新的实现时可能很有用。  
**具体示例需要暂时没有时间细究，略过**。  
需要注意的是当客户端已经可以接收帧缓冲，现在我们在客户端显示这些信息，使客户端能够查看来自RFB服务器（在我们的例子中是TightVNC）的GUI信息。＜canvas＞是HTML5中最重要的新元素之一。＜canvas＞元素支持一个2D绘图API，使HTML5应用程序具备操纵像素图形的能力。  
在RFB协议中，PointerEvent表示移动或者指点设备（pointing device）按钮的按下或者释放。PointerEvent消息是二进制事件消息，由一个指定发送到服务器的消息种类（如指点设备点击、指点设备移动等）的消息类型字节，一个按钮掩码（携带指点设备1～8号按钮的当前状态，由第0位到第7位表示，0代表按钮弹起，1代表按钮按下），以及两个位置值（由表示屏幕相对的X坐标和Y坐标的无符号短整型组成）组成。

# WebSocket安全性  
在Web上部署应用程序带来了安全性上的挑战，在决定使用WebSocket时你必须加以考虑。
  
| 攻击类型 | 解决问题的WebSocketAPI或者协议特性 |
| ------ | ------ |
| 拒绝服务 | Origin(源)首标 |
| 连接洪泛拒绝服务 | 用Origin首标限制新连接 |
| 代理服务器供给 | 屏蔽 |
| 中间人，窃听 |WebSocket安全(wss://) |
### WebSocket是什么（从安全性的角度）  
WebSocket有许多可取的特性。它是一个简单、标准的全双工协议，开销很低，可以用于构建可伸缩、几乎实时的网络服务器。如果你是一位聪明的计算机专业学生或者稍作回忆，就会知道上述的好处也适用于普通而纯粹的TCP/IP。也就是说，它们是套接字（特别是SOCK_STREAM）的好处，而非WebSocket独有的。那么为什么要在“套接字”（Socket）前加上Web？为什么不简单地在TCP基础上构建传统的互联网应用程序呢？  
要回答这个问题，我们需要区分“非特权”和“特权”应用程序代码。非特权代码是运行在已知源中的代码，通常是网页中运行的JavaScript。在Web安全性模型中，TCP连接无法被“非特权”代码安全地打开。如果允许非特权应用程序代码打开TCP连接，脚本就可以用伪造的首标发出HTTP请求，使其看上去好像来自不同的源。有了这种伪造首标的能力，相同的脚本可以重新在TCP上实现HTTP以避开规则，从而使控制脚本发起HTTP连接方式的规则变得毫无意义。允许从Web应用程序中发起TCP连接将会破坏源模型。  
WebSocket连接从非特权代码中发起，因此和AJAX及其他允许在非特权代码中使用的网络功能遵循相同的模型。HTTP握手使连接的初始化在浏览器的控制下进行，允许浏览器设置源首标和保持源模型所需的首标。这样，WebSocket允许应用程序利用轻量级的双向连接实现互联网风格的网络，同时与HTTP应用程序、服务器和沙箱规则并存。  
从特权代码建立的WebSocket连接一般可以打开任何网络连接；这种能力不是问题（至少对于Web不是问题），因为特权应用总是可以使用任何网络连接，必须由用户安装和执行。原生应用程序不运行于Web源，所以源规则不适用于从特权代码建立的WebSocket。
### 限制新连接  
尽管源首标在HTTP层上阻止连接洪泛（flood），但是在TCP层上仍然有连接洪泛DoS攻击的潜在威胁。即使打开的TCP连接没有携带数据资源，最终被服务器拒绝，大量的客户端也可能淹没服务器。为了避免连接超载，WebSocket API要求浏览器限制打开的新连接。  
### 屏蔽
WebSocket帧是WebSocket消息的组成部分。对从浏览器发往服务器的WebSocket帧进行屏蔽，以混淆帧内容，因为拦截流量的代理服务器可能被WebSocket流量所迷惑。屏蔽还有另一个不常见的微妙原因—它和安全性有关。我们知道有三类代理服务器：
+ 转发代理服务器：通常由服务器管理员安装、配置和控制。转发代理服务器将出站请求从内联网转发到互联网。  
+ 反向代理服务器：通常由服务器管理员安装、配置和控制。反向代理服务器（或者防火墙）一般部署在服务器之前的停火区（DMZ），执行安全功能，以保护内部服务器免遭来自互联网的攻击。  
+ 透明代理服务器：通常由网络操作员控制。透明代理服务器通常拦截网络通信用于缓冲，或者阻止公司内联网用户因为特定目的而访问Web。网络操作员可能使用透明代理服务器缓冲经常访问的网站，减少网络负载。  
屏蔽主要是针对透明代理服务器的安全性。*HTTP缓存中毒*是一种攻击类型，攻击者尝试控制HTTP缓存，用危险的内容代替请求的资源。在WebSocket标准化进程中，一组研究人员撰写了一篇论文，概述了使用HTTP升级请求对透明拦截代理进行理论上的进攻，之后，缓存中毒变成了一个重大的问题。如何避免中毒？WebSocket协议中加入了屏蔽（masking），屏蔽是一种混淆（但不是加密）协议内容，以避免透明拦截代理发生混乱的技术。屏蔽将从浏览器发往WebSocket服务器的每条消息内容与随机字节进行异或运算，转换它们的载荷。  
和源一样，屏蔽是一种安全特性，它不需要用于对付窃听的加密安全。通信双方和中间人在需要的时候能够理解屏蔽的载荷。然而，对于不理解屏蔽载荷的“中间人”，它们可以避免将WebSocket消息的内容错误地解释为HTTP请求和响应。
### TLS加强WebSocket安全性  
HTTPS和WSS协议非常相似，两者都在通过TCP连接的TLS基础之上运行。为WebSocket线路流量配置TLS加密的方法与HTTP相同：使用证书。WebSocket协议在TCP（和HTTP类似）上运行，WSS连接运行在TLS上，而TLS运行在TCP之上。WebSocket协议与HTTP兼容，所以WebSocket连接使用相同的端口：WebSocket默认端口为80，而WebSocket安全（WSS）默认使用443端口。  

### 验证  
为了确认通过WebSocket连接到我们服务器的用户身份，WebSocket握手可以包含cookie首标。使用cookie首标使服务器能够在WebSocket验证中使用和验证HTTP请求中相同的cookie。作为替代，验证可以通过应用层协议，在WebSocket升级完成之后进行。XMPP和STOMP等协议层次中内建了识别用户和交换凭据的语义。
####应用级安全性  
应用级安全性描述的是应用程序如何保护自身免遭可能暴露私有信息的攻击。这一级别的安全性保护由应用程序暴露的资源。如果你使用XMPP、STOMP或AMQP（Advanced Message QueueingProtocol，高级消息队列协议）等标准协议，可以在基于WebSocket的系统上利用应用级安全性。权限的配置和具体的服务器相关。  

# 部署的考虑  
WebSocket特别相关的部署特性，比如WebSocket模拟、代理和防火墙、负载均衡和容量规划。


### 用传输层安全（TLS或SSL）穿越代理和防火墙。
代理通常分为两类：显式代理和透明代理。当浏览器显式配置为使用代理时，该代理服务器就是显式代理。对于显式代理，你必须向浏览器提供代理主机名称、端口号以及可选的用户名和密码。当浏览器没有意识到流量被代理拦截时，代理服务器就是透明的。即使流量必须穿越显式和透明代理，使用WSS也可能显著增加WebSocket成功的几率。  

### 部署TLS   
部署TLS需要用于识别WebSocket服务器的加密数字证书。在生产环境中，这些证书必须由Web浏览器知晓并信任的证书机构（CA）签发 

### WebSocket ping和pong  
连接可能因为许多你无法控制的原因而意外关闭。因为WebSocket连接处于TCP连接的上层，发生在TCP级别的连接问题会影响WebSocket连接。在客户端和WebSocket服务器之间的全双工连接中，有时候连接上可能没有数据流。在这个时候，网络中介可能中止连接。  
使用WebSocket ping和pong能够保持连接打开，为数据流动做好准备。ping和pong可以从打开的WebSocket连接的任一端发起。WebSocket协议支持客户端发起和服务器发起的ping和pong。浏览器或服务器（也可以是两者）都可以在合适的时间间隔内发起ping和pong，保持连接活跃。**注意，我们说的是浏览器而不是WebSocket客户端**：**WebSocket API目前不支持客户端发起的ping和pong**。

### WebSocket缓冲和流量控制  
由于WebSocket应用程序使用全双工连接，你可以控制应用程序向服务器发送数据的速率，这也称作“流量控制”（throttling）。流量控制有助于避免可能受到其他限制影响的网络饱和或者瓶颈，例如互联网带宽和服务器CPU的限制。WebSocket API可以用WebSocket bufferedAmount特性（我们在第2章中讨论过）控制应用程序向服务器发送数据的速率。bufferedAmount特性表示在队列中尚未传输到服务器的字节数。  
你还可以限制客户端到服务器的连接数，允许服务器根据预先定义的设置确定是接受还是拒绝客户端连接。  
### 监控  
为了评估系统的性能，你还可以在必要的时候配置监控工具，跟踪用户活动、服务器性能，以及终止客户端会话  
###  容量规划  
在架构中实施WebSocket可以建立灵活、可伸缩的框架。尽管有这样的灵活性，你仍然必须为部署需求做计划，包括规模的考虑，特别是与硬件容量相关的因素。这些领域包括服务器的内存和CPU（不管是启用WebSocket的后端服务器，还是使WebSocket流量能够在客户端和后端服务器之间流动的网关），以及网络优化。  
此外，有许多基于云的WebSocket服务产品，它们实际上让WebSocket开发人员不需要考虑容量规划。  
### 套接字限制  
WebSocket服务器同时打开许多连接。你应该意识到，如果运行一台没有更改过操作系统设置的服务器，可能无法维持几千个以上的套接字。即使还有丰富的CPU和内存资源，你也将会看到无法打开更多文件的错误报告。（记住，在UNIX中，几乎所有资源都是文件，包括套接字！即使没有使用磁盘，也可能看到关于文件的错误信息。）操作系统限制每个用户打开的文件数，默认情况下，这一限制值相当低。这些限制是为了避免滥用许多用户争用相同资源的共享系统。  
例如，在Liunx上，ulimit-a命令显示当前用户限制，包括最大允许打开文件数。很幸运，在Linux上可以提高这一限制。
### WebSocket应用程序部署检查列表  
| 规划项目|备注 |
| ------ | ------ |
| WebSocket模拟和备用手段|确定用户的浏览器和版本确定备用策略是否必要。如果是，使用填充、插件后者Comet备用措施 |
|反向代理和负载平衡 |确定你想要公开的端口，确定必须使用反向代理的服务器，确定是否可以在firewall上打开一个端口，从而确定是否应该使用反向连接性，确定网络资源负载需求，包括服务器和客户端连接 |
| 用TLS穿越代理和防火墙|确定可能扰乱WebSocket流量的代理和防火墙；确定因为安全或者连接性原因使用TLS |
| 持续连接|确定你想要监控的连接，设置ping间隔以避免连接超时 |
| 缓存和流量控制| 确定流量控制能够改进性能的地方|
|监控 |确定监控区域 |
|硬件容量规划 |确定WebSocket服务器的内存和cpu需求；确定宽带需求；确定服务器的磁盘需求；确定基于云超是否更适合于你的应用部署 |
| 套接字限制| 确定系统需要的鬓发套接字连接数，确定服务器的套接字限制|  



# WebSocket流量和资源  
### WebSocket流量  
研究3个方便的工具：
+ Google Chrome开发者工具：Chrome自带的一组HTML5应用程序，你可以用它检查、调试和优化Web应用程序。
+ Google Chrome Network Internals（或称“net-internals”）：一组检查网络行为（包括DNS查找、SPDY、HTTP缓存和WebSocket）的工具。
在地址栏中键入**chrome：//net-internals**。网络内部工具（net-internals）的用处之一是检查TCP套接字事件。这些TCP套接字用于传输WebSocket和浏览器使用的其他通信协议。
说明 在Google Chrome中，URL about：about重定向到chrome：//about。其他浏览器如Mozilla Firefox在它们的about：about页面中列出有用的URL。该页面显示下列内部Chrome实用工具：  
  chrome：//appcache-internals  
  chrome：//blob-internals  
  chrome：//bookmarks  
  chrome：//cache  
  chrome：//chrome-urls  
  chrome：//crashes  
  chrome：//credits  
  chrome：//dns  
  chrome：//downloads  
  chrome：//extensions  
  chrome：//flags  
  chrome：//flash  
  chrome：//gpu-internals  
  chrome：//history  
  chrome：//ipc  
  chrome：//inspect  
  chrome：//media-internals  
  chrome：//memory  
  chrome：//nacl  
  chrome：//net-internals  
  chrome：//view-http-cache  
  chrome：//newtab  
  chrome：//omnibox  
  chrome：//plugins  
  chrome：//policy  
  chrome：//predictors  
  chrome：//profiler  
  chrome：//quota-internals  
  chrome：//settings  
  chrome：//stats  
  chrome：//sync-internals  
  chrome：//terms  
  chrome：//tracing  
  chrome：//version  
  chrome：//print
+ Wireshark：用于分析网络协议流量的工具
### WebSocket服务器
虽然你可以让服务器接受WebSocket连接或者编写自己的Websocket服务器，但是有一些现有的实现能够让你在开发WebSocket应用程序时更加轻松。书中列出可用的WebSocket服务器（该列表由http：//refcardz.dzone.com/refcardz/html5-websocket提供）：
Alchemy-Websockets（.NET）：  
  http：//alchemywebsockets.net/Apache ActiveMQ：   
  http：//activemq.apache.org/apache-websocket（Apache模块）:  
  https：//github.com/disconnect/ apache-websocket#readmeAPE Project（C）:  
  http：//www.ape-project.org/Autobahn（虚拟用具）:  
  http：//autobahn.ws/Caucho Resin（Java）:  
  http：//www.caucho.com/Cowboy:  
  https：//github.com/extend/cowboyCramp（Ruby）:  
  http：//cramp.in/Diffusion（商业产品）http：//www.pushtechnology.com/homeEM-WebSocket（Ruby）:  
  https：//github.com/igrigorik/em-websocketExtendible WebSocket Server（PHP）:  
  http：//github.com/wkjagt/Extendible-Web-Socket-Servergevent-websocket（Python）:  
  http：//www.gelens.org/code/gevent-websocket/GlassFish（Java）:  
  https：//glassfish.java.net/Goliath（Ruby）:  
  https：//github.com/postrank-labs/goliathJetty（Java）:  
  http：//jetty.codehaus.org/jetty/jWebsocket（Java）:  
  http：//jwebsocket.org/Kaazing WebSocket Gateway（商业产品）:  
  http：//kaazing.com/libwebsockets（C）:  
  http：//git.warmcat.com/cgi-bin/cgit/libwebsockets/Misultin（Erlang）:  
  https：//github.com/ostinelli/misultinnet.websocket（Go）:  
  http：//code.google.com/p/go.net/websocketNetty（Java）:  
  http：//netty.io/Nugget（.NET）:  
  http：//nugget.codeplex.com/phpdaemon（PHP）:  
  http：//phpdaemon.net/Pusher（云服务）:  
  http：//pusher.com/pywebsockets（Python）:  
  http：//code.google.com/p/pywebsocket/RabbitMQ（Erlang）:  
  https：//github.com/videlalvaro/rabbitmq-websocketsSocket.io（Node.js）:  
  http：//socket.io/SockJS-node（Node）:  
  https：//github.com/sockjs/sockjs-node