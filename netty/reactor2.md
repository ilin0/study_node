### reactor.md
关于reactor的论文，描述的是日志服务器

#### 意图
    它是处理多个客户端并发的服务请求，应用当中的每一个服务都可能包含多个方法，都会有单独的事件处理器，它是由特定的服务来分发，是由initiation dispatcher。服务的请求是由一个同步的事件分离器来执行。

![image](https://github.com/ilin0/study_node/raw/master/netty/image/reactor2018040901.png)

1. 第一步是线程一部分，先执行accept方法等待客户端的连接，客户端向Logging Acceptor（日志接受器）发起请求，当连接成功时调用create()方法创建一个Logging Handler来处理，日志处理器创建成功之后，客户端就直接向它发送请求，接下来就是接受日志，并写入

![image](https://github.com/ilin0/study_node/raw/master/netty/image/reactor2018040902.png)

1. Synchronous Event Demultiplexer
    同步事件分离器，是由操作系统完成，它是底层的select()方法，等待事件的发生
    它本身是一个系统调用，用于等待事件的发生，调用方在调用它的时候会被阻塞，一直阻塞到同步分离器上有事件产生为止。
    对于linux来说，同步事件分离器指的就是常用的I/O多路复用机制，
    比如说：select、poll、epoll等，在java NIO领域中，对应的组件是Selector；
    对应的阻塞方法就是select方法。

2. Handle
    它是一个句柄，表示一个socket句柄
    本质上表示一种资源，是由操作系统提供的；该资源用于表示一个个的事件，
    比如：文件描述符，socket描述符。事件即可以来自于外部，也可以来自于内部；
    外部事件比如说客户端的连接请求，客户端发送过来的数据等；
    内部事件比如说操作系统产生的定时器，它本质上就是一个文件描述符。它是事件产生的发源地。

3. Initiation Dispatcher
    初始化分发器，监听事件的到来，
    实际上就是Reactor角色。它本身定义了一些规范，这些规范用于控制事件的调试方法，同时又提供了应用进件事件处理器的注册、删除等设施。它本身是整个事件处理器的核心所在，Initiation Dispatcher会通过同步事件分离器来等待事件的发生。一旦事件发生，Initiation Dispatcher首先会分离出每一个事件（也就是遍历set,取出每一个selectionkey），然后调用事件处理器，最后去调用相关的回调方法来处理这些事件。

4. Event Handler
    业务实现的事件处理器
    本身由多个回调方法构成，这些回调方法构成了与应用相关的对于某个事件的反馈。Netty相比于Java NIO来说，在事件处理器这个角色上进行了一个升级，它为我们开发者提供了大量的回调方法，供我们在特定事件产生时实现相应回调方法进行业务逻辑处理。

5. Concrete Event Handler(具体的事件处理器)
    它是Event Handler的一个实现。它本身实现了事件处理器所提供的各个回调方法，从而实现了特定于业务的逻辑。它本质上就是我们所编写的一个个处理器实现。（类似于我们实现SimpleChannelInboundHandler的业务类）

![image](https://github.com/ilin0/study_node/raw/master/netty/image/reactor2018040903.png)
#### 建立连接时
1. 注册，将Logging Acceptor注册到Initiation Dispatcher
2. 启动，调用handle_events()方法启用。
3. select() 是调用操作系统的select方法阻塞，监听。
4. 客户端，发起连接请求，select()方法得到返回了。
5. handle_event()，通知Logging Acceptor有新的请求发起。 
6. Logging Acceptor，调用accept()接受这个连接，并调用create()创建Logging Handler
7. Logging Handler就会注册到Initiation Dispatcher当中，等待着客户端发送数据。

![image](https://github.com/ilin0/study_node/raw/master/netty/image/reactor2018040904.png)
#### 发送数据时
1. send()，客户端发送数据，
2. Initiation Dispatcher，它监听到有事件发送，根据返回的连接找到与之关联的Handle，回调Handle里的handle_event()方法，读取日志数据 ，处理完返回结果到客户端，等待第二次请求。





































