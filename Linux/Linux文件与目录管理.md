# Linux 文件与目录管理

## 一、目录和路径

 ###1、目录的相关操作

#### 特殊的目录：

```
. 代表此层目录
.. 代表上一层目录
- 代表前一个工作目录
~ 代表“目前使用者身份”所在的主文件夹
~account 代表 account 这个使用者的主文件夹（account是个帐号名称）
```

#### 常见的处理目录指令：

```
cd：变换目录
pwd：显示目前的目录
mkdir：创建一个新的目录
	  -p ：递归创建，目的是一次性创建多个，没有会显示没有这个目录的
	  	   错误。
		eg:mkdir -p test1/test2/test3

rmdir：删除一个空的目录，一定要空，所以要先删除该目录下的其他文件
	  -p ：递归删除,连同上一层也删除。
```

### 2、可执行文件路径的变量：$PATH

```shell
1、echo $PATH ： 查看系统定义的系统路径变量
2、echo有“显示、印出”的意思，而 PATH 前面加的 $ 表示后面接的是变
	量，所以会显示出目前的 PATH ！
3、PATH（一定是大写）这个变量的内容是由一堆目录所组成的，每个目录中
	间用冒号（:）来隔开， 每个目录是有“顺序”之分的。
4、想自己添加目录到可执行/root下面，可：
[root@study ~]# PATH="${PATH}:/root"  

```



## 二、文件与目录管理

​	文件与目录的管理上，不外乎“显示属性”、 “拷贝”、“删除文件”及“移动文件或目录”等等。

### 1、 文件与目录的检视：ls

```
[root@study ~]# ls [-aAdfFhilnrRSt] 文件名或目录名称..
[root@study ~]# ls [--color={never,auto,always}] 文件名或目录名称..
[root@study ~]# ls [--full-time] 文件名或目录名称..
选项与参数：
-a ：全部的文件，连同隐藏文件（ 开头为 . 的文件） 一起列出来（常用）
-A ：全部的文件，连同隐藏文件，但不包括 . 与 .. 这两个目录
-d ：仅列出目录本身，而不是列出目录内的文件数据（常用）
-f ：直接列出结果，而不进行排序 （ls 默认会以文件名排序！）
-F ：根据文件、目录等信息，给予附加数据结构，例如：
*:代表可可执行文件； /:代表目录； =:代表 socket 文件； &#124;:代表 FIFO 文件；
-h ：将文件大小以人类较易读的方式（例如 GB, KB 等等）列出来；
-i ：列出 inode 号码，inode 的意义下一章将会介绍；
-l ：长数据串行出，包含文件的属性与权限等等数据；（常用）
-n ：列出 UID 与 GID 而非使用者与群组的名称 （UID与GID会在帐号管理提到！）
-r ：将排序结果反向输出，例如：原本文件名由小到大，反向则为由大到小；
-R ：连同子目录内容一起列出来，等于该目录下的所有文件都会显示出来；
-S ：以文件大小大小排序，而不是用文件名排序；
-t ：依时间排序，而不是用文件名。
--color=never ：不要依据文件特性给予颜色显示；
--color=always ：显示颜色
--color=auto ：让系统自行依据设置来判断是否给予颜色
--full-time ：以完整时间模式 （包含年、月、日、时、分） 输出
--time={atime,ctime} ：输出 access 时间或改变权限属性时间 
					（ctime）而非内容变更时间 （modification 
					time）
```



### 2、复制、删除、移动(可用来更名)：cp , rm  , mv

#### cp（复制文件或目录），要r权限。

```shell
[root@study ~]# cp [-adfilprsu] 来源文件（source） 目标文件（destination）
[root@study ~]# cp [options] source1 source2 source3 .... directory
选项与参数：
-a ：相当于 -dr --preserve=all 的意思，至于 dr 请参考下列说明；（常用），可以把文件的特性一起复制过来
-d ：若来源文件为链接文件的属性（link file），则复制链接文件属性而非文件本身；
-f ：为强制（force）的意思，若目标文件已经存在且无法打开，则移除后再尝试一次；
-i ：若目标文件（destination）已经存在时，在覆盖时会先询问动作的进行（常用）
-l ：进行硬式链接（hard link）的链接文件创建，而非复制文件本身；
-p ：连同文件的属性（权限、用户、时间）一起复制过去，而非使用默认属性（备份常用）；
-r ：递回持续复制，用于目录的复制行为；（常用）
-s ：复制成为符号链接文件 （symbolic link），亦即“捷径”文件；
-u ：destination 比 source 旧才更新 destination，或 destination 不存在的情况下才复制。
--preserve=all ：除了 -p 的权限相关参数外，还加入 SELinux 的属性, links, xattr 等也复制了。
最后需要注意的，如果来源文件有两个以上，则最后一个目的文件一定要是“目录”才行！
```



