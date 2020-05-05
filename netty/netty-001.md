NioEventLoopGroup 

实际使用的时候 会设置参数1 ，作为bosswork,

不做实际业务处理，只做转发处理。不设置的话 默认是cpu核心数*2

完成基本配置





重要结论：在netty中，channel的实现一定是线程安全的；因此，我们可以存储一个channel的引用，并在需要向远程断电发送数据时，通过这个引用来调用channel相应的方法；即便当时有很多线程都在使用它也不会出现多线程的问题；而且消息一定是按顺序发送出去的



我们在业务开发时，不要将长时间执行的耗时任务放入到eventloop的执行队列中，因为他将会一直阻塞该线程所对应的所有channel上的其它执行任务，如果我们需要进行阻塞调用或者是耗时操作，那么我们需要使用一个专门的eventexecutor（业务线程池）

通常有以下两种方式：

1在channelhandler的回调方法中，使用自定义的业务线程池

2借助于netty提供的向channelpipeline时调用的addlast方法传递的eventexcutor

说明：默认情况下（调用addlast(handler)）,channelhandler中的回调方法都是由i/o线程所执行，如果调用了channelPipeline addLast(EventExecutorGrop group,ChannelHandler ...handlers)方法，那么ChannelHanler中的回调方法就是由参数中的group线程来执行的





jdk提供的future只能通过手工方式检查执行结果，而这个操作是会阻塞的；netty则对channelfuture进行了增强，通过channelfuture以回调的方式来获取执行结果，去除了手工检查阻塞的操作；值得注意的是：channelfurelisener的operatorcomplete方法由i/o线程执行的，因此要注意的是不要在这里执行耗时的操作，否则需要通过另外的线程池来执行





