### 类加载器工作机制： 
    1. 装载：将Java二进制代码导入jvm中，生成Class文件。 
    2. 连接：
        a）校验：检查载入Class文件数据的正确性
            - 文件结构检查
            - 语义检查
            - 字节码验证
            - 二进制兼容性的验证 
        b）准备：给类的静态变量分配存储空间 
        c）解析：将符号引用转成直接引用 
    3. 初始化：对类的静态变量，静态方法和静态代码块执行初始化工作。

### 初始化
    为类的静态变量赋予正确的初始值 ，比如 int a =1；在准备阶段它是0，初始化时才会变成实际的值1.
    1. ：先加载 再连接
    2. ：类存在直接的父类，先初始化父类
        - ：初始化类，并不会初始化它实现的接口
        - ：初始化一个接口时，并不会先初始化它的父接口
        - ：只有当程序首次使用特定接口的静态变量时
    3. ：类中存在初始化语句，依次执行这些初始化语句。

    
### 示例
    说明：只有当静态变量与静态方法确实在类中时，才能叫做对类的主动使用
    也就是说从父类继承下来的变量，不能叫做对类的主动使用。

```
class Parent3{
    static int a = 3;
    
    static{
        System.out.println("Parent3 static block");
    }
    
    static void doSomething(){
        System.out.println("do something");
    }
}

class Child3 extends Parent3{
    static{
        System.out.println("Child3 static block");
    }
}
public class Test6{
    public static void main(String[] args){
    // 这行定义的 child.a 变量时，并不会对 child 主动使用，
    // 它是父类中的变量，所以父类会被初始化，子类并不会被初始化
        System.out.println(Child3.a);  
        Child3.doSomething();
    }
}
```

### 主动使用（类的初始化时机）
    1. 创建类的实例  new Object();
    2. 访问某个类静态变量
    3. 调用类的静态方法
    4. 反射
    5. 初始化一个类的子类
    6. java虚拟机启动时标明为启动类的类
    除了以上的6 种方法，都叫被动使用，都不会初始化类。

### 类初始化示例
    说明：调用ClassLoader 只是加载一个类，并不会导致类的初始化。
```
class CL {
    static{
        System.out.println("Class CL");
    }
}

public class Test7{
    public static void main(String[] args) throws Exception{ 
        
        // 创建一个类加载器
        ClassLoader loader = ClassLoader.getSystemClassLoader();
        
        // 执行加载一个类，但类并不会初始化，CL类中的system并不会打印
        Class<?> clazz = loader.loadClass("com.shengsiyuan.classloader.CL");
        
        System.out.println("----------------------------");
        
        // 反射才会初始化它，打印类中的语句
        clazz = Class.forName("com.shengsiyuan.classloader.CL");
    }
}
```

    说明：此示例中的二种为 静态变量 x 赋值的方式不同，也就决定了类是否会被初始化。
```
class FinalTest
{
    // 因为它在编译时就能确定，所以不会导致类的初始化，静态代码块也就不会被执行的 
    public static final int x = 6 / 3;

    // 这行它是一个运行时才能确定它的值，所以必需初始化才能知道它的值，所以这个类会被初始化，
    public static final int x = new Random().nextInt(100);
    
    static
    {
        System.out.println("FinalTest static block");
    }
}

public class Test2
{
    public static void main(String[] args)
    {
        System.out.println(FinalTest.x);
    }
}
```

```
class Parent
{
    static int a = 3;
    
    static
    {
        System.out.println("Parent static block");
    }
}

class Child extends Parent
{
    static int b = 4;
    
    static
    {
        System.out.println("Child static block");
    }
}

public class Test4
{
    static
    {
        System.out.println("Test4 static block");
    }
    
    public static void main(String[] args)
    {
        System.out.println(Child.b);
    }
}

---
class Parent2
{
    static int a = 3;
    
    static
    {
        System.out.println("Parent2 static block");
    }
}

class Child2 extends Parent2
{
    static int b = 4;
    
    static
    {
        System.out.println("Child2 static block");
    }
}


public class Test5
{
    static
    {
        System.out.println("Test5 static block");
    }
    
    public static void main(String[] args)
    {
    // 当前类的 static 代码块先执行
        Parent2 parent;
        
        System.out.println("-------------------------");
    
    // 这里主动使用 ，会初始化父类，但不会初始化子类  
        parent = new Parent2();
        
        System.out.println(Parent2.a);
    
    // 最后主动使用子类， 子类 才会被初始化  
        System.out.println(Child2.b);
    }
}
```


###双亲委派模型：
    类加载器收到类加载请求，首先将请求委派给父类加载器完成 
    用户自定义加载器->应用程序加载器->扩展类加载器->启动类加载器。

### 为什么这样加载：
    处于安全原因考虑。

### 示例
```
public class Test1
{
    public static void main(String[] args) throws Exception
    {
        Class clazz = Class.forName("java.lang.String");
    
        System.out.println(clazz.getClassLoader());
        
        Class clazz2 = Class.forName("com.shengsiyuan.classloader.C");
        
        System.out.println(clazz2.getClassLoader());
        
        // 这二个就不是一个加载器加载 
        // String 是由根类加载 器加载 C 是由系统类加载 器加载  ？？？为什么不一样
    }
}

class C
{
   
}
```

### JVM (java virtual mackine)
    java 虚拟机的生命周期 以下情况会结束
    1. System.exit();
    2. 程序正常执行结束了
    3. 异常终止
    4. 操作系统错误