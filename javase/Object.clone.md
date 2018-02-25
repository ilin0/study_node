### Object.clone.md
    #### java中对象的克隆
    1. Object类的clone()。
    2. 在派生类中覆盖基类的clone()方法，并声明为 public【Object类中的clone方法为protected】。
    3. 在派生类的clone方法中，调用super.clone（）。
    4. 在派生类中实现Cloneable接口。

#### clone示例
```
public class Student implements Cloneable{
    private int age;
    private String name;

    @Override
    public Object clone() throws CloneNotSupportedException{
        // 如果该类有引用对象，在这里也调用引用的clone方法，就能实现深复制
        return super.clone();
    }
}
```