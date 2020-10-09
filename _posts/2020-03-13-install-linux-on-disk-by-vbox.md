---
layout: post
title: 使用vbirtualbox安装Linux到本地硬盘(图解、详细)
date: 2020-03-13 20:00:00 +800
description: 使用vbirtualbox安装Linux到本地硬盘， Virtualbox使用物理机                                                                                   # Add post description (optional)
img:  # Add image post (optional)
fig-caption: # Add figcaption (optional)
categories: [效率工具、实用技能, 桌面版Linux]
tags: [安装桌面版Linux] # add tag
seo:
  date_modified: 2020-04-09 00:51:24 +0800
---


### 本次demo材料： 
华硕顽石4代(16年买的机器)、virtualbox、gnome版manjaro 19.02  
(2020 4月8日更：我又用gnome版的20.0-pre1在linux上用vbox装了一次，没毛病。)
>若你的材料不一样，可能会grub安装失败，百度下就好，百度了还不行可以邮箱我  
### 分区最好都加上label  
>（在第二轮测试中，我用了xfce19.02,并且没有新建efi分区而是用了旧的，最后步骤的‘安装grub’失败，原因不明。安装grub可以百度一下，很多答案，暂不post，搞不定可以私我～）  

### 以WIN10为例：  
>思路：让我们要装系统的硬盘脱离当前系统的管理，起码要有一个分区是脱离了win10的接管了的。  

<!-- [高清pdf文档]({{site.baseurl}}/assets/others/win10上使用virtualbox安装linux到本地物理硬盘.pdf) -->
1、进入磁盘管理，如果你是打算给一部分空间linux用，压缩一个空卷出来：
![](/post_images/vbox/Picture1.png)  
2、接下来右键左下角菜单键，管理员身份打开PowerShell，使用VBoxManage创建一个指向物理硬盘的虚拟磁盘文件，比如我这里用的是1T机械硬盘：  
![](/post_images/vbox/Picture2.png)  
3、创建好了指向本地物理磁盘的虚拟磁盘文件，接下来打开vbox新建虚拟机，如图把刚才创建的指向本地物理硬盘的虚拟磁盘添加进去：  
![](/post_images/vbox/Picture3.png)  
4、创建好了，结果如下图，点击设置：  
![](/post_images/vbox/Picture4.png)  
5、看图：  
![](/post_images/vbox/Picture5.png)  
![](/post_images/vbox/Picture6.png)  
![](/post_images/vbox/Picture7.png)  
![](/post_images/vbox/Picture8.png)  
6、这里需要注意，我们需要再添加一个虚拟磁盘（总之就是要加一个非待安装盘进去）：  
![](/post_images/vbox/Picture9.png)  
7、点击创建，然后下一步下一步就行：  
![](/post_images/vbox/Picture10.png)  
![](/post_images/vbox/Picture11.png)  
![](/post_images/vbox/Picture12.png)  
8、设置完，点击确定，然后启动该虚拟机：  
![](/post_images/vbox/Picture13.png)  
9、按着图解的步骤进行安装就好了，看图集：  
![](/post_images/vbox/Picture14.png)  
![](/post_images/vbox/Picture15.png)  
![](/post_images/vbox/Picture16.png)  
10、若采用方案二手动分区，需要注意，挂载点/boot/efi记得勾上“boot”标记，否则会安装不了引导程序，下一步之后没报错那就没问题：  
![](/post_images/vbox/Picture17.png)  
![](/post_images/vbox/Picture18.png)  
![](/post_images/vbox/Picture19.png)  
11、没弹警告框，后面继续点next/下一步就行，继续看图：  
![](/post_images/vbox/Picture20.png)  
![](/post_images/vbox/Picture21.png)  
![](/post_images/vbox/Picture22.png)  
![](/post_images/vbox/Picture23.png)  
![](/post_images/vbox/Picture24.png)  
![](/post_images/vbox/Picture25.png)  
12、最后重启完事。但是，比如说你之前有装过其它系统，重启后还是进去之前那个系统的引导了，你重启时候选择进入刚装的系统的引导就好了，比如我这里是“EFI：”开头，比如我的华硕，开机按`esc`键EFI或BIOS菜单就出来了。  
13、致Manjaro用户：如果你不懂archlinux、想稳定地用这个系统，不要添加archlinux源！要么你装完要装的软件把源去掉！Manjaro's source is a stable Archlinux source, but Archlinux's source is not Manjaro's source.  
