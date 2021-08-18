---
title: Java NIO通信基础
header-img: /img/header_img/squirrel.jpg
catalog: true
top:
tags:
  - IO
categories:
  - work
abbrlink: 57bd
date: 2021-03-31 18:45:33
subtitle:
---

## Java NIO通信基础

### 一、NIO简介

> 在1.4版本之前，Java IO类库是阻塞IO；从1.4版本开始，为了支持非阻塞IO，引进了新的异步IO库，被称为Java New IO类库，简称为JAVA NIO。Java NIO属于 IO多路复用模型。

**NIO和OIO（old IO）的区别：**

- OIO面向流（Stream Oriented），NIO面向缓冲区（BufferOriented）
- OIO的操作是阻塞的，NIO是非阻塞的。
- OIO没有选择器（Selector）概念，NIO的选择器，需要底层操作系统提供支持。

Java NIO由以下三个核心组件组成：· Channel（通道）· Buffer（缓冲区）· Selector（选择器）

#### 1. 通道（Channel）

​		在OIO中，同一个网络连接会通过输入流（Input Stream）和输出流（Output Stream）不断地进行输入和输出的操作。在NIO中，同一个网络连接使用一个通道表示，所有的NIO的IO操作都是从通道开始的。一个通道类似于OIO中的两个流的结合体，既可以从通道读取，也可以向通道写入。

#### 2. 选择器（Selector）

> IO多路复用指的是一个进程/线程可以同时监视多个文件描述符（一个网络连接，操作系统底层使用一个文件描述符来表示），一旦其中的一个或者多个文件描述符可读或者可写，系统内核就通知该进程/线程。

​		通过选择器，一个线程可以查询多个通道的IO事件的就绪状态，即监视多个文件描述符。具体的开发层面来说，首先把通道注册到选择器中，然后通过选择器内部的机制，可以查询（select）这些注册的通道是否有已经就绪的IO事件（例如可读、可写、网络连接完成等）。一个选择器只需要一个线程进行监控，系统不必为每一个网络连接（文件描述符）创建进程/线程，从而大大减小了系统的开销。

#### 3. 缓冲区（Buffer）

​		应用程序与通道（Channel）主要的交互操作，就是进行数据的read读取和write写入。通道的读取，就是将数据从通道读取到缓冲区中；通道的写入，就是将数据从缓冲区中写入到通道中。

### 二、Buffer类及其属性

> Buffer（缓冲区）本质上是一个内存块（数组），既可以写入数据，也可以从中读取数据。

#### Buffer类

​		Buffer类是一个抽象类，位于java.nio包中，线程不安全。在NIO中有8种缓冲区类：ByteBuffer、CharBuffer、DoubleBuffer、FloatBuffer、IntBuffer、LongBuffer、ShortBuffer、MappedByteBuffer，其中MappedByteBuffer是专门用于内存映射的一种ByteBuffer类型。使用最多的是ByteBuffer。

#### Buffer类的重要属性

​		为了记录读写的状态和位置，Buffer类提供了一些重要的属性：capacity（容量）、position（读写位置）、limit（读写的限制）、mark（标记）。

