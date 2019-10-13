---
title: Docker安装centos8并使用SSH远程连接
tags: 
  - linux 
  - centos8
category: Docker
date: 2019-10-13 11:17:38
---




> 相信身为程序员的你肯定为了学习，自己安装过虚拟机，然后在里面安装你喜欢的linux系统，进而安装各种软件，一系列的配置，繁琐的不要不要的。。

> 今天给大家带来Docker这款神器，不需要自己在安装虚拟机和下载iso系统安装文件，进行网络配置。 Docker可谓是真正解放了程序员的生产力，我们再也不需要为了每次安装环境花费大量时间，我工作这三年前前后后也折腾了不下数十次，每次还是得依靠搜索引擎，真的学不到啥东西还浪费时间。

接下来就进入今天的主题吧！



# 安装centos8系统



1. 搜索并下载镜像

```shell
docker search centos
docker pull centos
```



2. 启动容器,建立本机对应centos8镜像端口映射

```powershell
ssh默认的端口为22,我们将docker中centos的22端口映射到宿主机的5022端口
docker run -d -p 5022:22 --name centos8 --privileged=true -v h:/docker/data:/data centos /usr/sbin/init
```

- -d,后台启动
- -v,挂载共享磁盘，其中**h:/docker/data**表示宿主机的目录，冒号后面的**/data**表示虚拟机中的挂载点。这个挂载点会由docker容器自动创建。

如果对docker基础命令不熟悉的同学,可以参考这篇文章 [Docker入门](<http://coderluo.top/2019/08/17/docker/docker-ru-men/>);


3. 进入容器

```shell
docker exec -it centos8 /bin/bash
```



# 安装常用工具



1. 安装常用工具

```shell
yum install -y openssh-server vim lrzsz wget gcc-c++ pcre pcre-devel zlib zlib-devel ruby openssl openssl-devel patch bash-completion zlib.i686 libstdc++.i686 lsof unzip zip net-tools
```



2. service安装

```shell
yum install initscripts  
```



3. ifconfig安装

```shell
yum install net-tools.x86_64
```



4. ssh安装

```shell
sshd rpm -qa | grep ssh
yum install openssh-server 
service sshd restart
netstat -antp | grep sshd
```



# 开启docker-centos8 ssh远程连接




> aliyun/aws 云服务器,需要在安全组打开 5022端口对外访问权限




```shell
1.修改sshd_config 为密码登录
    vim /etc/ssh/sshd_config
    #打开注释 PermitRootLogin yes, 允许密码登录,保存退出

2.设置root用户密码
    passwd root

3.换个服务器远程登录
    ssh root@宿主机ip -p 5022
```





# 结束语



本文介绍了如果使用Docker安装Centos8系统,从而避免在使用安装vmware等虚拟机软件.接着我们配置了ssh服务,可以使用远程连接。 相信大家一定已经开始迫不及待的想尝试了，just do it！





















