### classLoader-doc.md

#### getSystemClassLoader
    返回用于委托的系统类加载器，它是默认的委托双亲（自定义的类加载器）实例，通常它也是用于启动应用的类加载器。（业务代码都是系统类加载器加载）
     这个方法在运行开始的时候就会被调用，在这个时刻它会创建系统类加载器（即上面返回的必需先创建），而且设置为调用线程的上下文类加载器。
     默认的系统类加载 器，与这个类的实现相关的实例
     如果 系统属性『java.system.class.loader』被定义了，那个这个定义的值就是所返回的系统类加载 器的名字。这个类是被默认的系统类加载器加载，它必须要定义一个public 的构造方法，接受一个Classloader的参数。它把系统类加载器作为它的双亲。我们自定义的加载器就是系统类加载器了。

```java
                try {
                    scl = AccessController.doPrivileged(
                        new SystemClassLoaderAction(scl));
                } catch (PrivilegedActionException pae) {
                    oops = pae.getCause();
                    if (oops instanceof InvocationTargetException) {
                        oops = oops.getCause();
                    }
                }

//SystemClassLoaderAction 就是处理我们我们自定义的系统类加载器

// 该中的parent决定了我们自定的加载 器是由哪个来加载 的
class SystemClassLoaderAction
    implements PrivilegedExceptionAction<ClassLoader> {
    private ClassLoader parent;

    SystemClassLoaderAction(ClassLoader parent) {
        this.parent = parent;
    }

    public ClassLoader run() throws Exception {
        String cls = System.getProperty("java.system.class.loader");
        if (cls == null) {
            return parent;
        }

        Constructor<?> ctor = Class.forName(cls, true, parent)
            .getDeclaredConstructor(new Class<?>[] { ClassLoader.class });
        ClassLoader sys = (ClassLoader) ctor.newInstance(
            new Object[] { parent });
        Thread.currentThread().setContextClassLoader(sys);
        return sys;
    }
}

    
```