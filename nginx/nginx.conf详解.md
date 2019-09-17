整理于 ： https://blog.csdn.net/wangbin_0729/article/details/82109693

# nginx.conf详解



#### ps：查找文件位置：

`ps -ef|grep nginx`  可查看配置文件位置 ； 同时注意nginx此时监听的端口，可能出冲突

#### ps ： 日志文件很重要，报错可以看



## 一、标准的nginx.conf

```.conf
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
         
    access_log  /var/log/nginx/access.log  main;
         
    sendfile        on;
    #tcp_nopush     on;
             
    keepalive_timeout  65;

    #gzip  on;
    
    include /etc/nginx/conf.d/*.conf;

    server{ 
        server_name nginx.demo.com ;
    
        root /tmp/nginx/data/www/hello.html ;

        listen 8880 ; 
        location / {
                root /tmp/nginx/data/www;
        }
        location /images/ {
                root /tmp/nginx/data/images;
        }
    }
}

```

## 二、详解

### 1、大体

main(全局) 、 server(主机) 、 upstream(负载均衡服务器设置) 、 location(URL匹配特定位置的设置)

- main块设置的指令将影响其他所有设置；
- server块的指令主要用于指定主机和端口；
- upstream指令主要用于负载均衡，设置一系列的后端服务器；
- location块用于匹配网页位置。

### 2、nginx全局配置

```sql
--主模块指令，指定Nginx Worker进程运行用户以及用户组，默认由nobody账号运行。
user nobody nobody;
--指定了Nginx要开启的进程数。
worker_processes 2;
error_log logs/error.log notice;
pid logs/nginx.pid;
--绑定worker进程和CPU， Linux内核2.4以上可用。
worker_rlimit_nofile 65535;
 
events{
use epoll;
worker_connections 65536;
}
```



### 3、event事件

```sql
events {
	use epoll ;
	-- Nginx每个进程的最大连接数，默认是1024。
    worker_connections  1024;
    keepalive_timeout 60;
}
```



- 设定Nginx的工作模式及连接数上限

- use  ，事件模块指令，指定nginx工作模式，select  \ poll  \  epoll 等  

- worker_connections 是一个进程最大的链接数，那么整个程序最大连接数为：

  max_connections = worker_processes[ main中配置 ]  * worker_connections [events中配置] ;



### 4、HTTP服务器配置

```sql
http {
	--包含额外文件，避免主文件冗长
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
         
    access_log  /var/log/nginx/access.log  main;
   
    --设置允许客户端请求的最大的单个文件字节数；
     client_max_body_size 20m;
    --指定来自客户端请求头的headerbuffer大小，一般1k就够了
	client_header_buffer_size 32K;
	--指定客户端请求中较大的消息头的缓存最大数量和大小， “4”为个数，“128K”为大小，最大缓存量为4个128K；
	large_client_header_buffers 4 32k;
    --开启高效文件传输模式。将tcp_nopush和tcp_nodelay两个指令设置为on用于防止网络阻塞；        
    sendfile        on;
    #tcp_nopush     on;
    --客户端连接保持活动的超时时间，超时关闭连接。
    keepalive_timeout  65;

	
    #gzip on;
    --设置允许压缩的页面最小字节数，页面字节数从header头的Content-Length中获取。默认值是0，不管页面多大都进行压缩。建议设置成大于1K的字节数，小于1K可能会越压越大
    #gzip_min_length 1k;
    --表示申请4个单位为16K的内存作为压缩结果流缓存，默认值是申请与原始数据大小相同的内存空间来存储gzip压缩结果；
    #gzip_buffers 4 16k;
    --识别HTTP协议版本，默认是1.1，目前大部分浏览器已经支持GZIP解压，使用默认即可；
    #gzip_http_version 1.1;
    --指定GZIP压缩比，1 压缩比最小，处理速度最快；9 压缩比最大，传输速度快，但处理最慢，也比较消耗cpu资源；
    #gzip_comp_level 2;
    --指定压缩的类型，无论是否指定，“text/html”类型总是会被压缩的；
    #gzip_types text/plain application/x-javascript text/css application/xml;
    --可以让前端的缓存服务器缓存经过GZIP压缩的页面，例如用Squid缓存经过Nginx压缩的数据。
    #gzip_vary on;

    
    include /etc/nginx/conf.d/*.conf;

	-- 包含 server {}
	server { ... }
}

```

- HttpGzip模块。这个模块支持在线实时压缩输出数据流。



### 5、负载均衡配置

```sql
upstream lingz.com{
    ip_hash;
    server 192.168.8.11:80;
    server 192.168.8.12:80 down;
    server 192.168.8.13:8009 max_fails=3 fail_timeout=20s;
    server 192.168.8.146:8080;
}
```

- upstream 是 Nginx 的HTTP Upstream 模块，此模块通过一个简单的调度算法来实现客户端IP到后端服务器的负载均衡。

