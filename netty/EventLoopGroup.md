### EventLoopGroup.md
    它是一个事件循环组，用来在selector操作时注册Channel.
    它主要完成一些变量的赋值。线程数不传默认是cpu核心*2

```java
// MultithreadEventExecutorGroup 片断代码
        if (executor == null) {
            // 这里只是创建线程执行的任务
            executor = new ThreadPerTaskExecutor(newDefaultThreadFactory());
        }

        children = new EventExecutor[nThreads];

        for (int i = 0; i < nThreads; i ++) {
            boolean success = false;
            try {
                // 这里是创建线程执行器，与上面的任务解耦了
                children[i] = newChild(executor, args);
                success = true;
            } catch (Exception e) {
                // TODO: Think about if this is a good exception type
                throw new IllegalStateException("failed to create a child event loop", e);
            }
            //……………………
        }
```
