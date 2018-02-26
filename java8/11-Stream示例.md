### Stream 示例.md
    

```java
//拼接字符串， join 以遇到的顺序，一个接一个的拼接。
Stream<String> stream = Stream.of("hello", "world", "hello world");
String str = stream.collect(Collectors.joining()).toString();
```


```java
    //这种写法语义更加直观 
    public static void main(String[] args) {

        List<String> stringList = Arrays.asList("hello", "world", "hello world", "test");

        // 转大写 即小写映射成大写（map）
        stringList.stream().map(String::toUpperCase).collect(Collectors.toList()).forEach(System.out::println);

        System.out.println("-------我是分隔线-------");

        // 如果 流中的对象也是 集合时
        Stream<List<Integer>> listStream = Stream.of(Arrays.asList(1), Arrays.asList(2,3), Arrays.asList(4,5,6));

        // flatMap 可以理解成，把多个list 打平 合成一个的概念
        listStream.flatMap(theList -> theList.stream()).map(item -> item * item).forEach(System.out::println);
    }
```

