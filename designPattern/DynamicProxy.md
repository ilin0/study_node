### DynamicProxy

    java动态代理类位于java.lang.reflect包下，一般主要涉及到发下两个类：
1. Interface InvocationHandler:该接口中定义了一个方法public Object invoke(Object proxy, Method method, Object[] args);在实际使用时，第一个参数obj一般是指代理类，method是被代理的方法。
2. Proxy:该类即为动态代理类。


    它是在运行时生成的class,在生成它时你必须提供一组interface给它，然后该class就宣称它实现了这些interface。你当然可以把该class的实例当作这些interface中的任何一个来用。（这句话就是一个多态，生成该对象指向父类型的引用）。
    当然，这个DynamicProxy其实就是一个Proxy，它不会替你作实质性的工作，在生成它的实例时你必须提供一个handler，由它接管实际的工作


    步骤：
1. 创建一个实现接口InvocationHandler的类，它必须实现invoke方法
2. 创建被代理的类以及接口。
3. 通过Proxy的静态方法
newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h)
创建一个代理。
4. 通过代理调用方法。


#### 示例
```java
public interface Subject {
    public void request();
}
```

```java
public class RealSubject implements Subject {

    @Override
    public void request() {
        System.out.println("来自真实角色的功能！");
    }
}
/**
 * 动态代理必须实现InvocationHandler接口，
 * 这个动态代理类可以代理所有的真实角色完成各自的业务处理
 */
public class DynamicSubject implements InvocationHandler {

    private Object sub;

    public DynamicSubject(Object sub){
        this.sub = sub;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("before calling: " + method);

        method.invoke(sub, args);

        System.out.println("after calling: " + method);

        return null;
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        RealSubject real = new RealSubject();

        InvocationHandler dynamicSubject = new DynamicSubject(real);
        Class<?> classType = dynamicSubject.getClass();

        // 这里一定是返回它实现的接口，因为它不知道是哪个具体的类
        Subject subject = (Subject)Proxy.newProxyInstance(classType.getClassLoader()
                , real.getClass().getInterfaces(), dynamicSubject);

        subject.request();
    }
}

```