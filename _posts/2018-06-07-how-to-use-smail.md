---
layout: post
title: Android smali 使用
category: 技术
tags: [java, android, smali]
keywords: java, android, smali
description: 
---

> 用于记录Android逆向中smali的使用

## 下载smali工具
下载链接：https://bitbucket.org/JesusFreke/smali/downloads/

我们需要下载baksmali.jar和smali.jar两个工具jar包，并且这两个jar的版本号要一致。

## 如何使用

baksmali.jar 用于将apk中的classes.dex 解析成smali文件，命令如下：
```shell
java -jar baksmali.jar -o out_dir/ classes.dex
```

smali.jar 用于将修改后的smali文件重新打包成classes.dex文件，命令如下：
```shell
java -jar smali.jar -o classes.dex out_dir/
```

这样，我们就可以将原始apk中的部分代码逻辑修改并替换。

## smali语法
### 标志符

| 标志符 | 说明 | 
| - | - | 
| .field | 声明变量 | 
| .method | 定义方法 | 
| .parameter | 方法参数 |
| .prologue | 方法开始 |
| .line | 行数标记 |
| invoke-super | 调用父函数 |
| const/high16  v0, 0x7fo3 |　把0x7fo3赋值给v0 |
| invoke-direct | 调用函数 |
| return-void | 函数返回void |
| .end method | 函数结束 |
| new-instance | 创建实例 |
| iput-object | 对象赋值 |
| iget-object | 调用对象 |
| invoke-static | 调用静态函数 |

### 条件判断

基本形式: if-xx vA, vB, :cond_** 含义： 如果vA xx vB 则跳转到:cond_** , xx可以是如下情况：

| 表达式 | 说明 | 
| - | - | 
| if-eq | vA == vB |
| if-ne | vA != vB |
| if-lt | vA < vB |
| if-gt | vA > vB |
| if-le | vA <= vB |
| if-ge | vA >= vB | 
特殊形式，if-xx vA, :cond_**, 此时仅将vA与0比较，有如下情况：
| 表达式 | 说明 | 
| - | - | 
| if-eqz | vA == 0 |
| if-nez | vA != 0 |
| if-ltz | vA < 0 |
| if-gtz | vA > 0 |
| if-lez | vA <= 0 |
| if-gez | vA >= 0 | 

### 常用代码段
```java
sget-object v0, Ljava/lang/System;->out:Ljava/io/PrintStream;
const-string v1, "hello world!"
invoke-virtual {v0, v1}, Ljava/io/PrintStream;->println(Ljava/lang/String;)V
```
