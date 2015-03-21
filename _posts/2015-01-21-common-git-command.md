---
layout: post
title: 常用git命令
category: 技术
tags: [git]
keywords: git
description: 
---

> 常用的git命令

## 新建仓库

```bash
git init
git --bare init
```

## 设置远程仓库

```bash
git add remote origin ssh://username@hostname:/git_path.git
```

## 提交代码到远程仓库

```bash
git push origin master
```

## 下载git

```bash
git clone ssh://username@hostname:/git_path.git
```
