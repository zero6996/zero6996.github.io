---
title: NIO
date: 2020-04-09 21:00
categories: JavaAdvanced
tags: [NIO]
description: NIO基础，相关API及demo
urlname: NIO
---





# NIO相关API

## 1. NIO与IO的区别

Java NIO(New IO)是从Java1.4 版本开始引入的一个新的IO API，可以代替标准的Java IO API。

NIO与原来的IO有同样的作用和目的，但是使用方式完全不同，NIO支持面向缓冲区的、基于通道的IO操作。NIO将以更加高效的方式进行文件读写操作。

### 1.1 Java NIO与IO的主要区别

| IO                      | NIO                         |
| ----------------------- | --------------------------- |
| 面向流(Stream Oriented) | 面向缓冲区(Buffer Oriented) |
| 阻塞IO(Blocking IO)     | 非阻塞IO(Non Blocking IO)   |
| 无                      | 选择器(Selectors)           |

### 1.2 通道和缓冲区

Java NIO系统的核心在于：通道(Channel)和缓冲区(Buffer)。通道表示打开到IO设备(例如文件、套接字)的连接。若需使用NIO系统，需要获取用于连接IO设备的通道以及用于容纳数据的缓冲区。然后操作缓冲区，对数据进行处理。

> 简言之：Channel负责传输，Buffer负责存储。

## 2. 缓存区(Buffer)的数据存取

缓冲区是一个用于特定基本数据类型的容器，由`java.nio`包定义的，所有缓冲区都是Buffer抽象类的子类。

在Java NIO中负责数据的存取，缓冲区就是数组，存储不同数据类型的数据。根据数据类型的不同(除boolean外)，提供了相应类型的缓冲区，以下是Buffer常用子类：

- ByteBuffer
- CharBuffer
- ShortBuffer
- IntBuffer
- LongBufer
- FloatBuffer
- DoubleBuffer

上述缓冲区的管理方式几乎一致，通过`allocate()`获取缓冲区

### 2.1 缓冲区操作数据的两个核心方法

Buffer所有子类提供了两个用于数据操作的方法，get和put；

- 获取Buffer中的数据
  - `get()`：读取单个字节；
  - `get(byte[] dst)`：批量读取多个字节到dst中；
  - `get(int index)`：读取指定索引位置的字节。

- 存放数据到Buffer中
  - `put(byte b)`：将给定的单个字节写入到缓冲区的当前位置；
  - `put(byte[] src)`：将src中的字节写入到缓冲区的当前位置；
  - `put(int index,byte b)`：将指定字节写入到缓冲区的索引位置。

### 2.2 缓冲区的四个核心属性

- capacity(容量)：表示缓冲区中最大存储数据的容量，一旦声明不可改变；
- limit(界限)：表示缓冲区中可以操作数据的大小，位于limit后的数据不可读写，不能大于容量；
- position(位置)：表示缓冲区中正在操作数据的位置，不能大于界限；
- mark(标记)与重置(reset)：标记是一个索引，表示记录当前position的位置，可以通过`reset()`恢复到mark的位置。

> 以上属性必须遵循：0<=mark <= position <= limit <= capacity

### 2.3 Buffer常用方法

| 方法                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `int capacity()`         | 返回Buffer的容量大小；                                       |
| `int limit()`            | 返回Buffer的界限位置；                                       |
| `int position()`         | 返回缓冲区的当前位置；                                       |
| `Buffer limit(int n)`    | 将缓存区界限设置为n，并返回一个具有新limit的缓冲区对象；     |
| `Buffer position(int n)` | 将缓存区的当前位置设置为n，并返回修改后的Buffer对象；        |
| `Buffer mark()`          | 对缓冲区设置标记；                                           |
| `Buffer reset()`         | 将当前位置position转到以前设置的mark所在位置；               |
| `Buffer flip()`          | 将缓冲区界限设为当前位置，并将当前位置设为0，就是切换到读模式； |
| `Buffer clear()`         | 清空缓冲区并返回对缓冲区的引用；                             |
| `boolean hasRemaining()` | 判断缓冲区中是否还有元素；                                   |
| `int remaining()`        | 返回position和limit之间的元素个数，就是返回剩余可读元素个数； |
| `Buffer rewind()`        | 将当前位置设为0，并取消设置的mark。                          |

### 2.4 代码案例

