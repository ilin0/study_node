loader示例.md

#### 示例为类的初始化
```java
public class LoaderTest1 {

    public static void main(String[] args) {
        System.out.println(MyChild1.str);
    }
}

class MyParent1 {
    public static String str = "hello";

    static {
        System.out.println("Myparent static block");
    }
}

class MyChild1 extends MyParent1 {
    static {
        System.out.println("MyChild1 static block");
    }
}
```

运行结果
    Myparent static block
    hello
##### 说明：
    对于静态字段来说，只有直接定义了该字段的类才会被初始化


修改之后如下
```java
class MyChild1 extends MyParent1 {

    public static String str2 = "world";
    static {
        System.out.println("MyChild1 static block");
    }
}

    public static void main(String[] args) {
        System.out.println(MyChild1.str2);
    }
    
```
打印结果
    Myparent static block
    MyChild1 static block
    world
 
##### 说明：
    当一个类在初始化时，要求其父类必需先初始化完成。


#### 示例2
```java
public class LoaderTest2 {
    public static void main(String[] args) {
        System.out.println(MyParent2.str);
    }
}

class MyParent2 {
    public static String str = "hello";

    static {
        System.out.println("MyParent2 static block");
    }
}

```
1. 输出的结果是
    MyParent2 static block
    hello

2. 修改之后,将常量改成final时。
    public static final String str = "hello";

3. 输出的结果是
    hello
4. 原因
    final表示一个常量，赋值后就不可改变，在编译阶段这个常量就会存入到调用这个常量的类的常量池中。本质上调用类并没有直接引用到定义常量的类，因此并不会触发定义常量类的初始化。
    Myparent2都可以删除掉，因为它们之间并没有任何关系了。

5. 反编译
    javap -c 包名.类名
    ➜jvm javap -c build.classes.main.com.gs.jvm.loadering.LoaderTest2
??: ?????build.classes.main.com.gs.jvm.loadering.LoaderTest2??com.gs.jvm.loadering.LoaderTest2
Compiled from "LoaderTest2.java"
```java
  
public class com.gs.jvm.loadering.LoaderTest2 {
  public com.gs.jvm.loadering.LoaderTest2();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: ldc           #4                  // String hello
       5: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
       8: return
}

```
    ldc它是一个助记符
6. 如果再次修改它，增加一个常量
    public static final short a = 7;
7. 再次反编译它，助记符为下面所示。如果上面的值超过127为128时，那么助记符就是sipush
    3: bipush        7

#### 示例3
```java
public class LoaderTest3 {
    public static void main(String[] args) {
        System.out.println(MyParent3.str);
    }
}

class MyParent3 {
    public static final String str = UUID.randomUUID().toString();

    static {
        System.out.println("myParent3 static code");
    }
}
```
#### 结果
    如果改成编译阶段不能决定值的，输出如下
    myParent3 static code
    41c4861c-f4fd-40d2-a7e6-b3ce5cc22350
    当一个常量的值并非编译期间可以确定的，那么其值就不会放到调用类的常量池中，这时在程序运行时，会导致主动使用，类就会被初始化。

#### 示例4
```java
public class LoaderTest4 {
    public static void main(String[] args) {

        MyParent4[] mp4 = new MyParent4[1];
    }
}

class MyParent4 {
    static {
        System.out.println("myParent4 static block");
    }
}
```
###### 结果输出
    没有任何输出。表示没有初始化。
1. mp4.getClass();输出
    class [Lcom.gs.jvm.loadering.MyParent4;这是java虚拟机在运行期帮我们生成出来的。
2. 二维数组的定义
    class [[Lcom.gs.jvm.loadering.MyParent4;
3. 一维与二维数组的父类mp4.getClass().getSuperclass();
    class java.lang.Object

#### 示例6
```java
public class LoaderTest6 {
    public static void main(String[] args) {
        Singleton s = Singleton.getInstance();

        System.out.println("c1 :" + s.c1);
        System.out.println("c2 :" + s.c2);
    }
}

class Singleton {
    public static int c1 ;
    public static int c2 = 0;

    private static Singleton singleton = new Singleton();
    private Singleton(){
        c1++;
        c2++;
    }
    public static  Singleton getInstance(){
        return singleton;
    }
}

```

##### 输出结果 
    c1 :1
    c2 :1

1. 修改一下：
```java
class Singleton {
    public static int c1 ;

    private static Singleton singleton = new Singleton();
    private Singleton(){
        c1++;
        c2++;
    }
    public static int c2 = 0;
    public static  Singleton getInstance(){
        return singleton;
    }
}
```

##### 输出结果 
    c1 :1
    c2 :0