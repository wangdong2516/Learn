1. I/O多路复用有很多种的实现方式，之前主要是select和poll，现在住主要的是epoll，epoll已经成为了实现高性能服务器的必备技术
2. 高并发的核心解决方案是1个线程处理所有连接的等待消息准备好，之前select使用的方式是监听全部的连接，返回有效的连接，select在有报文到达的时候被调用
3. 内核中定义立刻一个参数用来定义每个FD_SET的句柄个数
4. 其次内核中的select使用的是轮询的方式，即每次检测都会遍历所有的句柄，select要检测的句柄数目越多的时候，就会越费时
5. poll和select在内部机制上面的实现并没有很大的区别，这是取消了最大监控文件描述符限制