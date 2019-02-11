# 鸟哥私房菜 第五章



## 导读

​	Linux一般将文件可存取的身份分为三类：owner/group/others

​	三种身份各位read/write/execute等权限



## 一、使用者与群组

1、

​	文件拥有者 owner

​		Linux是多用户任务的系统，每个角色独立使用的

​	群组概念 group

​		团队协作开发同一份文件时用上，非组成员不可使用

​	其他人 others

​		并不是这个群组的其中一个成员，而是外来的，那么，只能得到内部用户的

​		允许后才能进入群组。



​	万能者 root



2、Linux 使用者身份与群组记录的文件

​	在我们Linux系统当中，默认的情况下，所有的系统上的帐号与一般身份使用者，还有

​	那个root的相关信息， 都是记录在/etc/passwd这个文件内的。至于个人的密码则是记

​	录在/etc/shadow这个文件下。 此外，Linux所有的群组名称都纪录在/etc/group内！这

​	三个文件可以说是Linux系统里面帐号、密码、群组信息的集中地啰！





## 二、Linux文件权限概念

​	如何设置文件的权限给这些角色？

​	文件的权限和属性 ？

​	li -> list



### 1、 文件属性

​	- r w x r w x - - -

​	第一个字符： 

​		[ l ] 为 链接文件（link file）

​		[b] 为 设备文件里可供存储 的周边设备（可随机存取设备 RAM）

​		[c] 为 设备文件里 的序列埠设备，如键盘、鼠标，一次性读取设备

​	x : 代表可执行（execute）,如果是目录的话，代表可不可以进去这个目录



### 2、修改文件属性和权限

​	常用于群组、拥有者、各种身份的权限之修改的指令，如下所示：

* chgrp : 改变文件所属群组 

  ​	    **要被改变的群组名称必须要在/etc/group文件内存在才行，否则就会显示错**

  ​       	    **误！**

* chown：改变文件拥有者

* chmod ：改变文件的权限，SUID,SGID,SBIT等特性



	#### chgrp ：要修改的组名必须在 /etc/group文件内存才行

​	eg : 假设你已经知道在/etc/group里面已经存在一个名为users的群组， 但是testing这个

​		群组名字就不存在/etc/group当中了，此时改变群组成为users与

​		testing分别会有什么现象发生呢？

​	[root@study ~]# chgrp [-R] dirname/filename ...
​	选项与参数：
​	**-R : 进行递回（recursive）的持续变更**，亦即连同次目录下的所有文件、目录
​	都更新成为这个群组之意。常常用在变更某一目录内所有的文件之情况。

​	范例：

```shell
[root@study ~]# chgrp users initial-setup-ks.cfg
[root@study ~]# ls -l
-rw-r--r--. 1 root users 1864 May 4 18:01 initial-setup-ks.cfg
[root@study ~]# chgrp testing initial-setup-ks.cfg
chgrp: invalid group: `testing' == 错误，testing不在group中

