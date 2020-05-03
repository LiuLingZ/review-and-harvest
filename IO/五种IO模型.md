五种IO模型透彻分析：

​	https://www.cnblogs.com/f-ck-need-u/p/7624733.html

​	https://www.cnblogs.com/f-ck-need-u/p/7615914.html

# 一、基础



![img](https://images2017.cnblogs.com/blog/733013/201710/733013-20171003132425740-1897420439.png)

##1、理解这张图：外界请求程序的一个资源时发生的过程

```
1、程序（下面统称app。）先访问内存看是否将所需的数据载入。无则开始加载
2、从磁盘复制到内核缓存过程：
	从storage（磁盘存储）copy数据到 kernel buffer
	（同步的阻塞、非阻塞的首要区别，就在于此阶段是否是阻塞的）
3、从内核缓存复制数据到应用缓存
4、应用处理数据
5、将app buffer的数据复制到 socket buffer中
6、socket buffer再将数据通过网关发送到外界
```

## 2、拷贝方式

```
DMA copy：
	Direct Memory Access,直接内存访问，此步不需要借助CPU，因为其本身相当于一个小
	CPU，知道从哪里找数据。
CPU copy：
	需要CPU协助的，搜索数据地址、加载数据等行为，这里就需要占用CPU资源
	
DMA vs CPU :
	前者出现于  磁盘  于  内存的交互
	后者出现于  内存  与  内存的交互
```

## 3、如果数据通过TCP连接传输

![img](https://images2017.cnblogs.com/blog/733013/201710/733013-20171003141732271-400909384.png)

```
socket buffer :
	TCP\IP协议簇维护两个缓冲区：send buffer \ recv buffer.合称socketbuffer。
    发送数据前，将数据CPU copy到send buffer中，再DMA copy到网卡发送 ；
    接收数据，从网卡DMA copy到recv buffer中，再CPU copy到app buffer中。
```

### 零复制

```
网络数据一定要从kernel buffer复制到app buffer再复制到send buffer吗？
	并不是。如果进程不需要修改数据，就直接发送给TCP连接的另一端，可以不用从kernel buffer复制到app buffer，而是直接复制到send buffer。这就是零复制技术。
```

## 4、httpd进程处理文件类请求数据操作流程

![img](https://images2017.cnblogs.com/blog/733013/201710/733013-20171003145235536-131157754.png)

```
注：httpd 是 Apache超文本传输协议(HTTP)服务器的主程序
1、客户端发送请求，传到服务器
2、服务器收到，在网卡通过DMA copy请求到 Socket buffer中，这里是 recv buffer
3、此时app接收请求，recv buffer通过CPU copy到app buffer中
4、app将请求传到 httpd，httpd解析，知道了请求是哪个文件
5、httpd去内存查找，是否已经加载了此文件，如果没有，开始加载
6、从storage中通过DMA copy文件到 kernel buffer
7、kernel buffer通过 CPU copy到app buffer中
8、app将文件通过httpd规则封装
9、app将封装好的文件，从 app buffer中通过CPU copy到socket buffer中，这里是send buffer
10、socket buffer 通过 DMA copy数据到网卡
11、网卡通过TCP连接传回client
```



# 二、IO模型

​	所谓IO模型，描述的是  出现IO等待时 ， 进程的状态以及数据处理方式。

​	围绕 进程的状态、数据准备到kernel buffer ，再到 app buffer的两个阶段展开。

​	**数据准备** ：从 storage 加载到 kernel buffer

​	**数据复制** ： 从kernel buffer 复制到 app buffer



##1 、Blocking I/O模型

关键字：同步阻塞，全阻塞

## 2、Non-Blocking I/O模型

关键字：httpd自身轮询

## 3、I/O Multiplexing 模型

关键字：select \ poll \ epoll 代理 httpd监视消息

## 4、Signal-driven I/O模型

关键字：在数据准备过程，通过sigaction系统调用立刻返回；数据准备好发送信号提示，可数据复制阶段。

## 5、 Asynchronous I/O模式

关键字：真正异步，从read()开始到 **数据复制**结束，都是非阻塞的。



# 三、同/异 步IO、阻/非阻 塞的区分

## 1、区别

​	过程

​	哪里阻塞

​	同步异步的区别：根本在于，数据准备和数据复制阶段是否都是非阻塞的

## 2、 select() \ poll()  \ epoll()

​		三个函数都是文件描述符（fd）状态的监控函数，监控一系列文件的一系列事件，当出现满足条件的事件		后，就可被认为是就绪或者错误 。事件大致分3类：可读事件、可写事件、异常事件。它们通常放在循环		结构中进行循环监控。

### (1) 、select()  和 poll()

```
select() 
1、构建：
	通过FD_SET宏函数创建待监控的描述符集合。并作为select()函数的参数
	select()函数的参数可有：监控的描述符集合、阻塞时间间隔
	除了可以监控文件描述符，还可监控套接字，如recv buffer是否收到数据。
	select()默认最大监控： 32位 1024 ； 64位 2048 个 fd
2、select()时间间隔参数：（3种）
	- 设置 指定时间间隔内阻塞，除非之前有就绪事件发生。
	- 设置 永久阻塞，除非有就绪事件发生。
	- 设置 完全不阻塞，即立即返回，但因为select()通常在循环结构中，所以这是 轮询监控 的方式。
3、过程：
	当创建了监控对象后，由内核监控这些描述符集合，于此同时调用select()的进程被阻塞(或轮询)。当监控到满足就绪条件时(监控事件发生)，select()将被唤醒(或暂停轮询)，于是select()返回满足就绪条件的描述符数量，之所以是数量而不仅仅是一个，是因为多个文件描述符可能在同一时间满足就绪条件。由于只是返回数量，并没有返回哪一个或哪几个文件描述符，所以通常在使用select()之后，还会在循环结构中的if语句中使用宏函数FD_ISSET进行遍历，直到找出所有的满足就绪条件的描述符。最后将描述符集合通过指定函数拷贝回用户空间，以便被进程处理。
	
poll() vs select()
- 没什么太大的不同，实现的思想是差不多的，不过poll是通过链表放置被监控的文件描述符，所以poll函数可以监控没有数量限制的文件描述符。
```



### (2) 、epoll()

```
优势：
(1).epoll_create()创建的epoll实例可以随时通过epoll_ctl()来新增和删除感兴趣的文件描述符，不用再和select()每个循环后都要使用FD_SET更新描述符集合的数据结构。
(2).在epoll_create()创建epoll实例时，还创建了一个epoll就绪链表list。而epoll_ctl()每次向epoll实例添加描述符时，还会注册该描述符的回调函数。当epoll实例中的描述符满足就绪条件时将触发回调函数，被移入到就绪链表list中。
(3).当调用epoll_wait()进行监控时，它只需确定就绪链表中是否有数据即可，如果有，将复制到用户空间以被进程处理，如果没有，它将被阻塞。当然，如果监控的对象设置为非阻塞模式，它将不会被阻塞，而是不断地去检查。

也就是说，epoll的处理方式中，根本就无需遍历描述符集合。

劣势：
高并发情况下，很多线程争抢CPU资源，很容易导致资源匮乏，速度还不一定比前两者处理来得快。
```

   

# 四、五种模型核心过程

## （一）同步阻塞

```
httpd去read()内核缓冲区，发现没有数据，就从磁盘加载数据到内核缓存区，这一步阻塞 ；
接着，加载好后（数据准备好），就让引用程序去复制内核缓存区的数据到程序缓冲区，这一步httpd也阻塞。

最快速、简单
```

## （二）同步非阻塞

```
httpd去read()内核缓冲区，发现没有数据，就从磁盘加载数据到内核缓存区，这一步不阻塞!!!
当read()时，如果没有加载好，自动返回一个错误，不让httpd阻塞，httpd只能再次read()，重复这个步骤，直到加载好
这一步就是 polling ，轮询。
加载好后，通知app去复制，这一步，httpd阻塞。
```

## （三）IO多路复用

**扩展：IO多路模型：select \ poll \ epoll**

```
httpd不再直接read()，而是发起一个系统调用select() [ps：多路复用模型] ，由select()去监控fd_set .
select()会阻塞httpd，有三种阻塞：固定时间time , 0 , 一直等待就绪状态
	[ps：这里的就绪状态，指 是否可读、是否可写、异常；]
	[ps: 当三种情况中一种满足时，httpd就会恢复，尝试read()]
当某个fd ok时，httpd会再次read(),让app加载，如果，这里让CPU完全负责加载，就是阻塞，否则就是异步非阻塞！！
httpd在read之后，又按照上面select()规定阻塞的，进行阻塞。
```

## （四）信号量

```
Signal-driven IO , 即信号驱动IO模型

开启信号驱动模型，先建立信号处理的系统调用 sigaction() --→ 去准备数据 ， 这一步不阻塞
当data ok时，就会发送一个sigio信号，告诉httpd已经加载好了，httpd再让app去复制， 这一步阻塞。

```



## （五）异步AIO

```
Asynchronous I/O

在IO多路复用中，data ok后，如果从kernel buffer 到 app buffer是非阻塞的，那么整个过程就是异步的，只需要在copy ok 后告诉app 即可。

异步操作，就是 从数据准备 ，到数据复制完成，都是异步的，只有全部做好了，httpd才会通知app去使用。httpd全程不阻塞。

前面的同步，在磁盘到kernel buffer这里可阻塞可不阻塞，但是在 kernel buffer 到 app buffer,这里就是同步的意义，必须阻塞，而异步则也不需要阻塞。
```













