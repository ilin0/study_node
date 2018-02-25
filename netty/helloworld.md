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
```
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
```
    HttpRequest httpRequest = (HttpRequest)msg;
    URL url = new URL(httpRequest.uri());

    if("/favicon.ico".equals(url.getPath())){
        return;
    }
    // httpRequest.uri() 如果不是一个正确的url 会出错。 
```


    通过类的继承关系我们可以看出
    ![image](https://github.com/ilin0/study_node/raw/master/netty/image/2018.png)


