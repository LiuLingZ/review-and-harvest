## Session 与 Cookie 区别（未更完）

#### 存在的原因，效果

​	因为HTTP协议的无状态性（即服务器并不能知道发送来的两次服务是否是由同一客户端发出的），故需要一种机制来识别是否是同一客户端发送的请求。常见的例子就是“记住密码”，“7天内免登陆”之类。为了保存请求客户的信息，行之有效的方法就是通过 Session 或Cookie 存储该请求用户的信息，当同一用户再次访问时，可自动识别。



​	**在此不长篇大论其具体内容，直接给出介绍和比对。**



#### cookie

 - 是一种在客户端保存状态的机制。当用户访问服务器时，需要将存于本地对cookie一并发送。
 - cookie是通过扩展HTTP协议实现，在HTTP的响应头添加一些特殊的信息以提示生成响应的cookie。cookie是由 ￥￥￥￥￥￥ 生成。
 - cookie 是由浏览器按一定原则在后台发送给服务器，浏览器检查所有存储的cookie，如果某个cookie所声明的作用范围大于将要请求的资源的位置，则把该cookie附在请求的HTTP请求头上一并发给服务器。
 - cookie内容包括



 - cookie 有会话cookie和持久化的cookie。
    - 会话cookie不将数据存储于硬盘中，而是存于内存中。当此次会话结束后，此cookie也就销毁了，这是不合规范的。
    - 而将cookie存于硬盘上进行持久化，可以实现多个新开窗口共享此cookie。持久化的cookie也会因为设置了过期时间而过期。

​		



#### session 

- session是存于服务端的机制，服务器用一种类似散列表的键值对形式存储信息。而客户端会存有一个唯一的sessionId来对应服务端对应的session 。

- 当客户发送请求时，服务端检查其是否包含了sessionId，若有，取出已创建的，若无，创建一个session存于服务端，并存储相关的客户信息。同时返回给客户一个sessionId用来唯一标识某session。

- 发送sessionId可以存于cookie中，一并发给服务端，session和cookie两者并不矛盾。

- cookie可以被禁止，当cookie被禁止时，可以采用：

  - URL重写方案，即请求的url后面添上sessionId=xxxxx

  - 表单隐藏字段，服务器自动修改表单，添加一个隐藏的字段：

  - ```html
    <form name="formName" action="/url">
      <input type="hidden" name="jsessionid" value="ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764">
      <!--注：这个jessionid就是sessionId,是服务器Tomcat中的叫法-->
      ·······
      </form>
    ```

    ​

#### 比对

​	cookie放在客户端，是通过请求发送一些额外的数据到服务器端，这意味着需要占用更多的带宽。

​	session是存于服务器的信息，当较多用户请求时，无形中增加服务器的内存压力，容易OOM。不过session相对安全。

​	对于每个域名，对cookie的数量和大小有一定限制（<4k&&多数<=20个）。且cookie并不安全，可以通过分析本地cookie篡改进行cookie欺骗。

​	所以，重要信息放Session，普通非重要信息，Cookie是合理的。



#### 参考：

《深入分析Java Web》

[cookie 和session区别详解](https://www.cnblogs.com/shiyangxt/articles/1305506.html)	

[W3Cschool](https://www.w3cschool.cn/chenyh1/chenyh1-2zbg2krm.html)		