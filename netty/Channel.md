### Channel.md
    doc 它是一个网络连接socket,一个执行I/O操作的组件。如bind、connect、read、write.
    channel可以提供用户
        1.当前channel的状态，是read状态,write状态等
        2.channel的配置参数
        3.I/O操作的支持
        4.提供了channelPipeline可以处理与当前Channel的所有的I/O事件相关的请求

```java 
    //接口提供的相关方法，针对以上4点的示例方法

    /**
     * Returns {@code true} if the {@link Channel} is open and may get active later
     */
    boolean isOpen();--isRegistered、isActive
        /**
     * Returns the configuration of this channel.
     */
    ChannelConfig config();

    @Override
    Channel read();

/**
     * Return the assigned {@link ChannelPipeline}.
     */
    ChannelPipeline pipeline();
```

#### 所有的I/O操作都是异步的
    任何的io调用都会立即返回，不能保证在调用结束后是完成的（因为异步）。相关，会得到一个ChannelFuture实例，这个实例都会通知给你结果，如：请求失败、成功、取消

#### Channel是可继承的
    Channel可以有一个父亲的，取决于它是如何创建的。
    比如说：SocketChannel->由ServerSocketChannel创建，那么SocketChannel.parent()就会返回ServerSocketChannel。
    这种层次结构的语义是属于Channel传输实现来决定的。 例如，你可以编写一个新的Channel实现与子Channel共享连接。
```java
    /**
     * Returns the parent of this channel.
     *
     * @return the parent channel.
     *         {@code null} if this channel does not have a parent channel.
     */
    Channel parent();
```

#### Channel的创建
    这是由channel = channelFactory.newChannel();来创建
    这个channelFactory就是我们定义的NioServerSocketChannel.class
```java
    serverBootstrap.group(bossGroup, workerGroup).channel(NioServerSocketChannel.class).
                    childHandler(new NerryHttpServerInitializer());
```

#### ChannelPipeline的创建
    这是由NioServerSocketChannel创建的时候，初始化它父类AbstractChannel的时候在构造方法里对成员变量的创建。
    而在创建ChannelPipeline时，将当前的Channel作为它的构造方法参数赋值给ChannelPipeline的成员变量channel。所以它们之间就互为包含关系。你中有我，我中有你。
    
