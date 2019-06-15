# Shell 

## 一、Hello World

1、在 ./studyTest下创建文件 test.sh ，后缀名步骤不重要。命令： touch test.sh

2、通过vi编写test.sh

```shell
#!/bin/bash
echo 'Hello World'    
```

这里，echo是标准输出，输出到窗口。类似print

"#!" 是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行，即使用哪一种Shell。Shell有多个不同版本。

3、运行Shell

- 先给文件赋予可执行权限 chmod -x test.sh
- 执行文件 ./test.sh  。注意！！这里一定要加  ./ ,而不是test.sh，运行其它二进制的程序也一样，直接写test.sh，linux系统会去PATH里寻找有没有叫test.sh的，而只有/bin, /sbin, /usr/bin，/usr/sbin等在PATH里，你的当前目录通常不在PATH里，所以写成test.sh是会找不到命令的，要用./test.sh告诉系统说，就在当前目录找。



## 二、变量

### 1、shell变量

- 首个字符必须为字母（a-z，A-Z）。
- 中间不能有空格，可以使用下划线（_）。
- 不能使用标点符号。
- 不能使用bash里的关键字（可用help命令查看保留关键字）。

eg :  your_name='leikes'

### 2、定义，使用变量

- 通过 $ 符引用变量

- ```shelll
  !#/bin/bash
  your_name='liu'
  echo $your_name
  echo ${your_name}
  
  for skill in Ada Coffe Action Java do
  	echo "I am good at ${skill}Script"
  done
  ```

  {} 可加可不加，只是帮助解释器识别边界用，为了方便维护，还是加上比较好 。

- 重新赋值变量不需要使用$,除非是使用才需要。

### 3、字符串

#### 单引号

- 单引号内是什么就是什么，**无法使用变量**
- 单引号内**不能**再出现双引号或者**转义**的单引号。

#### 双引号

- 双引号里**可以有变量**
- 双引号里**可以出现转义字符**

​	