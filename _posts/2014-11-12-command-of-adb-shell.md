---
layout: post
title: 常用adb shell命令
category: 技术
tags: [keycode, android]
keywords: keycode, android
description: 
---

> 常用adb shell命令

##拨打电话
```bash
adb shell service call phone 2 s16 "10086"
adb shell am start -a android.intent.action.CALL -d tel:10086
```

##打开网页
```bash
am start -a android.intent.action.VIEW -d http://localhost
```

##启动已安装的应用
```bash
am start -n package_name/.class_name
```

##发送键值(模拟用户按键操作)
```bash
adb shell input keyevent 键值
```

##模拟用户点击操作
```bash
adb shell input tap x坐标 y坐标
```
