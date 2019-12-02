set 命令到底发生了什么



初始化re dis数据结构，监听事件的到来

已经触发的事件，则进行循环处理，如果有定时事件，则最后处理



aeApiPoll 进行事件处理的地方



initServer中redis注册了 acceptTcpHandler 回调函数

acceptCommandHandler 真正的处理地方



多命令模式，不是直接执行命令，而是让命令入队。

set. key value 是把数据存储在一个hash表中

addReply 注册写事件到事件队列

