# Mybatis的Mapper动态代理

https://blog.csdn.net/xiaokang123456kao/article/details/76228684

## 一、动态代理

```
JDK动态代理（也称“接口代理”）
- 顾名思义，JDK动态代理的被代理类需要实现一个或以上的接口
- 在JDK动态代理里面，有两个比较关键的类，一个是接口InvocationHandler,另外一个是类Proxy
- JDK动态代理生成的代理类是需要实现InvocationHandler接口的，这个接口里有个invoke方法，这个方法就是可以理解   为AOP的增强方法，也就是invoke方法里面会含代理的逻辑，并最终调用被代理的类的方法。
- 而有了这个代理类，就需要通过Proxy这个类的newProxyInstance()生成这个代理类的对象，这个代理类的对象由传入的接口	 参数决定类型，而这个接口参数，就是从我们被代理类来的，所以要求被代理的类需要实现接口。
- 这样，有了代理对象，就可以调用被代理的方法，实现Java的动态代理。

Cglib代理
- 也叫子类代理，是在内存中构建一个子类对象从而实现对目标对象功能的扩展 :
- JDK代理需要对象实现接口，Cglib不需要。
- Cglib是一个强大的高性能的代码生成包，可在运行期扩展Java类和接口。
- Cglib包底层是使用字节码处理框架ASM来转换字节码并生成新的类，由JVM指令编写。
- 因为是以继承的形式创建代理对象，所以对于final类、final方法都无法予以代理、拦截。
```





## 二、Mapper动态代理

### 1、导言

​	从我们现在习惯使用的mybatis定义同名的接口名、mapper命名空间便可实现操作DB的功能上看，我们只是定义了一个接口，并没有方法体，能做到这些是因为，**mybatis使用了JDK动态代理机制实现了代理类，替我们执行了配置好的sql语句。**

​	

### 2、解析

​	Mybatis关于包装Mapper的代码都在org.apache.ibatis.binding，其中有四个类需要重视：

​	- MapperRegistry (*注册Mapper接口与获取MapperProxyFactory*)

​	- MapperProxyFactory (*创建Mapper代理对象的工厂* )

​	- MapperProxy 

​        - MapperMethod



```java
MapperRegistry
- 这个类的作用是 （1）注册mapper接口 和 （2）获取生成代理类实例，即MapperProxyFactory 。
- 这个类有一个getMapper()，这里面是用来获取某个类型的mapper接口的方法，在这个方法里面同时会初始化 
  MapperProxyFactory，通过其来获得 用于生成该mapper接口代理对象的工厂。
  
MapperProxyFactory
- 这个类就是用来创建指定类型的mapper接口的代理对象
- 这个类的创建代理对象的方法，我们就可以看到是调用了 Proxy.newProxyInstance()方法，使用了JDK的动态代理

MapperProxy
- 上面的MapperProxyFactory是创建了代理对象，那么代理对象的方法细节，也就是为什么可以替我们调用sql、关闭资   源这些的固定操作，就是在这个MapperProxy里面定义的。
- 这个MapperProxy就是实现了InvocationHandler接口，就是代理类，实现了invoke方法，这个invoke方法就是代理方   法，包含了代理方法调用的细节。
- 这里的细节是指一些方法调用的细节，而不是具体代码的实现。
- MapperProxy的invoke方法最终，是使用MapperMethod去执行sql语句。

MapperMethod
- 现在，逻辑调用都有了，那么具体的模板细节，就交个MapperMethod实现。
- MapperMethod定义了SqlCommand和MethodSignature内部类，分别用来操作DB和对应XML定义的信息的。

```





### 3、总结

​	当我们定义mapper接口后，通过JDK动态代理实现自动映射功能。	

​	首先，MapperRegistry注册了Mapper接口（我们定义的），然后创建MapperProxyFactory对象 ； 接着通过MapperProxyFactory对象，创建指定的 mapper接口代理对象 mapperProxy。这个mapperProxyFacotry底层调用的就是Proxy.newProxyInstance() ；这个MapperProxy类，就是实现了InvocationHandler接口，其invoke方法就是代理的增强方法，而具体的方法实现，就是在invoke中调用 MapperMethod对象去实现。 MapperMethod内部定义了两个内部类，MethodSignature、SqlCommand，其职能就是：xml配置中定义的对应匹配，id啊，参数啊；和 真正执行DB操作。

所以调用链：MapperRegistry → MapperProxyFactory → MapperProxy → MapperMethod（MethodSignature和SqlCommand）















