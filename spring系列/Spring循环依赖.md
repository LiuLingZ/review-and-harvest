https://mp.weixin.qq.com/s/EY6KshHwLqq8QGDA4dTGbA

<https://www.cnblogs.com/yougewe/p/10146537.html>

# Spring 循环依赖



依赖分为强依赖和弱依赖：

​	强依赖：通过构造器生成的，或者说是需要这个依赖才能创建这个对象的。如构造器的传入的参数。

​	弱依赖：没有其也可以将对象本身初始化的。



Spring解决循环依赖

​	Spring的依赖调解只能解决弱依赖，对于强依赖没办法。

​	循环弱依赖解决：

​		创建A时，如果A依赖B，可以将A先放入一个Map中

​		这个map是 Map<String,ObjectFactory> ，key是BeanName，ObjectFactory的getBean可以获得这个A

​		递归创建B时，如果B要引用A，则会先从map里面取，如果有，此时将A的引用赋予B即可。

​		递归结束，将B的引用返回给A完成构建。



循环引用的解决过程是在 Bean生命流程的 init方法中的 。也就是建议初始化过程在构造器完成，注入在init完成。



## 解决

​	设计和编码上尽量避免循环依赖; 第二种是极个别的可以使用延迟加载。实际开发中，一般会避免循环依赖。