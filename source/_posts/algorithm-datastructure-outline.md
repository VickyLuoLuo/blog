---
title: 数据结构和算法大纲
header-img: /img/header_img/post-bg-desk.jpg
catalog: true
top: 1
tags:
  - algorithm
  - data structure
categories:
  - work
abbrlink: '2312'
date: 2020-11-26 17:33:24
subtitle:
---

# 数据结构

## 一、线性表

### 1. 数组

-  动态数组

### 2. 链表

-  单链表
-  双向链表
-  循环链表
-  双向循环链表
-  静态链表

### 3. 栈

-  顺序栈
-  链式栈

### 4. 队列

-  普通队列
-  双端队列
-  阻塞队列
-  并发队列
-  阻塞并发队列

## 二、散列表

### 1. 散列函数

### 2. 冲突解决

-  链表法
-  开放寻址
-  其他

### 3. 动态扩容

### 4. 位图

## 三、树

### 1. 二叉树

-  平衡二叉树
-  二叉查找树
-  平衡二叉查找树

    - AVL树
    - 红黑树

-  完全二叉树
-  满二叉树

### 2. 多路查找树

-  B树
-  B+树
-  2-3树
-  2-3-4树

### 3. 堆

-  小顶堆
-  大顶堆
-  优先级队列

    - 与普通队列区别：保证每次取出的元素是队列中优先级最高的，优先级别克自定义
    - 常用场景：从杂乱的数据中按照一定顺序筛选数据
    - 本质：二叉堆结构，binary heap. 利用数组来实现完全二叉树
    - 特性

        - 1.数组中第一个元素拥有最高优先级
        - 2.给定下标i，那么对于arr[i]：

            -  1）父节点：对应元素下标为（i - 1）/ 2
            -  2）左侧子节点：对应下标为2*i + 1
            -  3）右侧子结点：对应下标为2*i + 2

        - 3.每个元素的优先级都必须高于两侧子节点

    - 基本操作

        - 1.向上筛选（shift up / bubble up）
        - 2.向下筛选 （shift down / bubble down）

    - 时间复杂度：
        长度为n的数组，取前k个：O（logn）
        初始化：O(k)
    - Leetcode

        - [347. 前 K 个高频元素-中等](https://leetcode-cn.com/problems/top-k-frequent-elements/)

-  斐波那契堆
-  二项堆

-  其他

    - 树状数组 Binary Indexed Tree

        - 求数组中前K个元素的总和（或平均值）
        - LeetCode

    - 线段树Segment Tree

        - 按照二叉树的形式存储数据的结构，每个结点保存数组中某一段的总和
        - Leetcode 

            - [315. 计算右侧小于当前元素的个数-困难](https://leetcode-cn.com/problems/count-of-smaller-numbers-after-self/)

## 四、图

### 1. 图的存储

-  邻接矩阵
-  邻接表

### 2. 拓扑排序

### 3. 最短路径

### 4. 关键路径

### 5. 最小生成树

### 6. 二分图

### 7. 最大流

# 算法

## 一、复杂度

### 1. 时间复杂度

- 如何分析

    - 1. 只关注循环执行次数最多的一段代码
    - 2. 加法法则：总复杂度等于量级最大的那段代码的复杂度
    - 3. 乘法法则：嵌套代码的复杂度等于嵌套内外代码复杂度的乘积

- 常见时间复杂度实例分析

    ![image-20210917165515820](https://tva2.sinaimg.cn/large/006YzKDNly1gujq7ui7kaj60vq0fwadg02.jpg)

    - 1. O(1)

        - 一般情况下，只要算法中不存在循环语句、递归语句，即使有成千上万行的代码，其时间复杂度也是Ο(1)。

    - 2. O(logn)、O(nlogn)

        - i=1; while (i <= n) { i = i * 2; }
            在对数阶时间复杂度的表示方法里，我们忽略对数的“底”，统一表示为 O(logn)。
        - 如果一段代码的时间复杂度是 O(logn)，我们循环执行 n 遍，时间复杂度就是 O(nlogn) 了。归并排序、快速排序的时间复杂度都是 O(nlogn)

    - 3. O(m+n)、O(m*n)

        - 代码的复杂度由两个数据的规模来决定，无法事先评估 m 和 n 谁的量级大

### 2. 空间复杂度

- 时间复杂度的全称是渐进时间复杂度，表示算法的执行时间与数据规模之间的增长关系。类比一下，空间复杂度全称就是渐进空间复杂度（asymptotic space complexity），表示算法的存储空间与数据规模之间的增长关系。
- 常见的空间复杂度就是 O(1)、O(n)、O(n2 )，像 O(logn)、O(nlogn) 这样的对数阶复杂度平时都用不到。

### 3. Tips

- 越高阶复杂度的算法，执行效率越低。常见的复杂度并不多，从低阶到高阶有：O(1)、O(logn)、O(n)、O(nlogn)、O(n^2 )。

    ![image-20210917165700182](https://tvax2.sinaimg.cn/large/006YzKDNly1gujq9nxktgj60vq0hsgno02.jpg)

## 二、基本算法思想

### 1. 贪心算法

### 2. 分治算法

- 快排、归并
- 在O(n)内查找一个无序数组中第K大元素

### 3. 动态规划

### 4. 回溯算法

### 5. 枚举算法

## 三、排序

![image-20210917165757241](https://tva3.sinaimg.cn/large/006YzKDNly1gujqana3gej60vq0je42y02.jpg)

### 1. O(n^2) 

![image-20210917165853155](https://tvax2.sinaimg.cn/large/006YzKDNly1gujqbw4ot7j60vq0g8dk802.jpg)

- 冒泡排序

    - 原地排序、稳定排序

- 插入排序

    - 原地排序、稳定排序
    - leetcode:

    [147. 对链表进行插入排序-中等](https://leetcode-cn.com/problems/insertion-sort-list/)

- 选择排序

    - 原地、不稳定

- 希尔排序

### 2. O(nlogn)

![image-20210917165958632](https://tvax3.sinaimg.cn/large/006YzKDNly1gujqcqpdquj60vq0l1wl602.jpg)

-  归并排序

    - 分治思想、递归实现
    - 稳定排序、非原地排序，空间复杂度高O(n)
    - 先把数组从中间分成前后两部分，然后对前后两部分分别排序，再将排好序的两部分合并在一起，这样整个数组就都有序了。
    - 递推公式：merge_sort(p…r) = merge(merge_sort(p…q), merge_sort(q+1…r))
        终止条件：p >= r 不用再继续分解
    - 处理过程从下到上个，先处理子问题，再合并

-  快速排序

    - 分治思想、递归实现
    - 原地排序，空间复杂度O(1)，不稳定
    - 选择 p 到 r 之间的任意一个数据作为 pivot（分区点），遍历 p 到 r 之间的数据，将小于 pivot 的放到左边，大于 pivot 的放到右边， pivot 放到中间。根据分治、递归的处理思想，我们可以用递归排序下标从 p 到 q-1 之间的数据和下标从 q+1 到 r 之间的数据，直到区间缩小为 1
    - 递推公式：quick_sort(p…r) = quick_sort(p…q-1) + quick_sort(q+1… r)
        终止条件：p >= r
    - 处理过程从上到下，先分区，再处理子问题

-  堆排序

### 3. O(n)

- 计数排序
    - 1.算法原理
        1）计数其实就是桶排序的一种特殊情况。
        2）当要排序的n个数据所处范围并不大时，比如最大值为k，则分成k个桶
        3）每个桶内的数据值都是相同的，就省掉了桶内排序的时间。
    - 2.使用条件
        1）只能用在数据范围不大的场景中，若数据范围k比要排序的数据n大很多，就不适合用计数排序；
        2）计数排序只能给非负整数排序，其他类型需要在不改变相对大小情况下，转换为非负整数；
        3）比如如果考试成绩精确到小数后一位，就需要将所有分数乘以10，转换为整数。
    - 2. 案例分析：
            假设只有8个考生分数在0-5分之间，成绩存于数组A[8] = [2，5，3，0，2，3，0，3]。
            使用大小为6的数组C[6]表示桶，下标对应分数，即0，1，2，3，4，5。
            C[6]存储的是考生人数，只需遍历一边考生分数，就可以得到C[6] = [2，0，2，3，0，1]。
            对C[6]数组顺序求和则C[6]=[2，2，4，7，7，8]，c[k]存储的是小于等于分数k的考生个数。

- 基数排序

    - 1.算法原理（以排序10万个手机号为例来说明）
        1）比较两个手机号码a，b的大小，如果在前面几位中a已经比b大了，那后面几位就不用看了。
        2）借助稳定排序算法的思想，可以先按照最后一位来排序手机号码，然后再按照倒数第二位来重新排序，以此类推，最后按照第一个位重新排序。
        3）经过11次排序后，手机号码就变为有序的了。
        4）每次排序有序数据范围较小，可以使用桶排序或计数排序来完成。
    - 2.使用条件
        1）要求数据可以分割独立的“位”来比较；
        2）位之间由递进关系，如果a数据的高位比b数据大，那么剩下的地位就不用比较了；
        3）每一位的数据范围不能太大，要可以用线性排序，否则基数排序的时间复杂度无法做到O(n)。

- 桶排序

    - 1.算法原理：
        1）将要排序的数据分到几个有序的桶里，每个桶里的数据再单独进行快速排序。
        2）桶内排完序之后，再把每个桶里的数据按照顺序依次取出，组成的序列就是有序的了。
    - 2.使用条件
        1）要排序的数据需要很容易就能划分成m个桶，并且桶与桶之间有着天然的大小顺序。
        2）数据在各个桶之间分布是均匀的。
    - 3.适用场景
        1）桶排序比较适合用在外部排序中。
        2）外部排序就是数据存储在外部磁盘且数据量大，但内存有限无法将整个数据全部加载到内存中。
    - 4.应用案例
        1）需求描述：
        有10GB的订单数据，需按订单金额（假设金额都是正整数）进行排序
        但内存有限，仅几百MB
        2）解决思路：
        扫描一遍文件，看订单金额所处数据范围，比如1元-10万元，那么就分100个桶。
        第一个桶存储金额1-1000元之内的订单，第二个桶存1001-2000元之内的订单，依次类推。
        每个桶对应一个文件，并按照金额范围的大小顺序编号命名（00，01，02，…，99）。
        将100个小文件依次放入内存并用快排排序。
        所有文件排好序后，只需按照文件编号从小到大依次读取每个小文件并写到大文件中即可。
        3）注意点：若单个文件无法全部载入内存，则针对该文件继续按照前面的思路进行处理即可。

## 四、搜索

### 1. 深度优先搜索

### 2. 广度优先搜索

### 3. A*启发式搜索

## 五、查找

### 1. 线性表查找（二分查找）

### 2. 树结构查找

### 3. 散列表查找

## 六、字符串匹配

### 1. 朴素

### 2. KMP

### 3. Robin-karp

### 4. Boyer-Moore

### 5. AC自动机

### 6. 前缀树（字典树）Trie

- 用于字典查找：如给定一系列构成字典的字符串（个数为N），在字典中找出所有以“ABC”开头的字符串，其中最长的为M

    - 暴力搜索：O(M*N)
    - 前缀树：O(M)

- 经典应用：搜索框自动补全、拼写检查、IP 路由 (最长前缀匹配)、九宫格打字预测

- 特性

    - 1.每个结点至少包含两个基本属性

        - 1）children：数组或集合，罗列每个分支中包含的所有字符
        - 2）isEnd：布尔值，表示该结点是否为某个字符串的结尾

    - 2.根节点为空
    - 3.除根结点外，其他节点都可能是单词的结尾，叶子结点一定是单词的结尾

- 基本操作

    - 1.向 Trie 树中插入键

        ![image-20210917170051035](https://tvax3.sinaimg.cn/large/006YzKDNly1gujqdl7gvfj60xc0go42z02.jpg)

        - 算法分析

            - 从根开始搜索它对应于第一个键字符的链接。有两种情况：

                - 1.链接存在。沿着链接移动到树的下一个子层。算法继续搜索下一个键字符。
                - 2.链接不存在。创建一个新的节点，并将它与父节点的链接相连，该链接与当前的键字符相匹配
                    重复以上步骤，直到到达键的最后一个字符，然后将当前节点标记为结束节点，算法完成。

        - 复杂度分析

            - 时间复杂度：O(m)，其中 m为键长。在算法的每次迭代中，我们要么检查要么创建一个节点，直到到达键尾。只需要 m次操作。
            - 空间复杂度：O(m)。最坏的情况下，新插入的键和 Trie 树中已有的键没有公共前缀。此时需要添加 m 个结点，使用 O(m) 空间。

    - 2.在 Trie 树中查找键

        ![image-20210917170128796](https://tva4.sinaimg.cn/large/006YzKDNly1gujqe8598lj60lu0gon0902.jpg)

        - 算法分析

            - 每个键在 trie 中表示为从根到内部节点或叶的路径。我们用第一个键字符从根开始，。检查当前节点中与键字符对应的链接。有两种情况：

            - 1. 存在链接。我们移动到该链接后面路径中的下一个节点，并继续搜索下一个键字符。

                - 2. 不存在链接。若已无键字符，且当前结点标记为 isEnd，则返回 true。否则有两种可能，均返回 false :

                - 1）还有键字符剩余，但无法跟随 Trie 树的键路径，找不到键。
                    - 2）没有键字符剩余，但当前结点没有标记为 isEnd。也就是说，待查找键只是Trie树中另一个键的前缀。

        - 复杂度分析

            - 时间复杂度 : O(m)。算法的每一步均搜索下一个键字符。最坏的情况下需要 m 次操作。
            - 空间复杂度 : O(1)。

- Leetcode 

    - [208. 实现 Trie (前缀树) -中等](https://leetcode-cn.com/problems/implement-trie-prefix-tree/)
    - [211. 添加与搜索单词 - 数据结构设计-中等](https://leetcode-cn.com/problems/design-add-and-search-words-data-structure/)
    - [212. 单词搜索 II-困难](https://leetcode-cn.com/problems/word-search-ii/)

### 7. 后缀数组

## 七、其他

### 1. 数论

### 2. 计算几何

### 3. 概率分析

### 4. 并查集

### 5. 拓扑网络

### 6. 矩阵计算

### 7. 线性规划
