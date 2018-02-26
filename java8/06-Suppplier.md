## Suppplier.md
    提供者，每次调用并没有要求与之前提供不同的结果。

#### 示例
```java
        Supplier<String> supplier = () -> "hello world";

        System.out.println(supplier.get());
```

```java
        Supplier<Student> supplier = () -> new Student();

        System.out.println(supplier.get().getName());


        System.out.println("--------");

        Supplier<Student> supplier1 = Student::new;
        System.out.println(supplier1.get().getAge());
```