```java
public class TestBuffer {
    @Test
    public void test1(){
        String str = "abcde";
        // 1. 分配一个指定大小的缓冲区
        ByteBuffer buf = ByteBuffer.allocate(1024);
        showBufferInfo(buf);
        // 2. 使用put()存入数据到缓冲区中
        System.out.println("==========put()===========");
        buf.put(str.getBytes());
        showBufferInfo(buf);
        // 3. 切换到读取数据模式,不切换get时会报：BufferUnderflowException，缓冲区溢出异常
        System.out.println("==========flip()==========");
        buf.flip();
        showBufferInfo(buf);
        // 4. 利用get()读取缓冲区中的数据
        System.out.println("===========get()===========");
        byte[] dst = new byte[buf.limit()];
        buf.get(dst);
        System.out.println(new String(dst,0,dst.length));
        showBufferInfo(buf);
        // 5. rewind:可重复读
        System.out.println("===========rewind()===========");
        buf.rewind();
        showBufferInfo(buf);
        // 6. 清空缓冲区,缓冲区中的数据依然存在，但是无指针指向，处于“被遗忘”状态
        System.out.println("===========clear()===========");
        buf.clear();
        showBufferInfo(buf);
    }

    @Test
    public void testMark(){
        String str = "abcde";
        // 1. 分配一个指定大小的缓冲区
        ByteBuffer buf = ByteBuffer.allocate(1024);
        buf.put(str.getBytes());
        buf.flip();
        byte[] dst = new byte[buf.limit()];
        buf.get(dst,0,2);
        System.out.println(new String(dst,0,2));
        System.out.println("当前指针位置:"+buf.position());

        // mark:标记位置
        System.out.println("==========use mark()===========");
        buf.mark();
        buf.get(dst,2,2);
        System.out.println(new String(dst,2,2));
        System.out.println("当前指针位置:"+buf.position());
        // reset():恢复到mark位置
        System.out.println("===========use reset()===========");
        buf.reset();
        System.out.println("当前指针位置:"+buf.position());
        // 判断缓冲区中是否还有剩余可操作数据
        if (buf.hasRemaining()){
            // 输出可操作数据的数量
            System.out.println("余剩还可操作:"+buf.remaining());
        }
    }

    public static void showBufferInfo(Buffer buffer){
        System.out.println("当前位置:"+buffer.position());
        System.out.println("界限:"+buffer.limit());
        System.out.println("最大容量:"+buffer.capacity());
    }
}
```

## 3. 直接缓冲区与非直接缓冲区

字节缓冲区要么是直接的，要么是非直接的。如果为直接字节缓冲区，则Java虚拟机会尽最大努力直接在此缓冲区上执行本机I/O操作。也就是说，在每次调用基础操作系统的一个本机I/O操作之前(或之后)，虚拟机都会尽量避免将缓冲区的内容复制到中间缓冲区中(或从中间缓冲区中复制内容)。

### 3.1 非直接缓冲区

非直接缓冲区：通过`allocate()`方法分配缓冲区，将缓冲区建立在JVM的内存中，以下为操作原理图：



