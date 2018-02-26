

```java
class Singleton {
    
    // 这行在上面结果 是 1和0， 当放到 counter 2 下面时就变成了 1和1 
    private static Singleton singleton = new Singleton();
    public static int counter1;
    public static int counter2 = 0;
    
    private Singleton() {
        counter1++;
        counter2++;
    }
    public static Singleton getInstance() {
        return singleton;
    }
}


public class MyTest {

    public static void main(String[] args)
    {
    // 这行在调用时。类得到加载 singleton p初始化为null，counter1、counter2是int类型的默认值，都是为0；
    //这时初始化 singleton = new Singleton(),构造方法得到调用，所以counter1和counter2都是1， 
    //再初始化counter1 没有赋值还是是，初始化counter2时，重新赋值为0.
        Singleton singleton = Singleton.getInstance();
        System.out.println("counter1 = " + singleton.counter1);
        System.out.println("counter2 = " + singleton.counter2);
    }
}
```