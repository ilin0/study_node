### reactor.md
1. network service
    对于大多数网络服务都具有以下几步
    Read request
    Decode request 请求过来是二进制数据，需要解析成相应的内容。
    Process service 处理业务逻辑
    Encode reply 
    Send reply
    它们每一步的处理都是不一样的，成本也不一样
2. 传统的设计如图：
![image](https://github.com/ilin0/study_node/raw/master/netty/image/reactor2018031201.png)
    它的每个处理器（handler）都在自己的线程里处理。
3. 传统的service socket loop
    对于下面的示例会带来一些问题，就是线程数量太多，
```java
class Server implements Runnable {
  public void run() {
    try {
      ServerSocket ss = new ServerSocket(PORT);
      while (!Thread.interrupted())
        new Thread(new Handler(ss.accept())).start();
      // or, single-threaded, or a thread pool
    } catch (IOException ex) { /* ... */ }
  }
  static class Handler implements Runnable {
    final Socket socket;
    Handler(Socket s) { socket = s; }
    public void run() {
    try {
        byte[] input = new byte[MAX_INPUT];
        socket.getInputStream().read(input);
        byte[] output = process(input);
        socket.getOutputStream().write(output);
      } catch (IOException ex) { /* ... */ }
    }
    private byte[] process(byte[] cmd) { /* ... */ }
  }
}
```

4. 分解和攻克上面的问题
    * 将处理分解成多个小任务，每个任务都在不阻塞的情况下完成一个动作。
    * 但任务在可用的情况下，再去执行它（每个小任务都会有一个状态，比如但它在i/o操作时，cpu就不用等待它。）i/o事件是作为一种触发器来实现的。

5. 事件驱动设计
    通常它会比其它的方式效率会更高
    * 更少的资源，并不需要对每一个客户端产生一个线程
    * 成本低，因为线程少，锁资源也少，线程之间的切换也少
    * 分发会更高，需要手动的将事件绑定到动作上。
    * 编程会更高，
6. reactor pattern
    它通常运用在网络应用上。
    * 它是通过恰当的事件来处理i/o响应。
    * Handler是执行非阻塞的动作
    * 通过将handler绑定到事件的方式来管理（类似于awt的点击事件，当事件发生时，处理器就会得到调用）
7. 基础版本的reactor设计
![image](https://github.com/ilin0/study_node/raw/master/netty/image/reactor2018031202.png)
* step 1
```java
//  reactor 1: setup 
class Reactor implements Runnable {
    final Selector selector;
    final ServerSocketChannel serverSocket;
    Reactor(int port) throws IOException {
        selector = Selector.open();
        serverSocket = ServerSocketChannel.open(); 
        serverSocket.socket().bind(new InetSocketAddress(port)); 
        serverSocket.configureBlocking(false);
        SelectionKey sk =
          serverSocket.register(selector,SelectionKey.OP_ACCEPT);
        sk.attach(new Acceptor());
    }
/*
Alternatively, use explicit SPI provider: SelectorProvider p = SelectorProvider.provider(); selector = p.openSelector();
serverSocket = p.openServerSocketChannel();
 */
 // reactor 2 : dispatch loop
 // class Reactor continued
    public void run() {  // normally in a new Thread
        try {
            while (!Thread.interrupted()) {
                selector.select();
                Set selected = selector.selectedKeys(); 
                Iterator it = selected.iterator(); 
                while (it.hasNext()){
                    //这里只做任务的派发
                    dispatch((SelectionKey)(it.next()); 
                }
                selected.clear(); 
            }
        } catch (IOException ex) { /* ... */ }
    }
    void dispatch(SelectionKey k) {
        // 这里的attachment就是ACCEPT事件上附加的处理任务
        Runnable r = (Runnable)(k.attachment()); 
        if (r != null){
            r.run();
        }
    }
    // reactor 3 : acceptor
    // class Reactor continued
    class Acceptor implements Runnable { // inner
        public void run() {
          try {
            SocketChannel c = serverSocket.accept();
            if (c != null)
              new Handler(selector, c);
          }
          catch(IOException ex) { /* ... */ }
        }
    } 
}

//Reactor 4: Handler setup
final class Handler implements Runnable {
    final SocketChannel socket;
    final SelectionKey sk;
    ByteBuffer input = ByteBuffer.allocate(MAXIN); 
    ByteBuffer output = ByteBuffer.allocate(MAXOUT); 
    static final int READING = 0, SENDING = 1;
    int state = READING;
    Handler(Selector sel, SocketChannel c) throws IOException {
        socket = c; c.configureBlocking(false);
        // Optionally try first read now
        sk = socket.register(sel, 0); 
        sk.attach(this); 

        // 这一步对读事件感兴趣，把上面register 0的事件覆盖了。
        sk.interestOps(SelectionKey.OP_READ); 
        sel.wakeup();
    }


    boolean inputIsComplete() { /* ... */ } 
    boolean outputIsComplete() { /* ... */ } 
    void process() { /* ... */ }
    
    //http://gee.cs.oswego.edu
    // Reactor 5: Request handling
// class Handler continued
    public void run() {
        try {
            if (state == READING){
                read();
            } else if (state == SENDING){
                send();
            } 
        } catch (IOException ex) { /* ... */ }
    }
    void read() throws IOException {
        socket.read(input);
        if (inputIsComplete()) {
            process();
            state = SENDING;
        // Normally also do first write now 
            sk.interestOps( SelectionKey.OP_WRITE);
        } 
    }
    void send() throws IOException { 
        socket.write(output);
        if (outputIsComplete()) {
            sk.cancel();
        }
    } 
}
```

8. 多线程版本设计
    * 策略性的增加了一些线程可伸缩性，用于多线程环境下。
    * 快速的处理处理器，处理器会拖慢Reactor,将non-io移交其它处理器
    * 多reactor线程，它可以处理io,将负载转移其它，实现cpu与io匹配

9. Worker Threads
    * 将non-io移交出去，加快reactor线程
    * 使用线程池，可以调节和控制，比客户端的线程数量要少。

![image](https://github.com/ilin0/study_node/raw/master/netty/image/reactor2018032001.png)

10. 使用线程池
    主要的方法 execute(Runnable r)
    * 任务队列的种类，任何一个Channel
    * 最大、最小线程数
    * 活跃的间隔时间
    * 策略，阻塞、丢弃

11. 多线程 Reactor Threads
    使用Reactor线程池
    * 用于匹配cpu/io执行速率
    * 可以静态或动态的构件，每一个reactor 都有自己的 selector,thread,dispatch loop
    * 主接收器可以给reactor分发。
```java
    Selector[] selectors; // also create threads 
    int next = 0;
    class Acceptor { // ...
        public synchronized void run() { 
            //...
            Socket connection = serverSocket.accept(); 
            if (connection != null){
                new Handler(selectors[next], connection); 
            }
            
            if (++next == selectors.length) {
                next = 0;
            }
            
        } 
    }
```
##### using multiple reactors 多个reactor 
![image](https://github.com/ilin0/study_node/raw/master/netty/image/reactor2018032002.png)