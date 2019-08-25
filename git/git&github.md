# Git

### git 结构

​	工作区（写代码）

​	 ↓（git add ）

​	暂存区 （临时存储）

​	↓ （git commit）

​	本地库（历史版本）



### git 和 代码托管中心

​	代码托管中心任务：维护远程库

​	局域网：GitLab 服务器

​	外网：Github 、码云

### 设置本地仓库签名

​	1、项目级别、仓库级别：仅在当前本地库范围有效

​	作用：仅标识不同的开发人员，这里的签名和登陆认证什么的没有关系

​	操作：	

​		git config user.name lingz

​		git config user.email 5721xxxxx@qq.com

​	查看：（信息保存在.git/config中）

​		cat .git/config  //**这里也会显示一些 remote的信息**

​	

​	2、 系统级别：登陆当前操作系统用户的范围

​		git --global user.name lingz

​		git --global xxxx@qq.com

​		

​	级别优先级：

​		就近原则：项目优先于系统

​		如果只有系统级签名，用系统的

​		不允许一个都没有

## 基本操作

### 状态查看

​	git status 	查看工作区、暂存区状态

### 添加

​	git add [file name] 将工作区的有修改或者新建的文件添加到暂存区

### 提交

​	git commit -m '提交信息' [file name] 将暂存区的内容提交到本地库

### 查看历史记录

​	git log 

​		空格向下翻页 \ b 向上翻页 \ q 退出

​	**git log --pretty=oneline**		//一行显示，更好看

​	git log --oneline 

​		//上面是将记录全部打出来，但是只显示部分的hash码，够标识

​	git reflog 

```
可以查看所有分支的所有操作记录（包括已经被删除的 commit 记录和 reset 的操作）

	例如执行 git reset --hard HEAD~1，退回到上一个版本，用git log则是看不出来被删除的commitid，用git reflog则可以看到被删除的commitid，我们就可以买后悔药，恢复到被删除的那个版本。

	HEAD@{移动到当前版本需要多少步}
```



### 前进后退

```
git reflog ：
d958587 HEAD@{1}: commit: 修改
850552c HEAD@{2}: commit: 更新八大排序
bf164aa HEAD@{3}: commit: 新增算法/八大排序
cb684a6 HEAD@{4}: commit: 新增redis/redis使用布隆过滤器
4dc9fb2 HEAD@{5}: commit: D:/0softInitial/Git/Git/mysql_explain.md.
927c649 HEAD@{6}: commit: 新增java基础/Class类和Class对象.md
09755e5 HEAD@{7}: commit: 修改
02760fa HEAD@{8}: commit: 新增J2EE/Restful和状态码.md
1245699 HEAD@{9}: commit: 新增J2EE/Cookie和Session.md
7d6096c HEAD@{10}: commit: 更新
83937ec HEAD@{11}: commit: 更新 多线程/Java高并发程序设计.md
a2658df HEAD@{12}: commit: 新增 多线程/Java高并发程序设计.md
70e9a00 HEAD@{13}: commit: 添加红黑树的信息
7377bd8 HEAD@{14}: commit: 新增redis/redis持久化.md
0a21f99 HEAD@{15}: commit: 新增 redis 和 memcache使用场景和区别
16872d1 HEAD@{16}: commit: 更新
053b0a2 HEAD@{17}: commit: 新增 Java集合类/JDK1.8集合类整合.md.
40aabf8 HEAD@{18}: commit: 新增 分布式理论/CAP和BASE理论.md.
d6a33f2 HEAD@{19}: commit: redis和memcache区别.md
eeddd41 HEAD@{20}: commit: 新增/分布式/一致性hash算法.md
c2509ce HEAD@{21}: commit: 新增Java代码如何运行.md
965a624 HEAD@{22}: commit: 新增mysql索引

```

#### 基于索引值操作[推荐]

​	git reset --hard[局部索引值]

​	eg: git reset --hard 40aabf8     // 回退到 分布式理论/CAP和BASE理论.md.

### 使用^符号：只能后退

​	git reset --hard HEAD^[n]	

​	注：一个^回退一步 ， n个^回退n步

#### 使用~符号：只能后退

​	git reset --hard HEAD~n

​	注：表示后退n步	

#### reset 命令三个参数

```
-- soft  ： 仅本地库移动HEAD指针
-- mixed ： 在本地库移动HEAD指针，重置暂存区
--hard ：本地库移动HEAD，重置暂存区，重置工作区
```

#### 

#### 比较文件差异

​	git diff [文件名]  ： 将工作区的文件和暂存区进行比较

​	git diff[文件名] ： 将工作区中的文件和暂存区进行比较

​	git diff [本地库中历史版本] 【文件名】 : 将工作区中的文件和本地库历史记录比较

​	

## 分支

​	多线同时推进多个任务 , 一个分支的进行不会影响其他分支。

```
master      ○ → ○ → ○ → ○ → ○ → ○  
 	         ↘			 ↗
me     	       ○ → ○ → ○	
```

###分支操作

####创建分支

​	git branch [分支名]

#### 查看分支

​	git branch -v

#### 切换分支

​	git checkout [分支名]

#### 合并分支

```git
步骤：
	1、切换到接收修改的分支（被合并，增加新内容）上
		git checkout [被合并的分支名]
	2、执行merge命令
		git merge[有新内容分支名]
	3、冲突解决：
		① 、编辑冲突文件，删除特殊符号
		② 、修改文件后，保存退出
		③ 、 git add [冲突文件名]
		④ 、 git commit -m '日志信息'	//此时不带任何文件名！
		
```



# Github

### 创建远程库地址别名

​	git remote -v 查看当前所有远程地址别名

​	git remote add [别名] [远程地址名] :

​		eg: git remote add origin1 https://gitbub.com/LiuLingZ/xxxx.git

### 推送

​	git push [别名] [分支名]

​		eg : git push origin master

### 克隆

```
命令 ：git  clone [仓库远程地址]

效果：（3个）
	完整的把远程库下载到本地
	初始化了本地（自动 git init → .git）
	创建origin远程地址别名 git remote -v
```



### 团队成员邀请

​	在仓库的settings 点击 Collaboration ，输入被邀请者的 账户，点击Add  collaborator。获得邀请链接 。 通过线下将链接发给被邀请的人。

​	上被邀请的账号，登陆后，复制链接同意即可。



### 拉取

​	pull = fetch + merge

​	git fetch [远程库地址名] [远程分支名]

​	git merge [远程库地址名] [远程分支名]

​	git pull [远程库地址名] [远程分支名]

### 解决冲突

```
要点
 如果不是基于GitHub 远程库的最新版所做的修改，不能推送，必须先拉取。
 拉取下来后如果进入冲突状态，则按照“分支冲突解决”操作解决即可。
```



## 跨团队协作

