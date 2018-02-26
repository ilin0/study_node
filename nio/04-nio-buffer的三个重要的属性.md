### 04-nio-buffer的三个重要的属性.md
    关于 NIO Buffer 中的3个重要状态属性的含义， position, limit 与capacity。

    IntBuffer它是自动生成的代码。它继承Buffer类，

#### Buffer doc
    Buffer是特定的原生类型的容器。线性的、有限的序列，除了内容以外，本质的属性就是position,limit与capacity。

    capacity：元素的数量，不可变的（分配之后buffer的大小），不可以为负数，
    limit：永远不会超过capacity，也不可为负数，
    position：下一个读或写的索引，永远不会超过limit，不可以为负数。（读或写后，位置就会发生变化）
    0 <= mark <=porsition <= limit <= capacity
    新创建的Buffer，porsition=0，初始的limit可能是0，也可能是其它的值，这取决于它的创建方式，

    clear方法是将position=0，limit = capacity
    flip方法是将limit = position，position=0
    rewind方法是将limit保持不变，position=0


#### 图示
    当Buffer初始化6的时候三个属性的关系，position=0，limit = capacity = 6
![image](https://github.com/ilin0/study_node/raw/master/nio/image/nio2018022516.png)

    当position读了二个原素，再读二个原素时，它指向的是第4的索引位置，前4个元素是有值的
![image](https://github.com/ilin0/study_node/raw/master/nio/image/nio2018022517.png)

    调用flip()之后，position=0（它是表示将要读或写的下一个元素的位置），limit=4
![image](https://github.com/ilin0/study_node/raw/master/nio/image/nio2018022518.png)

    再写二个元素，position=2，它最大只能写到limit的值。
    如果当前position=3时，再次调用flip()方法之后，position=0，limit=3，也就是说limit会等于当前position的值。

#### IntBuffer的源码分析
    IntBuffer buffer = IntBuffer.allocate(10);
    allocate只是完成了一些值的初始化。
