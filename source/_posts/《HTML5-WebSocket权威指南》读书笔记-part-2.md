---
title: 《HTML5 WebSocket权威指南》读书笔记 part 2
date: 2020-01-31 09:25:40
tags: 
- websocket
categories: 
- web 
- booknotes
keywords: 
- websocket
excerpt: 《HTML5 WebSocket权威指南》读书笔记第二部分，主要是为什么要用websocket例子的搭建，最主要都是代码验证了构建远程JS控制台
---

# 用JS和Node.js编写WebSocket服务器API
用Node构建简单的WebSocket服务器，主要是用JS和Node.js编写WebSocket服务器API，最主要是实现一个供服务器使用的WebSocket的对象，而这个对象因为是在服务器使用的，所以需要提供的：  
1.构造函数完成握手过程，提供listen()方法构建connection成功  
2.外部方法主要是通过connection的send()和close()  
3.内部方法有handleFrame, processBuffer, doSend分别用于处理数据帧并发送 。
查了javascript中的WebSocket对象--**Provides the API for creating and managing a WebSocket connection to a server**, as well as for sending and receiving data on the connection.  

# 用的上述新服务器API构建一个回显服务器
listen端口，并在callback函数中完成对data和close事件的处理函数的定义和注册。在data事件中调用提供的WebSocket的API中的send方法进行数据回显。

# 应用：构建远程Javascript控制台  
控制台也称为REPL(Read Eval Print Loop)利用Node.js的repl模块，添加一个自定义的eval()函数。添加WebSocket，可以通过互联网远程控制一个Web应用程序。使用这个WebSocket驱动的控制台，我们将能从一个命令行接口远程求取表达式的值。更好的是，我们可以输入一个表达式，并看到每个并发连接的客户端求得的表达式值。

觉得这个例子很妙，但是书上的server code太多了，现在node有现成的websocket的模块ws，可以用于构建websocket server。改写书中的code，
WebSocket服务器的code：
```
const WebSocket = require('ws');
var repl = require("repl");
var connections = Object.create(null);
var remoteMultiEval = function(cmd, context, filename, callback) {
    for (var c in connections) {
        console.log("\t connection:" + c + " ready to send:\t" );
        console.log("\t connection is:" + connections[c] + "\t" );
        connections[c].send(cmd);
    }
    callback(null, "(result pending)");
};

const wssvr = new WebSocket.Server({host:"localhost", port: 9999 });
wssvr.on('connection', function connection(connsock, request) {
    connsock.id = Math.random().toString().substr(2);
    connections[connsock.id] = connsock;
    console.log("new connection: " + connsock.id);    
    connsock.on('message', function incoming(message) {
      console.log("\t" + connsock.id + ":\t" + message);
    });
    connsock.on("close", function() {
        //remove connection
        delete connections[connsock.id];
    });
});
 
repl.start({"eval": remoteMultiEval});
```

websocket客户端的code：
```
<!DOCTYPE html>
<titile>WebSocket REPL Client</titile>
<meta charset="utf-8">
<h2>Websocket REPL Client</h2>
<script>
var url = "ws://localhost:9999/repl";
var ws = new WebSocket(url);
ws.onmessage = function(e) {
    console.log("command", e.data);
    try{
        var result = eval(e.data);
        ws.send(result.toString());
    } catch (err) {
        ws.send(err.toString());
    }
}
</script>
```
在server端launch服务器，然后enter 命令“navigator.userAgent”，便可以看到client端的运行后回传回来的结果，如下
```
wssample> node .\websocket-repl-svr-ws.js
> new connection: 5649403465277831
> navigator.userAgent
         connection:5649403465277831 ready to send:
         connection is:[object Object]
'(result pending)'
>       5649403465277831:       Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:72.0) Gecko/20100101 Firefox/72.0
```
而在browser client端的console，output则是![browser结果](https://i.niupic.com/images/2020/01/31/6mca.PNG)
