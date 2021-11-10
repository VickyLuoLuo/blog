---
title: BIO NIO与Netty
header-img: /img/header_img/post-bg-desk.jpg
catalog: true
top: 0
tags:
  - IO
  - netty
categories:
  - work
abbrlink: 5dae
date: 2021-8-25 00:00:00
subtitle:
---
# BIO、NIO与Netty



> ### 网络编程：
>
> ​		网络编程的基本模型是Client/Server模型，也就是两个进程之间进行相互通信，其中服务端提供位置信息（绑定的IP地址和监听端口），客户端通过连接操作向服务端监听的地址发起连接请求，通过三次握手建立连接，如果连接建立成功，双方就可以通过网络套接字（Socket）进行通信。
>
> ### BIO:
>
> ​		在1.4版本之前，Java IO类库是阻塞IO（Blocking IO），建立网络连接的时候采用BIO模式。
>
> ### NIO:
>
> ​		为了支持非阻塞IO，Java引进了新的IO库，简称为JAVA NIO（Non-Blocking IO）。Java NIO属于 IO多路复用模型。
>
> ### Netty:
>
> ​		Netty是一个异步的、[事件驱动](https://baike.baidu.com/item/事件驱动/9597519)的网络应用程序框架，基于NIO，使用Netty 可以快速开发出一个网络应用，例如实现了某种协议的客户、[服务端](https://baike.baidu.com/item/服务端/6492316)应用。



## 1. Java BIO

### 1.1 Java BIO模型

- 同步阻塞IO，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，在高并发的应用场景下，需要大量的线程来维护大量的网络连接，内存、线程切换开销会非常巨大。

    ![bio-model](https://i.loli.net/2021/10/18/6jJxcVENwszR8Wd.png)

### 1.2 Java BIO 工作流程

1. 服务器端启动一个 `ServerSocket`，监听端口。
2. 客户端启动 `Socket` 与服务器进行通信，默认情况下服务器端需要对每个客户建立一个线程与之通讯。
3. 客户端发出请求后，先咨询服务器是否有线程响应，如果没有则会等待或被拒绝，如果有响应，客户端线程会等待请求结束后继续执行。

### 1.3 Java BIO代码示例([所有代码可前往github下载](https://github.com/VickyLuoLuo/netty.git))

- 最简单的BIO服务器：

```java
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

public class BioServer {

    public static void main(String[] args) throws IOException {
        // 创建ServerSocket
        ServerSocket serverSocket = new ServerSocket(9090);
        System.out.println("服务器启动");
        while (true) {
            //监听，等待客户端连接
            System.out.println("等待连接....");
            final Socket socket = serverSocket.accept(); // 阻塞
            System.out.println("连接到一个客户端");
            byte[] bytes = new byte[1024];
            socket.getInputStream().read(bytes); // 阻塞
            System.out.println("收到客户端消息：" + new String(bytes));
        }
    }
}
```

- 我们可以对上述代码进行优化，将`socket.getInputStream().read(bytes)`消息处理部分改成线程池异步处理：

```java
import java.io.IOException;
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class BioServer {

    public static void main(String[] args) throws IOException {
        // 创建ServerSocket
        ServerSocket serverSocket = new ServerSocket(9090);
        System.out.println("服务器启动");
        // 线程池机制,思路: 1. 创建一个线程池 2. 如果有客户端连接，就创建一个线程，与之通讯(单独写一个方法)
        ExecutorService newCachedThreadPool = Executors.newCachedThreadPool();
        while (true) {
            //监听，等待客户端连接
            System.out.println("等待连接....");
            final Socket socket = serverSocket.accept(); // 阻塞
            System.out.println("连接到一个客户端");
            newCachedThreadPool.execute(() -> {
                // 通讯处理
                handler(socket);
            });
        }
    }

    public static void handler(Socket socket) {
        try {
            byte[] bytes = new byte[1024];
            //通过socket获取输入流
            InputStream inputStream = socket.getInputStream();
            //循环的读取客户端发送的数据
            while (true) {
                System.out.println("handler线程：id = " + Thread.currentThread().getId() + "等待接收消息");
                int read = inputStream.read(bytes); // 阻塞
                if (read != -1) {
                    System.out.println("handler线程：id = " + Thread.currentThread().getId() + "收到消息：" + new String(bytes, 0, read));
                } else {
                    break;
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            System.out.println("handler线程：id = " + Thread.currentThread().getId() + "关闭连接"); 
            try {
                socket.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

​		这样可以同时处理多个连接请求，但是由于`inputStream.read(bytes)`是阻塞的，当有多个连接请求时，每个连接占用一个线程，此时如果大部分连接都没有发送消息，线程就一直被占用，造成资源浪费。

## 2. Java NIO

### 2.1 Java NIO模型

- 同步非阻塞IO，服务器实现模式为一个线程处理多个连接请求，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有 `I/O` 请求就进行处理。

    ![nio-model](https://i.loli.net/2021/10/18/ABy7qJoGhwaSzip.png)

### 2.2 Java NIO的三大核心组件

1. ####  通道（Channel）

    在BIO中，同一个网络连接会通过输入流（Input Stream）和输出流（Output Stream）不断地进行输入和输出的操作。在NIO中，同一个网络连接使用一个通道表示，所有的NIO的IO操作都是从通道开始的。一个通道类似于BIO中的两个流的结合体，既可以从通道读取，也可以向通道写入。

2. ####  选择器（Selector）

    > IO多路复用指的是一个进程/线程可以同时监视多个文件描述符（一个网络连接，操作系统底层使用一个文件描述符来表示），一旦其中的一个或者多个文件描述符可读或者可写，系统内核就通知该进程/线程。

    通过选择器，一个线程可以查询多个通道的IO事件的就绪状态，即监视多个文件描述符。具体的开发层面来说，首先把通道注册到选择器中，然后通过选择器内部的机制，可以查询（select）这些注册的通道是否有已经就绪的IO事件（例如可读、可写、网络连接完成等）。一个选择器只需要一个线程进行监控，系统不必为每一个网络连接（文件描述符）创建进程/线程，从而大大减小了系统的开销。

3. #### 缓冲区（Buffer）

    应用程序与通道（Channel）主要的交互操作，就是进行数据的read读取和write写入。通道的读取，就是将数据从通道读取到缓冲区中；通道的写入，就是将数据从缓冲区中写入到通道中。

### 2.3 Selector、Channel 和 Buffer的 关系

1. 每个 `Channel` 都会对应一个 `Buffer`。
2. `Selector` 对应一个线程，一个线程对应多个 `Channel`（连接）。
3. `Selector` 会根据不同的事件，在各个通道上切换。
4. `Buffer` 就是一个内存块，底层是一个数组。
5. 数据通过 `Buffer`进行读写，`BIO` 中要么是输入流，要么是输出流，不能双向，但是 `NIO` 的 `Buffer` 是可以读也可以写，需要 `flip` 方法切换 。`Channel` 是双向的，可以返回底层操作系统的情况，比如 `Linux`，底层的操作系统通道就是双向的。

![nio-element](https://i.loli.net/2021/10/18/4F5pSqTYbayDgsJ.png)

### 2.4 Java NIO 工作流程

1. 服务器端启动一个 `ServerSocketChannel`，监听端口。---类似BIO中的ServerSocket。

2. 客户端启动 `SocketChannel`与服务器进行通信。---类似BIO中的Socket。

3. 获取`Selector`选择器，并将`ServerSocketChannel`注册到`Selector`上，接收新连接。注册后返回一个 `SelectionKey`（集合），会和该 `Selector` 关联。一个 `Selector` 上可以注册多个 `SocketChannel`。

4. 当客户端连接时，服务端会通过 `ServerSocketChannel` 得到`SocketChannel`。

5. `Selector` 进行监听 `select` 方法，返回有事件发生的通道的个数。

6. 将 `socketChannel` 注册到 `Selector` 上，进一步得到各个 `SelectionKey`（有事件发生）。

7. 在通过 `SelectionKey` 反向获取 `SocketChannel`。

8. 可以通过得到的 `channel`，完成业务处理。

    ![nio-workflow](https://i.loli.net/2021/10/18/ybcGt7zRT459L8Z.png)

### 2.4 Java NIO 代码示例([所有代码可前往github下载](https://github.com/VickyLuoLuo/netty.git))

- 简单的NIO服务器，使用一个集合`List<SocketChannel>`来存放所有从客户端接收到的`SocketChannel`，并一直轮询处理IO事件

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class NioServer {

    static List<SocketChannel> channelList = new ArrayList<>();

    public static void startServer() throws IOException {

        // 获取通道
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        // 设置为非阻塞
        serverSocketChannel.configureBlocking(false);
        // 绑定连接
        serverSocketChannel.bind(new InetSocketAddress(5252));
        System.out.println("服务器启动成功");

        // 轮询感兴趣的IO就绪事件（选择键集合）
        while (true) {

            // 获取客户端连接
            SocketChannel socketChannel = serverSocketChannel.accept(); // 非阻塞
            if (null != socketChannel) {
                System.out.println("连接成功");
                // 设置SocketChannel为非阻塞模式。NIO的非阻塞是由操作系统内部实现的，底层调用了系统内核的accept方法
                socketChannel.configureBlocking(false);

                channelList.add(socketChannel);
            }
            // 获取选择键集合
            Iterator<SocketChannel> channels = channelList.iterator();
            while (channels.hasNext()) {
                // 获取单个的选择键并处理
                SocketChannel channel = channels.next();
                ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                // 调用通道的read方法，从通道读取数据写入缓冲区，并返回读取到的数据
                int length = channel.read(byteBuffer); // 非阻塞
                if (length > 0) {
                    System.out.println("接收到消息：" + new String(byteBuffer.array(), 0, length));
                } else if (length == -1) {
                    channels.remove();
                    System.out.println("客户端断开连接");
                }

            }
        }
    }

    public static void main(String[] args) throws IOException {
        startServer();
    }
}
```

- 上述代码有个很严重的问题，由于关键的两个方法：接收连接`serverSocketChannel.accept()`和读取消息`channel.read(byteBuffer)`都是非阻塞的，`while(true)`就会无限循环直到内存溢出，`Selector`很好的解决了这个问题，`selector.select()`会阻塞，直到有事件发生：

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;

public class NioSelectorServer {
    public static void startServer() throws IOException {

        // 获取通道
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        // 设置为非阻塞
        serverSocketChannel.configureBlocking(false);
        // 绑定连接
        serverSocketChannel.bind(new InetSocketAddress(5656));
        System.out.println("服务器启动成功");

        // 获取选择器
        Selector selector = Selector.open(); // epoll
        // 将通道注册的“接收新连接”IO事件注册到选择器上
        SelectionKey selectionKey = serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        // 轮询感兴趣的IO就绪事件（选择键集合）
        while (selector.select() > 0) { // 阻塞 epoll-wait
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
                    System.out.println("连接成功");
                } else if (selectedKey.isReadable()) {
                    // 若选择键的IO事件是“可读”，就读取数据
                    SocketChannel socketChannel = (SocketChannel) selectedKey.channel();
                    // 读取数据，然后丢弃
                    ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                    // 调用通道的read方法，从通道读取数据写入缓冲区，并返回读取到的数据
                    int length = socketChannel.read(byteBuffer);
                    if (length > 0) {
                        System.out.println("接收到消息：" + new String(byteBuffer.array(), 0, length));
                    } else if (length == -1) {
                        System.out.println("客户端断开连接");
                        socketChannel.close();
                    }
                }
                // 移除选择键
                selectedKeys.remove();
            }
        }
        // 关闭连接
        serverSocketChannel.close();
    }

    public static void main(String[] args) throws IOException {
        startServer();
    }
}
```

### 2.5 多路复用器`Selector`

​	通过`Selector`，可以使用一个线程查询多个通道的IO事件的就绪状态，并在没有事件发生时阻塞。其原理依赖于`epoll`系统调用(Linux系统为例)，`epoll`系统调用的主要方法：

1. epoll_create

    创建一个epoll文件描述符。可简单理解为channel 集合。

2. epoll_ctl

    添加/修改/删除需要侦听的文件描述符及其事件。可简单理解为channel事件集合。

3. epoll_wait

    接收发生在被侦听的描述符上的，用户感兴趣的IO事件。可简单理解为 channel事件集合为空时阻塞。

### 2.6 Java NIO 常用类及属性

​		参见另一篇博客：[Java NIO通信基础](https://vickyluoluo.github.io/2021/03/31/57bd.html)

## 3. Netty

### 3.1 原生NIO存在的问题

1. `NIO` 的类库和 `API` 繁杂，使用麻烦：需要熟练掌握 `Selector`、`ServerSocketChannel`、`SocketChannel`、`ByteBuffer`等。
2. 需要具备其他的额外技能：要熟悉 `Java` 多线程编程，因为 `NIO` 编程涉及到 `Reactor` 模式，你必须对多线程和网络编程非常熟悉，才能编写出高质量的 `NIO` 程序。
3. 开发工作量和难度都非常大：例如客户端面临断连重连、网络闪断、半包读写、失败缓存、网络拥塞和异常流的处理等等。
4.  `JDK NIO` 的 `Bug`：例如臭名昭著的 `Epoll Bug`，它会导致 `Selector` 空轮询，最终导致 `CPU100%`。直到 `JDK1.7` 版本该问题仍旧存在，没有被根本解决

### 3.2 Netty 的优点

`Netty` 对 `JDK` 自带的 `NIO` 的 `API` 进行了封装，解决了上述问题。

1. 设计优雅：适用于各种传输类型的统一 `API` 阻塞和非阻塞 `Socket`；基于灵活且可扩展的事件模型，可以清晰地分离关注点；高度可定制的线程模型-单线程，一个或多个线程池。
2. 使用方便：详细记录的 `Javadoc`，用户指南和示例；没有其他依赖项，`JDK5（Netty3.x）`或 `6（Netty4.x）`就足够了。
3. 高性能、吞吐量更高：延迟更低；减少资源消耗；最小化不必要的内存复制。
4. 安全：完整的 `SSL/TLS` 和 `StartTLS` 支持。
5. 社区活跃、不断更新：社区活跃，版本迭代周期短，发现的 `Bug` 可以被及时修复，同时，更多的新功能会被加入。

### 3.3 Netty工作原理

![](https://i.loli.net/2021/10/18/3cnCUJG4yerixqK.png)

`Netty` 线程模型基于主从 `Reactors` 多线程模型，`BossGroup` 线程维护 `Selector`，只关注 `Accecpt` 当接收到 `Accept` 事件，获取到对应的 `SocketChannel`，封装成 `NIOScoketChannel` 并注册到 `Worker` 线程（事件循环），并进行维护当 `Worker` 线程监听到 `Selector` 中通道发生自己感兴趣的事件后，由 `handler`进行处理。

![](https://i.loli.net/2021/10/18/chLyW8ko5aRuPqN.png)

说明：

1. `Netty` 抽象出两组线程池 `BossGroup` 专门负责接收客户端的连接，`WorkerGroup` 专门负责网络的读写
2. `BossGroup` 和 `WorkerGroup` 类型都是 `NioEventLoopGroup` 
3. `NioEventLoopGroup` 相当于一个事件循环组，这个组中含有多个事件循环，每一个事件循环是 `NioEventLoop` 
4. `NioEventLoop` 表示一个不断循环的执行处理任务的线程，每个 `NioEventLoop` 都有一个 `Selector`，用于监听绑定在其上的 `socket` 的网络通讯
5. `NioEventLoopGroup` 可以有多个线程，即可以含有多个 `NioEventLoop` 
6. 每个 `BossNioEventLoop` 循环执行的步骤有 `3` 步
    - 轮询 `accept` 事件
    - 处理 `accept` 事件，与 `client` 建立连接，生成 `NioScocketChannel`，并将其注册到某个 `worker` `NIOEventLoop` 上的 `Selector`
    - 处理任务队列的任务，即 `runAllTasks`
7. 每个 `Worker` `NIOEventLoop` 循环执行的步骤
    - 轮询 `read`，`write` 事件
    - 处理 `I/O` 事件，即 `read`，`write` 事件，在对应 `NioScocketChannel` 处理
    - 处理任务队列的任务，即 `runAllTasks`
8. 每个 `Worker` `NIOEventLoop` 处理业务时，会使用 `pipeline`（管道），`pipeline` 中包含了 `channel`，即通过 `pipeline` 可以获取到对应通道，管道中维护了很多的处理器

### 3.4 Netty 代码示例([所有代码可前往github下载](https://github.com/VickyLuoLuo/netty.git))

- 简单的Netty服务器实现：
    1. `NettyServer.java` 服务器启动类，通过链式编程配置启动参数并监听端口

```java
import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.*;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;

