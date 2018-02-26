### helloworld.md
    idea 创建一个gradle 项目
    步骤：
    File -> new -> Project
    选择 gradle -java 项目

    groupId:com.gs
    artifactId:netty-helloworld
    version:1.0.0-snapshot

    选择本地的gradle ,前提是本也已经安装了gradle。

#### 示例
```java
    public class NettyService {
    public static void main(String[] args) throws Exception {

        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {

            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup, workerGroup).channel(NioServerSocketChannel.class).
                    childHandler(new NerryServerInitializer());

            ChannelFuture channelFuture = serverBootstrap.bind(8999).sync();

            channelFuture.channel().closeFuture().sync();
        }finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}

public class NerryServerInitializer extends ChannelInitializer<SocketChannel> {
    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline channelPipeline = ch.pipeline();

        channelPipeline.addLast("httpServerCodec", new HttpServerCodec());
        channelPipeline.addLast("nettyHttpServerHandler", new NettyHttpServerHandler());
    }
}

public class NettyHttpServerHandler extends SimpleChannelInboundHandler<HttpObject> {

    /**
     * 读取 客户端请求，和响应返回
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, HttpObject msg) throws Exception {

        System.out.println(msg.getClass());

        System.out.println(ctx.channel().remoteAddress());// 查看远程请求的地址和端口号

        // msg 即 DefaultHttpRequest 它是 httpReuqest 的子类
        // 这里没有这个if 会出现异常情况『java.io.IOException: Connection reset by peer』
        if(msg instanceof HttpRequest){

            ByteBuf content = Unpooled.copiedBuffer("hello world", CharsetUtil.UTF_8);

            // http 响应对象
            FullHttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK, content);

            // http 响应头信息
            response.headers().set(HttpHeaderNames.CONTENT_TYPE, "text/plain");
            response.headers().set(HttpHeaderNames.CONTENT_LENGTH, content.readableBytes());

            // 调用write 只会把内容放到缓冲区中，是不会返回客户端的，再调用flush 才会推到客户端
            ctx.writeAndFlush(response);
        }
    }
}
```

    启动 helloworld 项目
    curl 'http://localhost:8999' 回车

    这个项目只监听了8999 端口，它没有对url 路由做处理。


    如果采用浏览器访问，它会多一个请求，favicon.ico

    处理url 路由.增加以下代码
```java
    HttpRequest httpRequest = (HttpRequest)msg;
    URL url = new URL(httpRequest.uri());

    if("/favicon.ico".equals(url.getPath())){
        return;
    }
    // httpRequest.uri() 如果不是一个正确的url 会出错。 
```


    通过类的继承关系我们可以看出
![image](https://github.com/ilin0/study_node/raw/master/netty/image/20180225145410.png)

    在 ChannelInboundHandlerAdapter 类中有很多回调事件。
    重写以上方法
```java
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        System.out.println("handler added");
        super.handlerAdded(ctx);
    }

    @Override
    public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
        System.out.println("channel registered");
        super.channelRegistered(ctx);
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("channel active");
        super.channelActive(ctx);
    }

    // active 之后就会执行 channelRead0
    //下面二个可能 不会执行，当浏览器退出时就会执行 这是http 协议决定的，
    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("channel inactive");
        super.channelInactive(ctx);
    }

    @Override
    public void channelUnregistered(ChannelHandlerContext ctx) throws Exception {
        System.out.println("channel unregistered");
        super.channelUnregistered(ctx);
    }

```

    通过 curl 命令请求时，它是按顺序打印出以下信息

![image](https://github.com/ilin0/study_node/raw/master/netty/image/20180225150014.png)

    而使用浏览器请求时，打印的信息是不规则的 channelInactive 和 channelUnregistered 将不一定会出现。

![image](https://github.com/ilin0/study_node/raw/master/netty/image/20180225150053.png)

    当关闭浏览器时『channelInactive 和 channelUnregistered 』就出现了
![image](https://github.com/ilin0/study_node/raw/master/netty/image/20180225150203.png)

    为什么curl就会有呢，因为它是一个完整的url请求工具。而浏览它不是这样的，对于http来说它是一个基于请求和响应的形式，
    对于http1.1协议增加keepAlive时间，保持连接一断时间，时间到了没有新的请求，服务器端会主动关闭
    http 1.0 是短连接协议，客户端请求之后，服务端即关闭了连接

    如果想要手动关闭连接，根据协议的类型，keepAlive的时间服务器端主动关闭请求
```
    ctx.channel().close();
```
