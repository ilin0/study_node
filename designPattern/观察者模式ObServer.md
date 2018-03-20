### 观察者模式ObServer

    定义了一种一对多的依赖关系，让多个观察者对同一个对象同时监听某一主题对象。
    这个主题对象在状态发生变化时，会通知所有观察者对象，让它们能够自动更新自己。

#### Observable 与 Observer
    这二个类是jdk对观察者模式提供的支持类，我们需要继承和实现它们。
##### 组成部分
1. 抽象主题角色：把所有对观察者对象的引用保存在一个集合中，每个抽象主题角色都可以有任意数量的观察者。抽象主题提供一个接口，可以增加和删除观察者角色。一般用一个抽象类或接口来实现。
2. 抽象观察者角色：为所有具体的观察者定义一个接口，在得到主题的通知时更新自己。
3. 具体主题角色：在具体主题内部状态改变时，给所有登记过的观察者发出通知。具体主题角色通常用一个子类实现。
4. 具体观察者角色：该角色实现抽象观察者角色所要求的更新接口，以便使本身的状态与主题的状态相协调。如果需要，具体观察者角色可以保存一个指向具体主题角色的引用。通常用一个子类实现

##### 示例1：
```java
/**
 *抽象主题角色
 */
public interface Watched {
    public void addWatcher(Watcher watcher);

    public void removerWatcher(Watcher watcher);

    public void notifyWatcher(String str);
}

/**
 * 具体主题角色
 */
public class ConcreteWatched implements Watched {

    private List<Watcher> list = new ArrayList<>();
    @Override
    public void addWatcher(Watcher watcher) {
        list.add(watcher);
    }

    @Override
    public void removerWatcher(Watcher watcher) {
        list.remove(watcher);
    }

    @Override
    public void notifyWatcher(String str) {
        //这里才是调用监听器的过程
        for (Watcher watcher : list){
            watcher.update(str);
        }
    }
}
```

```java
/**
 * 抽象观察者角色
 */
public interface Watcher {

    public void update(String str);
}

/**
 * 具体观察者角色
 */
public class ConcreteWatcher implements Watcher {
    @Override
    public void update(String str) {
        System.out.println(str);
    }
}
```

```java
/**
 *
 */
public class Test {

    public static void main(String[] args) {

        Watched girl = new ConcreteWatched();

        // 观察者角色，观察这个女孩
        Watcher w1 = new ConcreteWatcher();
        Watcher w2 = new ConcreteWatcher();
        Watcher w3 = new ConcreteWatcher();

        girl.addWatcher(w1);
        girl.addWatcher(w2);
        girl.addWatcher(w3);

        girl.notifyWatcher("开心");
    }
}
```

##### 示例2：
```java
class BeingWatched extends Observable{
    void counter(int number){
        for (; number >= 0; number--){
            this.setChanged();// 这二步最为关键，不能少
            this.notifyObservers(number);
        }
    }
}
```

```java
class Watcher1 implements Observer{
    @Override
    public void update(Observable o, Object arg) {
        System.out.println("Watcher1 count is: " + arg);
    }
}

class Watcher2 implements Observer{
    @Override
    public void update(Observable o, Object arg) {
        if ((Integer)arg <= 5){
            System.out.println("Watcher2 count is: " + arg);
        }
    }
}
```

```java
public class TwoObservers {

    public static void main(String[] args) {
        BeingWatched watched = new BeingWatched();

        Watcher1 w1 = new Watcher1();
        Watcher1 w2 = new Watcher1();

        watched.addObserver(w1);
        watched.addObserver(w2);

        watched.counter(10);
    }
}
```