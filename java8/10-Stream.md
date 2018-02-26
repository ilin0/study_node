### Stream.md
    doc
    一个原素的序列，支持串行或并行的操作。
    流操作会组合的流管道中，流管道会包含一个源，零个或多个中间操作（操作的是源），一个终止操作生成一个结果 。
    流操作的分类:
    1. 流是惰性的
    2. 及早求值


#### 创建 Stream
![image](https://github.com/ilin0/study_node/raw/master/netty/image/stream2018012301.png)
![image](https://github.com/ilin0/study_node/raw/master/netty/image/stream2018012302.png)
![image](https://github.com/ilin0/study_node/raw/master/netty/image/stream2018012303.png)

#### 区别
    面向对象是传递的数据，而函数式是传递的行为
![image](https://github.com/ilin0/study_node/raw/master/netty/image/stream2018012304.png)



#### collect 方法
    执行一个可变的汇聚或精简操作。指被汇聚的值是一个结果 容器，是根据更新结果 的状态来更新结果 -- 这是一个终止操作。
    api 中提供了二个示例代码

#### collect 方法示例：将流转成集合
```java
    Stream<String> stream = Stream.of("hello", "world", "hello world");

    //流转换成 list  Collectors.toList()参数 是一个封闭
    List<String> stringList = stream.collect(Collectors.toList());

        /**
         * 复杂点的方式为 使用重载接受三个参数 的方法
         * 第一参数 Supplier 它表示不接受参数 返回一个结果， 这里的结果 就是我们需要的结果list 所以new ArrayList<String>() 实现类
         * 第二个参数 BiConsumer 接受二个参数 不返回结果， 第一个参数 是个list 第二个参数是遍历的对象
         * 第三个参数 也是 BiConsumer 它接受的是二个参数 ，第一个参数是 supplier 返回的结果  第二个参数是每次遍历的list ,
         * 需要把每次遍历的lsit放到结果 中
         *
         */
        List<String> stringList1 = stream.collect(() -> new ArrayList<String>(), (theList, item) -> theList.add(item),
                (list1, list2) -> list1.addAll(list2));


        // 方法引用的方式实现
        List<String> stringList2 = stream.collect(LinkedList::new, LinkedList::add,LinkedList::addAll);

        stringList1.forEach(System.out::println);

        List<String> stringList3 = stream.collect(Collectors.toCollection(ArrayList::new));

        // 这里也可以转换成 set  参数 为 TreeSet::new

```

```java
// 生成 stream 方法
Stream<String> stream = Stream.generate(UUID.randomUUID()::toString);
stream.findFirst().get();
        stream.findFirst().isPresent(System.out::println);

        // findFirst 返回的Optional 目的是防止npe ,
```

#### iterate 方法示例
```
iterate方法接受二个参数 ，第二个参数 UnaryOperator （unary 表示一元的）它继承Function 
function 接受一个输入一个输出，这里输入与输出都是 T ,
iterate 这个方法返回一个无限的串行的有序的stream。 它由跌代函数F 对初始值 seed 跌代生成 。
对于第一个元素(stream中位置为0的 )它作为提供种子(seed)， 对于（n>0来说）位于 n 位置的元素是对于 n-1 位置上的元素应用 f 函数 的结果 。

Stream.iterate(1, item -> item +2) // 这样表示，从1 开始，每次对它+2，返回一个结果 ，如果只是这样不作处理，它会一直无限循环下去。
它会有一个配合 一般情况下有个 limit 它是一个中间操作
Stream.iterate(1, item -> item +2).limit(6).forEach(System.out::println);

```


Stream<Integer> stream1 = Stream.iterate(1, item -> item +2).limit(6);
stream1.filter(i -> i >2).mapToInt(i -> i *2).skip(2).limit(2).sum();
#### filter 方法示例
```
 filter 方是过滤的作用，它接受一个 Predicate 
 stream1.filter(i -> i >2) // 过滤流中大于 2 的元素
```

#### skip 跳过 忽略
```
stream1.skip(2)//  跳过前二个元素
```

#### min 取最小值
```
stream1.min(); // min 方法返回的是 OptionalInt 是int 的包装？ 为什么是这样？而 sum 普通的int. 原因就取决于它的值能不能为空。
1、当流中的数据 不存在时，求和返回 0就可以了，当如果 取最小值或最大值就不知道返回什么了。所以这里返回的就是Optional.empty
对于正确的写法应该是
stream1.filter(i -> i >2).mapToInt(i -> i *2).skip(2).limit(2).min().ifPresent(System.out::println);// 判断存在时执行
```

#### IntSummaryStatistics 总结统计
```
对于上面的示例中，如果要同时求 最小值、最大值、求和。不可以复制三行代码分别取出对应的结果 ，

        /**
         *  讲解 summaryStatistics 方法的使用
         */
        IntSummaryStatistics iss = stream1.filter(i -> i >2).mapToInt(i -> i *2).skip(2).limit(2).summaryStatistics();

        System.out.println(iss.getMin());
        System.out.println(iss.getMax());
        System.out.println(iss.getSum());

        注意事项：
        当对象流为空时，最小值或最大值都会有默认值。
```

#### 注意事项
```java
        System.out.println(stream1);
        System.out.println(stream1.filter(i -> i >2));// 第一次使用
        System.out.println(stream1.distinct());// 第二次使用  

        //这样会抛出IllegalStateException异常 stream has already been operated upon or closed。它与i/o 是一致的，一但被操作了是不能被重复使用了。可以采用方法链式的方式
```

#### stream 的中间操作
```java
    List<String> list = Arrays.asList("hello", "world", "hello world");

        list.stream().map(item -> item.substring(0,1).toUpperCase() + item.substring(1)).forEach(System.out::print);

        // 这段代码中的 print 是不会执行，那说明代码块是没有执行的，没有遇到终止操作时，它是不会执行的
        list.stream().map(item -> {
            String result = item.substring(0,1).toUpperCase() + item.substring(1);
            System.out.println("test");

            return result;
        });

        // 加上 forEach 的区别
        list.stream().map(item -> {
            String result = item.substring(0,1).toUpperCase() + item.substring(1);
            System.out.println("test");

            return result;
        }).forEach(System.out::print);
```

#### 陷进示例
```java
// 陷阱  limit 与 distinct 位置调换后，程序死循环
        IntStream.iterate(0, i -> (i+1) % 2).distinct().limit(6).forEach(System.out::println);
        上面的distinct 一直在去重，iterate一直在跌代产生01，所以它就死循环

    // 而下面取出前6个后，iterate 就停止了，再去重，所以在使用的时候一定要注意顺序。
        IntStream.iterate(0, i -> (i+1) % 2).limit(6).distinct().forEach(System.out::println);

```


