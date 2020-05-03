# 动态代理

https://www.cnblogs.com/xiaoluo501395377/p/3383130.html#undefined

https://www.cnblogs.com/cenyu/p/6289209.html

**代码见 \IDEAProject\JavaSE\project_test\dynamicProxyJDK**





## 一、JDK自带动态代理

是基于JDK实现的，需要被代理类实现接口，所以也叫 JDK动态代理、接口代理。



在java的动态代理机制中，有两个重要的类或接口，一个是 InvocationHandler(Interface)、另一个则是 Proxy(Class)	，这一个类和接口是实现我们动态代理所必须用到的 。



每一个动态代理类都必须要实现InvocationHandler这个接口，并且每个代理类的实例都关联到了一个handler，当我们通过代理对象调用一个方法的时候，这个方法的调用就会被转发为由InvocationHandler这个接口的 invoke 方法来进行调用。即：**invoke（）是完整的代理方法，其中包含了真实的方法调用以及额外的增强** 。 



Proxy这个类的作用就是用来动态创建一个代理对象的类，它提供了许多的方法，但是我们用的最多的就是 newProxyInstance 这个方法：这个方法的作用就是得到一个动态的代理对象，其接收三个参数

```java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) throws IllegalArgumentException

loader:　　一个ClassLoader对象，定义了由哪个ClassLoader对象来对生成的代理对象进行加载

interfaces:　　一个Interface对象的数组，表示的是我将要给我需要代理的对象提供一组什么接口，如果我提供了一组接口给它，那么这个代理对象就宣称实现了该接口(多态)，这样我就能调用这组接口中的方法了

h:　　一个InvocationHandler对象，表示的是当我这个动态代理对象在调用方法的时候，会关联到哪一个InvocationHandler对象上
```



### 1、三者关系



####被代理类（真实类）

* 需要实现一个接口

####代理类

* 需要实现一个InvovationHandler接口，其中提供的唯一方法 invoke(被代理的对象，Object args)
* invoke（）是实际的代理方法

####生成代理对象

* Proxy的newProxyInstance()方法创建真正的代理对象
* 可以调用被代理的业务方法，因为传入了被代理类实现的接口参数，但是，**不是被代理的类型，其类型可转换为我们传入的接口集合中的任何一个。**



一个代理对象和一个InvocationHandler对象关联，代理对象调用被代理对象的方法的时候，就会交给这个handler对象负责， 其invoke方法就是核心的代理方法。





## 二、Cglib实现动态代理



```
Cglib代理，也叫子类代理，是在内存中构建一个子类对象从而实现对目标对象功能的扩展 :
	- JDK代理需要对象实现接口，Cglib不需要。
	- Cglib是一个强大的高性能的代码生成包，可在运行期扩展Java类和接口。
	- Cglib包底层是使用字节码处理框架ASM来转换字节码并生成新的类，由JVM指令编写。
	- 因为是以继承的形式创建代理对象，所以对于final类、final方法都无法予以代理、拦
	  截。
```





## 三、静态代理

```
代理类需要和被代理类实现相同接口，这样类型相同，调用时才可传入代理类的类型。

缺点：
	当一个对象需要被代理时，就必须编写一个额外的代理类，容易产生很多和业务无关的类，并且当接口修改时，需要维护代理类。
```







## 四、对比JDK和Cglib

### JDK优势

* 最小化依赖关系，JDK本身支持
* 因为JDK本身支持，所以随着JDK版本升级而升级，稳定，新版JDK都能用。

###cglib框架优势

* 被代理的对象不需要实现额外接口。这是非侵入的
* 性能会高些，采用ASM框架。





### 画外音

并不是JDK性能就一定比cglib差，有的差不多，并且JDK的动态代理也开始使用ASM字节码框架进行操作。使用更多看实际情况有没有实现接口的。

```
Spring的AOP中，对象有接口就使用JDK动态代理。没有就使用Cglib动态代理

静态代理需要代理类实现相同接口
JDK动态代理只要求被代理对象实现接口
Cglib不拦截final类或者final方法
```









