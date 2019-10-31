1、全局配置

```git
git config --global user.name 'leikes'
git config --global user.email xxx@qq.com
```



2、二进制文档的修改在git是跟踪不了的，word也是二进制文档，所以也追踪不了。



3、提交文档到仓库

```
git add a.txt     //把文件添加到仓库中
git commit -m '提交了a.txt'   //将要添加到仓库的文件提交到仓库中
```

4、比较不同

```sql
git diff 文件名

-- 想要比较不同，这个文件必须已经 git commit 过，才有记录可与之匹配。
--eg: 
git add a.txt ;
git commit a.txt ;
-- 对a.txt进行修改，然后
git diff a.txt 
--就会显示出差异。
```



5、历史记录 、 操作记录

``` java
git log  //普通查看
    
git log --pretty=oneline   //格式比较ok的查看

//HEAD指的就是当前最新的，最近的一次提交
//HEAD的上一个就是 HEAD^ ，上上个就是 HEAD^^
//上2个 ： HEAD~2

git reflog //记录了我们的每一次操作，这个挺重要的


```



6、git reset 位置  （会删除日志）

```java
/*
	该操作是将记录回退到指定的地方
	并且删除原先的位置！！！，也就是git log就看不到了。
	而要恢复到刚刚最新的，可以通过git reflog 查看我们的所有操作，找到对应的序号，再git reset 序列号即可
*/

git reset HEAD^

//此时log已经没有最新记录了，

git reflog  //查看所有操作，找到刚刚最新的序列号

git reset 序列号  //完成谎还原


```



7、关于撤销

```java
//还未git add到暂存区，只是在工作区修改，如果想撤销，可以手动删除刚刚添加的，也可：
// -- 一定不能漏，这样a.txt会还原成刚打开工作区的样子。
git checkout -- a.txt    


//如果已经git add .到暂存区了,可以撤销掉暂存区的，回到工作区，想撤销工作区的，git checkout --
git reset HEAD a.txt

//如果git commit到仓库里，想回退，就用 git reset 旧版本即可
git reset HEAD^

//如果推送到远程库，就一定会被别人看见，虽然也能回退。

```



## 分支

​	git 用 master 指向最新的提交，HEAD指向 当前分支 。HEAD刚开始指向master，就能确定当前分支，以及分支的提交点

