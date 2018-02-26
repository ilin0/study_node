### 07-Buffer-Scattering-Gathering.md
    Scattering：把来自于一个Channel读到多个buffer中，它是把第一个读满，再读第二个
    场景：
    网络操作时，自定义协议，第一个传递过来的数据，头长度是10字节，第二个头长度是5字节，第三个请求是body,我们用Buffer的Scattering，将第一组信息读到第一个buffer,第二个读到第二个buffer中，消息体读到第三个buffe中。这样就天然的实现的数据的分门另类。如果信息都读取到一个buffer中，那么还需要解析这个数据。

#### 示例说明
```java
public class NioTest11 {

    public static void main(String[] args) throws Exception {

        ServerSocketChannel ssc = ServerSocketChannel.open();
        InetSocketAddress ica = new InetSocketAddress(8899);
        ssc.socket().bind(ica);

        int messageLength = 2+3+4;

        ByteBuffer[] buffers = new ByteBuffer[3];
        buffers[0] = ByteBuffer.allocate(2);
        buffers[1] = ByteBuffer.allocate(3);
        buffers[2] = ByteBuffer.allocate(4);

        SocketChannel sc = ssc.accept();

        while (true){
            int bytesRead = 0;

            // 如果读到的字节数<小数总的字节数时
            while (bytesRead < messageLength){
                long r = sc.read(buffers);// 接受的是个数组
                bytesRead += r;

                System.out.println("byteRead:" + bytesRead);

                Arrays.asList(buffers).stream().map(buffer -> "position:"+ buffer.position() + ", limit:" + buffer.limit())
                        .forEach(System.out::println);
            }

            Arrays.asList(buffers).forEach(byteBuffer -> byteBuffer.flip());
            long bytesWritten = 0;

            while (bytesWritten < messageLength){
                long r = sc.write(buffers);
                bytesWritten += r;
            }
            Arrays.asList(buffers).forEach(byteBuffer -> byteBuffer.clear());
        }
    }
}
```

    使用 mac  nc 功能与服务端连接发送消息
    ~ nc localhost 8899     | telnet localhost 8899
    12345678  回车占一位信息   |  12345678  回车占一位信息
    能够得到返回的信息  12345678  |  得到服务器返回的信息