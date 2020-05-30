<https://blog.csdn.net/gao_xu_520/article/details/76020365?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase>



## Vue

​	MVVM模式的框架。 

​	Model + View + ViewModel 

​	Vue的更新会更新到ViewModel上， ViewModel的更新也会更新到Vue上。避免频繁的DOM操作更新。

​	Vue本质还是封装DOM操作的，只是更高一层的MVVM架构。

**简而言之**：Vue.js是一个构建数据驱动的 web 界面的渐进式框架。Vue.js 的目标是通过尽可能**简单的 API 实现响应的数据绑定和组合的视图组件**。核心是一个**响应的数据绑定系统。**



渐进式的理解:
    不管是单页面还是多页面。首先都是通过声明式渲染声明每个字段，这是基本的要求。
基本要求不管什么页面，什么功能，都会使用声明式渲染，去渲染我们的字段，因为我们需要展现一些功能一些信息，那么就要通过渲染才可以。通常我们会把公共的头部和尾部抽出来，做成组件。这时候就需要使用组件系统。
单页面应用程序时往往是需要路由，这时候需要把vue的插件（vue-router）拉进来做路由，如果我们的项目足够复杂，大量的使用组件而且难以去管理组件的状态，这个时候我们使用vue-resource,vue-resource是集中来管理我们的状态的。项目完成后需要构建工具来build我们的系统，来提高我们的效果，最后形成完成的项目。



## JS

​	以DOM为基础的脚本。

