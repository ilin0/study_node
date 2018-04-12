loader示例.md

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
    对于表态字段来说，只有直接定义了该字段的类才会被初始化


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