![image](https://tvax2.sinaimg.cn/large/006YzKDNly1gp33iz38e4j30kk05pt9v.jpg)

#### Buffer类的重要方法

1. **allocate()创建缓冲区**

    ​		为了获取一个Buffer实例对象，并不是使用子类的构造器new来创建一个实例对象，而是调用子类的allocate()方法，并分配内存空间（capacity）。

    ```java
    IntBuffer intBuffer = intBuffer.allocate(10);
    ```

2. **put()写入到缓冲区**

    ```java
    intBuffer.put(1);
    ```

3. **flip()翻转**

    ​		向缓冲区写入数据之后，不能直接从缓冲区中读取数据，需要使用flip()将写入模式翻转成读取模式。flip()方法源码如下：

    ```java
     public final Buffer flip() {
         limit = position;
         position = 0;
         mark = -1;
         return this;
     }
    ```

    容量为10的intBuffer，在写入1个数据时，position=1，limit=10，capacity=10；在写完翻转后，position=0，limit=1，capacity=10；

    > 读取完成后，如何再一次将缓冲区切换成写入模式呢？可以调用Buffer.clear()清空或者Buffer.compact()压缩方法，它们可以将缓冲区转换为写模式。

    缓冲区读写模式的转换如图：

    ![image](https://tvax1.sinaimg.cn/large/006YzKDNly1gp34tey3npj30g1071aay.jpg)

4. **clear()清空**

    ​		清空缓冲区但不清除数据，数据将“被遗忘”，缓冲区切换为写入模式。源码如下：

    ```java
    public final Buffer clear() {
        position = 0;
        limit = capacity;
        mark = -1;
    	return this;
    }
    ```

5. **compact()压缩**

    ​		不覆盖未读的数据，将所有未读的数据拷贝到Buffer起始处，然后将position设到最后一个未读元素后面，limit设置为capacity，缓冲区切换为写入模式。源码如下：

    ```java
    public ByteBuffer compact() {
        System.arraycopy(hb, ix(position()), hb, ix(0), remaining()); // 拷贝未读数据
        position(remaining()); // remaining()返回limit - position
        limit(capacity()); // limit设置为capacity
        discardMark(); // mark = -1
        return this;
    }
    ```

6. **get()从缓冲区读取**

    ​		翻转后可读，读操作会改变可读位置position的值，而limit值不会改变。如果position==limit，表示所有数据读取完成，position指向了一个没有数据的元素位置，此时再读，会抛出BufferUnderflowException异常。

    ```java
    byteBuffer.get();
    ```

7. **rewind()倒带**

    ​		已读完的数据，如果需要再读一遍，可以调用rewind()方法。rewind()也叫倒带。源码如下：

    ```java
    public final Buffer rewind() {
        position = 0; // 重置
        mark = -1; // 清理标记
        return this;
    }
    ```

    > rewind()和flip()区别在于：rewind()不会影响limit；而flip()会重设limit属性值。

8.  **mark( )和reset( )**

    ​		Buffer.mark()和Buffer.reset()方法是配套使用的，比如读到第3个元素（i= =2时），调用mark()方法，把当前位置position的值保存到mark属性中，这时mark属性的值为2。接下来，就可以调用reset方法，将mark属性的值恢复到position中。然后可以从位置2（第三个元素）开始读。

**Buffer类的基本使用步骤**：

```java
IntBuffer intBuffer = intBuffer.allocate(10);
intBuffer.put(1);
intBuffer.flip();
intBuffer.get();
intBuffer.clear();||intBuffer.compact();
```

### 二、Channel类及其属性

> NIO中一个连接用一个Channel（通道）来表示，一个通道可以表示一个底层的文件描述符，例如硬件设备、文件、网络连接等。对于不同的网络传输协议类型，在Java中都有不同的NIO Channel（通道）实现。

#### Channel（通道）的主要类型

1. FileChannel文件通道，用于文件的数据读写。

2. SocketChannel套接字通道，用于Socket套接字TCP连接的数据读写。

3. ServerSocketChannel服务器嵌套字通道（或服务器监听通道），允许我们监听TCP连接请求，为每个监听到的请求，创建一个SocketChannel套接字通道。

4. DatagramChannel数据报通道，用于UDP协议的数据读写。

#### FileChannel的使用

> 通过FileChannel，既可以从一个文件中读取数据，也可以将数据写入到文件中。FileChannel为**阻塞模式**，不能设置为非阻塞模式。

1. **获取FileChannel通道**

- 通过文件输入输出流获取：

    ```java
    // 创建文件输入流
    FileInputStream fis = new FileInputStream(srcFile);
    // 获取文件流的通道
    FileChannel inChannel = fis.getChannel();
    
    // 创建文件输出流
    FileOutputStream fos = new FileOutputStream(destFile);
    // 获取文件流的通道
    FileChannel outChannel = fis.getChannel();
    ```

- 通过RandomAccessFile类获取：

    ```java
    // 创建RandomAccessFile随机访问对象
    RandomAccessFile rafile = new RandomAccessFile(srcFile, "rw");
    // 获取文件流的通道
    FileChannel raFileChannel = rafile.getChannel();
    ```

2. **读取FileChannel通道**

    ​		调用`public abstract int read(ByteBuffer src) throws IOException`方法读取通道数据，写入ByteBuffer缓冲区，并返回数据。**虽然对于通道来说是读取数据，但是对于ByteBuffer缓冲区来说是写入数据，这时，ByteBuffer缓冲区处于写入模式。**

    ```java
    ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
    int length = -1;
    while (-1 != (length = raFileChannel.read(byteBuffer))) {
    // TODO
    }
    ```

3. **写入FileChannel通道**

    ​		调用`public abstract int write(ByteBuffer src) throws IOException`方法读取缓冲区数据写入通道，并返回写入的字节数。**对于ByteBuffer缓冲区来说是读入数据，对通道来说是写入。**

    ```java
    // 刚写完要翻转成读取模式
    byteBuffer.flip();
    int outlength = 0;
    while (0 != (outlength = raFileChannel.write(byteBuffer))) {
    	System.out.println("写入字节数：" + outlength);
    }
    ```

4. **关闭通道**

    ```java
    inChannel.close();
    ```

5. **强制刷新到磁盘**

    ​		在将缓冲区写入通道时，出于性能原因，操作系统不可能每次都实时将数据写入磁盘。如果需要保证写入通道的缓冲数据，最终都真正地写入磁盘，可以调用FileChannel的force()方法。

    ```java
    inChannel.force(true);
    ```

#### SocketChannel和ServerSocketChannel的使用

> 在NIO中，涉及网络连接的通道有两个，一个是SocketChannel负责连接传输，另一个是ServerSocketChannel负责连接的监听。都**支持阻塞和非阻塞两种模式**。ServerSocketChannel应用于服务器端，而SocketChannel同时处于服务器端和客户端。对于一个连接，两端都有一个负责传输的SocketChannel传输通道。SocketChannel与OIO中的Socket类对应，ServerSocketChannel与OIO中的ServerSocket类对应。

1. **获取SocketChannel传输通道**

    ​		在**客户端**，先通过SocketChannel静态方法open()获得一个套接字传输通道；然后，将socket套接字设置为非阻塞模式；最后，通过connect()实例方法，对服务器的IP和端口发起连接。

    ```java
    // 获取通道
    SocketChannel socketChannel = SocketChannel.open();
    // 设置为非阻塞
    socketChannel.configureBlocking(false);
    // 对服务器的IP和端口发起连接
    socketChannel.connect(new InetSocketAddress(InetAddress.getLocalHost(),5252));
    ```

    ​		非阻塞情况下，与服务器的连接可能还没有真正建立，socketChannel.connect方法就返回了，因此需要不断地自旋，检查当前是否连接到了主机：

    ```java
    // 不断自旋，等待连接完成
    while (!socketChannel.finishConnect()) {
    }
    ```

    ​		在**服务器端**，当新连接事件到来时，ServerSocketChannel能成功地查询，通过accept()方法，来获取新连接的套接字通道：

    ```java
    // 新连接事件到来,通过事件（后面会讲到key）获取服务器监听通道
    ServerSocketChannel serverSocketChannel = （ServerSocketChannel）key.channel();
    // 获取新连接的套接字通道
    SocketChannel socketChannel = serverSocketChannel.accept();
    // 切换为非阻塞模式
    socketChannel.configureBlocking(false);
    ```

2. **读取SocketChannel传输通道**

    ​		与FileChannel一样

3. **写入SocketChannel传输通道**

    ​		与FileChannel一样

4. **关闭SocketChannel传输通道**

    ​		在关闭SocketChannel传输通道前，如果传输通道用来写入数据，则建议调用一次shutdownOutput()终止输出方法，向对方发送一个输出的结束标志（-1）。然后调用socketChannel.close()方法，关闭套接字连接。

    ```
    // 终止输出方法，向对方发送一个输出的结束标志
    socketChannel.shutdownOutput();
    // 关闭套接字连接
    socketChannel.close();
    ```

#### DatagramChannel的使用

> DatagramChannel数据报通道用来处理UDP协议的数据传输。和Socket套接字的TCP传输协议不同，UDP协议不是面向连接的协议。使用UDP协议时，只要知道服务器的IP和端口，就可以直接向对方发送数据。

1. **获取DatagramChannel传输通道**

    ​		调用DatagramChannel静态方法open()获得通道，然后设置为非阻塞模式，绑定数据报的监听端口：

    ```java
    // 获取通道
    DatagramChannel datagramChannel = DatagramChannel.open();
    // 设置为非阻塞
    datagramChannel.configureBlocking(false);		
    // 绑定监听IP和端口
    datagramChannel.socket().bind(new InetSocketAddress(InetAddress.getLocalHost(),5252));
    ```

2. **读取DatagramChannel传输通道**

    ​		不是调用read方法，而是调用receive：

    ```java
    ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
    SocketAddress clientAddr = datagramChannel.receive(byteBuffer);
    ```

    ​		通道读取receive（ByteBuffer buf）方法的返回值，是SocketAddress类型，表示返回发送端的连接地址（包括IP和端口）。

3. **写入DatagramChannel传输通道**

    ​		不是调用write方法，而是调用send方法，由于UDP是面向非连接的协议，因此，在发送数据的时候，需要指定接收方的地址：

    ```java
    byteBuffer.flip();
    datagramChannel.send(byteBuffer, new InetSocketAddress(InetAddress.getLocalHost(),5252));
    byteBuffer.clear();
    ```

4. **关闭DatagramChannel传输通道**

    ​		与FileChannel一样，close()即可。

### 三、Select类及其属性

> 非阻塞模式下，如何知道SocketChannel和DatagramChannel通道何时是可读的呢？这时就需要用到NIO的新组件——Selector通道选择器
>
> 简单地说：**选择器的使命是完成IO的多路复用**。一个通道代表一条连接，通过选择器可以同时监控多个通道的IO（输入输出）状况。选择器和通道的关系，是监控和被监控的关系。

#### Selector 选择器及注册

​		通道和选择器之间的关系，通过`Channel.register（Selector sel, int ops）`方法完成，需要传入待注册的选择器实例和待监控事件类型。

可供选择器监控的**通道IO事件类型**（就绪状态），包括以下四种：

1. 可读：SelectionKey.OP_READ
2. 可写：SelectionKey.OP_WRITE
3. 连接：SelectionKey.OP_CONNECT
4. 接收：SelectionKey.OP_ACCEPT

事件类型的定义在SelectionKey类中。如果选择器要监控通道的多种事件，可以用“按位或”运算符来实现。例如，同时监控可读和可写IO事件：

```java
int key = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```

#### SelectableChannel(可选择通道)

​		一条通道若能被选择，必须继承SelectableChannel类。所有网络链接Socket套接字通道，都继承了SelectableChannel类，都是可选择的。而FileChannel文件通道，并没有继承SelectableChannel，因此不是可选择通道。

#### SelectionKey(选择键)

​		一旦在通道中发生了某些IO事件（就绪状态达成），并且在选择器中注册过，就会被选择器选中，并放入SelectionKey的集合中。SelectionKey可以获得通道的IO事件类型，比方说SelectionKey.OP_READ，还可以获得发生IO事件所在的通道及选择器实例。

#### 选择器的使用

1. **获取选择器实例**

    ​		通过调用静态工厂方法open()来获取：

    ```java
    Selector selector = Selector.open();
    ```

    *open()的内部，是向选择器SPI（SelectorProvider）发出请求，通过默认的SPI对象，获取一个新的选择器实例。SPI全称为（Service Provider Interface，服务提供者接口），是JDK的一种可以扩展的服务提供和发现机制。Java通过SPI的方式，提供选择器的默认实现版本。也就是说，其他的服务提供商可以通过SPI的方式，提供定制化版本的选择器的动态替换或者扩展。*

2. **将通道注册到选择器中**

    ​		需要注意：注册到选择器的通道，必须处于非阻塞模式下，否则将抛出IllegalBlockingModeException异常。并且一个通道，并不一定支持所有的四种IO事件。例如服务器监听通道ServerSocketChannel，仅支持Accept（接收到新连接）IO事件；而SocketChannel传输通道，则不支持Accept（接收到新连接）IO事件。可以在注册之前，通过通道的validOps()方法，来获取该通道所有支持的IO事件集合。

    ```java
    // 获取通道
    ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
    // 设置为非阻塞
    serverSocketChannel.configureBlocking(false);
    // 绑定连接
    serverSocketChannel.bind(new InetSocketAddress(5252));
    System.out.println("服务器启动成功");
    // 将通道注册的“接收新连接”IO事件注册到选择器上
    serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
    ```

3. **选出感兴趣的IO就绪事件（选择键集合）**

    ```java
    // 轮询感兴趣的IO就绪事件（选择键集合）
    while (selector.select() > 0) {
        // 获取选择键集合
        Iterator<SelectionKey> selectedKeys = selector.selectedKeys().iterator();
        while (selectedKeys.hasNext()) {
            // 获取单个的选择键并处理
            SelectionKey selectedKey = selectedKeys.next();
            if (selectedKey.isAcceptable()) {
         		// 通道有新连接
            } else if (selectedKey.isConnectalbe()) {
                // 通道连接成功
            }  else if (selectedKey.isReadalbe()) {
                // 通道可读
            }  else if (selectedKey.isWritalbe()) {
                // 通道可写
            }
            // 处理完后移除选择键
            selectedKeys.remove();
        }
    }
    ```

    SelectionKey集合不能添加元素，如果试图向SelectionKey选择键集合中添加元素，则将抛出java.lang.UnsupportedOperationException异常。

    select()方法有三个重载的实现版本，具体如下：

    1. select()：阻塞调用，一直到至少有一个通道发生了注册的IO事件。
    2. select(long timeout)：和select()一样，但最长阻塞时间为timeout指定的毫秒数。
    3. selectNow()：非阻塞，不管有没有IO事件，都会立刻返回。

    select()方法返回的整数值，表示从上一次select到这一次select之间，有多少**通道**发生了注册的IO事件。强调一下，select()方法返回的数量，指的是通道数，而不是IO事件数。

#### 实践案例

使用NIO实现Discard服务器，Discard服务器的功能很简单，读取客户端通道的输入数据，读取完成后直接关闭客户端通道；并且读取到的数据直接抛弃掉。

**服务器端：**

```java
/**
 * 使用NIO实现Discard服务器端功能：
 *      仅读取客户端通道的输入数据，读取完成后直接关闭客户端通道，并且读取到的数据直接抛弃掉。
 */
public class NioDiscardServer {
    public static void startServer() throws IOException {

        // 获取选择器
        Selector selector = Selector.open();
        // 获取通道
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        // 设置为非阻塞
        serverSocketChannel.configureBlocking(false);
        // 绑定连接
        serverSocketChannel.bind(new InetSocketAddress(5252));
        System.out.println("服务器启动成功");
        // 将通道注册的“接收新连接”IO事件注册到选择器上
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        // 轮询感兴趣的IO就绪事件（选择键集合）
        while (selector.select() > 0) {
            // 获取选择键集合
            Iterator<SelectionKey> selectedKeys = selector.selectedKeys().iterator();
            while (selectedKeys.hasNext()) {
                // 获取单个的选择键并处理
                SelectionKey selectedKey = selectedKeys.next();
                if (selectedKey.isAcceptable()) {
                    // 若选择键的IO事件是“连接就绪”，就获取客户端连接
                    SocketChannel socketChannel = serverSocketChannel.accept();
                    // 切换为非阻塞模式
                    socketChannel.configureBlocking(false);
                    // 将新连接的通道可读事件注册到选择器上
                    socketChannel.register(selector, SelectionKey.OP_READ);
                } else if (selectedKey.isReadable()) {
                    // 若选择键的IO事件是“可读”，就读取数据
                    SocketChannel socketChannel = (SocketChannel) selectedKey.channel();
                    // 读取数据，然后丢弃
                    ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                    int length = 0;
                    // 调用通道的read方法，从通道读取数据写入缓冲区，并返回读取到的数据
                    while ((length = socketChannel.read(byteBuffer)) > 0) {
                        byteBuffer.flip();
                        System.out.println(new String(byteBuffer.array(), 0, length));
                        byteBuffer.clear();
                    }
                    socketChannel.close();
                }
                // 移除选择键
                selectedKeys.remove();
            }
        }
        // 关闭连接
        serverSocketChannel.close();
    }
}
```

**客户端：**

```java
/**
 * 使用NIO实现Discard客户端功能：
 *      客户端首先建立到服务器的连接，发送一些简单的数据，然后直接关闭连接。
 */
public class NioDiscardClient {

    public static void startClient() throws IOException {
        InetSocketAddress address = new InetSocketAddress(InetAddress.getLocalHost(),5252);
        // 获取通道
        SocketChannel socketChannel = SocketChannel.open(address);
        // 设置为非阻塞
        socketChannel.configureBlocking(false);
        // 不断自旋，等待连接完成
        while (!socketChannel.finishConnect()) {
        }
        System.out.println("客户端连接成功");
        // 分配指定大小的缓冲区
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        byteBuffer.put("hello nio".getBytes());
        byteBuffer.flip();
        // 发送到服务器
        socketChannel.write(byteBuffer);
        // 终止输出方法，向对方发送一个输出的结束标志
        socketChannel.shutdownOutput();
        // 关闭套接字连接
        socketChannel.close();
    }
}
```

**测试：**

​		先启动服务器，等到控制台出现“服务器启动成功”，再启动客户端，客户端连接成功后，发现服务器端出现“hello nio”则成功：

服务器端：

```
服务器启动成功
```

客户端：

```
客户端连接成功
```

服务器端：

```
服务器启动成功
hello nio
```



