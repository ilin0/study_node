
### netty示例
```java
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {

            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup, workerGroup).channel(NioServerSocketChannel.class).
                    childHandler(new NerryHttpServerInitializer());

            ChannelFuture channelFuture = serverBootstrap.bind(8999).sync();

            channelFuture.channel().closeFuture().sync();
        }finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
```

##### ServerBootstrap group 方法

    AbstractBootstrap 成员 volatile EventLoopGroup group  就是  bossGroup = new NioEventLoopGroup();
```
    public B group(EventLoopGroup group) {
        if (group == null) {
            throw new NullPointerException("group");
        }
        if (this.group != null) {
            throw new IllegalStateException("group set already");
        }
        this.group = group;
        return (B) this;
    }
```

##### ServerBootstrap channel 方法
    也是对ReflectiveChannelFactory 的成员private final Class<? extends T> clazz;赋值，而ReflectiveChannelFactory最后被赋给了AbstractBootstrap.channelFactory

##### ServerBootstrap bind 方法   
    创建一个新的channel 然后绑定它（端口）

    1、初始化 注册initAndRegister
            channel = channelFactory.newChannel();
            init(channel);
        这里channel就是NioServerSocketChannel.class对象，反射获取到它的对象。
        而channelFactory就是ReflectiveChannelFactory

##### NioServerSocketChannel的创建
    通过反射调用它的无参的构造方法创建。
```java
    private static final SelectorProvider DEFAULT_SELECTOR_PROVIDER = SelectorProvider.provider();

    /**
     * Create a new instance
     */
    public NioServerSocketChannel() {
        this(newSocket(DEFAULT_SELECTOR_PROVIDER));
    }

    // 最终是调用
            /**
             *  Use the {@link SelectorProvider} to open {@link SocketChannel} and so remove condition in
             *  {@link SelectorProvider#provider()} which is called by each ServerSocketChannel.open() otherwise.
             *
             *  See <a href="https://github.com/netty/netty/issues/2308">#2308</a>.
             */
            return provider.openServerSocketChannel();
            //1、同步代码块在什么地方，
            //2、为什么每个ServerChannel 创建SelectorProvider都会被调用
```







