# Spring IOC

## 广义IOC

1. IoC(Inversion of Control) 控制反转，即“不用打电话过来，我们会打给你”。

   两种实现： 依赖查找（DL）和依赖注入（DI）。

   IOC 和 DI 、DL 的关系（这个 DL，Avalon 和 EJB 就是使用的这种方式实现的 IoC）：

![img](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbudJ3ibFhbAgAqrE0GSLrBlBiandBUJKFOjUYvuETtDiaSoJQtTC6rkNopMqG8vXIBibbqmjNmk0hlLs7A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

2. DL 已经被抛弃，因为他需要用户自己去是使用 API 进行查找资源和组装对象。即有侵入性。
3. DI 是 Spring 使用的方式，容器负责组件的装配。

## Spring的IOC

Spring 的 IoC 设计支持以下功能：

1. 依赖注入
2. 依赖检查
3. 自动装配
4. 支持集合
5. 指定初始化方法和销毁方法
6. 支持回调某些方法（但是需要实现 Spring 接口，略有侵入）

其中，最重要的就是依赖注入，从 XML 的配置上说， 即 ref 标签。对应 Spring RuntimeBeanReference 对象。

###### 对于 IoC 来说，最重要的就是容器。容器管理着 Bean 的生命周期，控制着 Bean 的依赖注入。



那么， Spring 如何设计容器的呢？Spring 作者 Rod Johnson 设计了两个接口用以表示容器。

1. BeanFactory
2. ApplicationContext

BeanFactory 粗暴简单，可以理解为就是个 HashMap，Key 是 BeanName，Value 是 Bean 实例。通常只提供注册（put），获取（get）这两个功能。我们可以称之为 “低级容器”。

ApplicationContext 可以称之为 “高级容器”。因为他比 BeanFactory 多了更多的功能。他继承了多个接口。因此具备了更多的功能。

**该接口定义了一个 refresh 方法，用于刷新整个容器，即重新加载/刷新所有的 bean。**



## Spring IoC容器设计

```java
两条设计路线：


//BeanFactory <I> → HierarchicalBeanFactory <I> → ConfigurableBeanFactory <I> 。

- BeanFacory：
	最基本的IoC容器接口，定义了容器是否有Bean、Bean类型（单例、多例）、别名这些基本功能。
- HierachicalBeanFactory：
	继承了前者，并且定义了关于父容器相关的方法：getParentBeanFactory（），使BeanFactory具备双亲IoC容器的
	管理功能。
- ConfigurableBeanFactory:
	定义了BeanFactory的大多数配置功能，如添加Bean的后置处理器：addBeanPostProcessor()。
	
//最终层层叠加，定义了BeanFactory 简单IoC容器的基本功能。

-------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------

// BeanFactory → ListableBeanFactory → ApplicationContext → （我们常用的）WebApplicationContext/ConfigurableApplicationContext

- 这是以ApplicationContext为核心的接口设计
- 这个体系中，ApplicationContext继承了ListableBeanFactory 和 HierarchicalBeanFactory两接口，这两接口再	 继承于 BeanFactory，这两接口是此实现的桥梁。
- ListableBeanFactory：
	细化了很多BeanFactory接口的方法，如getBeanDefinitionNames（）；
- ApplicationContext：
	继承了其他接口如MessageSource和ApplicationEventPublisher等，在简单的BeanFactory上扩展了额外的高级容
	器的特性。



/*
-	上面是主要的接口关系，大部分具体的IoC容器实现都是实现某些接口的，如DefaultListableBeanFactory就是实现
	ConfigurableListableBeanFactory接口（其继承于ListableBeanFactory）。

-	这个系统是以BeanFactory和ApplicationContext为核心设计的。ApplicationContext除了继承了BeanFactory系	  列下的ListableBeanFactory和HierarchicalBeanFactory接口，还额外实现了其他如资源、环境相关的接口，赋予     更高级容器的特性。

*/

-------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------------------------

//ApplicationContext ：
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
	···
}
	











```



## Spring IoC 的初始化过程

1. 用户构造 ClassPathXmlApplicationContext（简称 CPAC）
2. CPAC 首先访问了 “抽象高级容器” 的 final 的 refresh 方法，这个方法是模板方法。所以要回调子类（低级容器）的 refreshBeanFactory 方法，这个方法的作用是使用低级容器加载所有 BeanDefinition 和 Properties 到容器中。
3. 低级容器加载成功后，高级容器开始处理一些回调，例如 Bean 后置处理器。回调 setBeanFactory 方法。或者注册监听器等，发布事件，实例化单例 Bean 等等功能，这些功能，随着 Spring 的不断升级，功能越来越多，很多人在这里迷失了方向 ：）。

简单说就是：

1. 低级容器 加载配置文件（从 XML，数据库，Applet），并解析成 BeanDefinition 到低级容器中。
2. 加载成功后，高级容器启动高级功能，例如接口回调，监听器，自动实例化单例，发布事件等等功能。

**所以，一定要把 “低级容器” 和“高级容器” 的区别弄清楚。**





## getBean 的流程



![img](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbudJ3ibFhbAgAqrE0GSLrBlBiac4HQF1hO8mSMiaZ9o9Uq1Kq5m8I4QQ2jhye54IWqPR5CzjzibUibia0BvA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



getBean的操作都是在低级容器里操作的。其中有个递归操作

```
假设：当 Bean_A 依赖着 Bean_B，而这个 Bean_A 在加载的时候，其配置的 ref = “Bean_B” 在解析的时候只是一个占位符，被放入了 Bean_A 的属性集合中，当调用 getBean 时，需要真正 Bean_B 注入到 Bean_A 内部时，就需要从容器中获取这个 Bean_B，因此产生了递归。
```

所以，Spring 将其分为了 2 个步骤：

1. 加载所有的 Bean 配置成 BeanDefinition 到容器中，如果 Bean 有依赖关系，则使用占位符暂时代替。
2. 然后，在调用 getBean 的时候，进行真正的依赖注入，即如果碰到了属性是 ref 的（占位符），那么就从容器里获取这个 Bean，然后注入到实例中 —— 称之为依赖注入。
   可以看到，依赖注入实际上，只需要 “低级容器” 就可以实现。

这就是 IoC。

所以 ApplicationContext refresh 方法里面的操作不只是 IoC，是高级容器的所有功能（包括 IoC），IoC 的功能在低级容器里就可以实现。