---
title: 数据结构与算法---二叉树基本算法及递归
header-img: /img/header_img/post-bg-desk.jpg
catalog: true
top:
tags:
  - algorithm
  - data structure
categories:
  - work
abbrlink: 7b21
date: 2020-12-19 00:00:00
subtitle:
---

# 数据结构与算法---二叉树基本算法及递归
1）假设以X节点为头，假设可以向X左树和X右树要任何信息
2）在上一步的假设下，讨论以X为头节点的树，得到答案的可能性（最重要）
3）列出所有可能性后，确定到底需要向左树和右树要什么样的信息
4）把左树信息和右树信息求全集，就是任何一棵子树都需要返回的信息S
5）递归函数都返回S，每一棵子树都这么要求
6）写代码，在代码中考虑如何把左树的信息和右树信息整合出整棵树的信息