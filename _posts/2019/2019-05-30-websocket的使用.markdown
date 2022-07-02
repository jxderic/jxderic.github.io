# WebSocket的使用

## 为什么要用WebSocket

项目中经常会有需要实时更新数据的需求，如果采用轮询接口的形式，一是达不到实时更新数据，二是浪费性能，毕竟轮询的时候不一定是数据更新的时候。WebSocket协议主要就是实现这种需求，它只有数据更新的时候才会推送数据，而且也可以客户端主动发送数据，实现了双向传输。在这边还是简单对比一下HTTP和WebSocket，能够更加清晰的了解其作用。

HTTP协议可以总结几个特点：

* 一次性的、无状态的短连接：客户端发起请求、服务端响应、结束。
* 被动性响应：只有当客户端请求时才被执行，给予响应，不能主动向客户端发起响应。
* 信息安全性：得在服务器添加 SSL 证书，访问时用 HTTPS。
* 跨域：服务器默认不支持跨域，可在服务端设置支持跨域的代码或对应的配置。

TCP协议可以总结几个特点：

* 有状态的长连接：客户端发起连接请求，服务端响应并建立连接，连接会一直保持直到一方主动断开。
* 主动性：建立起与客户端的连接后，服务端可主动向客户端发起调用。
* 信息安全性：同样可以使用 SSL 证书进行信息加密，访问时用 WSS 。
* 跨域：默认支持跨域。

如果在前端我们可以把 AJAX 请求当作一个 HTTP 协议的实现，那么，WebSocket 就是 TCP 协议的一种实现。

## 客户端的使用

WebSocket客户端的使用相当简单。

```javascript
var ws = new WebSocket("ws://localhost:8080");

ws.onopen = function(evt) { 
  console.log("Connection open ..."); 
  ws.send("Hello WebSockets!");
};

ws.onmessage = function(evt) {
  console.log( "Received Message: " + evt.data);
  ws.close();
};

ws.onclose = function(evt) {
  console.log("Connection closed.");
};      
```



## 服务端的实现

### WebSocketd

WebSocketd是一个很有用的，很小的命令行工具，能够通过各种语言（Bash, Python, Ruby, Perl, Bash, .NET, C, Go, PHP, Java等等）来实现WebSocket服务器的功能。这里就使用一段很简单的bash脚本来实现。

**count.sh**:

```bash
for ((COUNT = 1; COUNT <= 10; COUNT++)); do
  echo $COUNT
  sleep 1
done
```

通过WebSocketd使其转变成WebSocket服务：

```
$ websocketd --port=8080 ./count.sh
```

客户端接收数据

```html
<!DOCTYPE html>
<pre id="log"></pre>
<script>
  // helper function: log message to screen
  function log(msg) {
    document.getElementById('log').textContent += msg + '\n';
  }

  // setup websocket with callbacks
  var ws = new WebSocket('ws://localhost:8080/');
  ws.onopen = function() {
    log('CONNECT');
  };
  ws.onclose = function() {
    log('DISCONNECT');
  };
  ws.onmessage = function(event) {
    log('MESSAGE: ' + event.data);
  };
</script>
```

### Nodejs

* 安装第三方模块 ws:  `npm install ws`
* 开启一个WebSocket的服务器，端口为8080

```javascript
var socketServer = require('ws').Server;
var wss = new socketServer({
    port: 8080
});
```

- 用 on 来进行事件监听
- connection：连接监听，当客户端连接到服务端时触发该事件
- close：连接断开监听，当客户端断开与服务器的连接时触发
- message：消息接受监听，当客户端向服务端发送信息时触发该事件
- send: 向客户端推送信息

