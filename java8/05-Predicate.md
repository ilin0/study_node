#### Predicate.md
    根据给定的条件，返回 true 或 false
    Predicate<String> predicate = p -> p.length() > 5;

    使用
    System.out.println(predicate.test("hello"));


#### 场景应用示例
```java
public class PredicateTest2 {

    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);

        PredicateTest2 test = new PredicateTest2();

        // 按条件过滤
        test.conditionFilter(list, l -> l % 2 == 0);
        test.conditionFilter(list, l -> l % 2 != 0);
        test.conditionFilter(list, l -> l > 5);
        test.conditionFilter(list, l -> l < 3);
        test.conditionFilter(list, l -> true);

        System.out.println("-----");

        // 复合条件过滤
        test.conditionFilter(list, l -> l > 5, l -> l % 2 == 0);// 取大于5 并且是偶数的
    }

    public void conditionFilter(List<Integer> list, Predicate<Integer> predicate){
        list.stream().filter(predicate).forEach(l -> System.out.println(l));
    }

    public void conditionFilter(List<Integer> list, Predicate<Integer> predicate1, Predicate<Integer> predicate2){
        list.stream().filter(i -> predicate1.and(predicate2).test(i)).forEach(l -> System.out.println(l));
    }
}
```