#### rm(删除文件或目录)

```shell
[root@study ~]# rm [-fir] 文件或目录
选项与参数：
-f ：就是 force 的意思，忽略不存在的文件，不会出现警告讯息；
-i ：互动模式，在删除前会询问使用者是否动作
-r ：递回删除啊！最常用在目录的删除了！这是非常危险的选项！！！
```





#### mv(移动文件与目录，或更名)

```shell
[root@study ~]# mv [-fiu] source destination
[root@study ~]# mv [options] source1 source2 source3 .... directory
选项与参数：
-f ：force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖；
-i ：若目标文件 （destination） 已经存在时，就会询问是否覆盖！
-u ：若目标文件已经存在，且 source 比较新，才会更新 （update）
范例一：复制一文件，创建一目录，将文件移动到目录中
[root@study ~]# cd /tmp
[root@study tmp]# cp ~/.bashrc bashrc
[root@study tmp]# mkdir mvtest
[root@study tmp]# mv bashrc mvtest
# 将某个文件移动到某个目录去，就是这样做！
范例二：将刚刚的目录名称更名为 mvtest2
[root@study tmp]# mv mvtest mvtest2 &lt;== 这样就更名了！简单～
# 其实在 Linux 下面还有个有趣的指令，名称为 rename ，
# 该指令专职进行多个文件名的同时更名，并非针对单一文件名变更，与mv不同。请man rename。
范例三：再创建两个文件，再全部移动到 /tmp/mvtest2 当中
[root@study tmp]# cp ~/.bashrc bashrc1
[root@study tmp]# cp ~/.bashrc bashrc2
[root@study tmp]# mv bashrc1 bashrc2 mvtest2
# 注意到这边，如果有多个来源文件或目录，则最后一个目标文件一定是“目录！”
# 意思是说，将所有的数据移动到该目录的意思！
```



### 3、取得路径的文件名称和目录名称

```shell
[root@study ~]# basename /etc/sysconfig/network
network &lt;== 很简单！就取得最后的文件名～
[root@study ~]# dirname /etc/sysconfig/network
/etc/sysconfig &lt;== 取得的变成目录名了！
```



## 三、文件内容查阅

​	最常使用的显示文件内容的指令可以说是 cat 与 more 及 less 了！此外，如果我们要查看一个很大型的文件 （好几百MB时），但是我们只需要后端的几行字而已，那么该如何是好？呵呵！用 tail 呀，此外， tac 这个指令也可以达到这个目的喔！

```
cat 由第一行开始显示文件内容
tac 从最后一行开始显示，可以看出 tac 是 cat 的倒着写！
nl 显示的时候，顺道输出行号！
more 一页一页的显示文件内容
less 与 more 类似，但是比 more 更好的是，他可以往前翻页！
head 只看头几行
tail 只看尾巴几行
od 以二进制的方式读取文件内容！
```

### more(可翻页)

```
空白键 （space）：代表向下翻一页；
Enter ：代表向下翻“一行”；
/字串 ：代表在这个显示的内容当中，向下搜寻“字串”这个关键字；
:f ：立刻显示出文件名以及目前显示的行数；
q ：代表立刻离开 more ，不再显示该文件内容。
b 或 [ctrl]-b ：代表往回翻页，不过这动作只对文件有用，对管线无用。
```

### less(可翻页)

```
空白键 ：向下翻动一页；
[pagedown]：向下翻动一页；
[pageup] ：向上翻动一页；
/字串 ：向下搜寻“字串”的功能；
?字串 ：向上搜寻“字串”的功能；
n ：重复前一个搜寻 （与 / 或 ? 有关！）
N ：反向的重复前一个搜寻 （与 / 或 ? 有关！）
g ：前进到这个数据的第一行去；
G ：前进到这个数据的最后一行去 （注意大小写）；
q ：离开 less 这个程序；
```

### 修改文件时间或创建新文件：touch



## 四、文件与目录的默认权限和隐藏权限

### 1、文件默认权限 umask

#### 2、文件隐藏属性

#### 3、文件特殊权限 ：SUID, SGID, SBIT

## 五、指令与文件的搜索

