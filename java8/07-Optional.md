## Optional.md
    解决NullPointerException

    它可能包含一个null， 或者不是null。
    如果包含一个值，isPresent就返回true，get()就能得到值。

    https://docs.oracle.com/javase/8/docs/api/java/lang/doc-files/ValueBased.html  基于值的类

    它不能作为参数，或者成员变量，定义了就可能会出现问题，因为它不可以序列化。

#### 基本使用示例
```
public class OptionalTest {
    public static void main(String[] args) {

        //构造方式 分为 几种情况
        Optional<String> optional = Optional.of("hello");

        // 这种方式 并不是 推荐 的， 应该采用 函数式 风格
        if(optional.isPresent()){
            System.out.println(optional.get());
        }

        // 下面才是推荐的
        optional.ifPresent(item -> System.out.println(item));

        Optional<String> optional2 = Optional.empty();

        // 当optional为空时返回 world
        System.out.println(optional2.orElse("world"));

        // 当optional为空时返回 nihao
        System.out.println(optional2.orElseGet(() -> "nihao"));
    }
}
```
