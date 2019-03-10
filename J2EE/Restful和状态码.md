用URL描述资源  /user/1

使用HTTP方法描述行为，使用HTTP状态码表示不同的结果

使用json交互数据

只是一种风格，不是强制标准。



https://mp.weixin.qq.com/s/MY-ZMaYkls4lVS3DnNIXnQ



## 一 、URL设计

### 1、动词+宾语

```
- 如：GET 是动词， /user是宾语，必须是名词，url不要出现动词如 /getUser 

- 动词：
	GET ：获取
	POST：新增
	PUT：更新
	DELETE：删除
	PATCH：局部更新
	
	其中，客户端只能使用GET 、POST，DELETE和PUT需要在前端覆盖POST进行发送。

```

### 2、避免多级URL

```
eg :
	GET /user/12/categories/2   语义不清晰，需要想一会
	GET /user/12?categories=2	除了第一级，其他级别的查询用查询字符串表达
```





## 二、状态码

​	客户端每次请求，服务端都要给回应，回应包括 ： 状态码 和 数据  两部分。

### 1、状态码

```
HTTP状态码是一个三位数，分5类级别：
	1xx : 相关信息
	2xx : 操作成功
	3xx : 重定向
	4xx : 客户端错误
	5xx : 服务端错误

API 不需要 1xx状态码
```

### 2、详解：

#### 	2xx 

```
200表成功，不同方法可以返回更精确的状态码

GET 200 ok
POST 201 created
PUT 200 OK
PATCH 200 OK
DELETE 204 No Content  //删除成功，没有资源了

202 Accepted : 表服务器已经接受请求但为处理，常做异步操作
```



#### 	3xx

```
API 用不到301状态码（永久重定向）和302状态码（暂时重定向，307也是这个含义），因为它们可以由应用级别返回，浏览器会直接跳转，API 级别可以不考虑这两种情况。

API 用到的3xx状态码，主要是303 See Other，表示参考另一个 URL。它与302和307的含义一样，也是"暂时重定向"，区别在于302和307用于GET请求，而303用于POST、PUT和DELETE请求。收到303以后，浏览器不会自动跳转，而会让用户自己决定下一步怎么办。下面是一个例子。

	HTTP/1.1 303 See Other
	Location: /api/orders/12345
```



#### 	4xx 

```
表客户端错误

400  Bad Request 客户端请求的参数存在问题，服务器不理解不处理，如少传一个参数
401 Unauthorized 用户未提供身份验证凭据，或没有通过身份验证
403 Forbidden（禁止的） 用户通过了身份验证，但是不具有访问资源所需的权限。
404 Not Found 请求资源不存在或不可用
405 Method Not Allowed 用户已经通过身份验证，但是所用的 HTTP 方法不在他的权限之内。
					 通俗地说，就是有相同的uri，但是没有对应的请求类型。如：
					 有 /user/1  GET ，但是请求的是 DELETE 类型
410 Gone 所请求的资源已从这个地址转移，不再可用。
415 Unsupported Media Type：客户端要求的返回格式不支持。比如，API 只能返回 JSON 格
						  式，但是客户端要求返回 XML 格式。
422 Unprocessable Entity ：客户端上传的附件无法处理，导致请求失败。
429 Too Many Requests：客户端的请求次数超过限额。
```



#### 	5xx

```
5xx状态码表示服务端错误。

500 Internal Server Error：客户端请求有效，服务器处理时发生了意外。
503 Service Unavailable：服务器无法处理请求，一般用于网站维护状态。
```





## 三、服务器回应

### 1、不要返回纯文本

​	API 返回的数据格式，不应该是纯文本，而应该是一个 JSON 对象，因为这样才能返回标准的结构化数据。所以，服务器回应的 HTTP 头的Content-Type属性要设为application/json。

### 2、发送错误，不要返回200状态码

​	有一种不恰当的做法是，即使发生错误，也返回200状态码，把错误信息放在数据体里面，这种做法实际上取消了状态码，这是完全不可取的。正确的做法是，状态码反映发生的错误，具体的错误信息放在数据体里面返回。

错误：

![img](https://mmbiz.qpic.cn/mmbiz_png/0aWl00iaQWSGajBxI5a06FY88MaO28WAd9B7ibN9kUu67T4lXoO3lAP0SIX2aDIrTqNNGhOlTkaz49QuGPxZQKww/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

正确：

![img](https://mmbiz.qpic.cn/mmbiz_png/0aWl00iaQWSGajBxI5a06FY88MaO28WAdnOVwuzWeEXIvYL6fDVZQfMS9K63GBKicsicTwJMvHHdxGF4S7qLnq3WQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



