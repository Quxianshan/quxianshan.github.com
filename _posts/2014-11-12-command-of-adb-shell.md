---
layout: post
title: ����adb shell ����
category: ����
tags: [adb shell, android]
keywords: adb shell, android
description: 
---
> ���������ڼ�¼���õ�adb shell����

##����绰
```bash
adb shell service call phone 2 s16 "10086"
adb shell am start -a android.intent.action.CALL -d tel:10086
```

##����ҳ
```bash
am start -a android.intent.action.VIEW -d http://localhost
```

##�����Ѱ�װ��Ӧ��
```bash
am start -n package_name/.class_name
```

##���ͼ�ֵ(ģ���û���������)
```bash
adb shell input keyevent ��ֵ
```

##ģ���û��������
```bash
adb shell input tap x���� y����
```