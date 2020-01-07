​	一组ChannelHandler，用来处理Channel的入站事件和出站事件。ChannelPipeline实现一种高级的拦截过滤模式，使用户完全控制事件如何处理，以及管道中的ChannelHandler如何相互作用。

​	创建一个管道（pipeline）

​	每个通道（channel）都有自己的管道，当通道创建的时候，管道会被自动创建。

​	事件如何在管道中流动

​	下面的数据图描述了一个典型的i/o事件在ChannelPipeline中被ChannelHandler处理的流程。一个i/o事件由ChannelInboundHandler（入事件处理）或者ChannelOutboundHandler（出事件处理），并通过调用所定义的ChannelHandlerContext（通道处理器上下文）的事件传播方法，被发送到离当前调用最近的handler处理程序。例如：ChannelHandlerContext#fireChannelRead(Object) 和 ChannelHandlerContext#write(Object)  方法的调用，都会触发事件的传播

```java

*                                                 I/O 请求
*                                            通过 {@link Channel} 或
*                                        {@link ChannelHandlerContext}
*                                                      |
*  +---------------------------------------------------+---------------+
*  |                           ChannelPipeline         |               |
*  |                                                  \|/              |
*  |    +---------------------+            +-----------+----------+    |
*  |    | Inbound Handler  N  |            | Outbound Handler  1  |    |
*  |    +----------+----------+            +-----------+----------+    |
*  |              /|\                                  |               |
*  |               |                                  \|/              |
*  |    +----------+----------+            +-----------+----------+    |
*  |    | Inbound Handler N-1 |            | Outbound Handler  2  |    |
*  |    +----------+----------+            +-----------+----------+    |
*  |              /|\                                  .               |
*  |               .                                   .               |
*  | ChannelHandlerContext.fireIN_EVT() ChannelHandlerContext.OUT_EVT()|
*  |        [ method call]                       [method call]         |
*  |               .                                   .               |
*  |               .                                  \|/              |
*  |    +----------+----------+            +-----------+----------+    |
*  |    | Inbound Handler  2  |            | Outbound Handler M-1 |    |
*  |    +----------+----------+            +-----------+----------+    |
*  |              /|\                                  |               |
*  |               |                                  \|/              |
*  |    +----------+----------+            +-----------+----------+    |
*  |    | Inbound Handler  1  |            | Outbound Handler  M  |    |
*  |    +----------+----------+            +-----------+----------+    |
*  |              /|\                                  |               |
*  +---------------+-----------------------------------+---------------+
*                  |                                  \|/
*  +---------------+-----------------------------------+---------------+
*  |               |                                   |               |
*  |       [ Socket.read() ]                    [ Socket.write() ]     |
*  |                                                                   |
*  |  Netty Internal I/O Threads (Transport Implementation)            |
*  +-------------------------------------------------------------------+

```

​	如图的左侧所示，一个入站事件是由下往上方向的入站处理程序处理。入站处理器通常处理由从图的底部的I / O线程生成入站数据。入站数据通常从远程对等经由实际输入操作读取诸如{@link SocketChannel＃ByteBuffer}。如果入站事件超越了顶部入站处理程序，它被悄悄丢弃，或记录，如果它需要你的关注。 

