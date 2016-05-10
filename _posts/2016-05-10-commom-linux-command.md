---
layout: post
title: 常用linux命令
category: 技术
tags: [linux]
keywords: command, linux
description: 
---

> 常用linux命令

## 计算文件夹内文件个数
```bash
ls -l |grep "^-"|wc -l #linux shell

ls -l |grep "^-"|busybox wc -l #android shell
```
