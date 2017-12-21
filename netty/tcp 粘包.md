### tcp 粘包


### 示例
```
/**
* 客户端
*/
public class MyClientHandler extends SimpleChannelInboundHandler<ByteBuf> {

    private int count;

    @Override
    protected void channelRead0(ChannelHandlerContext channelHandlerContext, ByteBuf msg) throws Exception {


        byte[] buffer = new byte[msg.readableBytes()];
        msg.readBytes(buffer);

        String message = new String(buffer, Charset.forName("utf-8"));

        System.out.println("服务端接受的消息：" + message);
        System.out.println("服务端接受的消息数量：" + (++this.count));

    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
    
        // 连接上的时候发送10条消息
        for (int i = 0; i < 10; i++){
            ByteBuf buffer = Unpooled.copiedBuffer("send from client ", Charset.forName("utf-8"));

            ctx.writeAndFlush(buffer);
        }

    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}


public class MyServerHandler extends SimpleChannelInboundHandler<ByteBuf> {

    private int count;
    @Override
    protected void channelRead0(ChannelHandlerContext channelHandlerContext, ByteBuf msg) throws Exception {

        byte[] buffer = new byte[msg.readableBytes()];
        msg.readBytes(buffer);

        String message = new String(buffer, Charset.forName("utf-8"));

        System.out.println("服务端接受的消息：" + message);
        System.out.println("服务端接受的消息数量：" + (++this.count));

        ByteBuf responseByteBuf = Unpooled.copiedBuffer(UUID.randomUUID().toString(), Charset.forName("utf-8"));
        channelHandlerContext.writeAndFlush(responseByteBuf);
    }

    /**
     * 出现 异常时 关闭连接
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

#### 示例说明
    客户端连接上时 channelActive 发送10条消息 到 服务端
    服务端接受到时 channelRead0 显示消息内容及消息次数。
    这时客户端每次请求服务端的消息次数都不一样，这是由于tcp 消息字节流所决定的。
    ![日志打印：](https://github.com/ilin0/study_node/blob/master/netty/image/2017-12-20%2023.10.33.png)
    如图请示了三次，第一次只有一条消息，第二次有二条消息，第三次有八条消息。
