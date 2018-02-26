### 03-nio.md

    java.io中最为核心的一个概念是(Stream),面向流的编程。对于一个流不可能既是输入流又是输出流。

    java.nio中拥有3个核心的概念：Selector、Channel与Buffer，在java.nio中，我们是面向块（block）或是缓冲区（Buffer）编程的。Buffer本身就是一块内存，底层实现上，它实际是个数组，数据的读写都是通过Buffer来实现的。

##### 关系
    s:表示Selector、c表示Channel、B表示Buffer
    s包含多个c，而每个c都会有一个B
![image](https://github.com/ilin0/study_node/raw/master/nio/image/nio2018022515.png)

    除了数组之外，buffer还提供了对于数据结构访问方式，并且可以追踪到系统的读写过程。

    java中的7种原生数据类型都有各自对应的Buffer类型，如：IntBuffer，LongBuffer， ByteBuffer，charBuffer等等, 并没有BooleanBuffer类型。
    Channel指的是可以向其写入数据或是从中读取数据的对象，它类似于java.io中的Stream

    所有数据的读写都是通过Buffer来进行的，永远不会出现直接向Channel写入数据的情况，或是直接从Channel读取数据的情况。

    与Stream不同的是， Channel是双向的， 一个流只可能是InputStream或是OutputStream，Channel打开后则可以进行读取，写入或是读写。

    由于Channel 是双向的， 因此它能更好地反映底层操作系统的真实情况；
    在Linux系统中， 底层操作系统的通道就是双向系统。

#### 示例1
```java
public static void main(String[] args){
    IntBuffer buffer = IntBuffer.allocate(10);

    for(int i = 0; i < buffer.capacity(); i++){
        int randomNumber = new SecureRandom().nextInt(20);
        buffer.put(randomNumber);
    }
    buffer.flip();
    while(buffer.hasRemaining()){
        System.out.println(buffer.get());
    }
}
```


#### 示例2
        通过NIO读取文件涉及到3个步骤
    1、从FileInputStream 获取到 FileChannel对象
    2、创建Buffer
    3、将数据 从Channel读取到Buffer中。

```java
    public static void main(String[] args) throws Exception{
        FileInputStream fileInputStream = new FileInputStream("nioTest2.txt");
        FileChannel fileChannel = fileInputStream.getChannel();

        ByteBuffer byteBuffer = ByteBuffer.allocate(512);
        fileChannel.read(byteBuffer);

        byteBuffer.flip();

        while(byteBuffer.remaining() > 0){
            byte b = byteBuffer.get();
            System.out.println("character:" + (char)b);
        }

        fileInputStream.close();
    }
```

#### 示例3
```java
    public static void main(String[] args) throws Exception{

        FileOutputStream fileOutputStream = new FileOutputStream("nioTest3.txt");
        FileChannel fileChannel = fileOutputStream.getChannel();

        ByteBuffer byteBuffer = ByteBuffer.allocate(512);
        byte[] message = "hello world welcome, nihao".getBytes();

        for (int i = 0; i < message.length; i++){
            byteBuffer.put(message[i]);
        }

        byteBuffer.flip();
        fileChannel.write(byteBuffer);
        fileOutputStream.close();
    }
```




    绝对方法与相对方法的含义
    1、相对方法，limit 值与position值会在操作时被考虑到。
    2、绝对方法，完全忽略掉limit值与position值。




