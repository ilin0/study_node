### tcp 是transfer control protocol的简称
    是一种面向连接的保证可靠传输的协议。
    是通过tcp协议传输，得到的是一个顺序的无差错的数据流
    它是一个基于连接的协议。

### udp 是 user Datagram Protocol的简称
    是一种无连接的协议，每个数据报都是一个独立的信息，包括完整的源地址或目的地址。它是一个非面向连接的协议。

### tcp ip 模型包含4个层次
    应用层、传输层、网络层、网络接口层。

### TCP和UDP的区别，TCP为什么是三次握手，不是两次。
    1. 基于连接与无连接
    2. TCP要求系统资源较多，UDP较少。
    3. UDP程序结构较简单。
    4. 流模式（TCP）与数据报模式(UDP)。
    5. TCP保证数据正确性，UDP可能丢包。
    6. TCP保证数据顺序，UDP不保证。
    为了防止已失效的连接请求报文段突然又传送到了服务端，因而产生错误，即重要的事情说三遍
    url 是统一的资源定位