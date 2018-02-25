## 什么是lambda（1）
    在编程语言当中，是一种运算符，匿名函数或闭包的。

## 为什么需要lambda(2.40-10)
    在java 中， 我们无法将函数作为参数传递给一个方法，也无法声明返回一个函数的方法
    类比 ： 在javaScript中，函数参数是一个函数，返回值是另一个函数，返回值是另一个函数的情况是非常常见的；javaScript是一门非常典型的函数式语言

## 关于函数式接口：@FunctionalInterface
    1. 如果一个接口只有一个抽象方法（因为它可以有实现方法）
    2. 如果我们在某个接口上声明了FunctionalInterface注解，那么编译器就会按照函数式接口的定义来要求该接口
    3. 如果某个接口只有一个抽象方法，但我们并没有给接口声明FunctionalInterface注解，那么编译器依旧将该接口看作是函数式接口。
    4. 接口的任意一个实现肯定会继承Object,所以toString()方法是不会给抽象方法+1的。

#### 示例
```
@FunctionalInterface
interface MyInterface {
    void test();
    String toString(); //这里是二个接口，但他是对的（因为它是重写了Object的方法）
    // 如果改为下面这样
    String myString(); // 这样就提示出错了，因为这里有二个接口了。
}
// 扩展使用
public class Test2{
    public void myTsts(MyInterface myInterface){
        System.out.println("--1");
        myInterface.test();
        System.out.println("--2");
    }

    public static void main(String[] args) {

        Test2 test2 = new Test2();
        test2.myTsts(new MyInterface() {
            @Override
            public void test() {
                System.out.println("mytest");
            }
        });

        System.out.println("------------");

        test2.myTsts(() -> {
            System.out.println("lambda 表达式实现");
        });

        System.out.println("----------");

        MyInterface myInterface = () -> {
            System.out.println("hello");
        };
    }
}
```

### lambda 表达式的作用
    Lambda 表达式为java 添加了缺失的函数式编程的特性，使用我们能将函数当做一等公民看待。在将函数作为一等公民的语言中， Lambda表达式的类型是函数。但在java中， Lambda表达式是对象， 他们必须依附于一类特别的对象类型—-函数式接口

#### 示例 说明lambda是一个对象
```

@FunctionalInterface
interface MyInterface1 {

    void test();
}

@FunctionalInterface
interface MyInterface2 {

    void test();
}


public class Test3 {
    public static void main(String[] args) {

        // lambda 表达式实现函数声明
        MyInterface1 i1 = () -> {};// 没有参数 ，没有返回值
        System.out.println(i1.getClass().getInterfaces()[0]);

        /**
         * 
         * () -> {} 它们的表现形式是一样的，对于这样的 Lambda 式，是根据上下文，才能决定的
         * 这里的上下文是  MyInterface2 i2 如果没有这个上下文，那么（）这个不接受
         * 参数的方法就不知道是哪个方法了。
         */
        MyInterface2 i2 = () -> {};
        System.out.println(i2.getClass().getInterfaces()[0]);

        // 线程示例：
        new Thread(() -> System.out.println("Hello world")).start();

        List<String> list = Arrays.asList("hello", "world", "hello world");

        System.out.println("-转大写   1");
        list.forEach(item -> System.out.println(item.toUpperCase()));

        List<String> list2 = new ArrayList<>();
        list.forEach(item -> list2.add(item.toUpperCase()));
        list2.forEach(item -> System.out.println("转 list2 后：" + item));

        System.out.println("-转大写   2");
        list.stream().map(item -> item.toUpperCase()).forEach(item -> System.out.println(item));

        System.out.println("-转大写   3");
        list.stream().map(String::toUpperCase).forEach(item -> System.out.println(item));
    }
}



```

## 函数式接口实例的创建
    1. lambda 表达式 
    2. 方法引用的形式；System.out::print
    3. 构造方法引用


### Lambda概要
    Java Lambda 表达式是一种匿名函数，它是没有声明的方法，即没有访问修饰符、返回值声明和名字


### java lambda基本语法
    (argument) -> (body)
    比如：
    1. （int a , int b）-> {return a + b;}
    2. () -> System.out.println(“hello world”);
    3. (a, b) -> a+b
    4. () -> {return 42;}

### 结构
    一个lambda表达式可以有0个参数或多个参数
    参数类型可以明确声明，也可以根据上下文来推断
    空括号表示参数为空
    如果参数只有一个并且参数可以推断，那么圆括号可以省略。主体只有一条语句，花括号也可以省略。

## 第二个示例   集合的示例（35）
```
public class Test1 {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8);
        
        for(int i = 0; i < list.size(); i++){
            System.out.println(list.get(i));
        }
        System.out.println("------------1");
        
        for(Integer i : list) {
            System.out.println(i);
        }
        System.out.println("-----------2");

        // java 8 提供的方法
        list.forEach(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) {
                System.out.println(integer);
            }
        });

        System.out.println("-----------3");
        list.forEach(i -> {  // 声明类型   (Integer i) -> 
            System.out.println(i);
        });

        System.out.println("-----------4");
        list.forEach(System.out::print);// 方法引用的形式 创建一个函数式接口实例
    }
}
```