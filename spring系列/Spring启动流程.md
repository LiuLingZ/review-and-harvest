



<https://www.jianshu.com/p/280c7e720d0c>

# Spring 启动流程



1、配置 web.xml  （指定了spring的配置文件路径和一个上下文启动监听器 ContextLoaderListener）

2、这个 ContextLoaderListener 就是回去执行初始化 ApplicationContext 容器操作

3、这里的初始化主要有三步

 - 创建 WebApplicationContext （可以自定义的，只要实现ConfigurationApplicationContext）
 - 加载Spring配置文件的配置的Bean，因为在web.xml已经指定了spring.xml的位置。这里面就是配置了的详细内容，核心是调用  **refresh()** 方法完成所有的bean的加载。ps : 在 configureAndRefreshWebApplicationContext 方法
 - 最后将加载的web容器放到 servletContext 中即可（Web全局变量）



过程：

web.xml配置资源文件位置和配置一个Context启动监听器 → 启动触发监听器 → 创建 webApplicationContext → 

通过指定的资源文件加载bean （refresh）, 需要我们上面创建的那个容器 →  将容器放到 servletContext中