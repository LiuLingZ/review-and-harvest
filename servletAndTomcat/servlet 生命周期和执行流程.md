## servlet 生命周期和执行流程



### 一 、生命周期



servlet 声明周期可以分四个阶段： 

  - 类装载过程
  - init() 初始化过程
  - service() 服务过程，选择doGet \ doPost
  - destroy() 销毁过程



**servlet接口如下**

```java
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;

    ServletConfig getServletConfig();

    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    String getServletInfo();

    void destroy();
}
```



#### 1、创建servlet实例

时期：默认是第一个请求该servlet的时候就初始化此servlet，该servlet实例便一直存在，直到长			

​	   时间不调用、服务器关闭才销毁 或者 类文件更新后重新载入 。也可手动设置：在服务器

​	   启动时便加载此servlet 。

```Xml
<servlet>
	<servlet-name>Xxx</servlet-name>
     <servlet-class>com.lingz.Xxx</servlet-class>
     <load-on-startup>1</load-on-startup>
</servlet>
```



#### 2、init()初始化

​	servlet实例一创建出来，便调用init(ServletConfig var1) 进行初始化， 其中的ServletConfig参数对象携带了该servlet的配置信息，比如初始化参数，此ServletConfig参数由服务器创建。

（1）那么，如何配置 servlet的初始化参数？

```xml
<servlet>
	<servlet-name>Xxx</servlet-name>
	<servlet-class>com.lingz.Xxx</servlet-class>
  	<!--两个自定义的初始化参数-->
  	<init-param>
  		<param-name>value1</param-name>
  		<param-value>key1</param-value>
  	</init-param>
    <init-param>
  		<param-name>value2</param-name>
  		<param-value>key2</param-value>
  	</init-param>
</servlet>
```

​	通过这种配置方式，就不需要在Servlet中添加、修改，直接修改xml文件即可。

（2）如何读取上面的参数呢？

​	通过 ServletConfig类提供的 getInitParameter(String name) 获取初始化参数

```java
public interface ServletConfig {
    String getServletName();

    ServletContext getServletContext();

    String getInitParameter(String var1);

    Enumeration<String> getInitParameterNames();
}
```

（3）init(ServletConfig var1) 在Servlet生命周期中，只执行一次。并且是**单线程执行，不需要担心多线程安全。**



#### 3、service() 服务过程

（1）请求发到对应的Servlet，Servlet调用service（）,service() 根据请求调用doGet \ doPost

​	service方法是处理业务的核心。



（2）service() 与多线程

​	servlet 是单例的，当多个请求请求同一个servlet时，需要主要注意线程安全，不过也存在可以不必考虑线程安全的情况。

##### ①线程安全情况

 - 如果service()方法没有访问Servlet的成员变量也没有访问全局的资源比如静态变量、文件、数据库连接等，而是只使用了当前线程自己的资源，比如非指向全局资源的临时变量、request和response对象等。该方法本身就是线程安全的，不必进行任何的同步控制。
- 如果service()方法访问了Servlet的成员变量，但是对该变量的操作是只读操作，该方法本身就是线程安全的，不必进行任何的同步控制。

##### ②线程不安全情况

-  如果service()方法访问了全局的静态变量，如果同一时刻系统中也可能有其它线程访问该静态变量，如果既有读也有写的操作，通常需要加上同步控制语句。
- 如果service()方法访问了全局的资源，比如文件、数据库连接等，通常需要加上同步控制语句。





#### 4 、destroy()销毁

​	当web服务器认为此servlet没有存在的必要、类重新加载、服务器关闭、长时间未被访问，则可以从内存中销毁。而回收该Servlet内存前必须调用destroy()，web服务器保证该方法被调用时已经结束了请求调用的service()或等待剩余的请求执行完，并且不会再接收请求。当全部请求处理完并响应后，即可destroy() 并进行内存回收



### 执行流程

通过上面的描述，其实我们对执行流程已有了大体的认知：

 	1. 根据时机，Web容器加载对应的Servlet类,加载后进行init()初始化。
     - 设置了容器启动时初始化
     - 请求第一次请求此Servlet时初始化
     - Servlet类文件被更新后，重新装载Servlet
	2. 接收到请求，容器根据配置将请求交给对应的Servlet，同时创建HttpServletRequest 和 HttpServletResponse 对象，一并交给Servlet。
	3. 在service()中根据HttpServletRequest得请求类型等信息，调用doGet\doPost 进行业务处理。
	4. 处理后通过HttpServletResponse获得相应信息，返回给Web容器。
	5. Web容器再将响应返回给客户端。







参考：[https://www.cnblogs.com/Wonghy/p/5542277.html](https://www.cnblogs.com/Wonghy/p/5542277.html)





























