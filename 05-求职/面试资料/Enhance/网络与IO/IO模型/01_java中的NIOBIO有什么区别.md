# java中的NIO/BIO有什么区别

Java中的BIO和NIO是两种不同的IO模型，它们在处理网络通信和文件操作时有着根本性的区别：

## BIO (Blocking I/O) - 阻塞IO

**核心特点：**

- **同步阻塞模型**：线程在执行IO操作时会被阻塞，直到操作完成才能继续
- **一对一线程模型**：每个客户端连接需要一个独立的线程来处理
- **资源消耗大**：大量连接会导致线程数量激增，消耗大量内存和CPU资源

**工作原理：**

```java
// BIO服务器示例
ServerSocket serverSocket = new ServerSocket(8080);
while (true) {
    Socket clientSocket = serverSocket.accept(); // 阻塞等待连接
    // 为每个客户端创建新线程
    new Thread(() -> {
        try {
            InputStream input = clientSocket.getInputStream();
            byte[] buffer = new byte[1024];
            int bytesRead = input.read(buffer); // 阻塞读取
            // 处理数据...
        } catch (IOException e) {
            e.printStackTrace();
        }
    }).start();
}
```

## NIO (Non-blocking I/O) - 非阻塞IO

**核心特点：**

- **同步非阻塞模型**：线程不会被IO操作阻塞，可以处理其他任务
- **多路复用**：一个线程可以管理多个连接通道
- **基于缓冲区**：使用Buffer进行数据操作，更高效
- **选择器机制**：通过Selector监控多个Channel的状态

**三大核心组件：**

1. **Channel（通道）**：数据传输的载体，类似于流但支持双向操作
2. **Buffer（缓冲区）**：数据容器，所有数据都通过Buffer处理
3. **Selector（选择器）**：多路复用器，可以监听多个Channel的IO事件

**工作原理：**

```java
// NIO服务器示例
Selector selector = Selector.open();
ServerSocketChannel serverChannel = ServerSocketChannel.open();
serverChannel.configureBlocking(false);
serverChannel.bind(new InetSocketAddress(8080));
serverChannel.register(selector, SelectionKey.OP_ACCEPT);

while (true) {
    selector.select(); // 非阻塞选择就绪的Channel
    Iterator<SelectionKey> keys = selector.selectedKeys().iterator();
    
    while (keys.hasNext()) {
        SelectionKey key = keys.next();
        keys.remove();
        
        if (key.isAcceptable()) {
            // 处理新连接
        } else if (key.isReadable()) {
            // 处理读事件
        }
    }
}
```

## 主要区别对比

|特性|BIO|NIO|
|---|---|---|
|**阻塞性**|阻塞式|非阻塞式|
|**线程模型**|一连接一线程|一线程处理多连接|
|**资源消耗**|高（线程开销大）|低（线程复用）|
|**适用场景**|连接数少、处理简单|连接数多、高并发|
|**编程复杂度**|简单直观|相对复杂|
|**性能**|连接数少时性能好|高并发时性能优异|

## 扩展知识

### AIO (Asynchronous I/O)

Java 7引入了AIO，也称为NIO 2.0，是真正的异步IO模型。与NIO不同，AIO的读写操作是异步的，不需要通过Selector轮询。

### 选择建议

- **BIO**：适用于连接数较少（<1000）、连接持续时间较短的场景
- **NIO**：适用于连接数多、连接持续时间长的高并发场景
- **AIO**：适用于连接数多且连接持续时间长，且需要真正异步处理的场景

### 实际应用

- **Tomcat**：早期版本使用BIO，后续版本默认使用NIO
- **Netty**：基于NIO构建的高性能网络应用框架
- **Redis**：使用单线程+多路复用的IO模型

这些IO模型的理解对于构建高性能的Java应用程序至关重要，特别是在处理网络通信和大量并发连接时。