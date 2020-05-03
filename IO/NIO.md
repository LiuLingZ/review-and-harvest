浅析I/O模型 https://www.cnblogs.com/dolphin0520/p/3916526.html

NIO概述 https://www.cnblogs.com/dolphin0520/p/3919162.html

epoll \ select \ poll<https://www.cnblogs.com/aspirant/p/9166944.html>

<https://mp.weixin.qq.com/s/khyOVIqFp1vNK29OIMBBuQ>





select
- 线性存储结构，1024、 2048 
- 通过轮询访问每一个fd的状态，不论是否就绪
- 数量有限。


poll 
- 无最大连接数，链表形式存储，但是越来越多，效率下降
- 也是轮询整个链表
- 定时轮询，或者有就绪就轮询，期间经历了好几次没有意义的遍历。
- 水平触发的特点：上次报告没有处理的fd，下次还会报告。



select+poll 都会将fd_set加载到内核空间加以轮询，这是需要复制的，poll数量很大也要，耗费资源



epoll

<https://blog.csdn.net/daaikuaichuan/article/details/83862311>

​	LT epollLT 水平触发
​	ET epollET 边缘触发

	默认模式中，epoll监听的fd，有就绪通知时，即便这次没有来得及处理，每次epoll_wait（监听）都会返回事件，让程序去处理。（多次返回直到被处理）
	
	高速模式：就绪的fd只会返回一次事件提醒。目的在于，避免太多不关心的fd就绪事件，程序可以集中关系自己在意的几个fd描述符。避免大量事件的堆积。
优点

没有最大并发连接限制，可监听的FD上限大于1024

效率提升，通过fd注册事件,就绪回调提高效率，不会因为数量上升下降。

Epoll只管活跃的FD，对于连接总数无关，网络中效率远高于前二者。



























