### 08-selector.md
    selector主要涉及到网络，通过一个完整的例子（包含：selectot,channel,buffer）这个例子可以明显的感觉到于传统网络编程的差别。

    关于传递的socket网络编辑的流程与概念

#### 服务器端：
```
ServerSocket serverSocket = ......; 生成一个服务端Socket
serverSocket.bind(8899); 绑定端口,服务端监听着这个端口，当连接建立好之后，返回socket会选择当前系统中任意一个未被占用的端口（随机分配）与客户端通信

while(true){   循环获取

    Socket socket = serverSocket.accept();  //阻塞方法，返回一个客户端的socket

    new Thread(socket);  启一个线程，将对象传入

    run(){
       执行相应的业务
       socket.getInpuStream();
    }
}
```

#### 客户端
```
Socket socket = new Socket('localhost', 8899);
socket.connect();
```

    8899这个端口只是用于监听链接的，它会选择一个未被占用的端口与客户端交互。

    对于服务端的处理，当一个客户端与之链接之后，服务端都会启一个线程来处理，当和客户端多那么线程数就会很多，数量大这种编程模型就不恰当了。因为线程的切换会占用资源等情况。

    nio,它在一定程度上解决了传统io编程的一些问题，它并不是一个链接就会有一个线程，它是一个线程处理多个客户端。

    任何的异常编程都离不开event事件，nio它也是基于事件来完成的。

#### selector doc
    通过 open 方法创建，它是由系统默认一个 SelectorProvider  提供者创建,也可以提供 自定义的方式来创建。
    一个可选择的channel通道注册到selector上,它注册的关系是通过Selectiokey标识的。一个选择器会维护三种selectorkey的集合。

    key-set 包含了所有的key事件类型。当前注册到channel的集合
    selected-key 只包含了部分key的集合。
    cancellen-key 只包含了取消的key的集合。
    这三个集合在新创建的Selector中都是空的

    SelectableChannel#register方法注册Channel当中，会产生一个副作用，就是会增加一个key到selector's key 的集合中。
    Cancelled keys会被key 集合中被移除『是Selection 的操作中』 。key集合本身是不能被直接修改

    一个key当它被取消的时候，这个key会被添加到selector's cancelled-key集合中。
    有二种方式：关闭channel或调用SelectionKey#cancel方法。
    取消key会导致key所关联的Channel在下一次选择操作时被取消注册，这时key就会在集合中被移除

    选择操作时key会被添加到集合中，key不可能被直接添加到集合中的。

##### Concurrency 并发
    selected-key sets 并发线程去访问是不安全的。


##### 示例
```java
/**
 * 使用 nc localhost 5000
 */
public class NioNetWorkTest {

    public static void main(String[] args) throws Exception {
        int[] ports = new int[5];

        ports[0] = 5000;
        ports[1] = 5001;
        ports[2] = 5002;
        ports[3] = 5003;
        ports[4] = 5004;

        // 构造 selector ,使用open 方法
        Selector selector = Selector.open();

        for (int i = 0; i < ports.length; i++){
            ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

            serverSocketChannel.configureBlocking(false);// 将方法设置为不是阻塞的
            ServerSocket serverSocket = serverSocketChannel.socket();

            InetSocketAddress address = new InetSocketAddress(ports[i]);
            serverSocket.bind(address);

            // 注册
            serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
            System.out.println("监听端口：" + ports[i]);
        }

        while(true){
            int number = selector.select();
            System.out.println("number: " + number);

            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            System.out.println("selectionKeys: " + selectionKeys);

            Iterator<SelectionKey> iter = selectionKeys.iterator();

            while (iter.hasNext()){
                SelectionKey selectionKey = iter.next();

                if (selectionKey.isAcceptable()){
                    ServerSocketChannel ssc = (ServerSocketChannel)selectionKey.channel();

                    SocketChannel channel = ssc.accept();

                    channel.configureBlocking(false);
                    channel.register(selector, SelectionKey.OP_READ);

                    iter.remove();// 事件消费掉了，就一定要移除掉

                    System.out.println("获得客户端连接： " + channel);
                }else if (selectionKey.isReadable()){
                    SocketChannel channel = (SocketChannel)selectionKey.channel();

                    int bytesRead = 0;
                    while (true){
                        ByteBuffer byteBuffer = ByteBuffer.allocate(512);

                        byteBuffer.clear();
                        int read = channel.read(byteBuffer);

                        if (read <= 0){
                            break;
                        }
                        byteBuffer.flip();
                        channel.write(byteBuffer);

                        bytesRead += read;

                    }

                    System.out.println("读取： " + bytesRead + ", 来自于 " + channel);

                    iter.remove();
                }
            }
        }
    }
}

```


