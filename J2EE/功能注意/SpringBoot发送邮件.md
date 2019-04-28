注意：25端口默认被禁，可以通过第三方提供的加密465端口进行发送。否则无法找到smtp.qq.com

​	这里以QQ邮箱发送为例：



添加依赖：

```
		<!--配置邮箱发送功能，SpringBoot已经默认整合好-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>
```



配置application.yml ，使用465加密端口

```yml
spring:
	  mail:
    host: smtp.qq.com
    username: 572135877@qq.com
    password: # qq 邮箱配置的密码，非邮箱登录密码，在邮箱开启 smtp 的设置处配置，需要绑定手机并发短信才能设置
    defaultEncoding: UTF-8
    properties:
      mail:
        smtp:
        connectiontimeout: 5000
        timeout: 3000
        writetimeout: 5000
        auth: true
        ssl:
        trust: smtp.qq.com
        socketFactory:
          class: javax.net.ssl.SSLSocketFactory
          port: 465
          starttls:
          enable: true
          required: true
```



springboot已经自己整合好，直接测试

```java
	//自动注入
	@Autowired
    private JavaMailSender mailSender;

    @GetMapping("/email")
    public void emailDemo(){
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom("发送方邮箱@qq.com");
        message.setTo("接收方邮箱@qq.com");
        message.setSubject("主题：测试邮箱");
        message.setText("肚子饿了");
        mailSender.send(message);
    }
```