![git-br-initial](https://www.liaoxuefeng.com/files/attachments/919022325462368/0)



创建dev分支，指向master的提交，再把HEAD指向dev，就表示当前的分支 在dev上。

![git-br-create](https://www.liaoxuefeng.com/files/attachments/919022363210080/0)



现在提交都是在dev上

![git-br-dev-fd](https://www.liaoxuefeng.com/files/attachments/919022387118368/0)



1、**本地**分支的创建、开发、合并、删除

```java
//创建分支
git branch dev
//切换分支
git checkout dev

//上面两步可以合为一步
git checkout -b dev

/*
	现在在dev上开发，比如添加a.txt文件，
 	git add .
    git commit -m '添加了a.txt'
*/

//现在切回到master合并分支
git checkout master

//在master合并dev
git merge dev

//此时如果发生冲突，可以找到冲突文件，修改之后
//直接在本分支上面 
git add .
git commit -m '解决冲突'
//即可将本次合并、冲突解决一并提交

//删除掉dev分支
git branch -d dev



//查看图表式的分支合并
git log --graph --pretty=oneline --abbrev-commit 


```



3、关于Fast forward模式的merge

​	普通的merge ，不会保留多少被merge的信息，所以在merge时，添加 --no-ff 参数，比较好，能看到历史合并信息

```java
git merge --no-ff -m 'merge with no-ff'
```

​	发生冲突，可以通过合并解决。





4、分支管理

​	**一般的，master都是稳定版，发布用，不要开发。**

​	**开发都是在dev，每个人从master拉下，自己开发后整合到dev分支上，最终dev ok后再整合会master上。**

​	 ![git-br-policy](https://www.liaoxuefeng.com/files/attachments/919023260793600/0)





5、暂存一下工作台，来做其他事情：

```java
//将工作区的内容缓存。这样工作区就干净了，可以专门开发别的 。 可以缓存多个工作台。
git stash  

//开发完后，想要重新开发刚才的内容。
// list 可以查看缓存区的内容。回去有两种操作
git stash list 

//1、第一种，根据list提供的编码，回退到对应的（可以缓存多个）
git stash apply 编码   //这种方式不会删除缓存。需要手动删除
git stash drop 编码 //删除对应的缓存

//2、第二种，直接弹出最新的
git stash pop


```





6、删除一个没有被合并的分支：(强行删除分支)

```java
git branch -D 分支名
```





7、提交 到远程库

```java
//提交到远程dev分支
git push origin dev

//创建远程分支并且提交到这个分支上
git push --set-upstream origin dev

```



8、修复bug后，在其他的分支也想要这个修复的操作，不需要merge

```java
//这个操作可以将这个分支最新的commit拷贝一份过来并整合，不用merge全部，比较方便
git cherry-pick 提交的编号 //编号可以通过git log --pretty=oneline查看
 
//拷贝过来的commit会在本分支再commit一次，得到新的commit。 
```



9、简化日志，自动改变merge和commit的记录，使日志变成一条直线

```java
//在提交的之后
git rebase ;
//再次查看日志，发现好看多了

git log --graph --pretty=oneline --abbrev-commit
//再推到远程库，远程也变得简洁。
git push 
```



10、标签管理

**注意：标签总是和某个commit挂钩。如果这个commit既出现在master分支，又出现在dev分支，那么在这两个分支上都可以看到这个标签。**

```java
//标签 ：在某个commit打上标签，相比编号，更容易找到。本质也是指针，只是不可以移动。

//1、到某个分支，比如master
git checkout master
git tag v1.0 ;

//查看所有标签
git tag ;

//在某个commit上打标签，首先需要找到这个commit的编号
git log --pretty=oneline --abbrev-commit
git tag v0.9 编号


//标签可以添加额外信息， -a指定名字，-m指定额外信息
git tag -a v1.1 -m 'version 1.1 release' 编码

//查看说明信息
git show tagName


//删除本地的标签
git tag -d 标签名

//推送标签到远程
git push origin 标签名  //一个一个推送。
git push origin --tags  //一次性将未推送的tag全部推送。



//删除远程tag较麻烦。也可以直接在gitlab、 github直接删除.
//先在本地删除tag，再推送,格式如下：
git push origin :ref/tags/标签名




```



# github

1、关联远程仓库

```java
//查看远程连接 , -v是查看更详细的
git remote -v

//默认的 origin 就是远程库
git remote add origin git@github.com:LiuLingZ/learngit.git

//将本地库的内容推送到远程库，实际就是将master分支推送到远程。
git push -u origin master

//由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。

```

2、刚开始，克隆一个项目

```sql
--通过SSH
git clone git@github.com:LiuLingZ/review-and-harvest.git

--通过HTTPS
git clone https://github.com/LiuLingZ/review-and-harvest.git
```





## 3、多人协作开发

```java
//1、首先需要和项目创建连接
git remote add origin url

//2、克隆这个远程项目下来，这样clone到自己的master上
git clone 项目地址

//3、一般在dev开发， 就需要在远程项目创建 dev 分支，并且，将这个远程dev创建到本地
远程项目先创 ling.dev分支
git checkout -b dev origin/dev
//开始开发，此时在dev分支上
git add .
git commit -m '添加到远程dev上的各种操作'
    
/*4、此时想要push，如果此时远程dev一直只有自己的，就没问题，如果有别人提交的，就会push失败
    push失败是因为远程版本高于我们，需要先拉远程的来合并自己的。
    此时不能直接 pull，还需要和自己本地的dev 与 远程的 origin/dev 关联。
    所以步骤：
    i、关联本地与远程分支
    ii、拉取合并
*/

git branch --set-upstream-to=origin/dev dev     //关联远程与本地分支
git pull

//git pull成功后可能会有冲突，手动解决冲突 ，通过merge等，看上面,再次推送
git commit -m '已经在pull之后成功解决冲突，并且此时有自己的最新数据。' ；
git push origin dev ;


/**
	步骤：
		1、clone项目
		2、在远程项目创个分支
		3、将这个分支拉到本地的某个分支，开发
		4、提交
		5、git push冲突，，先关联本地的分支和远程的分支，拉取，手动合并，再git commit -m ''
		6、再次提交，ok。
*/

```



4、查看远端分支情况

​	git branch -r



5、.gitignore

```sql
如果要不提交一些文件，可以在.gitignore里面配置

如果在里面配置了，会让文件不提交被忽略，而如果有一小部分我们想要提交的并在名单之内的，可以通过

git add -f a.txt  --能够强制添加到git中，即使 .txt 是被添加到 .gitignore之中的。
```







































































