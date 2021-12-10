---
title: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.
header-img: /img/header_img/post-bg-desk.jpg
catalog: true
top: 
tags:
  - git
categories:
  - work
abbrlink: 9f70
date: 2021-06-04 00:00:00
---

#### git push错误

remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.

#### 解决方案：

1. ###### 登录github → Settings → Developer settings → Personal access tokens → Generate new token

2. ###### 复制新生成的token（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），执行：

    ```
    git remote set-url origin https://$new-token@github.com/$username/$repo-name.git
    ```

3. ###### 重新push即可