### jvm接口.md
```java
public class LoaderTest5 {

    public static void main(String[] args) {
        System.out.println(MyChild5.b);
    }
}

interface MyParent5 {
    public static int a = 3;
}

interface MyChild5 extends MyParent5 {
    public static int b = 8;
}
```
#### 结论
    当一个接口在初始化时，并不要求其父类都完成初始化
    只有在真正使用的到父接口的时候，才会被初始化

1. 输出，以上代码输出能正确输出8
2.  -XX:+TraceClassLoading加上这个参数
    我们能看到在加载过程中MyChild5与MyParent5都没有被加载。
    因为接口中的常量默认都是final的
3. 将MyChild5定义为class
    我们能回到在加载过程中，MyChild5与MyParent5都会被加载。
    而类如果也显示的定义为final时，那么结果将和接口是一样的，但这里没有定义所以，它会加载父类
4. 这是加载过程的区别
5. 如果int b = new Randow().nexInt(3); MyChild5还是interface
    我们能回到在加载过程中，MyChild5与MyParent5都会被加载。这个定义和类是一样的，在运行期无法确定时。

```java
public class LoaderTest5 {

    public static void main(String[] args) {
        System.out.println(MyChild5.b);
    }
}

interface MyParent5 {
    public static Thread thread = new Thread(){
        {
            System.out.println("MyParent5 invoked");
        }
    }
}

class MyChild5 implement MyParent5 {
    public static int b = 8;
}
```
#### 结论
    它是不会输出的。
    1.当一个类被初始化时，它所实现的接口是不会被初始化的。

```java
public class LoaderTest5 {

    public static void main(String[] args) {
        System.out.println(MyChild5.thread);
    }
}

interface MyParent5 {
    public static Thread thread = new Thread(){
        {
            System.out.println("MyParent5 invoked");
        }
    }
}

interface MyChild5 extends MyParent5 {
    public static Thread thread = new Thread(){
        {
            System.out.println("MyChild5 invoked");
        }
    }
}
```
#### 结论
    它是不会输出的。
    2.在初始化一个接口时，并不会初始化它的父接口。