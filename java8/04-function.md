### function 函数
    一个抽象方法
    apply（）

    两个默认方法
    /** 先执行 before 的function 再执行当前 this（也就是一个组合的function串联执行）
    */
    Function<V, R> compose(Function<? Super V, ? extends T> before)
    
    /** andThen该方法与compose类似，只不过它与compose正好相反，它是先执行当前函数
    * 返回的结果,作为参数传递给after.apply() 方法
    */
    Function<V, R> andThen(Function<? Super V, ? extends T> after)
    
    
    
    一个静态方法
    identity
    该方法 总是返回输入参数 ，即参数是什么就返回什么

#### 示例
```java
    Function<String, String> function = String::toUpperCase;
    System.out.println(function.getClass().getInterfaces()[0]);
```
#### 说明


#### 示例
    function 函数示例
```java
public class FunctionTest {

    public static void main(String[] args) {

        FunctionTest test = new FunctionTest();

        System.out.println(test.compute(1, value -> 2 * value));
        System.out.println(test.compute(2, value -> 5 + value));
        System.out.println(test.compute(3, value -> value * value));

        System.out.println(test.convert(5, value -> String.valueOf(value + "hello world")));

    }

    public int compute(int a , Function<Integer, Integer> function){
        return function.apply(a);
    }

    // 返回的结果类型与参数类型不一致
    public String convert(int a, Function<Integer, String> function){
        return function.apply(a);
    }

    // 下面三个方法 是指定好行为，上面一个方法就可以 解决
    public int method1(int a){
        return 2+ a;
    }

    public int method2(int a){
        return 5+ a;
    }

    public int method3(int a){
        return a+ a;
    }
}
```

```java
// 该示例只接受一个参数，返回一个结果
public class FunctionCompute {

    public static void main(String[] args) {

        FunctionCompute tests2 = new FunctionCompute();

        System.out.println(tests2.compute(2, value -> value * 3, value -> value * value));  // 结果是：12
        System.out.println(tests2.compute2(2, value -> value * 3, value -> value * value)); // 结果是：36
    }

    public int compute(int a, Function<Integer, Integer> function1, Function<Integer, Integer> function2) {
        return function1.compose(function2).apply(a);
    }

    public int compute2(int a, Function<Integer, Integer> function1, Function<Integer, Integer> function2) {
        return function1.andThen(function2).apply(a);
    }
}
```

## BiFunction
    function 函数只能接受一个参数，返回一个的结果。
    它只有一个默认方法
    default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after)

    BiFunction 相对于 Function 少了一个compose 方法

#### BiFunction 示例
```java
    public int compute3(int a, int b, BiFunction<Integer, Integer, Integer> bifunction){
        return bifunction.apply(a, b);
    }

    public int compute4(int a, int b, BiFunction<Integer, Integer,Integer>     biFunction, Function<Integer, Integer> function){

        return biFunction.andThen(function).apply(a, b);
    }

    // 调用
    System.out.println(tests2.compute3(1, 2, (a, b) -> a + b));
        System.out.println(tests2.compute3(1, 2, (a, b) -> a - b));

        System.out.println(tests2.compute4(1, 2, (a, b) -> a + b, value -> value * value));
```

```java
    // 特殊的实现，参数与返回都一样 简化了一点
    public static void main(String[] args) {
        BinaryOperatorTest test = new BinaryOperatorTest();
        System.out.println(test.compute(1,2, (a, b) -> a +b));
    }

    public int compute(int a, int b , BinaryOperator<Integer> binaryOperator){
        return binaryOperator.apply(a, b);
    }
```

### 为什么BiFunction 少了一个compose 的方法
    因为compose 方法是将参数的结果 作为调用者本身的参数 ，而 apply 只能返回一个结果 ，而BiFunction 是接受二个参数的，所以它没有compose 方法。
    反推：假如有 compose 方法，二个参数先执行参数before BiFunction函数apply方法，得到了一个结果，而执行当前apply方法是需要二个参数，这时我们并没有第二个参数， 所以BiFunction是没有 compose方法的。

#### 应用示例
```java
public class PersonTest {

    public static void main(String[] args) {
        Person person1 = new Person("zhangsan", 20);
        Person person2 = new Person("lishi", 40);
        Person person3 = new Person("wangwu", 45);

        List<Person> list = Arrays.asList(person1, person2, person3);

        PersonTest test = new PersonTest();
        List<Person> personResult = test.getPersonByName("lishi", list);

        personResult.forEach(person -> System.out.println(person.getName()));


        System.out.println("---------2");
        List<Person> personResult2 = test.getPersonByAge(20, list);

        personResult2.forEach(person -> System.out.println(person.getName()));

        System.out.println("-------3");
        List<Person> personResult3 = test.getPersonByAge2(20, list, (age, personList) -> {
            return personList.stream().filter(person -> person.getAge() <= age).collect(Collectors.toList());
                }
                );

        personResult3.forEach(person -> System.out.println(person.getName()));

    }
    // 将满足条件的结果返回
    public List<Person> getPersonByName(String name, List<Person> persons){

        // filter 接受的参数 Predicate 断定 返回为boolean 类型
        return persons.stream().filter(person -> person.getName().equals(name)).collect(Collectors.toList());
    }
    // 用BiFunction来实现上面的方法一样的逻辑
    public List<Person> getPersonByAge(int age, List<Person> persons){
        BiFunction<Integer, List<Person>, List<Person>> biFunction = (ageOfPerson, personList) ->
                personList.stream().filter(person -> person.getAge() > ageOfPerson).collect(Collectors.toList());

        return biFunction.apply(age, persons);
    }

    /**
     * 第三个参数 作为行为 动态 传递给方法，
     * @param age
     * @param persons
     * @param biFunction
     * @return
     */
    public List<Person> getPersonByAge2(int age, List<Person> persons, BiFunction<Integer, List<Person>, List<Person>> biFunction){
        return biFunction.apply(age, persons);
    }
}

```
