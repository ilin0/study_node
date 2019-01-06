### classLoader示例.md

#### 示例1
```java
public class LoaderTest7 {
    public static void main(String[] args) throws Exception {
        Class<?> clazz = Class.forName("java.lang.String");
        System.out.println(clazz.getClassLoader());

        Class<?> clazzc = Class.forName("com.gs.jvm.loadering.C");
        System.out.println(clazzc.getClassLoader());
    }
}

class C {

}
```
##### 输出结果
    null
    sun.misc.Launcher$AppClassLoader@18b4aac2

    说明：AppClassLoader是由系统反编译的

#### 示例2
```java
public class LoaderTest8 {

    public static void main(String[] args) {
        System.out.println(FinalTests.x);
        System.out.println(FinalTests.y);
    }
}

class FinalTests {
    public static final int x = 2;
    public static final int y = new Random().nextInt(3);
    static {
        System.out.println("finalTest static black");
    }
}
```

##### 反编译
    public static void main(java.lang.String[]);
    Code:
       0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: iconst_2
       4: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
       7: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
      10: getstatic     #5                  // Field com/gs/jvm/loadering/FinalTests.y:I
      13: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
      16: return
      从上面我们可以看到
      3: iconst_2 就是变量x 是存在当前类的常量池中的，那么和定义的类就没有关系，不会初始化定义类
      10: getstatic 它是一个引用变量，它在使用时就会主动使用它所定义的类，自然就会初始化类

#### 示例3
```java
public class LoaderTest9 {
    static {
        System.out.println("test9 static block");
    }

    public static void main(String[] args) {
        System.out.println(Child.c);
    }
}

class Parent {
    static int p = 2;

    static {
        System.out.println("parent static block");
    }
}

class Child extends Parent {
    static int c = 4;

    static {
        System.out.println("Child static block");
    }
}
```

##### 输出
    test9 static block  main方法所在的类第先加载 
    parent static block  再使用变量c时， 优先加载它的父类，所以它会被执行
    Child static block      再到当前类的初始化
    4

#### 示例5
```java
public class LoaderTest11 {
    public static void main(String[] args) {

        System.out.println(Child3.a);
        Child3.doSomething();
    }
}
class Parent3 {
    static int a = 3;
    static {
        System.out.println("parent3 static block");
    }
    static void doSomething(){
        System.out.println("do something");
    }
}
class Child3 extends Parent3 {
    static {
        System.out.println("child3 static block");
    }
}
```

##### 输出
    parent3 static block 因为a定义在Parent3类中，所以它被初始化
    3   打印的3 是a 的值，正常输出，而child3中的静态代码块没有打印是因为它没有初始化。
    do something

#### 示例6
```java
public class LoaderTest12 {
    public static void main(String[] args) throws Exception {
        ClassLoader loader = ClassLoader.getSystemClassLoader();

        Class<?> classcl = loader.loadClass("com.gs.jvm.loadering.CL");
        System.out.println(classcl);
        classcl = Class.forName("com.gs.jvm.loadering.CL");
        System.out.println(classcl);
    }
}

class CL {
    static {
        System.out.println("class cl");
    }
}
```

##### 输出  目的是验证反射可以导致类的初始化
    class com.gs.jvm.loadering.CL
    ---------             在这行上面 静态代码块并没有执行，而下面执行了，说明loadClass只是加载了类，并不会初始化
    class cl
    class com.gs.jvm.loadering.CL


#### 示例7
```java
public class LoaderTest15 {
    public static void main(String[] args) {
        String[] strings = new String[2];
        System.out.println(strings.getClass().getClassLoader());

        System.out.println("-------------");
        LoaderTest15[] lt15s = new LoaderTest15[2];
        System.out.println(lt15s.getClass().getClassLoader());
    }
}
```

##### 输出  目的是验证--数组类加载器是？
    null
    -------------
    sun.misc.Launcher$AppClassLoader@18b4aac2
    原因：它是由数组里的元素决定的。如：String类是由根加载器加载，它的数组类型也一样
    如果是原生类型它的结果也是null,但它与String类是不一样的。它是表示没有classLoader,
    对于原生类型它是没有类加载器的。java doc文档中说明的。

#### 示例8 类加载器之间的层次关系
```java
public class LoaderTest13 {

    public static void main(String[] args) {
        ClassLoader classLoader = ClassLoader.getSystemClassLoader();
        System.out.println(classLoader);

        while (null != classLoader){
            classLoader = classLoader.getParent();
            System.out.println(classLoader);
        }
    }
}
```

##### 输出  类加载器之间的层次关系
    sun.misc.Launcher$AppClassLoader@18b4aac2
    sun.misc.Launcher$ExtClassLoader@5451c3a8
    null

#### 示例9 如何通过字节码文件打印相应的信息
```java
public class LoaderTest14 {
    public static void main(String[] args) throws Exception {
        ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
        System.out.println(classLoader);

        Enumeration<URL> urls = classLoader.getResources("com/gs/jvm/loadering/LoaderTest12.class");
        while (urls.hasMoreElements()){
            System.out.println(urls.nextElement());
        }
    }
}
```

##### 输出  完整的资源位置信息
    file:/Users/ilin0/gettingStarted/jvm/build/classes/main/com/gs/jvm/loadering/LoaderTest12.class