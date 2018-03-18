### strategy策略模式
    两个非常基本的面向对象设计的原则
1. 封装变化的概念
2. 编程中使用接口，而不是对接口的实现
    定义
1. 定义一组算法，将每个算法都封闭起来，并且使它们之间可以互换。
2. 策略模式使这些算法在客户端调用它们的时候能够互不影响地变化。
    意义：
1. 策略模式使用开发人员能够开发出由许多可替换的部分组件的软件，并且和个部分之间是弱连接的关系。
2. 弱连接的特性使软件具有更强的可扩展性，易于维护；更重要的是，它大大提高了软件的可重用性。

    组成：
1. 抽象策略角色：策略类，通常由一个接口或者抽象类实现。
2. 具体策略角色：包装了相关的算法和行为。
3. 环境角色：持有一个策略类的引用，最终给客户端调用的。
    
    实现：
1. 策略模式的用意是针对一组算法，将每一个算法封闭到具有共同接口的独立的类中，从而使得它们可以相互替换。
2. 策略模式使得算法可以在不影响到客户端的情况下发生变化。使用策略模式可以把行为和环境分开来。
3. 环境类负责维护和查询行为为，各种算法则在具体策略中提供。由于算法和环境独立开来，算法的修改都不会影响环境和客户端。
    
    编写步骤：
1. 对策略对象定义一个公共接口。
2. 编写策略类，该类实现了上面的公共接口。
3. 在使用策略对象的类中保存一个对策略对象的引用。
4. 在使用策略对象的类中， 实现对策略对象的set和get方法（注入）或者使用构造方法完成赋值。


#### 示例
```java
/**
 * 抽象策略角色
 */
public interface Strategy {
    public int calculate(int a , int b);
}
```

```java
/**
 * 具体策略角色,完成加法计算
 */
public class AddStrategy implements Strategy {

    @Override
    public int calculate(int a, int b) {
        return a + b;
    }
}


/**
 * 具体策略角色,完成减法计算
 */
public class SubtractStrategy implements Strategy {

    @Override
    public int calculate(int a, int b) {
        return a - b ;
    }
}
```

```java
/**
 * 环境角色
 */
public class Environment {
    private Strategy strategy;

    public Environment(Strategy strategy){
        this.strategy = strategy;
    }

    public int calculate(int a, int b){
        return strategy.calculate(a, b);
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        SubtractStrategy subtract = new SubtractStrategy();

        Environment subtractenv = new Environment(subtract);

        int subi = subtractenv.calculate(3, 6);
        System.out.println(subi);

        AddStrategy add = new AddStrategy();

        Environment addenv = new Environment(add);

        int addi = addenv.calculate(3, 6);
        System.out.println(addi);
    }
}
```
