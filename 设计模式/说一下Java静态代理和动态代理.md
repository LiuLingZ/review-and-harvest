# 动态代理

[详细博客链接点我](https://www.cnblogs.com/xiaoluo501395377/p/3383130.html#undefined)

**代码见 \IDEAProject\JavaSE\project_test\dynamicProxyJDK**





## 一、JDK自带动态代理

在java的动态代理机制中，有两个重要的类或接口，一个是 InvocationHandler(Interface)、另一个则是 Proxy(Class)	，这一个类和接口是实现我们动态代理所必须用到的 。



每一个动态代理类都必须要实现InvocationHandler这个接口，并且每个代理类的实例都关联到了一个handler，当我们通过代理对象调用一个方法的时候，这个方法的调用就会被转发为由InvocationHandler这个接口的 invoke 方法来进行调用。即：**invoke（）是完整的代理方法，其中包含了真实的方法调用以及额外的增强** 。 



Proxy这个类的作用就是用来动态创建一个代理对象的类，它提供了许多的方法，但是我们用的最多的就是 newProxyInstance 这个方法：这个方法的作用就是得到一个动态的代理对象，其接收三个参数



### 1、三者关系



####被代理类（真实类）

* 需要实现一个接口

####代理类

* 需要实现一个InvovationHandler接口，其中提供的唯一方法 invoke(被代理的对象，Object args)
* invoke（）是实际的代理方法

####生成代理对象

* Proxy的newProxyInstance()方法创建真正的代理对象
* 可以调用被代理的业务方法，因为传入了被代理类实现的接口参数，但是，不**是被代理的类型，而是一个Proxy的内部类的类型！！！！！！**





## 二、Cglib实现动态代理

https://www.cnblogs.com/whgk/p/6627187.html

CGLib采用了非常底层的字节码技术，其原理是通过字节码技术为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑，所以说被代理的类不能有final关键字　

## 三、ASM简介





## 四、对比JDK和Cglib

### JDK优势

* 最小化依赖关系，JDK本身支持
* 因为JDK本身支持，所以随着JDK版本升级而升级，稳定，新版JDK都能用。

###cglib框架优势

* 被代理的对象不需要实现额外接口。这是非侵入的
* 性能会高些，采用ASM框架。



### 画外音

并不是JDK性能就一定比cglib差，有的差不多，并且JDK的动态代理也开始使用ASM字节码框架进行操作。使用更多看实际情况有没有实现接口的。







