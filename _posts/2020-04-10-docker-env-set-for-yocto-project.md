---
layout: post
title: Linux底层开发之——使用docker作为Yocto Project的环境
date: 2020-04-10 22:00:00 +800
description: Linux底层开发之使用docker作为Yocto Project的环境                                                                                  # Add post description (optional)
img:  # Add image post (optional)
fig-caption: # Add figcaption (optional)
categories: [软、硬件编程, Linux]
tags: [docker, Yocto] # add tag
seo:
  date_modified: 2020-04-11 01:58:49 +0800
---


>前言：想必在座用Yocto的各位都经历过一段“折磨”的使用阶段，~~它的高门槛让许多入门者望而却步，但在整个Linux的所有东西（UBOOT、Kernel、文件系统、包含GCC编译工具链的针对该Poject的SDK、众多软件包）编译成功之后，那时候感觉真的是太香了！！！这就是整个发行版都出来了啊！这么好用、强大的东西我怎么舍得放弃！回顾初学时候，其实主要就是卡在编译失败，顺利的起码编译7、8个小时，不顺利的比如我第一次编译了好多个星期。。。最终它被我孜孜不倦的精神和长久的坚持给拿了下来！~~ 进来的估计都不用我说了，废话不说了。。。  
　　如果你不了解docker，那么我现在告诉你，使用docker当容器跑linux，配置好一个环境，到处通用，像个隔离开的沙盒一样，详情请自行百度或出门左转’菜鸟教程‘。因为我们只需要一个Yocto的运行环境，虚拟机那玩意太大了，本机环境很难保证每次都一样。作为芯片原厂/开发板商，一个docker环境解决无数客户使用Yocto最大的痛点：编译环境。作为开发者也类似，就不多说了。  

———首先我说明一下，docker里主要的两个东西：image和container：  
　　image指文件系统镜像，container是这个运行镜像的载体。就像你发布一个ubuntu的image，全部人都可以装到自己电脑上，你也可以把你装了一堆软件的文件系统打包出来成image，即预装xx软件的镜像文件。如果对其中原理感兴趣我们可以私聊～  

### 1、安装配置
笔者机器环境：Manjaro Linux（Arch系）  
Arch系，直接  
`sudo pacman -S docker`  
其它Linux自行百度。教程满天飞，我就不写了。  

安装完，添加国内镜像（加速），路径和内容如下：  
```
[whqee@whqee docker]$ cat /etc/docker/daemon.json 
{
  "registry-mirrors": [
          "http://hub-mirror.c.163.com",
          "https://registry.docker-cn.com",
          "http://f1361db2.m.daocloud.io",
          "https://docker.mirrors.ustc.edu.cn",
          "https://mirror.ccs.tencentyun.com"
  ]
}
```
```
systemctl enable docker.service
systemctl start docker.service
sudo gpasswd -a 用户名 docker  //把自己添加到docker用户组，然后不需要‘sudo’来docker了
```
完事。  

### 2、先过一遍怎么自己配置个Yocto的环境


  
官方文档ubuntu，那就用ubuntu，省事，记得对应Yocto版本选不同版本ubuntu。  
拉取镜像：
```
$ docker pull ubuntu:18.04
```
以它作为镜像，为它开启一个独立的容器，加载运行。（-i是”交互式操作“，-t是”终端“，-d是”后台运行“，”ubuntu:18.04“是我们刚拉取的镜像，后面接着的是要运行的命令，没有指定container的名字，所以名字随机）：
```
$ docker run -itd ubuntu:18.04 /bin/bash 
```
```
$ docker ps -a  //列出所有容器
$ docker ps     //列出正在运行的容器。（不在运行态的，你可以类比它为休眠到swap分区里的系统文件）

$ docker exec -it thirsty_villani /bin/bash 
（此方式进入其shell退出时容器不会被挂起）
（我这里随机的这容器名字thirsty_villani，按tab自动补全）
```
现在我们有了一个ubuntu18.04环境，打开Yocto文档，该干嘛干嘛......  
```
# apt-get update && apt-get -y install gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat cpio python python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev pylint3 xterm
```
完事。现在只有一个ubuntu18.04的image。**下面注意，导入啥都好最终得到的都只有镜像**：
```
$ docker export thirsty_villani > yocto_env.tar  //导出容器为tar压缩包
$ cat yocto_env.tar | docker import - demo/ubuntu:demo //导入一个tar包内容创建一个文件系统镜像
```
```
$ docker save demo/ubuntu:demo -o yocto_env.tar //导出image为压缩包
$ docker load -i yocto_env.tar  //从压缩包加载一个image，注意是image不是容器
```
查看、删除：
```
$ docker ps -a   //列出所有容器
$ docker rm xxx  //删除container
$ docker images  //列出所有镜像
$ docker rmi xxx //删除image
```
从Dockerfile构建一个镜像(好处是远程拉取这文本文件很轻):  
```
# file: Dockerfile
FROM 本地或云端镜像名
RUN 命令...
ENV LANG en_US.UTF-8  //不设置语言Yocto会编译失败。
ENV LC_ALL en_US.UTF-8
...
（Dockerfile规则可以参考菜鸟教程之类的，就不写了）
```
直接给我的ubuntu16.04的模板，从哪里摘改的忘了......随便复制粘贴，请：  
```
FROM ubuntu:16.04

RUN apt-get update && apt-get -y install gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat cpio python python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping libsdl1.2-dev xterm tar locales

RUN rm /bin/sh && ln -s bash /bin/sh
RUN locale-gen en_US.UTF-8 && update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LC_ALL en_US.UTF-8

ENV USER_NAME whqee  //这里改你的用户名，其它没啥改的

ARG host_uid=1001
ARG host_gid=1001

RUN groupadd -g $host_gid $USER_NAME && useradd -g $host_gid -m -s /bin/bash -u $host_uid $USER_NAME
USER $USER_NAME
```
挂载本地文件夹到容器里：
```
$ docker run -d -v /home/whqee/Development/Yocto/:/root ubuntu:18.04  //本地路径:容器里路径
Yocto需要一样的路径，所以我们基本就是这样用了：
$ docker run -d -v /home/whqee/Development/Yocto/:/home/whqee/Development/Yocto/ ubuntu:18.04   //后台挂载
$ docker exec -it 容器名 /bin/bash   //进入容器bash命令行
# ...
```
停止容器：
```
$ docker stop 容器名
```
没了。需要掌握的就这些。
  

夜已深，关机洗澡睡觉。。