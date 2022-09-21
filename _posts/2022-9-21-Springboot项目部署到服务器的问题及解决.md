---
layout: post
title: "Springboot部署项目过程中的问题"
date:   2022-9-21
tags: [Server]
comments: true
author: Bo
---

## 服务器解压缩安装jdk

第一步，在官网上下载安装包，下载完后上传到服务器：

![image-20220811110042488](https://raw.sevencdn.com/Bo-Vane/picgo/main/img/202208111101656.png)

上传成功后，在命令行解压缩：

```bash
tar -xvzf jdk-8u341-linux-x64.tar.gz
```

解压缩完成后，会生成一个jdk目录，记住路径好配置环境变量。

```bash
vim /etc/profile
```

在文件末尾加上：

```bash
export JAVA_HOME=/home/bo/jdk/jdk1.8.0_341
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

配好环境后,遇到了以下问题：

```bash
java -version
-bash: /usr/bin/java: No such file or directory
```

这是因为如果是直接从官网下载的jdk解压安装，**那么/usr/bin/没有java的软连接**，即/usr/bin/java，所以即使在/etc/profile中配置了jdk的环境变量也没用，识别不到。

用java的执行路径配置/usr/bin/java软连接：

```bash
ln -s /home/bo/jdk/jdk1.8.0_341/bin/java /usr/bin/java
```

成功解决问题：

```bash
root@bo:/home/bo/jdk/jdk1.8.0_341/bin# java -version
java version "1.8.0_341"
Java(TM) SE Runtime Environment (build 1.8.0_341-b10)
Java HotSpot(TM) 64-Bit Server VM (build 25.341-b10, mixed mode)
```



## 部署项目

安装宝塔面板：

```bash 
wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && sudo bash install.sh ed8484bec
```

安装好后用网址登录面板

下载配置Tomcat，MySQL环境

然后开始部署：

> 将项目打包

在idea终端输入：

`mvn clean package`

打包完成后，将jar包和application配置文件传入到服务器文件夹。不需要更改任何代码。

> 添加数据库

在数据库面板添加数据库，注意**数据库名和用户名，密码需要和开发时写的一致**，如果是root，用户名可以不管，随便写，注意开放访问权限

将原来的数据库导出为sql文件，再导入新建的数据库即可！

> 运行jar文件

一开始运行时，我发现运行成功后，可以访问，但是关掉连接服务就终止了。使用以下命令使程序仍然运行：

```bash 
nohup java -jar xxx.jar
```





## 突然连接不上云服务器的问题解决

今天部署项目的时候，部署着突然就连不上云服务器（Ubuntu）里，安全组里的端口号都是打开的，查找了非常非常多的资料成功解决

由于我的xshell和阿里云都连不上服务器：

```bash
Could not connect to 'xxxxx' (port 22): Connection failed.
```

所以我只能用vnc来连接实例：

https://help.aliyun.com/document_detail/25433.html

操作之后，我想查看防火墙开放的端口号：

```bash
sudo ufw status
```

果然显示：

```
Status:inactive
```

然后再开启：

```bash 
sudo ufw enable
```

成功地显示了开放的端口号！

终于我也能成功地登录服务器了



## 开放端口的问题

然后呢，我在部署的时候发现，在阿里云控制台安全组设置里开放端口，并不能访问到，还需要在防火墙开放端口：

**1.启动firewalld服务**

```
systemctl start firewalld.service
```

**2.关闭firewalld服务**

```
systemctl stop firewalld.service
```

**3.重启firewalld服务**

```
systemctl restart firewalld.service
```

**4.查看firewalld状态**

```
systemctl status firewalld.service
```

**5.开机自启firewalld**

```
systemctl enable firewalld
```

**6.查看版本**

```
firewall-cmd --version
```

**7.查看帮助**

```
firewall-cmd --help
```

**8.显示状态**

```
firewall-cmd --state
```

**9.查看所有打开的端口**

```
firewall-cmd --zone=public --list-ports
```

**10.更新防火墙规则**

```
firewall-cmd --reload
```

**11.添加开放端口**

```
firewall-cmd --zone=public --add-port=80/tcp --permanent (permanent永久生效，没有此参数重启后失效)
```

**12.查看端口是否开放**

```
firewall-cmd --zone=public --query-port=80/tcp
```

**13.删除开放端口**

```
firewall-cmd --zone=public --remove-port=80/tcp --permanent
```

**注：每次更改firewall规则后需重新加载（firewall-cmd --reload）**

折腾了老半天。

``` bash
firewall-cmd: error: unrecognized arguments: --list-ports1
root@bo:~# firewall-cmd --zone=public --list-ports
8888/tcp
```

我们看到现在只开放了8888，所以要继续：

```bash 
root@bo:~# firewall-cmd --zone=public --add-port=80/tcp --permanent
success
root@bo:~# firewall-cmd --zone=public --add-port=81/tcp --permanent
success
root@bo:~# firewall-cmd --zone=public --add-port=8080/tcp --permanent
success
root@bo:~# firewall-cmd --zone=public --add-port=3306/tcp --permanent
success
root@bo:~# firewall-cmd --zone=public --add-port=443/tcp --permanent
success
root@bo:~# firewall-cmd --reload
success
```

每一次reload，都会导致我连不上22端口，然后又要重复一遍一级标题的步骤。