public class NettyServer {

    public static void main(String[] args) throws Exception {

        //创建两个线程组bossGroup和workerGroup，默认子线程个数为cpu核数 * 2，bossGroup只处理连接请求,真正的客户端业务处理会交给workerGroup
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup(8);

        try {
            //创建服务器端的启动对象，配置参数
            ServerBootstrap bootstrap = new ServerBootstrap();
            //使用链式编程来进行设置
            bootstrap.group(bossGroup, workerGroup) //设置两个线程组
                    .channel(NioServerSocketChannel.class) //使用NioSocketChannel 作为服务器的通道实现
                    .option(ChannelOption.SO_BACKLOG, 128) // 初始化服务器接收连接的队列大小
                    .childHandler(new ChannelInitializer<SocketChannel>() {//创建一个通道初始化对象(匿名对象)
                        //给workGroup的SocketChannel设置处理器
                        @Override
                        protected void initChannel(SocketChannel ch) {
                            System.out.println("客户socketchannel hashcode=" + ch.hashCode()); //可以使用一个集合管理 SocketChannel， 再推送消息时，可以将业务加入到各个channel 对应的 NIOEventLoop 的 taskQueue 或者 scheduleTaskQueue
                            ch.pipeline().addLast(new NettyServerHandler()); // 可添加多个
                        }
                    });

            System.out.println(".....服务器 is ready...");

            //启动服务器并绑定端口，bind是异步操作，可以通过isDone()等方法查看事件执行情况
            ChannelFuture cf = bootstrap.bind(6668).sync();

            //给cf 注册监听器，监控我们关心的事件
            cf.addListener((ChannelFutureListener) future -> {
                if (cf.isSuccess()) {
                    System.out.println("监听端口成功");
                } else {
                    System.out.println("监听端口失败");
                }
            });

            //对关闭通道进行监听
            cf.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

​			2. `NettyServerHandler.java `自定义`ChannelHandler`处理业务，只需根据需要继承指定类型的抽象类，通过重载父类方法实现业务处理：

```java
import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.*;
import io.netty.util.CharsetUtil;

public class NettyServerHandler extends ChannelInboundHandlerAdapter {

    /**
     * 当客户端连接服务器完成时触发
     * @param ctx: 上下文对象, 含有 管道pipeline , 通道channel, 地址
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) {
        System.out.println("客户端连接通道建立完成");
    }

    /**
     * 读取数据实际(这里我们可以读取客户端发送的消息)
     * @param ctx:上下文对象, 含有 管道pipeline , 通道channel, 地址
     * @param msg: 就是客户端发送的数据 默认Object
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

        Channel channel = ctx.channel();
        //将 msg 转成一个 ByteBuf,ByteBuf 是 Netty 提供的，不是 NIO 的 ByteBuffer.
        ByteBuf buf = (ByteBuf) msg;
        System.out.println("客户端发送消息是:" + buf.toString(CharsetUtil.UTF_8));
        System.out.println("客户端地址:" + channel.remoteAddress());
    }

    /**
     *  数据读取完毕
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) {
        //将数据写入到缓存，并刷新
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端", CharsetUtil.UTF_8));
    }

    /**
     * 处理异常, 一般关闭通道
     * @param ctx
     * @param cause
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        ctx.close();
    }
}

```

- `SpringBoot` + `Netty`实现网络聊天室核心代码([完整代码可前往github下载](https://github.com/VickyLuoLuo/netty.git))：

```java
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.socket.SocketChannel;
import io.netty.handler.codec.http.HttpObjectAggregator;
import io.netty.handler.codec.http.HttpServerCodec;
import io.netty.handler.codec.http.websocketx.WebSocketServerProtocolHandler;
import io.netty.handler.stream.ChunkedWriteHandler;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

@Component
@Qualifier("somethingChannelInitializer")
public class NettyWebSocketChannelInitializer extends ChannelInitializer<SocketChannel> {

    @Autowired
    private TextWebSocketFrameHandler textWebSocketFrameHandler;

    @Override
    public void initChannel(SocketChannel ch) throws Exception {//2
        ChannelPipeline pipeline = ch.pipeline();

        pipeline.addLast(new HttpServerCodec()); // http解编码器
        pipeline.addLast(new HttpObjectAggregator(64*1024)); // 补充http解编码器
        pipeline.addLast(new ChunkedWriteHandler()); // 保证队列中的每一个元素是一次性发送
        pipeline.addLast(new WebSocketServerProtocolHandler("/ws")); //
        pipeline.addLast(textWebSocketFrameHandler); // 自定义ChannelHandler

    }
}
```

```java
import com.lhl.netty.chat.util.RandomName;
import com.lhl.netty.chat.util.RedisDao;
import io.netty.channel.Channel;
import io.netty.channel.ChannelHandler;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.channel.group.ChannelGroup;
import io.netty.channel.group.DefaultChannelGroup;
import io.netty.handler.codec.http.websocketx.TextWebSocketFrame;
import io.netty.util.concurrent.GlobalEventExecutor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

@Component
@Qualifier("textWebSocketFrameHandler")
@ChannelHandler.Sharable
public class TextWebSocketFrameHandler extends SimpleChannelInboundHandler<TextWebSocketFrame> {

    public static ChannelGroup channels = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);

    @Autowired
    private RedisDao redisDao;

    @Override
    protected void channelRead0(ChannelHandlerContext ctx,
                                TextWebSocketFrame msg) throws Exception {
        Channel incoming = ctx.channel();
        String uName = redisDao.getString(incoming.id()+"");
        for (Channel channel : channels) {
            if (channel != incoming){
                channel.writeAndFlush(new TextWebSocketFrame("[" + uName + "]" + msg.text()));
            } else {
                channel.writeAndFlush(new TextWebSocketFrame("[我]" + msg.text() ));
            }
        }
    }

    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        System.out.println(ctx.channel().remoteAddress());
        String uName = new RandomName().getRandomName();

        Channel incoming = ctx.channel();
        for (Channel channel : channels) {
            channel.writeAndFlush(new TextWebSocketFrame("[新用户] - " + uName + " 加入群聊"));
        }
        redisDao.saveString(incoming.id()+"",uName);
        channels.add(ctx.channel());
    }

    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
        Channel incoming = ctx.channel();
        String uName = redisDao.getString(String.valueOf(incoming.id()));
        for (Channel channel : channels) {
            channel.writeAndFlush(new TextWebSocketFrame("[用户] - " + uName + " 离开"));
        }
        redisDao.deleteString(String.valueOf(incoming.id()));

        channels.remove(ctx.channel());
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        Channel incoming = ctx.channel();
        System.out.println("用户:"+redisDao.getString(incoming.id()+"")+"在线");
    }


    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        Channel incoming = ctx.channel();
        System.out.println("用户:"+redisDao.getString(incoming.id()+"")+"掉线");
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause)
            throws Exception {
        Channel incoming = ctx.channel();
        System.out.println("用户:"+redisDao.getString(incoming.id()+"")+"异常");
        cause.printStackTrace();
        ctx.close();
    }

}
```

### 3.5 Netty常用类及属性

#### Bootstrap、ServerBootstrap

1. `Bootstrap` 意思是引导，一个 `Netty` 应用通常由一个 `Bootstrap` 开始，主要作用是配置整个 `Netty` 程序，串联各个组件，`Netty` 中 `Bootstrap` 类是客户端程序的启动引导类，`ServerBootstrap` 是服务端启动引导类。
2. 常见的方法有
    - `public ServerBootstrap group(EventLoopGroup parentGroup, EventLoopGroup childGroup)`，该方法用于服务器端，用来设置两个 `EventLoop`
    - `public B group(EventLoopGroup group)`，该方法用于客户端，用来设置一个 `EventLoop`
    - `public B channel(Class<? extends C> channelClass)`，该方法用来设置一个服务器端的通道实现
    - `public <T> B option(ChannelOption<T> option, T value)`，用来给 `ServerChannel` 添加配置
    - `public <T> ServerBootstrap childOption(ChannelOption<T> childOption, T value)`，用来给接收到的通道添加配置
    - `public ServerBootstrap childHandler(ChannelHandler childHandler)`，该方法用来设置业务处理类（自定义的`handler`）
    - `public ChannelFuture bind(int inetPort)`，该方法用于服务器端，用来设置占用的端口号
    - `public ChannelFuture connect(String inetHost, int inetPort)`，该方法用于客户端，用来连接服务器端

####  Future、ChannelFuture

`Netty` 中所有的 `IO` 操作都是异步的，不能立刻得知消息是否被正确处理。但是可以过一会等它执行完成或者直接注册一个监听，具体的实现就是通过 `Future` 和 `ChannelFutures`，他们可以注册一个监听，当操作执行成功或失败时监听会自动触发注册的监听事件

常见的方法有

- `Channel channel()`，返回当前正在进行 `IO` 操作的通道
- `ChannelFuture sync()`，等待异步操作执行完毕

#### Channel

1. `Netty` 网络通信的组件，能够用于执行网络 `I/O` 操作。
2. 通过 `Channel` 可获得当前网络连接的通道的状态
3. 通过 `Channel` 可获得网络连接的配置参数（例如接收缓冲区大小）
4. `Channel` 提供异步的网络 `I/O` 操作(如建立连接，读写，绑定端口)，异步调用意味着任何 `I/O` 调用都将立即返回，并且不保证在调用结束时所请求的 `I/O` 操作已完成
5. 调用立即返回一个 `ChannelFuture` 实例，通过注册监听器到 `ChannelFuture` 上，可以 `I/O` 操作成功、失败或取消时回调通知调用方
6. 支持关联 `I/O` 操作与对应的处理程序
7. 不同协议、不同的阻塞类型的连接都有不同的 `Channel` 类型与之对应，常用的 `Channel` 类型：
    - `NioSocketChannel`，异步的客户端 `TCP` `Socket` 连接。
    - `NioServerSocketChannel`，异步的服务器端 `TCP` `Socket` 连接。
    - `NioDatagramChannel`，异步的 `UDP` 连接。
    - `NioSctpChannel`，异步的客户端 `Sctp` 连接。
    - `NioSctpServerChannel`，异步的 `Sctp` 服务器端连接，这些通道涵盖了 `UDP` 和 `TCP` 网络 `IO` 以及文件 `IO`。

#### Selector

1. `Netty` 基于 `Selector` 对象实现 `I/O` 多路复用，通过 `Selector` 一个线程可以监听多个连接的 `Channel` 事件。
2. 当向一个 `Selector` 中注册 `Channel` 后，`Selector` 内部的机制就可以自动不断地查询（`Select`）这些注册的 `Channel` 是否有已就绪的 `I/O` 事件（例如可读，可写，网络连接完成等），这样程序就可以很简单地使用一个线程高效地管理多个 `Channel`

#### ChannelHandler 

1. `ChannelHandler` 是一个接口，处理 `I/O` 事件或拦截 `I/O` 操作，并将其转发到其 `ChannelPipeline`（业务处理链）中的下一个处理程序。
2. `ChannelHandler` 本身并没有提供很多方法，因为这个接口有许多的方法需要实现，方便使用期间，可以继承它的子类

#### Pipeline 和 ChannelPipeline

1. `ChannelPipeline` 是一个 `Handler` 的集合，它负责处理和拦截 `inbound` 或者 `outbound` 的事件和操作，相当于一个贯穿 `Netty` 的链。（也可以这样理解：`ChannelPipeline` 是保存 `ChannelHandler` 的 `List`，用于处理或拦截 `Channel` 的入站事件和出站操作）
2. `ChannelPipeline` 实现了一种高级形式的拦截过滤器模式，使用户可以完全控制事件的处理方式，以及 `Channel` 中各个的 `ChannelHandler` 如何相互交互
3. 在 `Netty` 中每个 `Channel` 都有且仅有一个 `ChannelPipeline` 与之对应，它们的组成关系如下

![](https://i.loli.net/2021/10/18/oZARqLONHzbaGiP.png)

​	![](https://i.loli.net/2021/10/18/KeR8OM6uTVqhjFZ.png)

4. 常用方法
    `ChannelPipeline addFirst(ChannelHandler... handlers)`，把一个业务处理类（`handler`）添加到链中的第一个位置`ChannelPipeline addLast(ChannelHandler... handlers)`，把一个业务处理类（`handler`）添加到链中的最后一个位置

#### ChannelHandlerContext

1. 保存 `Channel` 相关的所有上下文信息，同时关联一个 `ChannelHandler` 对象
2. 即 `ChannelHandlerContext` 中包含一个具体的事件处理器 `ChannelHandler`，同时 `ChannelHandlerContext` 中也绑定了对应的 `pipeline` 和 `Channel` 的信息，方便对 `ChannelHandler` 进行调用。
3. 常用方法
    - `ChannelFuture close()`，关闭通道
    - `ChannelOutboundInvoker flush()`，刷新
    - `ChannelFuture writeAndFlush(Object msg)`，将数据写到 
    - `ChannelPipeline` 中当前 `ChannelHandler` 的下一个 `ChannelHandler` 开始处理（出站）

![](https://i.loli.net/2021/10/18/qRSk7ezATLQtKV5.png)

#### ChannelOption

1. `Netty` 在创建 `Channel` 实例后，一般都需要设置 `ChannelOption` 参数。
2. `ChannelOption` 参数如下：

![](https://i.loli.net/2021/10/18/Z6pAFyCjhOU5dvg.png)

#### EventLoopGroup

1. `EventLoopGroup` 是一组 `EventLoop` 的抽象，`Netty` 为了更好的利用多核 `CPU` 资源，一般会有多个 `EventLoop` 同时工作，每个 `EventLoop` 维护着一个 `Selector` 实例。
2. `EventLoopGroup` 提供 `next` 接口，可以从组里面按照一定规则获取其中一个 `EventLoop` 来处理任务。在 `Netty` 服务器端编程中，我们一般都需要提供两个 `EventLoopGroup`，例如：`BossEventLoopGroup` 和 `WorkerEventLoopGroup`。
3. 通常一个服务端口即一个 `ServerSocketChannel` 对应一个 `Selector` 和一个 `EventLoop` 线程。`BossEventLoop` 负责接收客户端的连接并将 `SocketChannel` 交给 `WorkerEventLoopGroup` 来进行 `IO` 处理，如下图所示

![](https://i.loli.net/2021/10/18/zoQgpkliDMshRN1.png)

4. 常用方法
    `public NioEventLoopGroup()`，构造方法
    `public Future<?> shutdownGracefully()`，断开连接，关闭线程

#### Unpooled 

1. `Netty` 提供一个专门用来操作缓冲区（即 `Netty` 的数据容器）的工具类
2. 常用方法如下所示

![](https://i.loli.net/2021/10/18/yQbHkAGRs7fqYX2.png)

3.  `Unpooled` 获取 `Netty` 的数据容器 `ByteBuf` 的基本使用

![](https://i.loli.net/2021/10/18/CpM4xhrFYE1IdVB.png)