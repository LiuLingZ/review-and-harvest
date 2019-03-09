https://www.cnblogs.com/linguoguo/p/5106618.html



**HTTP协议是无状态的协议。一旦数据交换完毕，客户端与服务器端的连接就会关闭，再次交换数据需要建立新的连接。这就意味着服务器无法从连接上跟踪会话**。

为了保证同一用户的所有请求都是同一个会话，引入了Cookie 和 Session。



## Cookie

## 1、机制

- 一小段文本，保存在客户端，客户端保存用户状态的方式。每次发送请求时带上cookie，由服务器解析验证，以此辨认用户状态，服务器还可根据需要修改Cookie的内容。



## 2、内容

- 不可跨域名性。不同域名使用不同的，自己的Cookie，不能混淆使用
- 中文转码。中文Unicode 占4个字符 ； 英文属于ASCII字符，内存只占2个字节，Cookie使用Unicode字符需要对Unicode字符进行编码，否则乱码（这就是为什么Cookie保存中文需要转码）
- 尽量保存精要信息 。
- Cookie设有有效期
- Cookie的安全性。可以明文也可加密后传输。cookie.setSecure(true) 后Cookie仅在HTTPS和SSL等安全协议传输。





## Session

​	Session是服务器端使用的一种记录客户端状态的机制，会增加服务器的存储压力。

## 1、机制

​	Session是另一种记录客户状态的机制，不同的是Cookie保存在客户端浏览器中，而Session保存在服务器上。客户端浏览器访问服务器的时候，服务器把客户端信息以某种形式记录在服务器上。这就是Session。客户端浏览器再次访问时只需要从该Session中查找该客户的状态就可以了。

​	Cookie相当于客户身上的“通行证” ， 而Session则是保存在服务端的“客户明细表”，来访时查看。



## 2、内容

- Session生命周期

  ```
  - 为了获得更高的存取速度，服务器一般把Session放在内存里。每个用户都会有一个独立的
    Session。如果Session内容过于复杂，当大量客户访问服务器时可能会导致内存溢出。因   此，Session里的信息应该尽量精简。
  - Session在客户第一次访问服务器时自动创建，但是必须是访问Servlet\jsp这些，静态资 
    源是不会加载的。Session生成后，用户只要访问，服务器就会更新Session时间，超过一 
    段时间就失效。
  - Session也需要Cookie协助。客户端通过Cookie保存SessionID，发送请求时一并发送 
    SessionID,服务端通过这个Cookie识别是否为同一用户。
    如果客户端禁止Cookie，可以通过 URL地址重写 传输SessionID，将SessionID添加到 
    URL中发送到服务器。
    
    
  ```

  



## Cookie 和Session对比

1、cookie数据存放在客户的浏览器上，session数据放在服务器上。

2、cookie不是很安全，别人可以分析存放在本地的COOKIE并进行COOKIE欺骗
   考虑到安全应当使用session。

3、session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能
   考虑到减轻服务器性能方面，应当使用COOKIE。

4、单个cookie保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie。