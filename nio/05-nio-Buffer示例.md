### 05-nio示例.md
    通过下面这个例子来更好的掌握Buffer的三个重要属性
```java
public class NioTest4 {

    public static void main(String[] args) throws Exception{
        FileInputStream fileInputStream = new FileInputStream("input.txt");
        FileOutputStream fileOutputStream = new FileOutputStream("output.txt");

        FileChannel inputChannel = fileInputStream.getChannel();
        FileChannel outputChannel = fileOutputStream.getChannel();

        // 初始的 l  c  =  1024
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        while(true){
            buffer.clear();// 如果注释这行，会发生什么情况？
            // 如果 没有这行，由于第一次循环p = l，那么不管『inputChannel』是否有内容，当它尝试读取时p就会大于l了，那么就直接返回0，
            // 再次执行到 buffer.flip()时，p = 0，这时buffer里的内容又会写入outputChannel 中，那么将一直循环下去。
            int read = inputChannel.read(buffer);//  读到这行  p  就是文件的大小或l 一样 1024

            if(-1 == read){
                break;
            }
            buffer.flip();// 这行之后， p = 0  , l = 1024或 文件大小（文件小于1024时）
            outputChannel.write(buffer); // 当这里写完后  p = l ,

        }

        inputChannel.close();
        outputChannel.close();
    }
}
```

    buffer底层是字节类型，如果存储的是原生类型，那么在们占用的字节数肯定是不一样的，而且放置的是什么类型，取出来肯定也是相应的类型

```java
    public static void main(String[] args) {
        ByteBuffer buffer = ByteBuffer.allocate(64);

        buffer.putInt(15);
        buffer.putLong(345L);
        buffer.putDouble(13.1415);
        buffer.putChar('你');
        buffer.putShort((short)2);

        buffer.flip();

        System.out.println(buffer.getInt()); 
        System.out.println(buffer.getLong());
        System.out.println(buffer.getDouble());
        System.out.println(buffer.getChar());
        System.out.println(buffer.getShort());
    }
```

    ByteBuffer类型化的get与put
    这时候是没有问题的，如果修改一下
```java
    // 第4次get时修改成这样，
    System.out.println(buffer.getInt());
```  
![image](https://github.com/ilin0/study_node/raw/master/nio/image/nio2018022601.png)
![image](https://github.com/ilin0/study_node/raw/master/nio/image/nio2018022602.png)