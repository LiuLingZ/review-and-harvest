## 1、@PropertySource

```java
这个注解配合@ConfigurationProperties使用，用来找到prefix所在的文件位置。
默认是application.yml  或 application.properties

但是！！！！！！！！！！！！！

一、
@PropertySource（"classpath:book.properties"）仅默认支持名为 application 的yml，其他名字找不到！！！
想多yml就需要改配置。所以老实用 .properties 吧 。

二、
含有@ConfigurationProperties的类，一定要有个 无参 的构造器 ！！！！！！！
```

