BeanDefinition的信息来源：Resource类

```java
信息来源被封装为Spring中的Resource类。 Resource类是Spring用来封装I/O操作的类。

	//我们的BeanDefinition信息以XML形式存在，就可以：
	ClassPathResource res = new ClassPathResource("beans.xml");
	//然后将res作为构造参数给具体的容器，来完成容器的初始化和依赖注入过程
	//容器内部用对应的Reader类解析Resource
	//loadBeanDefinitions是一个重要的过程
	/*
		Resource定位BeanDefinition ;
		用Resource构建容器；
		Reader加载Resource → loadBeanDefinition 过程；
	*/

	eg :
	ClassPathResource res = new ClassPathResource("beans.xml");
	DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
	XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
	reader.loadBeanDefinitions(res) ;
	//此时，容器已经建立，可以直接使用。

```



