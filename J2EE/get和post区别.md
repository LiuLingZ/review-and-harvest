常规区别：

```
1、传输位置：
	get传参暴露在url中 ，post则存于request请求体中发送 ； 
2、长度限制：
	倒不是说get本身能传的长度有限，而是因为get是通过url传，而不同浏览器本身限制了，url的传输长度， 	  间接导致get传的参数长度受限。
3、效率上
	get请求会被浏览器缓存，post不会，比如刷新界面时，get不发请求，直接使用缓存，post就重新提交请	 求。	

```



特殊：

````
get 和 post 本质上都是一样的 。 

	因为他们都是http请求，http请求是建立在TCP/IP上的，get和post发送的包都是相同的，只是http制定了规则，服务于不同的情况。
	有篇博客说到是  get是发一个数据包，post发两个，一个header,对方接收到再发body。
	抓包了一波，没发现，也有人对此进行论证，最后确实是有。
````



​	博客：https://www.cnblogs.com/logsharing/p/8448446.html

​	论证：https://blog.csdn.net/zerooffdate/article/details/78962818