```javascript
wss.on('connection', function (client) {
    client.on('message', function (_message) {
        var _messageObj = JSON.parse(_message);
        //status = 1 表示正常聊天
        _messageObj.status = 1;
        this.message = _messageObj;
        //把客户端的消息广播给所有在线的用户
        wss.broadcast(_messageObj);
    });

    // 退出聊天  
    client.on('close', function() {  
        try{
            this.message = this.message || {};
            // status = 0 表示退出聊天
            this.message.status = 0;
            //把客户端的消息广播给所有在线的用户
            wss.broadcast(this.message);  
        }catch(e){  
            console.log('刷新页面了');  
        }  
    });  
});

//定义广播方法
wss.broadcast = function broadcast(_messageObj) {  
    wss.clients.forEach(function(client) { 
        client.send(JSON.stringify(_messageObj))
    });  
}; 
```



## 心跳的实现

在前后端保持长连接的过程中，有可以因为一些原因（断网、后台服务异常等）会造成连接断开，这时不会触发WebSocket的任何事件，前端也就无法得知当前连接是否已经断开，因此需要实现心跳功能，重新连接。

一般情况如果希望WebSocket连接一直保持，我们只需要在close或者error上绑定重新连接方法。

```javascript
ws.onclose = function () {
    reconnect();
};
ws.onerror = function () {
    reconnect();
};
```

那么针对断网情况的心跳重连，怎么实现呢，我们只需要定时的发送信息，去触发send方法，如果网络断开，浏览器便会触发onclose。简单的实现：

```javascript
var heartCheck = {
    timeout: 60000,//60ms
    timeoutObj: null,
    reset: function(){
        clearTimeout(this.timeoutObj);
　　　　 this.start();
    },
    start: function(){
        this.timeoutObj = setTimeout(function(){
            ws.send("HeartBeat");
        }, this.timeout)
    }
}

ws.onopen = function () {
   heartCheck.start();
};
ws.onmessage = function (event) {
    heartCheck.reset();
}
```

定时器的时间可以根据项目情况自行设置，该思路同样可以使用于后台没有推送数据，多久之后需要前端执行一些业务操作的需求，桐庐园区透视的人员管理就通过这种思路实现后台没有数据推送5s之后隐藏人员信息弹窗。

后端的异常情况导致连接中断，前端是不会感应到，需要前端发送心跳一定时间后，后端既没有返回心跳响应信息，也没收到任何其他消息的话，我们就可以断定后端发生异常断开了。重新改造一下代码：

```javascript
var heartCheck = {
    timeout: 60000,//60ms
    timeoutObj: null,
    serverTimeoutObj: null,
    reset: function(){
        clearTimeout(this.timeoutObj);
        clearTimeout(this.serverTimeoutObj);
　　　　 this.start();
    },
    start: function(){
        var self = this;
        this.timeoutObj = setTimeout(function(){
            ws.send("HeartBeat");
            self.serverTimeoutObj = setTimeout(function(){
                ws.close();//如果onclose会执行reconnect，我们执行ws.close()就行了.如果直接执行reconnect 会触发onclose导致重连两次
            }, self.timeout)
        }, this.timeout)
    },
}

ws.onopen = function () {
   heartCheck.start();
};
ws.onmessage = function (event) {
    heartCheck.reset();
}
ws.onclose = function () {
    reconnect();
};
ws.onerror = function () {
    reconnect();
};
```



## 项目中使用的注意事项

1. 项目中有很多地方需要实时更新数据，可以要求后端合并websocket，通过返回消息的字段来区分各个模块的推送数据。
2. 看板小部件的实时推送数据要求业务后台推送给门户的可视化框架前端，之后由可视化框架前端通过一个统一的WS链接推送，之后根据事件名称分发。事件名称规则为组件标识：消息特征码。具体可以查看[待办消息组件业务接口]([https://git.hikvision.com.cn/users/zhangzhitao/repos/wiportal/browse/tlnc%20%E5%BE%85%E5%8A%9E%E6%B6%88%E6%81%AF%E7%BB%84%E4%BB%B6%E4%B8%9A%E5%8A%A1%E6%8E%A5%E5%8F%A3.md](https://git.hikvision.com.cn/users/zhangzhitao/repos/wiportal/browse/tlnc 待办消息组件业务接口.md))



