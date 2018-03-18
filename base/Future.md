### Future.md
    代码的是一个异步计算的结果。
    提供了一些方法，检查计算是否完成，获取计算结果。只能通过get方法获取，它是阻塞的，只到计算完成返回结果。
    取消是通过cancel方法，它还提供一些方法确定任务是正常完成还是取消的情况，一但计算完成就不能被取消。使用它用于取消目的可以返回null.

### netty Future
    它比jdk提供的多了几个方法，对一些状态表现更明确，还增加了一个listener,观察者模式

    它比jdk提供的有哪些好处？
    1.5提供的它在结束时是通过调用get方法获取，但我们什么时候调用这个方法，我们也不清楚。
    任务是异步的，但后面的业务逻辑执行完后，这个任务也许还没有结束，这时我们调用get方法还是阻塞在那。这就是它的缺陷。
    而neety在某种程度上就为了解决这个缺陷，才有了自己的实现。
    它是通过listener来解决的，它在isDone方法完成后，listener里的方法会被调用，它是由Future对象调用的GenericFutureListener触发的

### netty ChannelFuture 
    它表示的是一个异步的channel的i/o操作的未来对象。未来的执行结果。
    netty当中所有的异常i/o操作都是异步的。调用一个i/o操作时，会返回一个channelFutrue。
```
 * <pre>
 *                                      +---------------------------+
 *                                      | Completed successfully    |
 *                                      +---------------------------+
 *                                 +---->      isDone() = true      |
 * +--------------------------+    |    |   isSuccess() = true      |
 * |        Uncompleted       |    |    +===========================+
 * +--------------------------+    |    | Completed with failure    |
 * |      isDone() = false    |    |    +---------------------------+
 * |   isSuccess() = false    |----+---->      isDone() = true      |
 * | isCancelled() = false    |    |    |       cause() = non-null  |
 * |       cause() = null     |    |    +===========================+
 * +--------------------------+    |    | Completed by cancellation |
 *                                 |    +---------------------------+
 *                                 +---->      isDone() = true      |
 *                                      | isCancelled() = true      |
 *                                      +---------------------------+
 * </pre>
```
    推荐使用addListener不推荐使用await
    channelHandler中的事件处理方法通常都是被i/o线程所调用。如果await在事件处理中所调用，它所等待的i/o操作永远不会完成，因为这个await方法是阻塞的，所以这个事件处理方法和await一直等待者对方。









    
