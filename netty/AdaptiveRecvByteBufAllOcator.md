### AdaptiveRecvByteBufAllOcator 可适配的接受
    它是由NioSocketChannel创建时的config对象。
```java
    public NioSocketChannel(Channel parent, SocketChannel socket) {
        super(parent, socket);
        config = new NioSocketChannelConfig(this, socket.socket());
    }

```

    根据反馈自动的增加或减少可预测的Buf 大小，（与channel关联的ButeBuf）
    它会优雅的增加可期望的字节数，上一次读取的字节数完全填满了时，
    如果连续二次没有填充满时，就可适当的减小

    默认的构造方法是从1024开始，但最小不低于64，最大不会超过 65536（字节）。
```java

// 按从小到大的顺序，分配缓冲区的大小取决于数组里的值，这就是整型数组的含义
private static final int[] SIZE_TABLE;

    static {
        List<Integer> sizeTable = new ArrayList<Integer>();
        for (int i = 16; i < 512; i += 16) {
            sizeTable.add(i);
        }

        for (int i = 512; i > 0; i <<= 1) {// 向左位移1位即*2，一直循环就溢出就小于0循环结束
            sizeTable.add(i);
        }

        SIZE_TABLE = new int[sizeTable.size()];
        for (int i = 0; i < SIZE_TABLE.length; i ++) {
            SIZE_TABLE[i] = sizeTable.get(i);
        }
    }
```

#### 如何申请缓存区？
    它是由HandleImpl私有内部类，它父类里有个这样的方法
```java
        public ByteBuf allocate(ByteBufAllocator alloc) {
            return alloc.ioBuffer(guess());
            //guess()方法就是每次记录的字节数大小，用于猜测下一次的缓冲区大小。
        }

        --AbstractByteBufAllocator 转到这个类
    public ByteBuf ioBuffer(int initialCapacity) {
        if (PlatformDependent.hasUnsafe()) {
            return directBuffer(initialCapacity);// 直接缓冲区（堆外缓冲区）
        }
        return heapBuffer(initialCapacity);// 堆内缓冲区

        而堆外缓冲区最终使用的是Nio的ByteBuffer
        而堆内缓冲就是字节数组
    }
```

    if (PlatformDependent.hasUnsafe())这里的判断主要是否包含『sum.misc.Unsafe』这是由操作系统决定的。

```java
    private static boolean hasUnsafe0() {
        if (isAndroid()) { //判断是不是android系统，如果是就不使用堆外内存 
            logger.debug("sun.misc.Unsafe: unavailable (Android)");
            return false;
        }

        // 是不是明确表示不使用堆外内存，即判断是否有『io.netty.noUnsafe』配置项
        if (PlatformDependent0.isExplicitNoUnsafe()) {
            return false;
        }

        try {
            boolean hasUnsafe = PlatformDependent0.hasUnsafe();
            logger.debug("sun.misc.Unsafe: {}", hasUnsafe ? "available" : "unavailable");
            return hasUnsafe;
        } catch (Throwable ignored) {
            // Probably failed to initialize PlatformDependent0.
            return false;
        }
    }
```