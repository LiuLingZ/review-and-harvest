# Spring的Bean的作用域和生命周期



## 1、作用域

### singleton

```
- Spring IOC容器中只有一个，并且容器负责其创建、运行、销毁的整个生命周期。
- 默认作用域：singleton
- 容器中仅此一个bean， 建议放的是无状态的bean，否则线程不安全
- 容器初始化就加载 。 
- 可以懒加载 ： lazy-init="true" 
```

### prototype

```
- 意为“原型” 。 每次请求此bean，都会获得一个新创建的bean 
- 容器指负责其初始化，并赋给请求，便再也不负责其生命周期，完全交由请求去处理。
- 可以放有状态的对象，并发安全。但是也会加重内存负担。
```

### request

```
- 针对每个 HTTP 请求，每个请求都会提供一个属于自己的bean。并且此bean只在当前HTTP Request中有效。
- 请求结束，销毁bean
- request作用域仅基于 web 的SpringApplicationContext有效（仅spring容器有效）
```

### session

```
- 针对每个 Session，每个请求都会提供一个属于自己的bean。并且此bean只在当前Session中有效。
- Session过期，销毁bean
- Session作用域仅基于 web 的SpringApplicationContext有效（仅spring容器有效）
```

### global session

```
- 针对的是一个portlet的全局session，在这个portlet的Session都会共享这个bean
- global session仅在spring容器中起作用
```



## 2、声明周期



![img](https://img-blog.csdn.net/20160417164808359?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

上图也约等于：

![img](https://upload-images.jianshu.io/upload_images/4638441-05bf2b9b2f2a01d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Bean实例生命周期的执行过程如下：

Spring对bean进行实例化，默认bean是单例；

Spring对bean进行依赖注入；

如果bean实现了BeanNameAware接口，spring将bean的id传给setBeanName()方法；

如果bean实现了BeanFactoryAware接口，spring将调用setBeanFactory方法，将BeanFactory实例传进来；

如果bean实现了ApplicationContextAware接口，它的setApplicationContext()方法将被调用，将应用上下文的引用传入到bean中；

如果bean实现了BeanPostProcessor接口，它的postProcessBeforeInitialization方法将被调用；

如果bean实现了InitializingBean接口，spring将调用它的afterPropertiesSet接口方法，类似的如果bean使用了init-method属性声明了初始化方法，该方法也会被调用；

如果bean实现了BeanPostProcessor接口，它的postProcessAfterInitialization接口方法将被调用；

此时bean已经准备就绪，可以被应用程序使用了，他们将一直驻留在应用上下文中，直到该应用上下文被销毁；

若bean实现了DisposableBean接口，spring将调用它的distroy()接口方法。同样的，如果bean使用了destroy-method属性声明了销毁方法，则该方法被调用；



**当然，很多时候我们并不会真的去实现上面说描述的那些接口**