![非直接缓冲区](http://yanxuan.nosdn.127.net/b2c1547d89c6ee9b6b0b82d3747e7ab0.png)





### 3.2 直接缓冲区

直接缓冲区：通过`allocateDirect()`方法分配直接缓冲区，将缓冲区建立在物理内存中，可以提高效率。

![UTOOLS1586338196458.png](http://yanxuan.nosdn.127.net/74f44e2c46233b635a3fecbc8ce64ee6.png)

此方法返回的缓冲区进行**分配和取消分配所需成本通常高于非直接缓冲区**。直接缓冲区的内容可以驻留在常规的垃圾回收堆之外，因此，它们对应用程序的内存需求量造成的影响可能并不明显。所以，建议将直接缓冲区主要分配给那些易受基础系统的本机I/O 操作影响的大型、持久的缓冲区。一般情况下，最好仅在直接缓冲区能在程序性能方面带来明显好处时分配它们。

字节缓冲区是直接缓冲区还是非直接缓冲区可通过调用其`isDirect()`方法来确定。提供此方法是为了能够在性能关键型代码中执行显式缓冲区管理。

## 4. 通道(Channel)的原理与获取

通道由`java.nio.channels`包定义，表示IO源与目标打开的连接。Channel类似于传统的“流”，只不过Channel本身不能直接访问数据，而是提供了一个通道用以传输数据，Channel只能与Buffer进行交互。

![Channel](http://yanxuan.nosdn.127.net/1729e787306e62154017bd3a078b1b65.png)

### 4.1 通道的主要实现类

Java为Channel接口提供的最主要实现类如下：

- FileChannel：用于读取、写入、映射和操作文件的通道；
- SocketChannel：通过TCP读写网络中的数据；
- ServerSocketChannel：可以监听新进来的TCP连接，对每个新连接都会创建一个SocketChannel；
- DatagramChannel：通过UDP读写网络中的数据通道。

### 4.2 获取通道

Java针对支持通道的类提供了`getChannel()`方法，支持通道的类如下：

- 本地IO
  - FileInputStream
  - FileOutputStream
  - RandomAccessFile
- 网络IO
  - Socket
  - ServerSocket
  - DatagramSocket

还可使用Files工具类的`newByteChannel()`方法获取字节通道，或使用通道的静态方法`open()`打开并返回指定通道。

## 5. 通道的数据传输与内存映射文件

### 5.1 使用非直接缓冲区完成文件复制

```java
@Test
public void test1() throws Exception {
    long start = System.currentTimeMillis();
    // 获取文件IO流对象
    FileInputStream fis = new FileInputStream("D:\\bigFile.mp4");
    FileOutputStream fos = new FileOutputStream("D:\\test1.mp4");
    // ① 获取通道
    FileChannel inChannel = fis.getChannel();
    FileChannel outChannel = fos.getChannel();
    // ② 分配缓冲区
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    // ③ 将通道中的数据存入缓冲区中
    while (inChannel.read(buffer) != -1){
        // 切换到读数据模式
        buffer.flip();
        // ④ 将缓冲区中的数据写入通道中
        outChannel.write(buffer);
        // 清空缓冲区
        buffer.clear();
    }
    // 释放资源
    outChannel.close();
    inChannel.close();
    fos.close();
    fis.close();
    long end = System.currentTimeMillis();
    System.out.println("耗费时间:" + (end - start)); // 17490
}
```

### 5.2 使用直接缓冲区完成文件的复制(内存映射文件)

直接字节缓冲区还可以通过 FileChannel 的`map()`方法将文件区域直接映射到内存中来创建。该方法返回
MappedByteBuffer 。Java 平台的实现有助于通过JNI(Java本地接口)从本机代码创建直接字节缓冲区。如果以上这些缓冲区 中的某个缓冲区实例指的是不可访问的内存区域，则试图访问该区域不会更改该缓冲区的内容，并且将会在访问期间或稍后的某个时间导致抛出不确定的异常。

```java
@Test
public void test2() throws Exception {
    long start = System.currentTimeMillis();

    FileChannel inChannel = FileChannel.open(Paths.get("D:\\bigFile.mp4"), StandardOpenOption.READ);
    // 创建一个通道，CREATE_NEW:文件不存在则创建，存在则报错
    FileChannel outChannel = FileChannel.open(Paths.get("D:\\test2.mp4"), StandardOpenOption.WRITE,StandardOpenOption.READ,StandardOpenOption.CREATE_NEW);
    // 创建内存映射文件
    MappedByteBuffer inMapBuffer = inChannel.map(FileChannel.MapMode.READ_ONLY, 0, inChannel.size());
    MappedByteBuffer outMapBuffer = outChannel.map(FileChannel.MapMode.READ_WRITE, 0, inChannel.size());
    // 直接对缓冲区进行数据的读写操作
    byte[] dst = new byte[inMapBuffer.limit()];
    inMapBuffer.get(dst);
    outMapBuffer.put(dst);
    // 释放资源
    inChannel.close();
    outChannel.close();
    long end = System.currentTimeMillis();
    System.out.println("耗费时间:" + (end - start)); // 2854
}
```

### 5.3 通道之间的数据传输(直接缓冲区)

```java
@Test
public void test3 ()throws Exception{
    long start = System.currentTimeMillis();
    FileChannel inChannel = FileChannel.open(Paths.get("D:\\bigFile.mp4"), StandardOpenOption.READ);
    FileChannel outChannel = FileChannel.open(Paths.get("D:\\test3.mp4"),StandardOpenOption.WRITE,StandardOpenOption.READ,StandardOpenOption.CREATE);

    // 将数据源从源通道传输到另一通道
    //        inChannel.transferTo(0,inChannel.size(),outChannel);
    outChannel.transferFrom(inChannel,0,inChannel.size());

    inChannel.close();
    outChannel.close();
    long end = System.currentTimeMillis();
    System.out.println("耗费时间:" + (end - start));
}
```

## 6. 分散读取与聚集写入

- 分散读取(Scattering Reads)：指将通道中的读取的数据分散到多个缓冲区中；
- 聚集写入(Gathering Writes)：将多个缓冲区中的数据聚集到通道中。

```java
@Test
public void test4() throws Exception{
    // 一、分散读取
    RandomAccessFile file = new RandomAccessFile("D:\\1.md", "rw");
    // 1. 获取通道
    FileChannel channel1 = file.getChannel();
    // 2. 分配指定大小的缓冲区
    ByteBuffer buf1 = ByteBuffer.allocate(250);
    ByteBuffer buf2 = ByteBuffer.allocate(1024);
    // 3. 分散读取
    ByteBuffer[] bufs = {buf1,buf2};
    channel1.read(bufs);
    // 4. 将所有缓冲区切换至读模式
    for (ByteBuffer byteBuffer:bufs){
        byteBuffer.flip();
    }
    // 5. 打印两个缓冲区中的数据内容
    System.out.println(new String(bufs[0].array(),0,bufs[0].limit()));
    System.out.println("===============================");
    System.out.println(new String(bufs[1].array(),0,bufs[1].limit()));

    // 二、聚集写入
    RandomAccessFile rw = new RandomAccessFile("D:\\2.md", "rw");
    FileChannel channel2 = rw.getChannel();
    channel2.write(bufs);

}
```

## 7. 字符集Charset

Charset字符集对象可以编解码数据，同类型编码的数据只能由同类型解码器解码。

```java
@Test
public void TestCharset() throws Exception{
    // 打印所有字符集
    //        SortedMap<String, Charset> map = Charset.availableCharsets();
    //        Set<Map.Entry<String, Charset>> set = map.entrySet();
    //        for ( Map.Entry<String, Charset> entry : set) {
    //            System.out.println(entry.getKey()+"="+entry.getValue());
    //        }
    Charset cs1 = Charset.forName("GBK");
    // 获取编解码器
    CharsetEncoder ce = cs1.newEncoder();
    CharsetDecoder cd = cs1.newDecoder();
    // 获取字符缓冲区
    CharBuffer cBuf = CharBuffer.allocate(1024);
    cBuf.put("字符串编解码");
    cBuf.flip();
    // 编码
    ByteBuffer bBuf = ce.encode(cBuf);
    for (int i = 0; i < 12; i++) {
        System.out.println(bBuf.get());
    }
    bBuf.flip();
    // 解码
    CharBuffer cb = cd.decode(bBuf);
    System.out.println(cb.toString());
    System.out.println("========GBK编码，UTF-8解码========");
    Charset u8 = Charset.forName("UTF-8");
    bBuf.flip();
    CharBuffer decode = u8.decode(bBuf);
    System.out.println(decode.toString());

}
```

> 注意：编解码前，要将缓冲区设置为读模式，不然读取不到数据。

## 8. 阻塞与非阻塞

### 8.1 阻塞

传统的IO流都是阻塞式的。也就是说，当一个线程调用`read()`或`write()`时，该线程会被阻塞，直到有些数据被读取或写入，该线程在此期间不能执行其他任务。

因此，在完成网络通信进行IO操作时，由于线程会阻塞，所以服务器端必须为每个客户端都提供一个独立的线程进行处理，当服务器端需要处理大量客户端请求时，性能会急剧下降。

### 8.2 非阻塞

Java NIO是非阻塞模式的。当线程从某通道进行读写数据时，若没有数据可用时，该线程可以进行其他任务。线程通常将非阻塞IO的空闲时间用于在其他通道上执行IO操作，所以单独的线程可以管理多个输入和输出通道。因此，NIO可以让服务器端使用一个或少数几个线程来同时处理连接到服务器端的所有客户端。

### 8.3 选择器(Selector)

选择器是SelectableChannle对象的多路复用器，Selector 可 以同时监控多个 SelectableChannel 的IO 状况，也就是说，利用 Selector  可使一个单独的线程管理多个 Channel。**Selector 是非阻塞IO 的核心。**

#### 8.3.1 选择器的应用

- 创建Selector：通过调用`Selector.open()`方法创建一个Selector。
- 向选择器注册通道：`SelectableChannel.register(Selector sel,int ops)`
- Selector的常用方法如下：

|            方法            | 描述                                                         |
| :------------------------: | :----------------------------------------------------------- |
| `Set<SelectionKey> keys()` | 所有的 SelectionKey 集合。代表注册在该Selector上的Channel    |
|       selectedKeys()       | 被选择的 SelectionKey 集合。返回此Selector的已选择键集       |
|        int select()        | 监控所有注册的Channel，当它们中间有需要处理的 IO 操作时， 该方法返回，并将对应得的 SelectionKey 加入被选择的 SelectionKey 集合中，该方法返回这些 Channel 的数量。 |
|  int select(long timeout)  | 可以设置超时时长的 select() 操作                             |
|      int selectNow()       | 执行一个立即返回的 select() 操作，该方法不会阻塞线程         |
|     Selector wakeup()      | 使一个还未返回的 select() 方法立即返回                       |
|        void close()        | 关闭该选择器                                                 |

#### 8.3.2 SelectionKey

SelectionKey表示SelectableChannel 和 Selector 之间的注册关系。每次向选择器注册通道时就会选择一个事件(选择键)。选择键包含两个表示为整数值的操作集。操作集的每一位都表示该键的通道所支持的一类可选择操 作。  

  - 当调用`register(Selector sel,int ops)`将通道注册选择器时，选择器对通道的监听事件，需要通过第二个参数ops指定。
- 可以监听的事件类型(可使用SelectionKey的四个常量表示)：
  - 读：SelectionKey.OP_READ(1)
  
  - 写：SelectionKey.OP_WRITE(4)
  
  - 连接：SelectionKey.OP_CONNECT(8)
  
  - 接收：SelectionKey.OP_ACCEPT(16)
  
- 若注册时不止监听一个时间，则可以使用“位或”操作符连接：`SelectionKey.OP_READ|SelectionKey.OP_WRITE`

- SelectionKey的常用方法

|            方法             |               描述               |
| :-------------------------: | :------------------------------: |
|      int interestOps()      |        获取感兴趣事件集合        |
|       int readyOps()        | 获取通道已经准备就绪的操作的集合 |
| SelectableChannel channel() |           获取注册通道           |
|     Selector selector()     |            返回选择器            |
|    boolean isReadable()     |   检测Channel中读事件是否就绪    |
|    boolean isWritable()     |   检测Channel中写事件是否就绪    |
|   boolean isConnectable()   |    检测Channel中连接释放就绪     |
|   boolean isAcceptable()    |    检测Channel中接收是否就绪     |

### 8.4 SocketChannel

- SocketChannel：在Java NIO中SocketChannel是一个连接到TCP网络套接字的通道。
- ServerSocketChannel ：Java NIO中的 ServerSocketChannel 是一个可以 监听新进来的TCP连接的通道，就像标准IO中 的ServerSocket一样。

## 9. 阻塞式

网络通信有以下几个核心：

- 通道：负责传输数据，来自于`java.nio.channels.Channel`接口，主要有以下几个实现类
  - SocketChannel
  - ServerSocketChannel
  - DatagramChannel
- 缓冲区：负责数据的存取
- 选择器(Selector)：是SelectableChannel的多路复用器，用于监控SelectableChannel的IO状况

### 9.1 阻塞式简单案例

```java
public class TestBlockingNIO {
    // 客户端
    @Test
    public void client() throws Exception{
        // 1. 获取网络通道
        SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1",8899));
        // 2. 打开本地文件通道
        FileChannel fileChannel = FileChannel.open(Paths.get("D:\\IDEADemos\\Interview\\src\\Java基础\\Nio\\1.jpg"), StandardOpenOption.READ);
        // 3. 分配缓冲区
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        // 4. 读取本地文件，通过网络通道发送到服务端
        while (fileChannel.read(buffer) != -1){
            buffer.flip();
            socketChannel.write(buffer);
            buffer.clear();
        }

        // 传统IO模式下，传输完毕,需手动关闭连接
        socketChannel.shutdownOutput();

        // 5. 接收来自服务器端的反馈
        int len = 0;
        while ((len = socketChannel.read(buffer)) != -1){
            buffer.flip();
            System.out.println(new String(buffer.array(),0,len));
            buffer.clear();
        }

        // 6. 释放资源
        fileChannel.close();
        socketChannel.close();
    }

    // 服务端
    @Test
    public void server() throws Exception{
        System.out.println("服务器端已启动...监听中");
        // 1. 打开服务器网络通道
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        // 2. 绑定端口号
        serverSocketChannel.bind(new InetSocketAddress(8899));
        // 3. 获取客户端连接的通道
        SocketChannel socketChannel = serverSocketChannel.accept();
        // 4. 打开本地文件通道
        FileChannel fileChannel = FileChannel.open(Paths.get("D:\\IDEADemos\\Interview\\src\\Java基础\\Nio\\4.jpg"), StandardOpenOption.WRITE, StandardOpenOption.CREATE);
        // 5. 分配缓冲区
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        // 6. 接收客户端的数据，并保存到本地
        while (socketChannel.read(buffer) != -1){
            buffer.flip();
            fileChannel.write(buffer);
            buffer.clear();
        }

        // 7. 发送反馈给客户端
        buffer.put("服务器接收数据成功！".getBytes());
        buffer.flip();
        socketChannel.write(buffer);

        // 8. 释放资源
        fileChannel.close();
        socketChannel.close();
        serverSocketChannel.close();
        System.out.println("服务器关闭...");
    }
}
```

## 10. 非阻塞式

- Java NIO Demo如下：

```java
public class TestNonBlockingNIO {

    // 客户端
    @Test
    public void client() throws Exception{
        // 1. 获取网络通道
        SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1",8888));
        // 2. 切换到非阻塞模式
        socketChannel.configureBlocking(false);
        // 3. 分配缓冲区
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        // 4. 发送数据
        Scanner scan = new Scanner(System.in);
        while (scan.hasNext()){
            String str = scan.next();
            buffer.put((new Date().toString() + "\n" + str).getBytes());
            buffer.flip();
            socketChannel.write(buffer);
            buffer.clear();
        }
        socketChannel.close();
    }

    // 服务端
    @Test
    public void server() throws Exception{
        System.out.println("服务器端已启动...监听中");
        // 1. 打开服务器网络通道
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        // 2. 切换到非阻塞模式
        serverSocketChannel.configureBlocking(false);
        // 3. 绑定端口号
        serverSocketChannel.bind(new InetSocketAddress(8888));
        // 4. 获取选择器
        Selector selector = Selector.open();
        // 5. 将通道注册到选择器里，并指定“监听接收事件”
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        // 6. 轮询的获取选择器中已经“准备就绪”的事件
        while (selector.select() > 0){
            // 7. 获取选择器中所有的“准备就绪”事件迭代器
            Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
            while (iterator.hasNext()){
                // 8. 获取准备就绪的事件
                SelectionKey key = iterator.next();
                // 9. 判断具体是什么事件准备就绪
                if (key.isAcceptable()){
                    // 10. 若是“接收就绪”，则获取客户端连接
                    SocketChannel socketChannel = serverSocketChannel.accept();
                    // 11. 将客户端连接通道也切换为非阻塞模式
                    socketChannel.configureBlocking(false);
                    // 12. 将该通道注册到选择器中
                    socketChannel.register(selector,SelectionKey.OP_READ);
                }else if (key.isReadable()){
                    // 13. 获取当前选择器上“读就绪”状态的通道
                    SocketChannel sChannel = (SocketChannel) key.channel();
                    // 14. 分配缓冲区
                    ByteBuffer buffer = ByteBuffer.allocate(1024);
                    int len = 0;
                    while ((len = sChannel.read(buffer)) > 0){
                        buffer.flip();
                        System.out.println(new String(buffer.array(),0,len));
                        buffer.clear();
                    }
                }
                // 15. 取消已使用的选择键
                iterator.remove();
            }
        }
        System.out.println("服务器关闭...");
    }
}
```

### 10.1 关于IDEA的Junit环境下无法使用Scanner

打开`Help`->`Edit Custom VM Options...`，在最后加上一句：`-Deditable.java.test.console=true`，然后重启IDEA即可。

## 11. DatagramChannel

lJava NIO中的DatagramChannel是一个能收发UDP包的通道。

### 11.1 简单案例

```java
public class TestNonBlockingNIO2 {

    @Test
    public void send() throws Exception{
        DatagramChannel datagramChannel = DatagramChannel.open();
        datagramChannel.configureBlocking(false);
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        Scanner scan = new Scanner(System.in);
        while (scan.hasNext()){
            String str = scan.next();
            buffer.put((new Date().toString()+":\n"+str).getBytes());
            buffer.flip();
            datagramChannel.send(buffer,new InetSocketAddress("127.0.0.1",8899));
            buffer.clear();
        }
        datagramChannel.close();
    }

    @Test
    public void receive() throws Exception{
        DatagramChannel datagramChannel = DatagramChannel.open();
        datagramChannel.configureBlocking(false);
        datagramChannel.bind(new InetSocketAddress(8899));
        Selector selector = Selector.open();
        datagramChannel.register(selector,1);
        while (selector.select()>0){
            Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
            while (iterator.hasNext()){
                SelectionKey key = iterator.next();
                if (key.isReadable()){
                    ByteBuffer buffer = ByteBuffer.allocate(1024);
                    datagramChannel.receive(buffer);
                    buffer.flip();
                    System.out.println(new String(buffer.array(),0,buffer.limit()));
                    buffer.clear();
                }
            }
            iterator.remove();
        }
    }
}

```

## 12. Pipe管道

lJava NIO 管道是2个线程之间的单向数据连接。Pipe有一个source通道和一个sink通道。数据会被写到sink通道，从source通道读取。

![Pipe](http://yanxuan.nosdn.127.net/9f9acc38a52ad45ebe1b9abb4b4078d2.png)

### 12.1 通过管道传输数据

- 通过`pipe.sink()`返回一个入口通道，写数据；
- 从管道读取数据，需要访问source通道，使用`pipe.source()`；

```java
public class TestPipe {
    public static void main(String[] args) throws Exception{
        // 1. 获取管道
        Pipe pipe = Pipe.open();
        // 2. 分配缓冲区
        ByteBuffer buf = ByteBuffer.allocate(1024);
        // 3. 将缓冲区中的数据写入管道
        Pipe.SinkChannel sink = pipe.sink();
        buf.put("通过单向管道发送数据".getBytes());
        buf.flip();
        sink.write(buf);

        // 4. 读取缓冲区中的数据
        Pipe.SourceChannel source = pipe.source();
        buf.flip();
        int len = source.read(buf);
        System.out.println(new String(buf.array(),0,len));

        // 5. 释放资源
        source.close();
        sink.close();
    }
}
```

## 13. BIO、AIO、NIO的区别

### 13.1 同步和异步的概念

同步和异步是针对应用程序和内核交互而言的，同步是指用户进程触发I/O操作并等待或轮询去查看I/O操作是否就绪；而异步是指用户进程触发I/O操作以后便开始做自己的事情，而当I/O操作完成时会得到I/O完毕的通知。

### 13.2 阻塞和非阻塞的概念

阻塞和非阻塞是针对于进程在访问数据时，根据I/O操作的就绪状态来采取不同的方式，简单说就是读取或写入操作函数的一种实现方式；阻塞方式下读写函数一直等待，而非阻塞方式下，读写函数会立即返回一个状态值。

### 13.3 三类IO

所以，I/O操作可分为3类：同步阻塞(即早期的BIO操作)、同步非阻塞(NIO)、异步非阻塞(AIO)。

- 同步阻塞(Block IO，简称BIO)
  - 此种方式下，用户进程在发起一个I/O操作后，必须等待I/O操作的完成，只有当真正完成了I/O操作之后，用户进程才能运行。Java传统的I/O模式就属于此种方式，特点是模式简单实用方便，但并发处理能力低。
- 同步非阻塞(New IO，简称NIO)
  - 在此种方式下，用户进程发起一个I/O操作后便可以返回做其他事情了，但用户进程需要时不时地去询问I/O操作是否就绪，这就要求用户周期性地去询问，从而引入不必要的CPU资源浪费。目前Java的NIO就属同步非阻塞IO，是传统IO的升级，实现了多路复用，但会占用CPU资源。
- 异步非阻塞(Asynchronous IO，简称AIO)
  - 此种方式下，用户进程发起一个I/O操作后，不必等待内核I/O操作完成，内核完成I/O操作后会主动通知用户进程。此时用户进程只需对数据进行处理即可，无需进行实际的I/O读写操作，真正的I/O读写操作已经由内核完成了。

