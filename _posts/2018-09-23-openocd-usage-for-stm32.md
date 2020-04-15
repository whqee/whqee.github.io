---
layout: post
title: openocd的使用
date: 2018-09-24 00:00:00 +800
description: openocd的使用                                                                                 # Add post description (optional)
img:  # Add image post (optional)
fig-caption: # Add figcaption (optional)
categories: [软、硬件编程, STM32]
tags: [OpenOCD] # add tag
seo:
  date_modified: 2020-04-09 00:51:24 +0800
---

来自我的CSDN，原文： https://blog.csdn.net/baidu_36482169/article/details/81005060

一、openocd安装使用遇到的问题：

`sudo apt-get install` 的openocd-0.10.0，玩着Nucleo STM32L496，flash erase...不行：
```
> flash erase_sector 0 0 255    
Cannot identify target as a STM32L4 family.    
auto_probe failed     
in procedure 'target' 
```
![]()


查得结果说是openocd的问题，遂至官网下载源码编译。


库更新了，源码编译不通过，自己捣鼓了两个多小时，一路找依赖，修改部分源码，啊啊啊终于编译好～～～

开箱即用成品：https://pan.baidu.com/s/1muJCRbFJMLSZbu2IvkXRMA

（修改后的源码：https://pan.baidu.com/s/1Vh5TcL-4SjbweTy1FoouLw ）

解压直接放到想放的地方，比如/opt/openocd，然后自己到/etc/profile或者~/.bashrc添加环境变量即是

至于编译出来的pdf手册，呃不知道为什么没有出来，里面的man1感觉没用，所以还是看官网吧--此文章最后面。

换了后如图：

[Click and drag to move]

nice～
二、openocd的基本使用：

1、烧录前要连接板子，连接板子命令，例如：

openocd -f  board/st_nucleo_l4.cfg 

或者拆开两条：

openocd -f interface/stlink.cfg -f target/st_nucleo_l4xx.cfg

board/  interface/  target/  等都在openocd的share目录下的script文件夹里，讲到这可以了，自己捣鼓下就ok 就不一一讲了。

上一张图：

[Click and drag to move]

成功后打开新命令行窗口

2、烧录代码：

flash下载基本操作：
```bash
telnet localhost 4444

>reset halt                         //要先挂起才能开始擦除
>flash erase_sector 0 0 last        //全部擦除
>flash write_image xxx.hex/bin/elf  //完成
>reset
或者
>halt
>flash write_image erase xxx.bin/hex/elf 0  //自动擦除并从0地址开始                  （1）
>reset                                      //啦啦啦快看你的板子～
```

[Click and drag to move]

下面这个操作最舒服，一条搞定：

>\>program xx.elf                                                                        （2）
>\>reset            // look your board

[Click and drag to move]

上面标记的（1）（2）命令，烧录elf文件和hex文件后面不用写擦除地址，因为它们包含有这些信息，而bin文件只有要执行的二进制文件，后面要加地址项。

熟悉之后，可以直接做成脚本： （待续）

shell脚本编写可参考: https://github.com/RIOT-OS/RIOT/commit/7713bede996a1fb45894ab9e852e38c4fa1809f3


官方参考资料：

Openocd User's Guide:  http://openocd.org/doc-release/html/Command-and-Driver-Index.html#Command-and-Driver-Index
