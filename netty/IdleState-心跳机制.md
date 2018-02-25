### IdleState-心跳机制.md
    集群之间的tcp通信，为什么要使用这种通信，主实现读写，从实现读。因为它们不是实时一致性，都是最终一致性。保证通信之间是否还是活着的没有挂掉，可以通过发送心跳包来获得感知。

    服务端通过IdleStateHandler来检测客户端是否还存在
#### 示例
```
// ChannelInitializer部分代码实现
    @Override
    protected void initChannel(SocketChannel socketChannel) throws Exception {

        ChannelPipeline cp = socketChannel.pipeline();

        // 空闲状态检测
        cp.addLast(new IdleStateHandler(5,7,10, TimeUnit.SECONDS));
        cp.addLast(new NettyIdleServerHandler());

    }


//Handler类的部分代码如下：
    @Override
    protected void channelRead0(ChannelHandlerContext channelHandlerContext, String s) throws Exception {

        System.out.println("得到的内容是：" + s);
    }

    @Override
    public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {

        Channel channel = ctx.channel();

        if(evt instanceof IdleStateEvent){
            IdleStateEvent event = (IdleStateEvent) evt;

            String eventType = null;
            switch (event.state()){
                case READER_IDLE:
                    eventType = "读空闲";
                    break;
                case WRITER_IDLE:
                    eventType = "写空闲";
                    break;
                case ALL_IDLE:
                    eventType = "读写空闲";
                    break;
            }

            if (null == eventType){
                ctx.fireUserEventTriggered(evt);
            }else {

                System.out.println(channel.remoteAddress() + "超时事件：" + eventType);

                channel.close();
            }

        }

    }
```

    这时我们启动服务超过5秒发现：
    控制台打印出：/127.0.0.1:64961超时事件：读空闲
    如果客户端不断向服务器端发送消息时，就会出现 写空闲，
    一旦检测到空闲状态时，就会关闭channel如图：
![image](https://github.com/ilin0/study_node/raw/master/netty/image/idleState01.png)
![image](https://github.com/ilin0/study_node/raw/master/netty/image/idleState02.png)

    当出现写空闲时客户端的消息就不能发送到服务器端了，因为出现空闲时被服务器端关闭了连接。

