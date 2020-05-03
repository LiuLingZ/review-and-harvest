<https://mp.weixin.qq.com/s/I_Qb508B7jhLHAaUxPAjJA>

一个消息两个部分：
	payload 有效载荷 ：真实的数据内容 
	label 标签 ： Exchange 的名字或者一个tag，描述 payload 【通过label将消息发给对应的Consumer】


Consumer订阅消息队列。

Consumer 和 Product 通过 TCP 连接到 Broker上的，
但不是通过TCP传输，而是在TCP之上建立多个channel，在channel上面传输。
因为频繁建立断开TCP消耗资源，并且建立的TCP连接数量有限

一个 channel代表一个会话任务，一个TCP上可建立多个channel

过程：
	生产者将消息发到 Exchange ， 再有 Exchange 投递到绑定的 Queue 中。

Exchange 有四种类型：

routing key + [ exchange type + binding key ]

routing key 长度255 bytes

exchange 和 queue 绑定会指定一个 binding key
当rounting key == binding key时，就会将消息从exchange递交到对应的queue上。



Exchange Types ： 
	fanout 
		所有绑定的queue全部发送，无视binding key
	direct 
		binding key 完全匹配 routing key时发送。
	topic 
		支持以 . 为分隔符的 模糊匹配
		* 匹配一个单词 *.*.red  → green.yellow.red √

匹配多个单词 #.red  → sss.xxx.aa.red √

 		一定要有单词去模糊匹配*，#无规定，否则无法投递
	headers
		不采用 routing key ，而是通过取消息的header头部，也是键值对，然后匹配binding key。
		完全匹配就会投递。

### **使用 ack确认Message的正确传递**