​	如图的右侧所示，一个出站事件是由上往下方向的出站处理程序处理。出站处理器通常生成或变换的出站通信，如写请求。如果出站事件超出了底部出站处理程序，它通过相关的I / O线程的{@link  Channel}进行处理。I / O线程常常执行实际的输出操作，例如{@link SocketChannel#write(ByteBuffer)}。

​	例如，让我们假设，我们创建了以下管道：

```java
ChannelPipeline} p = ...;
p.addLast("1", new InboundHandlerA());
p.addLast("2", new InboundHandlerB());
p.addLast("3", new OutboundHandlerA());
p.addLast("4", new OutboundHandlerB());
p.addLast("5", new InboundOutboundHandlerX());
```

​	如上述示例所示，名字开始于{@code Inbound}意味着它是入站处理程序的类的实例。 类的名字开始与{@code Outbound}意味着它是一个出站处理程序。

​	在给出的示例配置中，对于一个入站事件，程序的处理顺序是1，2，3，4，5；对于一个出站事件，程序的处理顺序是5,4，3，2，1。基于这个原则，{@link ChannelPipeline} 跳过某些处理程序来缩短堆调用的深度：

- ​	3 and 4 没有实现{@link ChannelInboundHandler}, 因此一个入站事件的处理顺序是: 1, 2, and 5.
- 1 and 2 没有实现 {@link ChannelOutboundHandler}, 因此一个出站事件的处理顺序是: 5, 4, and 3.
- 因为5即同时实现了入站和出站事件，所以5在出站和入站处理流程中都会出现

将事件转发到下一个处理（事件传播）

正如你可能在图显示注意到，处理程序必须在调用{@linkChannelHandlerContext}事件传播方法事件转发到其下一个处理程序。这些方法包括：

- 入站事件传播的方法：

```java
*     <li>{@link ChannelHandlerContext#fireChannelRegistered()}</li>
*     <li>{@link ChannelHandlerContext#fireChannelActive()}</li>
*     <li>{@link ChannelHandlerContext#fireChannelRead(Object)}</li>
*     <li>{@link ChannelHandlerContext#fireChannelReadComplete()}</li>
*     <li>{@link ChannelHandlerContext#fireExceptionCaught(Throwable)}</li>
*     <li>{@link ChannelHandlerContext#fireUserEventTriggered(Object)}</li>
*     <li>{@link ChannelHandlerContext#fireChannelWritabilityChanged()}</li>
*     <li>{@link ChannelHandlerContext#fireChannelInactive()}</li>
*     <li>{@link ChannelHandlerContext#fireChannelUnregistered()}</li>
```

- 出站事件传播的方法：

```java
*     <li>{@link ChannelHandlerContext#bind(SocketAddress, ChannelPromise)}</li>
*     <li>{@link ChannelHandlerContext#connect(SocketAddress, SocketAddress, ChannelPromise)}</li>
*     <li>{@link ChannelHandlerContext#write(Object, ChannelPromise)}</li>
*     <li>{@link ChannelHandlerContext#flush()}</li>
*     <li>{@link ChannelHandlerContext#read()}</li>
*     <li>{@link ChannelHandlerContext#disconnect(ChannelPromise)}</li>
*     <li>{@link ChannelHandlerContext#close(ChannelPromise)}</li>
*     <li>{@link ChannelHandlerContext#deregister(ChannelPromise)}</li>
```

下面的示例中展示了事件传播通常是如何完成的：

```java
 public class MyInboundHandler extends {@link ChannelInboundHandlerAdapter} {
     @Override
     public void channelActive({@link ChannelHandlerContext} ctx) {
         System.out.println("Connected!");
         ctx.fireChannelActive();
     }
 }

 public class MyOutboundHandler extends {@link ChannelOutboundHandlerAdapter} {
     @Override
     public void close({@link ChannelHandlerContext} ctx, {@link ChannelPromise} promise) {
         System.out.println("Closing ..");
         ctx.close(promise);
     }
 }
```

​	建立一个管道

​	在一个管道中，用户应该有一个或多个 {@link ChannelHandler}去处理i/o事件（例如 read）和i/o请求操作（例如write and close）。一个典型的服务器，通常在每个通道的管道中有以下的处理程序，但您的里程可能取决于协议的复杂性和业务逻辑的特点有所不同：

```java
<li>Protocol Decoder - translates binary data (e.g. {@link ByteBuf}) into a Java object.</li>
* <li>Protocol Encoder - translates a Java object into binary data.</li>
* <li>Business Logic Handler - performs the actual business logic (e.g. database access).</li>
```

​	并且它可以通过以下例子表示：

```java
static final {@link EventExecutorGroup} group = new {@link DefaultEventExecutorGroup}(16);
 ...
 {@link ChannelPipeline} pipeline = ch.pipeline();
 pipeline.addLast("decoder", new MyProtocolDecoder());
 pipeline.addLast("encoder", new MyProtocolEncoder());
/**
告诉管道在不同的线程中执行MyBusinessLogicHandler's事件处理方法，而不是在一个i/o线程中执行耗时
的i/o任务。
	如果你的业务逻辑是完全异步或完成得非常快，则不需要指定线程组(EventExecutorGroup)
**/
 pipeline.addLast(group, "handler", new MyBusinessLogicHandler());
```

​	线程安全

​	在ChannelPipeline 中添加和删除ChannelHandler是线程安全的。例如，你可以当敏感信息即将被交换插入一个加密处理程序，并在交换之后将其删除。

```java
public interface ChannelPipeline
        extends ChannelInboundInvoker, ChannelOutboundInvoker, Iterable<Entry<String, ChannelHandler>> {
```

