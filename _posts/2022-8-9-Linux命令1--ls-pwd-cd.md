---
layout: post
title: "Linux命令1--ls pwd cd"
date:   2022-8-9
tags: [Linux]
comments: true
author: Bo
---

## pwd

显示当前处于哪个路径,我们开启一个终端演示：

```bash
bo@bo-virtual-machine:~$ pwd
/home/bo
```



## ls

列出当前路径下所有的文件：

```bash
bo@bo-virtual-machine:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  snap  Templates  Videos
```

ls可以加上路径参数来罗列某个文件夹下的文件:

```bash
bo@bo-virtual-machine:~$ ls /home/Desktop/a
```



### ls的三个option

```bash
bo@bo-virtual-machine:~$ ls -l
total 36
drwxr-xr-x 3 bo bo 4096  8月  9 09:46 Desktop
drwxr-xr-x 2 bo bo 4096  8月  8 19:20 Documents
drwxr-xr-x 3 bo bo 4096  8月  8 19:21 Downloads
drwxr-xr-x 2 bo bo 4096  8月  8 19:20 Music
drwxr-xr-x 2 bo bo 4096  8月  8 19:20 Pictures
drwxr-xr-x 2 bo bo 4096  8月  8 19:20 Public
drwx------ 4 bo bo 4096  8月  8 19:21 snap
drwxr-xr-x 2 bo bo 4096  8月  8 19:20 Templates
drwxr-xr-x 2 bo bo 4096  8月  8 19:20 Videos
```

表示以列表的形式罗列



```bash
~$ ls -a
```

表示显示隐藏的文件夹。



```bash
~$ ls -h
```

和 -l 一起使用，可以直接写成：

```bash
~$ ls -lh
```

### ls通配符

```bash
~$ ls *.txt
```

用*这样写可以只罗列出以.txt结尾的文件

注：*可以代表**1个或多个字符**，**可以加在任何位置**，所以可以这样来筛选：

```bash
~$ ls 1*
~$ ls 1*.txt
```

第一行可以罗列出以1开头的文件，第二行可以罗列以1开头，结尾为.txt的文件。



```bash
~$ ls ???.txt
```

**?代表单个字符**，比如这一行就是罗列三个字符的以.txt结尾的文件



```bash
~$ ls [1234]23.txt
```

表示罗列第一个字符是1234中的**一个**，后面是23.txt的文件。**可以连续使用**

## cd

打开路径下的某个文件夹:

```bash
bo@bo-virtual-machine:~$ cd desktop
bash: cd: desktop: No such file or directory
bo@bo-virtual-machine:~$ cd Desktop
bo@bo-virtual-machine:~/Desktop$ 
```

如事例所示，注意：**这个命令区分大小写**！

返回当前目录的上一级：

```bash
bo@bo-virtual-machine:~$ cd ..
```

注意cd后面是可以写多级目录的：

```bash
bo@bo-virtual-machine:~$ cd Desktop/a
bo@bo-virtual-machine:~/Desktop/a$ 
```

Tab键自动补全功能：

比如我们只输入以下字符命令，再按下tab键，后面的文件夹名称就会自动补全

```bash
~$ cd Des
```

按tab：

```bash
~$ cd Desktop/
```

但是要注意：文件夹前面的字母要给够，不然会混淆，比如现在路径下有两个以Des开头的文件夹“Desktop”和‘Desk'，tab就没用了

不过你可以**按两次tab键**来让终端提示你有哪些以该字符开头的文件夹：

```bash
~$ cd D
Desktop/   Documents/ Downloads/ 
```

~表示家（home）目录，直接cd到home目录：

```bash
~$ cd ~
bo@bo-virtual-machine:~$ 
```



在某路径下打开是cd相对路径，可以cd打开绝对路径(以/开头)：

```bash
~$ cd /home/Desktop/a
&
~$ cd ~/Desktop/a
```

