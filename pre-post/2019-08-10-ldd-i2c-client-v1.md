---
layout: post
title: Linux i2c client driver （1）
date: 2019-08-10 20:00:00 +800
description: You’ll find this post in your `_posts` directory. Go ahead and edit it
  and re-build the site to see your changes.                                                                                   # Add post description (optional)
img:  # Add image post (optional)
fig-caption: # Add figcaption (optional)
categories: [软、硬件编程, Linux驱动]
tags: [MYS-6ULX-IOT, Linux驱动, Sensor Driver] # add tag
seo:
  date_modified: 2020-03-18 02:24:22 +0800
---
(Kernel: 4.1.15)
本次实验使用max30102光电心率传感器作为client device，我们的关注点是i2c client驱动的编写，所以仅仅实现的是最基本的demo，本次为misc driver。
首先，修改device tree：
![max30102 dev reg]({{site.baseurl}}/assets/img/MYS-6ULX-IOT/i2c_dts.png)<br>
如图，i2c-2已有默认引脚组，新加max30102使用该引脚组，故只需添加compatible和reg，注意，这里的reg是去掉读/写位的7个位。
@57的由来如下图：
![max30102 dev reg]({{site.baseurl}}/assets/img/MYS-6ULX-IOT/Max30102_reg.png)<br>

然后是driver：