- upstream指令指定了一个负载均衡器的名称 lingz.com , 名字可以随意指定，只要后面调用的地方一致即可。

- 负载均衡策略：轮询、Weight 、iphash 、f最短时间响应 、 最少次数 、 url_hash 

- 可以设置服务器在负载均衡中的状态

  ```sql
  eg : server 192.168.8.12:80 down;
  
  - down：表示当前的server暂时不参与负载均衡；
  - backup：预留的备份机器。当其他所有的非backup机器出现故障或者忙的时候，才会请求backup机器，因此这台机			器的压力最轻；
  - max_fails：允许请求失败的次数，默认为1。当超过最大次数时，返回proxy_next_upstream 模块定义的错误；
  - fail_timeout：在经历了max_fails次失败后，暂停服务的时间。max_fails可以和fail_timeout一起使用。
  
  -- PS : 注意，当负载调度算法为ip_hash时，后端服务器在负载均衡调度中的状态不能是weight和backup。
  ```



### 6、server虚拟主机配置

ps : 建议将对虚拟主机进行配置的内容写进另外一个文件，然后通过include指令包含进来，这样更便于维护和管理。

```sql
server{
	--，listen用于指定虚拟主机的服务端口,监听的端口。
    listen 80;
    -- 指定IP地址或者域名，多个域名通过空格分开。
    server_name 192.168.8.18 lingz.com;
    -- index用来设定访问的默认首页地址。
    index index.html index.htm ;
    -- root指令用于指定虚拟主机的网页根目录,可以是相对路径，也可以是绝对路径。
    root /var/data/nginx/www/
    -- 设置网页的默认编码格式.
    charset utf8;
    -- 指定此虚拟主机的访问日志存放路径，最后的main用于指定访问日志的输出格式。这里和main全局设置的一样
    access_log logs/www.ixdba.net.access.log main;
}
```



### 7、location URL匹配配置

​	server块中。location支持正则表达式匹配。支持条件判断。可以通过location 实现Nginx**对动、静页面进行过滤处理** 、 **还可实现动态解析或负载均衡**。

```sql
-- 以.gif、.jpg、.jpeg、.png、.bmp、.swf结尾的静态文件都交给nginx处理，而expires用来指定静态文件的过期时间，这里是30天。
location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ {
    root /wwwroot/www.lingz.com;
    expires 30d;
}


--将upload和html下的所有文件都交给nginx来处理，当然，upload和html目录包含在/web/wwwroot/www.lingz.com目录中。
location ~ ^/(upload|html)/ {
root /web/wwwroot/www.lingz.com;
expires 30d;
}

--location是对此虚拟主机下动态网页的过滤处理，也就是将所有以.jsp为后缀的文件都交给本机的8880端口处理。
location ~ .*.jsp$ {
index index.jsp;
proxy_pass http://localhost:8880;
}
```





### 8、StubStatus模块配置

StubStatus模块能够获取Nginx自上次启动以来的工作状态，此模块非核心模块，需要在Nginx编译安装时手工指定才能使用此功能。以下指令实指定启用获取Nginx工作状态的功能。

```sql
location /NginxStatus {
    -- 启动StubStatus工作状态统计功能
    stub_status on;
    -- 指定StubStatus模块的访问日志
    access_log logs/NginxStatus.log;
    -- auth_basic是Nginx的一种认证机制。
    auth_basic "NginxStatus";
    --auth_basic_user_file用来指定认证的密码文件，由于Nginx的auth_basic认证采用的是与Apache兼容的密码文件，因此需要用Apache的htpasswd命令来生成密码文件，例如要添加一个test用户，可以使用下面方式生成密码文件：
    auth_basic_user_file ../htpasswd;
}
```

（1）生成密码文件：`/usr/local/apache/bin/htpasswd -c  /opt/nginx/conf/htpasswd test` , 后输两次密码进行确认。

（2）查看Nginx运行状态，可以输入 http://ip/NginxStatus 。输入用户名 test 和 密码。

```sql
Active connections: 1
server accepts handled requests
34561 35731 354399
Reading: 0 Writing: 3 Waiting: 0

Active connections表示当前活跃的连接数，第三行的三个数字表示 Nginx当前总共处理了34561个连接， 成功创建次握手， 总共处理了354399个请求。最后一行的Reading表示Nginx读取到客户端Header信息数， Writing表示Nginx返回给客户端的Header信息数
，“Waiting”表示Nginx已经处理完，正在等候下一次请求指令时的驻留连接数。

```



### 9、错误信息返回

```sql
--error_page指令可以定制各种错误信息的返回页面。在默认情况下，Nginx会在主目录的html目录中查找指定的返回页面，特别需要注意的是，这些错误信息的返回页面大小一定要超过512K，否者会被ie浏览器替换为ie默认的错误页面。

error_page 404 /404.html;
error_page 500 502 503 504 /50x.html;
location = /50x.html {
	root html;
}

```









