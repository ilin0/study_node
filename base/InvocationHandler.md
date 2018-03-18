### InvocationHandler.md
	它是一个接口，是由一个代理实例的调用处理器实现。
    每一个动态代理实例都会有一个与之关联的调用处理器（invocation handler）（因为它本身不会调用，都是由关联的处理器invoke完成方法调用）.
    当我们调用一个代理实例上的某一个方法，它就是被派发到invoke方法调用。

    invoke:处理代理实例上的方法调用并返回结果。这个方法在一个调用处理上被调用，是调用处理器的invoke方法帮助我们调用。
    参数：
1. proxy,代理实例
2. method,对应于代理实例上的接口方法
3. args,代理实例被调用方法的参数 。

### Proxy
    动态代理类的父类。
    它有一个成员变量 protected InvocationHandler h;

