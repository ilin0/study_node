### websocket.md
    由来
    它是一种规范，html5的一部分，解决http本身存在的一种不足。
    http它是一个无状态的、基于请求和响应的。
    http它的二次请求，它不能确定是否是同一个客户端，cookie一并代入到服务端session获取到相应的信息。
    http1.0协议  服务器数据响应之后，连接就断了，这时如果再要请求时，还需要重新建立连接
    对于http1.1协议增加keepAlive时间，保持连接一断时间，时间到了没有新的请求，服务器端会主动关闭，如果在这段时间内是不需要重新建立新的连接。

    对于网页聊天是无法实现的。
    无法实时消息到客户端，它只能在下一次轮询时才会发送消息。
    http请求本身包含了头信息。头信息占据的大小会超过消息内容本身，得不偿失的方式。

    而长连接只需要在初次连接时需要发送头信息，而在后续使用时就不需要发送头消息，只要发送我们业务关心的数据本身，webSocket是http的一种升级形式。

    tomcat、spring也支持 
    ios facebook websocket库、原生js

    HttpObjectAggregator它是将http分段的请求合成一个完成的http请求。

#### 示例
```
    // ChannelInitializer部分代码
    @Override
    protected void initChannel(SocketChannel socketChannel) throws Exception {

        ChannelPipeline cp = socketChannel.pipeline();

        cp.addLast(new HttpServerCodec());
        cp.addLast(new ChunkedWriteHandler());// 以块的方式写
        cp.addLast(new HttpObjectAggregator(8192));// 对http请求的聚合操作
        cp.addLast(new WebSocketServerProtocolHandler("/ws"));

        cp.addLast(new NettyWebSocketTextFrameHandler());
    }


public class NettyWebSocketTextFrameHandler extends SimpleChannelInboundHandler<TextWebSocketFrame> {

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, TextWebSocketFrame msg) throws Exception {
        System.out.println("收到消息：" + msg.text());

        ctx.channel().writeAndFlush(new TextWebSocketFrame("服务器时间" + LocalDateTime.now() + msg.text()));
    }

    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        System.out.println("handlerAdded: " + ctx.channel().id().asLongText());
    }

    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
        System.out.println("handlerRemoved: " + ctx.channel().id().asLongText());
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        System.out.println("发生异常");
        ctx.channel().close();
    }
}
```
#### 客户端示例
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>websocket客户端</title>
</head>
<body>
<script type="text/javascript">

    var socket;
    if(window.WebSocket){

        socket = new WebSocket("ws://localhost:8899/ws");
        socket.onmessage = function (event) {
            var ta = document.getElementById("responseText");
            ta.value = ta.value + "\n" + event.data;
        }
        socket.onopen = function (p1) {
            var ta = document.getElementById("responseText");
            ta.value = ta.value + "\n" + "连接开启";
        }
        socket.close = function (p1) {
            var ta = document.getElementById("responseText");
            ta.value = ta.value + "\n" + "连接断开";
        }
    }else{
        alert("浏览器不支持Websocket");
    }

    function send(message) {
        if(!window.WebSocket){
            return;
        }
        if(socket.readyState == WebSocket.OPEN){
            socket.send(message);
        }else {
            alert("连接尚未开启");
        }
    }
</script>
<form onsubmit="return false;">
    <textarea name="message" style="width: 400px; height: 50px;"></textarea>
    <input type="button" value="发送数据" onclick="send(this.form.message.value)">

    <h3>服务端输出：</h3>
    <textarea id="responseText" style="width: 400px; height: 300px;"></textarea>
    <input type="button" value="清空内容" onclick="javascript:document.getElementById('responseText').value=''">
</form>
</body>
</html>
```

    http协议是以http开头的，而websocket是:ws://server:port/context_path如：
    ws:localhost:8080/ws

    我们通过下图可以看到Upgrade:websocket表示请求以升级为websocket长连接了。
![image](https://github.com/ilin0/study_node/raw/master/netty/image/websocket01.png)
