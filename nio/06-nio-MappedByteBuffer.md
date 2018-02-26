### 06-nio-MappedByteBuffer.md
    是一个直接缓冲区，是一个内存映射区域。
    通过fileChannel.map实现,就是文件的内容在内存中，我们只需要和内存打交道，用于内存映射是堆外内存。

```java
public class NitTest8 {

    public static void main(String[] args) throws Exception {
        RandomAccessFile file = new RandomAccessFile("nioTest9.txt", "rw");

        FileChannel fileChannel = file.getChannel();
        MappedByteBuffer mappedByteBuffer = fileChannel.map(FileChannel.MapMode.READ_WRITE, 0, 5);

        // 这里直接修改后，文件流关闭 文件的内容就发生了变化了
        mappedByteBuffer.put(0, (byte) 'a');
        mappedByteBuffer.put(3, (byte) 'g');

        file.close();
    }
}
```

#### 文件锁的理解
```java
public class NioTest10 {

    public static void main(String[] args) throws Exception {
        RandomAccessFile file = new RandomAccessFile("nioTest10.txt", "rw");
        FileChannel fileChannel = file.getChannel();
        // true 表示共享锁，flase 表示排它锁
        FileLock fileLock = fileChannel.lock(3,6,true);

        System.out.println("valid:" + fileLock.isValid());
        System.out.println("lock type: " + fileLock.isShared());

        fileLock.release();

        fileChannel.close();
    }
}
```