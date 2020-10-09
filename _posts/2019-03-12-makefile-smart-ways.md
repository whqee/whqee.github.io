---
layout: post
title: Makefile技巧记录/收录
date: 2019-03-12 20:00:00 +800
description: c语言 Makefile技巧                                                                                   # Add post description (optional)
img:  # Add image post (optional)
fig-caption: # Add figcaption (optional)
categories: [软、硬件编程, 自动化构建]
tags: [Makefile] # add tag
seo:
  date_modified: 2020-04-09 00:51:24 +0800
---

## wildcard递归列出目标
```makefile
# Make does not offer a recursive wildcard function, so here's one:
rwildcard=$(wildcard $1$2) $(foreach d,$(wildcard $1*),$(call rwildcard,$d/,$2))

# How to recursively find all files with the same name in a given folder
ALL_MAIN_C := $(call rwildcard,,main.c)

# How to recursively find all files that match a pattern
ALL_SOURCE_C := $(call rwildcard,,*.c)
all:
	echo $(ALL_MAIN_C)
	echo $(ALL_SOURCE_C)
```