```

​	   

#### chown 改变文件拥有者

​	**使用者必须在系统账号中，即/etc/passwd此文件夹中**。

```shell
[root@study ~]# chown [-R] 帐号名称 文件或目录
[root@study ~]# chown [-R] 帐号名称:群组名称 文件或目录
选项与参数：
-R : 进行递回（recursive）的持续变更，亦即连同次目录下的所有文件都变更
范例：将 initial-setup-ks.cfg 的拥有者改为bin这个帐号：
[root@study ~]# chown bin initial-setup-ks.cfg
[root@study ~]# ls -l
-rw-r--r--. 1 bin users 1864 May 4 18:01 initial-setup-ks.cfg
范例：将 initial-setup-ks.cfg 的拥有者与群组改回为root：
[root@study ~]# chown root:root initial-setup-ks.cfg
[root@study ~]# ls -l
-rw-r--r--. 1 root root 1864 May 4 18:01 initial-setup-ks.cfg
```



#### chmod 改变权限

​	权限的设置方法有两种， 分别可以使用数字或者是符号来进行权限的变更。

* 数字类型改变文件权限

  > r:4 > w:2 > x:1
  > 每种身份（owner/group/others）各自的三个权限（r/w/x）分数是需要累加的，例如当权
  > 限为： [-rwxrwx---] 分数则是：
  > owner = rwx = 4+2+1 = 7 > group = rwx = 4+2+1 = 7 > others= --- = 0+0+0 = 0
  > 所以等一下我们设置权限的变更时，该文件的权限数字就是770啦！变更权限的指令
  > chmod的语法是这样的：
  >
  > **[root@study ~]# chmod [-R] xyz 文件或目录**
  > **选项与参数：**
  > **xyz : 就是刚刚提到的数字类型的权限属性，为 rwx 属性数值的相加。**
  > **-R : 进行递回（recursive）的持续变更，亦即连同次目录下的所有文件都会变更**

```shell
[root@study ~]# ls -al .bashrc
-rw-r--r--. 1 root root 176 Dec 29 2013 .bashrc
[root@study ~]# chmod 777 .bashrc
[root@study ~]# ls -al .bashrc
-rwxrwxrwx. 1 root root 176 Dec 29 2013 .bashrc
```

* 符号类型改变文件权限

  ​	基本上就九个权限分别是
  ​	（1）user （2）group （3）others三种身份啦！那么我们就可以借由u, g, o来代表	

  ​	三种身份的权限！此外， a 则代表 all 亦即全部的身份！那么读写的权限就可以写		

  ​	成r, w, x

  格式：

  | chmod | u g o a | +（加入） -（除去） =（设置） | r w x | 文件或目录 |

  ```shell
  #案例：要“设置”一个文件的权限成为“-rwxr-xr-x”时
  [root@study ~]# chmod u=rwx,go=rx .bashrc
  # 注意喔！那个 u=rwx,go=rx 是连在一起的，中间并没有任何空白字符！
  [root@study ~]# ls -al .bashrc
  -rwxr-xr-x. 1 root root 176 Dec 29 2013 .bashrc
  ```


假如是“ -rwxr-xr-- ”这样的权限呢？可以使用“ chmod u=rwx,g=rx,o=r #filename ”来设置

如果我不知道原先的文件属性，而我只想要增加.bashrc这个文件的每个人均

可写入的权限， 那么我就可以使用：

  [root@study ~]# ls -al .bashrc
  -rwxr-xr-x. 1 root root 176 Dec 29 2013 .bashrc
  [root@study ~]# chmod a+w .bashrc
  [root@study ~]# ls -al .bashrc
  -rwxrwxrwx. 1 root root 176 Dec 29 2013 .bashrc

而如果是要将权限去掉而不更动其他已存在的权限呢？如去掉全部人的可执行权限

  [root@study ~]# chmod a-x .bashrc
  [root@study ~]# ls -al .bashrc
  -rw-rw-rw-. 1 root root 176 Dec 29 2013 .bashrc
  [root@study ~]# chmod 644 .bashrc # 测试完毕得要改回来喔！



#### 修改文件的时机

最常见的例子就是在复制文件给你之外的其他人时， 我们使用最简单的cp指令来说明好了：

复制给其他用户时还是会把属性和权限复制过去，这是就该修改了







### 3、目录与文件权限的意义

* 对于目录，很多动作中，你只要具有 x 即可！r 是非必备的！只是，没有 r 的话，使用 [tab] 时，他就无法自动帮你补齐文件名了！
* 目录中，r是能看到目录名，x是能进入到目录中，w是对目录名进行修改、转移什么的。







### 4、 Linux文件种类和扩展名

**任何设备在Linux下面都是文件， 不仅如此，连数据沟通的接口也有专属的文件在负责。**



##### 文件种类

* 正规文件（regular file）: 一般的进行存取的类型的文件，第一个字符为‘-’

  ​	按照文件的内容，又可分：

  * 纯文本文件（ASCII） : cat 指令查看
  * 二进制档（binary）
  * 数据格式文件（data）

* 目录（directory） [d]

* 链接文件（link）[l]

* 设备和设备文件（device）

  * 区块（block）文件 : RAM\ROM设备 [b]
  * 字符：如键盘鼠标等一次性读取，[c]

* 数据接口文件（sockets）: 监听用户，通过socket沟通 。 [s]

* 数据输送档（FIFO，pipe）：防止多程序同时读写文件导致错误。



##### Linux文件扩展名

​	Linux文件能不能执行由其十个字符权限决定（[x]），与后缀名没有任何关系

​	可以被执行跟可以执行成功是不一样的，因为他得具备可执行的程序码 。

​	x代表这个文件具有可执行的能力， 但是能不能执行成功，当然就得要看该文件的内容

​	啰（本身内容有没有可以执行的数据）

​	.sh : 脚本或批处理文件（shell写）

​	Z,.tar , .tar.gz , .zip , *.tgz :  压缩文件，压缩软件不同，结果不同





## Linux 目录配置

### 一、Linux目录配置的依据 -- FHS

#### 1、FHS

​	Filesystem Hierarchy Standard （FHS），文件分层标准 。 ，FHS的重点在于规范每个特定的目录下应该要放置什么样子的数据而已。

**鸟哥私房菜 5.3章**

#### 2、目录树

#### 3、绝对路径和相对路径

- 绝对路径：absolute，以 / 开始

- 相对路径：relative ,

-  .  ： 当前目录，也可 ./表示

- ..  ：上一层，也可 ../

  ​