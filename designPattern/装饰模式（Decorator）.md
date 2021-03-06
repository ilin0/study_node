### 装饰模式（Decorator）
    装饰模式又名包装（Wrapper）模式
    装饰模式以对客户端透明的方式扩展对象的功能，是继承关系的一个替代方案
    装饰模式以对客户端透明方式动态的给一个对象附加更多的责任。换言之，客户端并不会觉得对象在装饰前和装饰后有什么不同
    装饰模式可以在不创造更多子类的情况下将对象的功能加以扩展。

    装饰模式把客户端的调用委派到被装饰类。装饰模式的关键在于这种扩展完全是透明的。
    装饰模式是在不必改变原类文件和使用继承的情况下，动态扩展一个对象的功能。它是通过创建一个包装对象，也就是装饰来包裹真实的对象

    角色：
1. 抽象构件角色：（Component）给出一个抽象接口，以规范准备接收附加责任的对象。
2. 具体构件角色：（Concrete Component）定义一个将要接收附加责任的类。
3. 抽象装饰角色：（Decorator）持有一个构件（Component）对象的引用，并定义一个与抽象构件接口一致的接口
4. 具体装饰角色：（Concrete Decorator）负责构件对象『贴上』附加的责任。

    特点：
1. 装饰对象与真实的对象有相同的接口。这样客户端对象就可以以和真实对象相同的方式和装饰对象交互。
2. 装饰对象包含一个真实对象的引用（reference）.
3. 装饰对象接收所有来自客户端的请求。把这些请求转发给真实的对象。
4. 装饰对象可以在转发这些请求以前或以后增加一些附加功能。这样就确保了在运行时，不用修改给定对象的结构就可以在外部增加附加的功能。在面向对象的设计中，通常是通过继承来实现对给定类的功能扩展。

##### 比较
    装饰模式
1. 用来扩展特定对象的功能
2. 不需要子类 
3. 动态
4. 运行时分配职责
5. 防止由于子类而导致的复杂和混乱
6. 更多的灵活性
7. 对于一个给定的对象，同时可能有不同的装饰对象，客户端可以通过它的需要选择合适的装饰对象发送消息
    继承
1. 用来扩展一类对象的功能
2. 需要子类
3. 静态
4. 编译时分派职责
5. 导致很多子类产生
6. 缺乏灵活性
