
### 异常分为二类
    1. 对于非运行时异常(checked exception)，必须要对其进行处理，处理方式有两种:
        第一种是使用 try.. catch...finally 进行捕获;
        第二种是在调用该会产生异常的方法所在的方法声明 throws Exception

    2. 对于运行时异常(runtime exception)
        我们可以不对其进行处理，也可以对其进行处理。推荐不对其进行处理。

### 自定义异常
    通常就是定义了一个继承自Exception类的子类，那么这个类就是一个自定义异常类。通常情况下，我们都会直接继承自 Exception 类，一般不会继承 某个运行时的异常类。