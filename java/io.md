# 网络

## unix网络模型有哪些？
阻塞、轮询非阻塞、多路复用、信号驱动和异步(AIO)。

Linux系统2.6版本后的epoll系统调用支持多路复用模型。相对于select系统调用的优点包括：支持一个进程打开的FD不受限制，IO效率不会因FD数量过大而下降(适合活跃连接数比重小的情况，只有活跃的连接才会回调)，使用mmap加速内核与用户空间的消息传递(内核和用户空间映射为同一块内存)，api更简单。

JDK7的NIO2.0有3个改进：提供批量获取文件属性的API和标准文件系统的SPI，提供AIO功能，完成通道功能(支持配置和多播数据报)。

## IO模型有哪些？

同步阻塞IO(BIO)模型：由一个独立的Acceptor线程监听客户端的连接，接收到客户端的连接请求后为每个客户端创建一个新的线程进行链路处理，处理完成，通过输出流返回后，线程销毁。缺点是内存占用与并发数和处理时间成正比。

伪异步IO(同步)模型：客户端的socket连接请求被封装为任务，使用线程池处理。优点是避免了内存耗尽的问题，但底层通信仍是同步阻塞模型。

非阻塞(同步的NIO多路复用)模型：Channel注册到多路复用器Selector上，发生读写事件的Chennel会处于就绪状态，被Selector不断轮询出来，通过SelectionKey获取就绪Channel的集合，进行后续的IO操作。优点是异步的连接和读写，同时处理上万连接。

异步IO(AIO)模型：系统直接回调，不需要Selector. Windows系统支持，Linux底层暂时不支持。

## Reactor模型是什么？

## TCP保证可靠传输的三次握手过程是什么？

在TCP的连接中，数据流必须以正确的顺序送达对方。TCP的可靠性是通过顺序编号和确认（ACK）来实现的。TCP 连接是通过三次握手进行初始化的。三次握手的目的是同步连接双方的序列号和确认号并交换 TCP 窗口大小信息。第一次是客户端发起连接；第二次表示服务器收到了客户端的请求；第三次表示客户端收到了服务器的反馈。

## 什么是一致性哈希？

常用的hash算法有哪些？

1. 加法hash：所谓的加法Hash就是把输入元素一个一个的加起来构成最后的结果。

2. 位运算hash：这类型Hash函数通过利用各种位运算（常见的是移位和异或）来充分的混合输入元素。

3. 乘法hash：33*hash + key.charAt(i)。

## JDK的io

字符流、字节流、输入流、输出流

## NIO的特性/NIO与IO区别

1)Non-blocking IO（非阻塞IO）

IO流是阻塞的，NIO流是不阻塞的。

Java NIO使我们可以进行非阻塞IO操作。比如说，单线程中从通道读取数据到buffer，同时可以继续做别的事情，当数据读取到buffer中后，线程再继续处理数据。写数据也是一样的。另外，非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。

Java IO的各种流是阻塞的。这意味着，当一个线程调用 read() 或 write() 时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了
2)Buffer(缓冲区)

IO 面向流(Stream oriented)，而 NIO 面向缓冲区(Buffer oriented)。

Buffer是一个对象，它包含一些要写入或者要读出的数据。在NIO类库中加入Buffer对象，体现了新库与原I/O的一个重要区别。在面向流的I/O中·可以将数据直接写入或者将数据直接读到 Stream 对象中。虽然 Stream 中也有 Buffer 开头的扩展类，但只是流的包装类，还是从流读到缓冲区，而 NIO 却是直接读到 Buffer 中进行操作。

在NIO厍中，所有数据都是用缓冲区处理的。在读取数据时，它是直接读到缓冲区中的; 在写入数据时，写入到缓冲区中。任何时候访问NIO中的数据，都是通过缓冲区进行操作。

最常用的缓冲区是 ByteBuffer,一个 ByteBuffer 提供了一组功能用于操作 byte 数组。除了ByteBuffer,还有其他的一些缓冲区，事实上，每一种Java基本类型（除了Boolean类型）都对应有一种缓冲区。
3)Channel (通道)

NIO 通过Channel（通道） 进行读写。

通道是双向的，可读也可写，而流的读写是单向的。无论读写，通道只能和Buffer交互。因为 Buffer，通道可以异步地读写。
4)Selectors(选择器)

NIO有选择器，而IO没有。

选择器用于使用单个线程处理多个通道。因此，它需要较少的线程来处理这些通道。线程之间的切换对于操作系统来说是昂贵的。 因此，为了提高系统效率选择器是有用的。

## NIO 读数据和写数据方式是什么？

通常来说NIO中的所有IO都是从 Channel（通道） 开始的。

- 从通道进行数据读取 ：创建一个缓冲区，然后请求通道读取数据。
- 从通道进行数据写入 ：创建一个缓冲区，填充数据，并要求通道写入数据。

## NIO核心组件简单介绍

NIO 包含下面几个核心的组件：

- Channel(通道)
- Buffer(缓冲区)
- Selector(选择器)

## http/1.0 http/1.1 http/2之间的区别

## cookie被禁用，如何实现session

## 用Java写一个简单的静态文件的HTTP服务器

## 什么是DNS 、记录类型:A记录、CNAME记录、AAAA记录等

#### Servlet

生命周期

线程安全问题