##### 示例2
```java
// 服务端
/**
 *
 * 启动服务   nc localhost 8899 调试 启动多个客户端
 *
 * 这里并没有处理 客户端关闭后的逻辑，比如clientMap 里会 存在已关闭的客户端连接
 *
 *
 *
 */
public class NioServer {

    // 保存 连接的 socket
    private static Map<String, SocketChannel> clientMap = new HashMap<>();

    public static void main(String[] args) throws Exception {

        // 它是绑定服务端的 端口
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.configureBlocking(false);// 设置 为 非阻塞的

        ServerSocket serverSocket = serverSocketChannel.socket();// 获取 服务端 socket对象

        serverSocket.bind(new InetSocketAddress(8899));//  以上4 步是创建 socket 标准的步骤

        Selector selector = Selector.open();
        // 把channel注册到selector上
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        while (true){
            try {
                selector.select();// 阻塞的方法

                // 当上面这个方法返回时，我们就能通过下面这行获取它所有注册事件的key
                Set<SelectionKey> selectionKeys = selector.selectedKeys();

                // 遍历它
                selectionKeys.forEach(selectionKey -> {

                    final SocketChannel client;
                    try {

                        // 客户端向服务器端 发起的连接， 当前 的对象就是发起连接的对象 channel
                        if (selectionKey.isAcceptable()){
                            ServerSocketChannel channel = (ServerSocketChannel) selectionKey.channel();

                            client = channel.accept();// 这行表示服务器端真正的接受客户端的连接通信
                            client.configureBlocking(false);
                            client.register(selector, SelectionKey.OP_READ);// 再把新的cHannel 注册到selector上

                            String key = "【" + UUID.randomUUID().toString() + "】";
                            clientMap.put(key, client);
                        }else if (selectionKey.isReadable()){
                            client = (SocketChannel) selectionKey.channel();

                            ByteBuffer readBuffer = ByteBuffer.allocate(1024);

                            // 这里是将客户端的数据读到Buffer中
                            int count = client.read(readBuffer);
                            if (count > 0){
                                readBuffer.flip();// 如果大于0表示有数据，需要写到其它客户端
                                Charset charset = Charset.forName("utf-8");
                                String receviedMsg = String.valueOf(charset.decode(readBuffer).array());

                                System.out.println(client + ": " + receviedMsg);

                                for (Map.Entry<String, SocketChannel> entry : clientMap.entrySet()){
                                    SocketChannel v = entry.getValue();

                                    ByteBuffer writeBuffer = ByteBuffer.allocate(1024);

                                    writeBuffer.put(("发送的内容：" + receviedMsg).getBytes());
                                    writeBuffer.flip();

                                    v.write(writeBuffer);
                                }
                            }
                        }
                    }catch (Exception e1){

                        e1.printStackTrace();
                    }
                });

                selectionKeys.clear();//清空集合，没有清空就会出现npe
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
}


// 客户端
public class NioClient {

    public static void main(String[] args) {
        try {
            SocketChannel socketChannel = SocketChannel.open();
            socketChannel.configureBlocking(false);

            Selector selector = Selector.open();
            socketChannel.register(selector, SelectionKey.OP_CONNECT);
            socketChannel.connect(new InetSocketAddress("127.0.0.1", 8899));

            while (true){
                selector.select();

                Set<SelectionKey> keySet = selector.selectedKeys();

                for (SelectionKey selectionKey : keySet){
                    if (selectionKey.isConnectable()){
                        SocketChannel client = (SocketChannel)selectionKey.channel();

                        // 判断客户的连接是否是进行中的状态
                        if (client.isConnectionPending()){
                            client.finishConnect();//如果是，完成这连接，建立好连接

                            ByteBuffer writerBuffer = ByteBuffer.allocate(1024);

                            writerBuffer.put((LocalDateTime.now() + "连接成功").getBytes());

                            writerBuffer.flip();
                            client.write(writerBuffer);// 把数据写出去。

                            // 线程 监听 输入
                            ExecutorService executorService = Executors.newSingleThreadExecutor(Executors.defaultThreadFactory());

                            executorService.submit(() -> {
                                while (true){
                                    try {
                                        writerBuffer.clear();

                                        InputStreamReader input = new InputStreamReader(System.in);
                                        BufferedReader br = new BufferedReader(input);

                                        String sendMsg = br.readLine();

                                        writerBuffer.put(sendMsg.getBytes());
                                        writerBuffer.flip();
                                        client.write(writerBuffer);

                                    }catch (Exception e){
                                        e.printStackTrace();
                                    }
                                }
                            });
                        }

                        client.register(selector, SelectionKey.OP_READ);
                    } else if (selectionKey.isReadable()) {
                        SocketChannel client = (SocketChannel)selectionKey.channel();

                        ByteBuffer readBuffer = ByteBuffer.allocate(1024);

                        int count = client.read(readBuffer);

                        if (count > 0){
                            String reveivedMsg = new String(readBuffer.array(), 0, count);
                            System.out.println(reveivedMsg);
                        }
                    }
                }

                keySet.clear();
            }




        }catch (Exception e){
            e.printStackTrace();
        }
    }
}

```