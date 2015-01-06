---
layout: post
title: android sdk 中编译jar文件
category: 技术
tags: [java, android, jar]
keywords: java, android, jar
description: 
---

>介绍在android sdk中编译jar文件的方法

###直接编译源码生成jar文件
在进行系统定制时，可能会将自己定义的类与接口封装到一起作为公共部分供其他apk调用，因此需要将这类文件打包编译成jar文件。    
其中，Android.mk文件配置如下：    

```bash
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_SRC_FILES := $(call all-java-files-under, src/)
LOCAL_JAVA_LIBRARIES := bouncycastle core
LOCAL_MODULE := jar_name
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_CLASS := JAVA_LIBRARIES
LOCAL_MODULE_PATH := $(TARGET_OUT_JAVA_LIBRARIES)
include $(BUILD_JAVA_LIBRARY)
```
配置项说明：    
LOCAL_PATH ---- 表示当前Android.mk文件所在的路径    
include $(CLEAR_VARS) ----  清空source\build\core\config.mk中定义的变量值(不包括LOCAL_PATH)    
LOCAL_SRC_FILES ---- 表示需要将哪个目录下的文件编译进jar文件当中     
LOCAL_JAVA_LIBRARIES ---- 依赖的本地java 库     
LOCAL_MODULE ---- 编译出的jar包名称     
LOCAL_MODULE_TAGS ---- 编译选项：user eng tests optional    
  user 表示模块仅在user版本下编译 eng 表示模块仅在eng版本下编译 tests表示仅在tests版本下编译 optional 则所有版本都编译    
配置完成后，手动mm即可在out目录下生成对应LOCAL_MODULE配置的jar文件。
使用时，在apk编译的Android.mk的LOCAL_JAVA_LIBRARIES中添加此jar文件的LOCAL_MODULE 名称即可。
按照上述配置使用后，发现编译时没有报错，等apk运行时报NoClassDefFoundError，搜索到如下结果：    
In short NoClassDefFoundError will come if a class was present during compile time but not available in java classpath during runtime. 
所以需要将我们编译的jar包放到系统的CLASS_PATH当中，在device目录下，有对应的init.rc文件，其中有定义CLASS_PATH的部分，
将编译的jar文件所在的路径及文件名添加到字段尾部以":"分隔即可：    

```bash
export BOOTCLASSPATH ...:/system/framework/jar_name.jar
```

###编译eclipse导出的第三方jar包
在android sdk中引用第三方的jar包时，需按照如下配置Android.mk：    

```bash
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES := new_name:libs/source_jar.jar
include $(BUILD_MULTI_PREBUILT)
```
说明：    
LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES 定义要编译jar模块的名称以及原始jar包所在的路径(相对于Android.mk文件的位置)    

在apk的Android.mk文件中， 在LOCAL_STATIC_JAVA_LIBRARIES 后添加new_name即可编译运行。
