---
layout: post
title: ApkTool的使用
category: 技术
tags: [java, android]
keywords: java, android
description: 
---
> 用于记录如何使用ApkTool工具 反编译apk 回编译等操作。

## 操作步骤

1. 需要下载ApkTool工具：https://ibotpeaches.github.io/Apktool/ 
2. 执行apktool d source.apk 反编译apk
3. 对smali文件进行修改
4. 执行apktool b source_dir/ 回编译apk
5. 回编译apk后，需要对apk进行签名操作，执行：
```shell
java -jar signapk.jar platform.x509.pem platform.pk8 source_modified.apk dist.apk
```
