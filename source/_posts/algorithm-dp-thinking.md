---
title: 数据结构与算法---暴力递归到动态规划
header-img: /img/header_img/post-bg-desk.jpg
catalog: true
top:
tags:
  - algorithm
  - 递归
  - dp
categories:
  - work
abbrlink: f6ae
date: 2021-04-29 00:00:00
subtitle:
---

# 数据结构与算法-暴力递归到动态规划

## 暴力递归

### 暴力递归设计思路

> 暴力递归就是尝试 

1. 把问题转化为规模缩小了的同类问题的子问题 
2. 有明确的不需要继续进行递归的条件(base case) 
3. 有当得到了子问题的结果之后的决策过程 
4. 不记录每一个子问题的解 

### 常见的递归

- [打印n层汉诺塔从最左边移动到最右边的全部过程（非递归）](#打印n层汉诺塔从最左边移动到最右边的全部过程（非递归）)

- [打印n层汉诺塔从最左边移动到最右边的全部过程（递归）](#打印n层汉诺塔从最左边移动到最右边的全部过程（递归）)

    ==> 结论: N层汉诺塔问题移动的最优步数: $2^N -1 $ 步

> **字符串的子串、子序列 & 全排列**
>
> 子串：必须是连续的
>
> 子序列：在原始字符串中从左往右依次拿字符, 可以不连续, 但是相对次序不能乱
>
> 全排列：可以打乱顺序
>
> 一个字符串，子串个数是$N^2$个，子序列个数是$2^N$ 个，全排列个数是$N!$ 个

- [打印一个字符串的全部子序列](#打印一个字符串的全部子序列)

- [打印一个字符串的全部子序列，要求不要出现重复字面值的子序列](#打印一个字符串的全部子序列，要求不要出现重复字面值的子序列)

- [打印一个字符串的全部排列](#打印一个字符串的全部排列)

- [打印一个字符串的全部排列，要求不要出现重复的排列](#打印一个字符串的全部排列，要求不要出现重复的排列)

- [递归逆序一个栈](#递归逆序一个栈)
    - 逆序一个栈，不能申请额外的数据结构，只能使用递归函数。 

### 暴力递归的优化

> 有重复调用同一个子问题的解，这种递归可以优化
>
> 如何分析有没有重复解：列出调用过程，可以只列出前几层，有没有重复解，一看便知
>

#### 记忆化搜索

- [面值数组组成面值的方法数-张数不限](#面值数组组成面值的方法数-张数不限)

    - arr是面值数组，其中的值都是正数且没有重复。再给定一个正数aim。
        每个值都认为是一种面值，且认为张数是无限的，返回组成aim的方法数。
        例如：arr = {1,2}，aim = 4 

        方法如下：1+1+1+1、1+1+2、2+2
        一共就3种方法，所以返回3

## 暴力递归和动态规划的关系

- 某一个暴力递归，有解的重复调用，就可以把这个暴力递归优化成动态规划
- 任何动态规划问题，都一定对应着某一个有解的重复调用的暴力递归
- 但不是所有的暴力递归，都一定对应着动态规划

### 如何找到某个问题的动态规划方式

1. 设计暴力递归：[重要原则](#设计尝试过程的原则)+[4种常见尝试模型](#常见的尝试模型)
2. 分析有没有重复解：[套路解决](#暴力递归到动态规划的套路)
3. 用记忆化搜索 -> 用严格表结构实现动态规划：[套路解决](#暴力递归到动态规划的套路)
4. 看看能否继续优化：[套路解决](#暴力递归到动态规划的套路)

### 设计尝试过程的原则

1. 每一个可变参数的类型，一定不要比int类型更加复杂
2. 原则1可以违反，让类型突破到一维线性结构，那必须是单一可变参数
3. 如果发现原则1被违反，但不违反原则2，只需要做到记忆化搜索即可
4. 可变参数的个数，能少则少

### 常见的尝试模型

#### 1. 从左往右的尝试模型

- [把数字翻译成字符串](#把数字翻译成字符串)  [leetcode M](https://leetcode-cn.com/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/)
    - 规定1和A对应、2和B对应、3和C对应...那么一个数字字符串比如"111”就可以转化为:
        "AAA"、"KA"和"AK"
        给定一个只有数字字符组成的字符串str，返回有多少种转化结果 
- [背包能装下最多的价值](#背包能装下最多的价值)
    - 给定两个长度都为N的数组weights和values，weights[i]和values[i]分别代表 i号物品的重量和价值。
        给定一个正数bag，表示一个载重bag的袋子，装的物品不能超过这个重量。
        返回能装下最多的价值是多少? 

#### 2. 范围上的尝试模型

- [A,B玩家从左右两边拿纸牌,返回最后获胜者的分数](#A,B玩家从左右两边拿纸牌,返回最后获胜者的分数)

    - 给定一个整型数组arr，代表数值不同的纸牌排成一条线，玩家A和玩家B依次拿走每张纸牌， 

        规定玩家A先拿，玩家B后拿， 但是每个玩家每次只能拿走最左或最右的纸牌，玩家A和玩家B都绝顶聪明。

        请返回最后获胜者的分数。

- [N皇后问题](#N皇后问题)  [leetcode H](https://leetcode-cn.com/problems/n-queens/)

    - 在N\*N的棋盘上要摆N个皇后，要求任何两个皇后不同行、不同列， 也不在同一条斜线上

        给定一个整数n，返回n皇后的摆法有多少种。

        n=1，返回1，n=2或3，2皇后和3皇后问题无论怎么摆都不行，返回0；n=8，返回92。

#### 3. 多样本位置全对应的尝试模型（一个样本做行一个样本做列的对应模型）

#### 4. 寻找业务限制的尝试模型

### 暴力递归到动态规划的套路

1. 你已经有了一个不违反原则的暴力递归，而且的确存在解的重复调用
2. 找到哪些参数的变化会影响返回值，对每一个列出变化范围
3. 参数间的所有的组合数量，意味着表大小
4. 记忆化搜索的方法就是傻缓存，非常容易得到
5. 规定好严格表的大小，分析位置的依赖顺序，然后从基础填写到最终解
6. 对于有枚举行为的决策过程，进一步优化

### 动态规划的进一步优化

1. 空间压缩
2. 状态化简
3. 动态规划中的四边形不等式优化

## 相关题目题解

#### [打印n层汉诺塔从最左边移动到最右边的全部过程（非递归）](#常见的递归)

```java
// Hanoi.class
public static void hanoi1(int n) {
    leftToRight(n);
}

public static void leftToRight(int n) {
    if (n == 1) {
        System.out.println("Move 1 from left to right");
        return;
    }
    leftToMid(n - 1);
    System.out.println("Move " + n + " from left to right");
    midToRight(n - 1);
}

public static void leftToMid(int n) {
    if (n == 1) {
        System.out.println("Move 1 from left to mid");
        return;
    }
    leftToRight(n - 1);
    System.out.println("Move " + n + " from left to mid");
    rightToMid(n - 1);
}

public static void rightToMid(int n) {
    if (n == 1) {
        System.out.println("Move 1 from right to mid");
        return;
    }
    rightToLeft(n - 1);
    System.out.println("Move " + n + " from right to mid");
    leftToMid(n - 1);
}

public static void midToRight(int n) {
    if (n == 1) {
        System.out.println("Move 1 from mid to right");
        return;
    }
    midToLeft(n - 1);
    System.out.println("Move " + n + " from mid to right");
    leftToRight(n - 1);
}

public static void midToLeft(int n) {
    if (n == 1) {
        System.out.println("Move 1 from mid to left");
        return;
    }
    midToRight(n - 1);
    System.out.println("Move " + n + " from mid to left");
    rightToLeft(n - 1);
}

public static void rightToLeft(int n) {
    if (n == 1) {
        System.out.println("Move 1 from right to left");
        return;
    }
    rightToMid(n - 1);
    System.out.println("Move " + n + " from right to left");
    midToLeft(n - 1);
}
```

#### [打印n层汉诺塔从最左边移动到最右边的全部过程（递归）](#常见的递归)

```java
// Hanoi.class
public static void hanoi2(int n) {
    if (n > 0) {
        func(n, "left", "right", "mid");
    }
}

// 1~i 圆盘 目标是from -> to， other是另外一个
public static void func(int N, String from, String to, String other) {
    if (N == 1) { // base
        System.out.println("Move 1 from " + from + " to " + to);
    } else {
        func(N - 1, from, other, to);
        System.out.println("Move " + N + " from " + from + " to " + to);
        func(N - 1, other, to, from);
    }
}
```

#### [打印一个字符串的全部子序列](#常见的递归)

```java
// PrintAllSubsquences.class
public static List<String> subs(String s) {
    char[] str = s.toCharArray();
    String path = "";
    List<String> ans = new ArrayList<>();
    process1(str, 0, ans, path);
    return ans;
}

public static void process1(char[] str, int index, List<String> ans, String path) {
    if (index == str.length) {
        ans.add(path);
        return;
    }
    String no = path;
    process1(str, index + 1, ans, no);
    String yes = path + String.valueOf(str[index]);
    process1(str, index + 1, ans, yes);
}
```

#### [打印一个字符串的全部子序列，要求不要出现重复字面值的子序列](#常见的递归)

```java
// PrintAllSubsquences.class
public static List<String> subsNoRepeat(String s) {
    char[] str = s.toCharArray();
    String path = "";
    HashSet<String> set = new HashSet<>();
    process2(str, 0, set, path);
    List<String> ans = new ArrayList<>();
    for (String cur : set) {
        ans.add(cur);
    }
    return ans;
}

public static void process2(char[] str, int index, HashSet<String> set, String path) {
    if (index == str.length) {
        set.add(path);
        return;
    }
    String no = path;
    process2(str, index + 1, set, no);
    String yes = path + String.valueOf(str[index]);
    process2(str, index + 1, set, yes);
}
```

#### [打印一个字符串的全部排列](#常见的递归)

```java
// PrintAllPermutations.class
public static ArrayList<String> permutation(String str) {
    ArrayList<String> res = new ArrayList<>();
    if (str == null || str.length() == 0) {
        return res;
    }
    char[] chs = str.toCharArray();
    process(chs, 0, res);
    return res;
}

public static void process(char[] str, int i, ArrayList<String> res) {
    if (i == str.length) {
        res.add(String.valueOf(str));
    }
    for (int j = i; j < str.length; j++) {
        swap(str, i, j);
        process(str, i + 1, res);
        swap(str, i, j);
    }
}
```

#### [打印一个字符串的全部排列，要求不要出现重复的排列](#常见的递归)

```java
// PrintAllPermutations.class
public static ArrayList<String> permutationNoRepeat(String str) {
    ArrayList<String> res = new ArrayList<>();
    if (str == null || str.length() == 0) {
        return res;
    }
    char[] chs = str.toCharArray();
    process2(chs, 0, res);
    return res;
}

public static void process2(char[] str, int i, ArrayList<String> res) {
    if (i == str.length) {
        res.add(String.valueOf(str));
    }
    boolean[] visit = new boolean[26]; // visit[0 1 .. 25]
    for (int j = i; j < str.length; j++) {
        if (!visit[str[j] - 'a']) {
            visit[str[j] - 'a'] = true;
            swap(str, i, j);
            process2(str, i + 1, res);
            swap(str, i, j);
        }
    }
}
```

#### [递归逆序一个栈](#常见的递归)

```java
// ReverseStackUsingRecursive.class
public static void reverse(Stack<Integer> stack) {
    if (stack.isEmpty()) {
        return;
    }
    int i = f(stack);
    reverse(stack);
    stack.push(i);
}

public static int f(Stack<Integer> stack) {
    int result = stack.pop();
    if (stack.isEmpty()) {
        return result;
    } else {
        int last = f(stack);
        stack.push(result);
        return last;
    }
}
```

#### [把数字翻译成字符串](#1. 从左往右的尝试模型)

```java
// ConvertToLetterString.class
public static int number(String str) {
    if (str == null || str.length() == 0) {
        return 0;
    }
    return process(str.toCharArray(), 0);
}

// i之前的位置，如何转化已经做过决定了, 不用再关心
// i... 有多少种转化的结果
public static int process(char[] str, int i) {
    if (i == str.length) { // base case
        return 1;
    }
    // i没有到终止位置
    if (str[i] == '0') {
        return 0;
    }
    // str[i]字符不是‘0’
    if (str[i] == '1') {
        int res = process(str, i + 1); // i自己作为单独的部分，后续有多少种方法
        if (i + 1 < str.length) {
            res += process(str, i + 2); // (i和i+1)作为单独的部分，后续有多少种方法
        }
        return res;
    }
    if (str[i] == '2') {
        int res = process(str, i + 1); // i自己作为单独的部分，后续有多少种方法
        // (i和i+1)作为单独的部分并且没有超过26，后续有多少种方法
        if (i + 1 < str.length && (str[i + 1] >= '0' && str[i + 1] <= '6')) {
            res += process(str, i + 2); // (i和i+1)作为单独的部分，后续有多少种方法
        }
        return res;
    }
    // str[i] == '3' ~ '9'
    return process(str, i + 1);
}
```

#### [把数字翻译成字符串-递归改动态规划](#1. 从左往右的尝试模型)

```java
// ConvertToLetterString.class
public static int dpWays(String s) {
    if (s == null || s.length() == 0) {
        return 0;
    }
    char[] str = s.toCharArray();
    int N = str.length;
    int[] dp = new int[N + 1];
    dp[N] = 1;
    for (int i = N - 1; i >= 0; i--) {
        if (str[i] == '0') {
            dp[i] = 0;
        } else if (str[i] == '1') {
            dp[i] = dp[i + 1];
            if (i + 1 < N) {
                dp[i] += dp[i + 2];
            }
        } else if (str[i] == '2') {
            dp[i] = dp[i + 1]; 
            if (i + 1 < str.length && (str[i + 1] >= '0' && str[i + 1] <= '6')) {
                dp[i] += dp[i + 2];
            }
        } else {
            dp[i] = dp[i + 1];
        }
    }
    return dp[0];
}
```

#### [背包能装下最多的价值](#1. 从左往右的尝试模型)

```java
// Knapsack.class
public static int getMaxValue(int[] w, int[] v, int bag) {
    return process(w, v, 0, 0, bag);
}

// index... 最大价值
public static int process(int[] w, int[] v, int index, int alreadyW, int bag) {
    if (alreadyW > bag) {
        return -1;
    }
    // 重量没超
    if (index == w.length) {
        return 0;
    }
    int p1 = process(w, v, index + 1, alreadyW, bag);
    int p2next = process(w, v, index + 1, alreadyW + w[index], bag);
    int p2 = -1;
    if (p2next != -1) {
        p2 = v[index] + p2next;
    }
    return Math.max(p1, p2);

}
```

#### [背包能装下最多的价值-递归改动态规划](#1. 从左往右的尝试模型)

```java
// Knapsack.class
public static int dpWay(int[] w, int[] v, int bag) {
    int N = w.length;
    int[][] dp = new int[N + 1][bag + 1];
    for (int index = N - 1; index >= 0; index--) {
        for (int rest = 1; rest <= bag; rest++) {
            dp[index][rest] = dp[index + 1][rest];
            if (rest >= w[index]) {
                dp[index][rest] = Math.max(dp[index][rest], v[index] + dp[index + 1][rest - w[index]]);
            }
        }
    }
    return dp[0][bag];
}
```

#### [A,B玩家从左右两边拿纸牌,返回最后获胜者的分数](#2. 范围上的尝试模型)

```java
// CardsInLine.class
public static int win1(int[] arr) {
    if (arr == null || arr.length == 0) {
        return 0;
    }
    return Math.max(f(arr, 0, arr.length - 1), s(arr, 0, arr.length - 1));
}

public static int f(int[] arr, int i, int j) {
    if (i == j) {
        return arr[i];
    }
    return Math.max(arr[i] + s(arr, i + 1, j), arr[j] + s(arr, i, j - 1));
}

public static int s(int[] arr, int i, int j) {
    if (i == j) {
        return 0;
    }
    return Math.min(f(arr, i + 1, j), f(arr, i, j - 1));
}
```

#### [A,B玩家从左右两边拿纸牌,返回最后获胜者的分数-递归改动态规划](#2. 范围上的尝试模型)

```java
// CardsInLine.class
public static int win2(int[] arr) {
    if (arr == null || arr.length == 0) {
        return 0;
    }
    int[][] f = new int[arr.length][arr.length];
    int[][] s = new int[arr.length][arr.length];
    for (int j = 0; j < arr.length; j++) {
        f[j][j] = arr[j];
        for (int i = j - 1; i >= 0; i--) {
            f[i][j] = Math.max(arr[i] + s[i + 1][j], arr[j] + s[i][j - 1]);
            s[i][j] = Math.min(f[i + 1][j], f[i][j - 1]);
        }
    }
    return Math.max(f[0][arr.length - 1], s[0][arr.length - 1]);
}
```

#### [N皇后问题](#2. 范围上的尝试模型)

```java
// NQueens.class
public static int num1(int n) {
    if (n < 1) {
        return 0;
    }
    // record[0] ?  record[1]  ?  record[2]
    int[] record = new int[n]; // record[i] -> i行的皇后，放在了第几列
    return process1(0, record, n);
}

// 潜台词：record[0..i-1]的皇后，任何两个皇后一定都不共行、不共列，不共斜线
// 目前来到了第i行
// record[0..i-1]表示之前的行，放了的皇后位置
// n代表整体一共有多少行
// 返回值是，摆完所有的皇后，合理的摆法有多少种
public static int process1(int i, int[] record, int n) {
    if (i == n) { // 终止行
        return 1;
    }
    int res = 0;
    for (int j = 0; j < n; j++) { // 当前行在i行，尝试i行所有的列  -> j
        // 当前i行的皇后，放在j列，会不会和之前(0..i-1)的皇后，不共行共列或者共斜线，
        // 如果是，认为有效
        // 如果不是，认为无效
        if (isValid(record, i, j)) {
            record[i] = j;
            res += process1(i + 1, record, n);
        }
    }
    return res;
}

// record[0..i-1]你需要看，record[i...]不需要看
// 返回i行皇后，放在了j列，是否有效
public static boolean isValid(int[] record, int i, int j) {
    for (int k = 0; k < i; k++) { // 之前的某个k行的皇后
        if (j == record[k] || Math.abs(record[k] - j) == Math.abs(i - k)) {
            return false;
        }
    }
    return true;
}
```

#### [N皇后问题-递归改动态规划](#2. 范围上的尝试模型)

```java
// NQueens.class
// 请不要超过32皇后问题
public static int num2(int n) {
    if (n < 1 || n > 32) {
        return 0;
    }
    int limit = n == 32 ? -1 : (1 << n) - 1;
    return process2(limit, 0, 0, 0);
}

// colLim 列的限制，1的位置不能放皇后，0的位置可以
// leftDiaLim 左斜线的限制，1的位置不能放皇后，0的位置可以
// rightDiaLim 右斜线的限制，1的位置不能放皇后，0的位置可以
public static int process2(int limit, 
                           int colLim, 
                           int leftDiaLim,
                           int rightDiaLim) {
    if (colLim == limit) { // base case
        return 1;
    }
    // 所有候选皇后的位置，都在pos上
    int pos = limit & (~(colLim | leftDiaLim | rightDiaLim));
    int mostRightOne = 0;
    int res = 0;
    while (pos != 0) {
        mostRightOne = pos & (~pos + 1);
        pos = pos - mostRightOne;
        res += process2(limit, 
                        colLim | mostRightOne,
                        (leftDiaLim | mostRightOne) << 1,
                        (rightDiaLim | mostRightOne) >>> 1);
    }
    return res;
}
```

#### [面值数组组成面值的方法数-张数不限](#记忆化搜索)

```java
// CoinsWay.class
// arr中都是正数且无重复值，返回组成aim的方法数
public static int ways(int[] arr, int aim) {
    if (arr == null || arr.length == 0 || aim < 0) {
        return 0;
    }
    return process(arr, 0, aim);
}

// 如果自由使用arr[index...]的面值，组成rest这么多钱，返回方法数 （1 , 6）
public static int process(int[] arr, int index, int rest) {
    if (index == arr.length) { // 无面值的时候
        return rest == 0 ? 1 : 0;
    }
    // 有面值的时候
    int ways = 0;
    // arr[index] 当钱面值
    for (int zhang = 0; zhang * arr[index] <= rest; zhang++) {
        ways += process(arr, index + 1, rest - zhang * arr[index]);
    }
    return ways;
}
```

#### [面值数组组成面值的方法数-张数不限-递归改动态规划](#记忆化搜索)

```java
// CoinsWay.class
public static int waysdp(int[] arr, int aim) {
    if (arr == null || arr.length == 0 || aim < 0) {
        return 0;
    }
    int N = arr.length;
    int[][] dp = new int[N + 1][aim + 1];
    dp[N][0] = 1;
    for (int i = N - 1; i >= 0; i--) { // 大顺序 从下往上
        for (int rest = 0; rest <= aim; rest++) {
            dp[i][rest] = dp[i + 1][rest];
            if (rest - arr[i] >= 0) {
                dp[i][rest] += dp[i][rest - arr[i]];
            }
        }
    }
    return dp[0][aim];
}
```

