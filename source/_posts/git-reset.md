---
title: git 回滚指定版本并推送到远程分支
header-img: /img/header_img/post-bg-desk.jpg
catalog: true
top:
tags:
  - git
categories:
  - work
abbrlink: '6186'
---

1. ###### 回退

    ```
    git log //查看提交的历史
    git reset --hard $要回退的commit号
    ```

2. ###### 切换到临时分支推送并删除原分支

    ```
    git checkout -b temp            // 新建临时分支并切换
    git push origin temp:temp       // 将代码push到temp分支
    git push origin --delete main   // 删除远端分支main，此处注意如果要删除的远端分支为默认分支则不可删除，需将默认分支切换到其他分支
    git branch -d main              // 删除本地分支main
    ```

3. ###### 新建并切换回原分支并删除临时分支

    ```
    git checkout -b main            // 新建本地分支main并切换
    git push origin main            // 提交分支main到远端
    git branch -d temp				// 删除本地临时分支
    git push origin --delete temp	// 删除远端临时分支，此处注意如果要删除的远端分支为默认分支则不可删除，需将默认分支切换到其他分支
    ```

    > ###### github切换默认分支：
    >
    > 	进入对应repo → Settings → Branches → Switch to another branch