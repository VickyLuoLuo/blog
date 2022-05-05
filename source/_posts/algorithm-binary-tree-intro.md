---
title: 数据结构与算法---二叉树的相关概念
catalog: true
tags:
  - algorithm
  - data structure
  - 二叉树
abbrlink: ac01
date: 2020-12-12 22:34:53
subtitle:
header-img:
categories:
mathjax: true
---

# 数据结构与算法-二叉树的相关概念

## 平衡二叉树

> 定义:在一个二叉树中, 每一颗子树, 左树的高度和右树的高度差不超过1 

以X为头的树, 想保持平衡性, 要满足以下条件: 

1. 左树是平衡二叉树 
2. 右树是平衡二叉树 
3. 左树和右树的高度差不要超过1 

![Pasted image 20201218004452.png](https://s2.loli.net/2022/05/05/xsaYJP1IUOudvk6.png)

## 二叉搜索树/搜索二叉树/BST
> 定义: 整棵树没有重复值, 左树比当前节点小, 右树比当前节点大, 每一个颗子树都满足上面条件 

检测原则:

1. X左树是BST
2. X右树是BST
3. X左树最大值max < X
4. X右树最小值min  > X  

![Pasted image 20201218004619.png](https://s2.loli.net/2022/05/05/UEpxRK87d4nAHjb.png)
Note: 空树是BST

![Pasted image 20201120190236](https://s2.loli.net/2022/05/05/4dfQeLrFEHtKSiT.png)

以 5, 3, 7 为头都达标

![Pasted image 20201218004535](https://s2.loli.net/2022/05/05/iMnP7sDTHtW3OSI.png)




## 平衡搜索二叉树

1. 不管输入状况如何, 都要保证每次做完插入, 删除后能够自平衡, 
2. 自平衡代价不能超过logN, 而且做完调整之后,每次查询,删除, 插入代价都能够做到logN
3. 具有这样性质的的搜索二叉树种类很多，如AVL树、红黑树

### 平衡搜索二叉树的左旋, 右旋操作
左旋,右旋是对头节点说的
对哪个节点实行左旋还是右旋

对谁进行右旋, 谁就倒向右方
对谁进行左旋, 谁就倒向左方


![Pasted image 20201217182853](https://s2.loli.net/2022/05/05/gUTMcIWn49dZVPA.png)

## 完全二叉树

> 定义:每一层要么是满的，要么最后一层不满;而且最后一层即便是不满的, 也是从左到右依次变满的

大根堆, 小根堆一定是完全二叉树

判断标准:  
1. 遇到的每一个节点, 如果有右孩子,没有左孩子的话-->一定不是完全二叉树, 否则继续判断 
    因为完全二叉树不可能出现有右无左的情况
2. 一旦遇到第一个左右孩子不双全的节点, 则后序遇到的所有节点都必须是叶节点  

![Pasted image 20201120200550](https://s2.loli.net/2022/05/05/byfw3Q28mBhloST.png)
Note: 上图中数值大小关系不重要

## 满二叉树

> 定义: 二叉树每一层的节点都是满的 

性质: 高度是h, 节点个数是N, 存在如下关系: 

`2^h -1 == N`

NOTE: 满二叉树有国内, 国外两种不同的定义
ref: https://baike.baidu.com/item/%E6%BB%A1%E4%BA%8C%E5%8F%89%E6%A0%91  

#### 国外的满二叉树定义   
如果一棵二叉树的结点要么是叶子结点，要么它有两个子结点，这样的树就是满二叉树。
```java
public boolean isFullTree(TreeNode root) {  
    // write your code here  
    if (root == null) {  
        return true;  
    }  
    if (root.left == null ^ root.right == null) {  
        return false;  
    }  
    return isFullTree(root.left) && isFullTree(root.right);  
}
```

#### 国内的满二叉树定义   
一个二叉树，如果每一个层的结点数都达到最大值，则这个二叉树就是满二叉树。

## 后继节点, 前序节点
> 后继节点：中序遍历排序中相对的后一个。
>
> 前序节点：中序遍历排序中相对的前一个。

后继节点查找策略：

1. 每次找后继节点都是先找当前节点的右子树中的最左节点
2. 如果没有右子树，则找当前节点的父节点，并且当前节点是父节点的左子树的节点。
3. 如果一直没有一个节点是父节点的左孩子节点, 那么这个节点是最右的节点, 没有后继, 返回null。
