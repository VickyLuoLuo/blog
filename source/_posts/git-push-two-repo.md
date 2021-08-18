---
title: Git同时push到Github和Gitee
header-img: /img/header_img/post-bg-desk.jpg
catalog: true
top:
tags:
  - git
categories:
  - work
abbrlink: 1af5
date: 2020-12-18 00:00:00
subtitle:
---

### 目的

Github有时访问会很慢，想用Gitee做个备份，如何将已有的Github项目同步到Gitee，并将修改同时push到自己的Github和Gitee上呢？

### 1. 在Gitee上新建仓库，并关联Github仓库

新建gitee仓库
![create_gitee_repo](/img/1-work/tools/create_gitee_repo.png)

复制github仓库的https地址并粘贴
![import-github-repo](/img/1-work/tools/import-github-repo.png)

创建，等待同步。

### 2. 拉取Gitee项目，修改.git配置文件

克隆创建好的Gitee项目：
`git clone $项目地址`

git默认远程仓库为origin，可用git remote查看。
添加gitee地址到origin：
`git remote set-url --add origin $项目gitee地址`

添加github地址到origin：
`git remote set-url --add origin $项目github地址`

也可直接打开配置文件修改，文件位于.git/config
```
[remote "origin"]
 	url = https://gitee.com/$项目gitee地址
 	fetch = +refs/heads/*:refs/remotes/origin/*
 	url = https://github.com/$项目github地址
```

### 3. 修改完成，push一下看看吧

若push不成功，可以检查一下自己是否有仓库权限，配置地址是否正确

#### push时记住用户名密码的方法

配置文件中使能保存密码功能：
`git config credential.helper store`

提交时输入用户名密码，再次提交就记住啦。