
### 线程
    1. 不能依靠线程的优先级来决定线程的执行顺序。

    2. Thread 类也实现了 Runnable 接口，因此实现了 Runnable 接口中的 run 方法;
        1)  当生成一个线程对象时，如果没有为其设定名字，那么线程对象的名字将使用如下形式:Thread-number,该 number 将是自动增加的，并被所有的 Thread 对象所共享(因为它是 static 的成员变量)。
        2) 当使用第一种方式来生成线程对象时，我们需要重写 run 方法，因为 Thread 类的 run方法此时什么事情也不做。
        3）当使用第二种方式来生成线程对象时，我们需要实现 Runnable 接口的 run 方法，然 后使用 new Thread(new MyThread())(假如 MyThread 已经实现了 Runnable 接口)来 生成线程对象，这时的线程对象的 run 方法就会调用 MyThread 类的 run 方法，这样 我们自己编写的 run 方法就执行了。

### 多线程的同步 synchronized 
    1. Java 中的每个对象都有一个锁(lock)或者叫做监视器(monitor)，
        当访问某个对象的synchronized方法时，表示将该对象上锁，此时其他任何线程都无法再去访问该synchronized 方法了，直到之前的那个线程执行方法完毕后(或者是抛出了异常),那么将该对象的锁释放掉，其他线程才有可能再去访问该 synchronized 方法。
    2. 如果一个对象有多个 synchronized 方法，某一时刻某个线程已经进入到了某个 synchronized 方法，那么在该方法没有执行完毕前，其他线程是无法访问该对象的任 何 synchronized 方法的。
        当其中一个是静态方法时，同一个对象在执行它的synchronized方法时，也是无序的，因为它们锁的不是一个对象。一人是类Class对应的对象，另一个实例对象 
    3. 如果某个 synchronized 方法是 static 的，那么当线程访问该方法时，它锁的并不是    synchronized 方法所在的对象，而是 synchronized 方法所在的对象所对应的 Class 对 象，因为 Java 中无论一个类有多少个对象，这些对象会对应唯一一个 Class 对象， 因此当线程分别访问同一个类的两个对象的两个 static，synchronized 方法时，他们 的执行顺序也是顺序的，也就是说一个线程先去执行方法，执行完毕后另一个线程 才开始执行。

    被synchronized 保护的数据应该是私有的
### synchronized 块
#### 写法: 
    synchronized(object){
    }
    表示线程在执行的时候会对 object 对象上锁。

### synchronized 方法与synchronized代码块的区别
    1. synchronized方法是一种粗粒度的并发控制，某一时刻，只能有一个线程执行该synchronized方法;
    2. synchronized 块则是一种细粒度的并发控制，只会将块中的代码同 步，位于方法内、synchronized 块之外的代码是可以被多个线程同时访问到的。

### 死锁(deadlock)
    线程的相互作用
    1. wait 与 notify 方法都是定义在 Object 类中，而且是 final 的，因此会被所有的 Java
    类所继承并且无法重写。这两个方法要求在调用时线程应该已经获得了对象的锁， 因此对这两个方法的调用需要放在 synchronized 方法或块当中。当线程执行了 wait 方法时，它会释放掉对象的锁。
    2. 另一个会导致线程暂停的方法就是 Thread 类的 sleep 方法，它会导致线程睡眠指定的毫秒数，但线程在睡眠的过程中是不会释放掉对象的锁的。

#### wait
    当前线程必需要拥有对象的锁。调用wait() 一定是在synchronized方法或方法块中

### wait 与 sleep 的区别
    wait方法等待时会释放掉对象的锁，在被通知后，不一定会执行，需要竞争才可能会执行。而sleep则不会，它在定时器到了之后，则会继续执行，因为它拥有对象的锁

#### notify
    notify方法调用将会任意唤醒一个线程。

#### 线程之间通信示例
```java
public class Sample {
    private int number;
    public synchronized void increase() throws Exception{
        while(o != number){
            wait();
        }
        number++;
        System.out.println(number);
        notify();
    }
    public synchronized void decrease() throws Exception{
        while(o == number){
            wait();
        }
        number--;
        System.out.println(number);
        notify();
    }
}
```

```java
public class IncreaseThread extends Thread {
    private Sample sample;
    public IncreaseThread(Sample sample){
        this.sample = sample;
    }
    @Override
    public void run() {
        for(int i = 0; i < 20; i++){
            Thread.sleep((long)Math.random() * 1000)
            sample.increase();
        }
    }
}
```

```java
// 做加法的线程与减法的类似
public class DecreaseThread extends Thread {

}
```

```java
    Sample sample = new Sample();
    Thread t1 = new IncreaseThread();
    Thread t2 = new DecreaseThread();

    t1.start();
    t2.start();
```