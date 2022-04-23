---
title: 数据结构与算法---线段树
catalog: true
abbrlink: f82e
date: 2022-04-10 15:35:07
subtitle:
header-img: /img/header_img/post-bg-desk.jpg
top: 
tags:
    - algorithm
    - data structure
categories:
    - work 
---

# 数据结构与算法---线段树

## 什么是线段树：

一种支持范围整体修改和范围整体查询的数据结构

## 解决问题的范畴：

大范围信息可以只由左、右两侧信息加工出，而不必遍历左右两个子范围的具体状况

## 线段树实例：

### 实例一：SegmentTree

#### 题目描述:

​	给定一个数组arr，用户希望你实现如下三个方法
​	1）void add(int L, int R, int V) :  让数组arr[L…R]上每个数都加上V
​	2）void update(int L, int R, int V) :  让数组arr[L…R]上每个数都变成V
​	3）int sum(int L, int R) :让返回arr[L…R]这个范围整体的累加和
​	怎么让这三个方法，时间复杂度都是O(logN)

#### 思路：

1. 将原始数组`origin`第`0`位弃用（为了使用位运算），从`1`开始，放入数组`arr`中

   ```java
   int MAXN = origin.length + 1;
   arr = new int[MAXN]; // arr[0] 不用  从1开始使用
   for (int i = 1; i < MAXN; i++) {
   	arr[i] = origin[i - 1];
   }
   ```

2. 计算累加和。将数组`arr`完全二分，每一段区间的和放入数组`sum`中，`sum`为满二叉树结构，长度为`4*MAXN`（`arr`长度满足`2^n`时，满二叉树长度为`2*MAXN`，不满足时需要补满）

   例如：`arr[x, 3, 2, 5, 7]，sum[x, 17, 5, 12, 3, 2, 5, 7]` 

   ​															`arr(1-4)`的和放入`sum[1],` 

   ​										`arr(1-2)`的和放入`sum[2]`，`arr(3-4)`的和放入`sum[3]`，

   `arr(1-1)`的和放入`sum[4]`，`arr(2-2)`的和放入`sum[5]`，`arr(3-3)`的和放入`sum[6]`，`arr(4-4)`的和放入`sum[7]`

   ```java
   int[] sum = new int[MAXN << 2];
   
   // 递归填充某一区间的累加和信息
   // 在arr[l~r]范围上，去build，1~N，(l: 1, r: origin.length)
   // rt :  这个范围在sum中的下标（root）
   public void build(int l, int r, int rt) {
     if (l == r) {
       sum[rt] = arr[l];
       return;
     }
     int mid = (l + r) >> 1;
     build(l, mid, rt << 1); // 左孩子 2*rt
     build(mid + 1, r, rt << 1 | 1); // 右孩子 2*rt + 1
     pushUp(rt);
   }
   
   /**
    * 当前节点值=左孩子值+右孩子值
    * @param rt
    */
   private void pushUp(int rt) {
     sum[rt] = sum[rt << 1] + sum[rt << 1 | 1];
   }
   ```

3. 懒增加。实现在`l～r`范围内所有的元素都加`C`，引入`int[] lazy`数组， 标记每个节点需要下发的增加数量，当`l~r`范围覆盖了当前表达的范围时直接在当前层累加，当覆盖不了时，往下级下发攒下的lazy add任务，根据任务范围决定左右孩子是否都需要下发，下发完毕后汇总父节点`sum`值。下发时将左右子树的`lazy`值累加上父节点值(`lazy[rt << 1] += lazy[rt];`），`sum`值累加上子节点个数*当前lazy值(`sum[rt << 1] += lazy[rt] * ln;`)，并将父节点`lazy`值置为`0`。

   ```java
   // L..R -> 任务范围 ,所有的值累加上C
   // l,r -> 表达的范围(l: 1, r: origin.length)
   // rt  去哪找l，r范围上的信息（root）
   public void add(int L, int R, int C,
   	int l, int r, 
   	int rt) {
     // 任务的范围彻底覆盖了，当前表达的范围
     if (L <= l && r <= R) {
       sum[rt] += C * (r - l + 1);
       lazy[rt] += C;
       return;
     }
     // 要把任务往下发
     // 任务  L, R  没有把本身表达范围 l,r 彻底包住
     int mid = (l + r) >> 1;
     // 下发之前所有的懒任务
     pushDown(rt, mid - l + 1, r - mid);
     // 左孩子是否需要接到任务
     if (L <= mid) {
     	add(L, R, C, l, mid, rt << 1); // 递归增加左孩子
     }
     // 右孩子是否需要接到任务
     if (R > mid) {
     	add(L, R, C, mid + 1, r, rt << 1 | 1); // 递归增加右孩子
     }
     // 左右孩子做完任务后，我更新我的sum信息
     pushUp(rt); // 见2.
   }
   
   // 之前的所有懒增加，从父范围发给左右两个子范围
   // ln表示左子树元素结点个数，rn表示右子树结点个数
   private void pushDown(int rt, int ln, int rn) {
     if (lazy[rt] != 0) {
       lazy[rt << 1] += lazy[rt];
       sum[rt << 1] += lazy[rt] * ln;
       lazy[rt << 1 | 1] += lazy[rt];
       sum[rt << 1 | 1] += lazy[rt] * rn;
       lazy[rt] = 0;
     }
   }
   ```

4. 懒更新。实现在`l～r`范围内所有的元素更新为`C`，引入`int[] change`和`boolean[] update`数组，配合使用标记是否需要更新及更新的值。当`l~r`范围覆盖了当前表达的范围时直接在当前层更新，当覆盖不了时，往下级下发攒下的lazy update任务，根据任务范围决定左右孩子是否都需要下发，下发完毕后汇总父节点`sum`值。任务下发时，将左右子树的update值置为父节点值(`update[rt << 1] = true;change[rt << 1] = change[rt];`），`sum`值置为子节点个数*当前`update`值(`sum[rt << 1] = change[rt] * ln;`)，并将左右子节点`lazy`值置为0，父节点`update`置为`false`。

   ```java
   // L..R -> 任务范围 ,所有的值累加上C
   // l,r -> 表达的范围(l: 1, r: origin.length)
   // rt  去哪找l，r范围上的信息（root）
   public void update(int L, int R, int C, int l, int r, int rt) {
     if (L <= l && r <= R) {
       update[rt] = true;
       change[rt] = C;
       sum[rt] = C * (r - l + 1);
       lazy[rt] = 0;
       return;
     }
     // 当前任务躲不掉，无法懒更新，要往下发
     int mid = (l + r) >> 1;
     pushDown(rt, mid - l + 1, r - mid);
     if (L <= mid) {
     	update(L, R, C, l, mid, rt << 1);
     }
     if (R > mid) {
     	update(L, R, C, mid + 1, r, rt << 1 | 1);
     }
     pushUp(rt);
   }
   
   // 之前的所有懒更新，从父范围发给左右两个子范围
   // ln表示左子树元素结点个数，rn表示右子树结点个数
   private void pushDown(int rt, int ln, int rn) {
     if (update[rt]) {
       update[rt << 1] = true;
       update[rt << 1 | 1] = true;
       change[rt << 1] = change[rt];
       change[rt << 1 | 1] = change[rt];
       lazy[rt << 1] = 0;
       lazy[rt << 1 | 1] = 0;
       sum[rt << 1] = change[rt] * ln;
       sum[rt << 1 | 1] = change[rt] * rn;
       update[rt] = false;
     }
   }
   ```

5. 任务下发时先执行懒更新任务，再执行懒增加。因为懒更新任务直接将节点值修改为更新值，如果后执行懒更新，更新操作后可能存在积攒的增加任务，无法正常执行。

   ```java
   // 之前的所有懒增加和懒更新，从父范围发给左右两个子范围
   // ln表示左子树元素结点个数，rn表示右子树结点个数
   private void pushDown(int rt, int ln, int rn) {
     if (update[rt]) {
       update[rt << 1] = true;
       update[rt << 1 | 1] = true;
       change[rt << 1] = change[rt];
       change[rt << 1 | 1] = change[rt];
       lazy[rt << 1] = 0;
       lazy[rt << 1 | 1] = 0;
       sum[rt << 1] = change[rt] * ln;
       sum[rt << 1 | 1] = change[rt] * rn;
       update[rt] = false;
     }
     if (lazy[rt] != 0) {
       lazy[rt << 1] += lazy[rt];
       sum[rt << 1] += lazy[rt] * ln;
       lazy[rt << 1 | 1] += lazy[rt];
       sum[rt << 1 | 1] += lazy[rt] * rn;
       lazy[rt] = 0;
     }
   }
   ```

6. 懒查询。返回在`l～r`范围内所有元素的累加和。当`l~r`范围覆盖了当前表达的范围时直接返回`sum`，当覆盖不了时，往下级下发攒下的lazy add及lazy update任务，根据任务范围决定左右孩子是否都需要下发，下发完毕后汇总父节点`sum`值。

   ```java
   // l,r -> 表达的范围(l: 1, r: origin.length)
   // rt  去哪找l，r范围上的信息（root）
   public long query(int L, int R, int l, int r, int rt) {
     if (L <= l && r <= R) {
       return sum[rt]; // 任务范围覆盖了可表达范围，直接返回当前sum
     }
     int mid = (l + r) >> 1;
     pushDown(rt, mid - l + 1, r - mid); // 执行懒更新及懒增加任务
     long ans = 0;
     if (L <= mid) {
       ans += query(L, R, l, mid, rt << 1);
     }
     if (R > mid) {
       ans += query(L, R, mid + 1, r, rt << 1 | 1);
     }
     return ans;
   }
   ```

#### 完整题解：

```java
public class Code01_SegmentTree {

	public static class SegmentTree {
		
		private int MAXN;
		private int[] arr; // arr[]为原序列的信息从0开始，但在arr里是从1开始的
		private int[] sum; // sum[]0
		private int[] lazy; // lazy[]为累加懒惰标记
		private int[] change; // change[]为更新的值
		private boolean[] update; // update[]为更新慵懒标记

		public SegmentTree(int[] origin) {
			MAXN = origin.length + 1;
			arr = new int[MAXN]; // arr[0] 不用  从1开始使用
			for (int i = 1; i < MAXN; i++) {
				arr[i] = origin[i - 1];
			}
			sum = new int[MAXN << 2]; // 用来支持脑补概念中，某一个范围的累加和信息
			lazy = new int[MAXN << 2]; // 用来支持脑补概念中，某一个范围沒有往下傳遞的纍加任務
			change = new int[MAXN << 2]; // 用来支持脑补概念中，某一个范围有没有更新操作的任务
			update = new boolean[MAXN << 2]; // 用来支持脑补概念中，某一个范围更新任务，更新成了什么
		}

		/**
		 * 当前节点值=左孩子值+右孩子值
		 * @param rt
		 */
		private void pushUp(int rt) {
			sum[rt] = sum[rt << 1] + sum[rt << 1 | 1];
		}

		// 之前的所有懒增加和懒更新，从父范围发给左右两个子范围
		// ln表示左子树元素结点个数，rn表示右子树结点个数
		private void pushDown(int rt, int ln, int rn) {
			if (update[rt]) {
				update[rt << 1] = true;
				update[rt << 1 | 1] = true;
				change[rt << 1] = change[rt];
				change[rt << 1 | 1] = change[rt];
				lazy[rt << 1] = 0;
				lazy[rt << 1 | 1] = 0;
				sum[rt << 1] = change[rt] * ln;
				sum[rt << 1 | 1] = change[rt] * rn;
				update[rt] = false;
			}
			if (lazy[rt] != 0) {
				lazy[rt << 1] += lazy[rt];
				sum[rt << 1] += lazy[rt] * ln;
				lazy[rt << 1 | 1] += lazy[rt];
				sum[rt << 1 | 1] += lazy[rt] * rn;
				lazy[rt] = 0;
			}
		}

		// 在初始化阶段，先把sum数组，填好。递归填充某一区间的累加和信息
		// 在arr[l~r]范围上，去build，1~N，(l: 1, r: origin.length)
		// rt :  这个范围在sum中的下标
		public void build(int l, int r, int rt) {
			if (l == r) {
				sum[rt] = arr[l];
				return;
			}
			int mid = (l + r) >> 1;
			build(l, mid, rt << 1); // 左孩子 2*rt
			build(mid + 1, r, rt << 1 | 1); // 右孩子 2*rt + 1
			pushUp(rt);
		}

		// L..R -> 任务范围 ,所有的值累加上C
		// l,r -> 表达的范围(l: 1, r: origin.length)
		// rt  去哪找l，r范围上的信息(root)
		public void add(int L, int R, int C,
						int l, int r,
						int rt) {
			// 任务的范围彻底覆盖了，当前表达的范围
			if (L <= l && r <= R) {
				sum[rt] += C * (r - l + 1);
				lazy[rt] += C;
				return;
			}
			// 要把任务往下发
			// 任务  L, R  没有把本身表达范围 l,r 彻底包住
			int mid = (l + r) >> 1;
			// 下发之前所有的懒任务
			pushDown(rt, mid - l + 1, r - mid);
			// 左孩子是否需要接到任务
			if (L <= mid) {
				add(L, R, C, l, mid, rt << 1);
			}
			// 右孩子是否需要接到任务
			if (R > mid) {
				add(L, R, C, mid + 1, r, rt << 1 | 1);
			}
			// 左右孩子做完任务后，我更新我的sum信息
			pushUp(rt);
		}

		// L..R -> 任务范围 ,所有的值更新为C
		// l,r -> 表达的范围(l: 1, r: origin.length)
		// rt  去哪找l，r范围上的信息(root)
		public void update(int L, int R, int C, int l, int r, int rt) {
			if (L <= l && r <= R) {
				update[rt] = true;
				change[rt] = C;
				sum[rt] = C * (r - l + 1);
				lazy[rt] = 0;
				return;
			}
			// 当前任务躲不掉，无法懒更新，要往下发
			int mid = (l + r) >> 1;
			pushDown(rt, mid - l + 1, r - mid);
			if (L <= mid) {
				update(L, R, C, l, mid, rt << 1);
			}
			if (R > mid) {
				update(L, R, C, mid + 1, r, rt << 1 | 1);
			}
			pushUp(rt);
		}
		
		// L..R -> 任务范围 ,查询范围上累加和
		// l,r -> 表达的范围(l: 1, r: origin.length)
		// rt 去哪找l，r范围上的信息(root)
		public long query(int L, int R, int l, int r, int rt) {
			if (L <= l && r <= R) {
				return sum[rt];
			}
			int mid = (l + r) >> 1;
			pushDown(rt, mid - l + 1, r - mid);
			long ans = 0;
			if (L <= mid) {
				ans += query(L, R, l, mid, rt << 1);
			}
			if (R > mid) {
				ans += query(L, R, mid + 1, r, rt << 1 | 1);
			}
			return ans;
		}

	}

	/**
	 * 暴力方法，做对数器
	 */
	public static class Right {
		public int[] arr;

		public Right(int[] origin) {
			arr = new int[origin.length + 1];
			for (int i = 0; i < origin.length; i++) {
				arr[i + 1] = origin[i];
			}
		}

		public void update(int L, int R, int C) {
			for (int i = L; i <= R; i++) {
				arr[i] = C;
			}
		}

		public void add(int L, int R, int C) {
			for (int i = L; i <= R; i++) {
				arr[i] += C;
			}
		}

		public long query(int L, int R) {
			long ans = 0;
			for (int i = L; i <= R; i++) {
				ans += arr[i];
			}
			return ans;
		}

	}

	public static int[] genarateRandomArray(int len, int max) {
		int size = (int) (Math.random() * len) + 1;
		int[] origin = new int[size];
		for (int i = 0; i < size; i++) {
			origin[i] = (int) (Math.random() * max) - (int) (Math.random() * max);
		}
		return origin;
	}

	/**
	 * 对数器
	 * @return
	 */
	public static boolean test() {
		int len = 100;
		int max = 1000;
		int testTimes = 5000;
		int addOrUpdateTimes = 1000;
		int queryTimes = 500;
		for (int i = 0; i < testTimes; i++) {
			int[] origin = genarateRandomArray(len, max);
			SegmentTree seg = new SegmentTree(origin);
			int S = 1;
			int N = origin.length;
			int root = 1;
			seg.build(S, N, root);
			Right rig = new Right(origin);
			for (int j = 0; j < addOrUpdateTimes; j++) {
				int num1 = (int) (Math.random() * N) + 1;
				int num2 = (int) (Math.random() * N) + 1;
				int L = Math.min(num1, num2);
				int R = Math.max(num1, num2);
				int C = (int) (Math.random() * max) - (int) (Math.random() * max);
				if (Math.random() < 0.5) {
					seg.add(L, R, C, S, N, root);
					rig.add(L, R, C);
				} else {
					seg.update(L, R, C, S, N, root);
					rig.update(L, R, C);
				}
			}
			for (int k = 0; k < queryTimes; k++) {
				int num1 = (int) (Math.random() * N) + 1;
				int num2 = (int) (Math.random() * N) + 1;
				int L = Math.min(num1, num2);
				int R = Math.max(num1, num2);
				long ans1 = seg.query(L, R, S, N, root);
				long ans2 = rig.query(L, R);
				if (ans1 != ans2) {
					return false;
				}
			}
		}
		return true;
	}

	public static void main(String[] args) {
		int[] origin = { 2, 1, 1, 2, 3, 4, 5 };
		SegmentTree seg = new SegmentTree(origin);
		int S = 1; // 整个区间的开始位置，规定从1开始，不从0开始 -> 固定
		int N = origin.length; // 整个区间的结束位置，规定能到N，不是N-1 -> 固定
		int root = 1; // 整棵树的头节点位置，规定是1，不是0 -> 固定
		int L = 2; // 操作区间的开始位置 -> 可变
		int R = 5; // 操作区间的结束位置 -> 可变
		int C = 4; // 要加的数字或者要更新的数字 -> 可变
		// 区间生成，必须在[S,N]整个范围上build
		seg.build(S, N, root);
		// 区间修改，可以改变L、R和C的值，其他值不可改变
		seg.add(L, R, C, S, N, root);
		// 区间更新，可以改变L、R和C的值，其他值不可改变
		seg.update(L, R, C, S, N, root);
		// 区间查询，可以改变L和R的值，其他值不可改变
		long sum = seg.query(L, R, S, N, root);
		System.out.println(sum);

		System.out.println("对数器测试开始...");
		System.out.println("测试结果 : " + (test() ? "通过" : "未通过"));
	}

}
```



### 实例二：Falling Squares

#### 题目描述：

​	[leetcode 699. Falling Squares](https://leetcode.com/problems/falling-squares/ ) 想象一下标准的俄罗斯方块游戏，X轴是积木最终下落到底的轴线。

下面是这个游戏的简化版：
	1）只会下落正方形积木
	2）[a,b] -> 代表一个边长为b的正方形积木，积木左边缘沿着X = a这条线从上方掉落
	3）认为整个X轴都可能接住积木，也就是说简化版游戏是没有整体的左右边界的
	4）没有整体的左右边界，所以简化版游戏不会消除积木，因为不会有哪一层被填满。

给定一个N*2的二维数组matrix，可以代表N个积木依次掉落，返回每一次掉落之后的最大高度。

#### 思路：

1. 将给定的二维数组转化为方块在X轴左右的边界(`[a, b]`从`x=a`位置掉落长为`b`的方块，转化为`[a, a+b-1]`表示方块在X轴的左右边界)
2. 离散化所有边界，从小到大放入`map`，将所有的边界从`1`开始编号
3. 后续步骤参考实例一思路，将累加和的计算修改为计算最大高度

#### 完整题解：

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.TreeSet;

public class Code02_FallingSquares {

	public static class SegmentTree {
		private int[] max;
		private int[] change;
		private boolean[] update;

		public SegmentTree(int size) {
			int N = size + 1;
			max = new int[N << 2];
			change = new int[N << 2];
			update = new boolean[N << 2];
		}

		private void pushUp(int rt) {
			max[rt] = Math.max(max[rt << 1], max[rt << 1 | 1]);
		}

		// ln表示左子树元素结点个数，rn表示右子树结点个数
		private void pushDown(int rt, int ln, int rn) {
			if (update[rt]) {
				update[rt << 1] = true;
				update[rt << 1 | 1] = true;
				change[rt << 1] = change[rt];
				change[rt << 1 | 1] = change[rt];
				max[rt << 1] = change[rt];
				max[rt << 1 | 1] = change[rt];
				update[rt] = false;
			}
		}

		public void update(int L, int R, int C, int l, int r, int rt) {
			if (L <= l && r <= R) {
				update[rt] = true;
				change[rt] = C;
				max[rt] = C;
				return;
			}
			int mid = (l + r) >> 1;
			pushDown(rt, mid - l + 1, r - mid);
			if (L <= mid) {
				update(L, R, C, l, mid, rt << 1);
			}
			if (R > mid) {
				update(L, R, C, mid + 1, r, rt << 1 | 1);
			}
			pushUp(rt);
		}

		public int query(int L, int R, int l, int r, int rt) {
			if (L <= l && r <= R) {
				return max[rt];
			}
			int mid = (l + r) >> 1;
			pushDown(rt, mid - l + 1, r - mid);
			int left = 0;
			int right = 0;
			if (L <= mid) {
				left = query(L, R, l, mid, rt << 1);
			}
			if (R > mid) {
				right = query(L, R, mid + 1, r, rt << 1 | 1);
			}
			return Math.max(left, right);
		}

	}

  // positions
  // 将给定的二维数组转化为方块在X轴左右的边界([a, b]从x=a位置掉落长为b的方块，转化为[a, a+b-1]表示方块在X轴的左右边界)
  // [2, 7] -> 2, 8
  // [3, 10] -> 3, 12
	public HashMap<Integer, Integer> index(int[][] positions) {
		TreeSet<Integer> pos = new TreeSet<>();
		for (int[] arr : positions) {
			pos.add(arr[0]);
			pos.add(arr[0] + arr[1] - 1);
		}
		HashMap<Integer, Integer> map = new HashMap<>();
		int count = 0;
		for (Integer index : pos) {
			map.put(index, ++count);
		}
		return map;
	}

	public List<Integer> fallingSquares(int[][] positions) {
		HashMap<Integer, Integer> map = index(positions);
    // 离散化所有边界，从小到大放入map，将所有的边界从1开始编号
    // 100 -> 1   306 -> 2   403 -> 3
    // [100, 403]   1 ~ 3
		int N = map.size(); // 1 ~ N
		SegmentTree segmentTree = new SegmentTree(N);
		int max = 0;
		List<Integer> res = new ArrayList<>();
    // 每落一个正方形，收集一下，所有方块组成的图形，最大高度是多少
		for (int[] arr : positions) {
			int L = map.get(arr[0]);
			int R = map.get(arr[0] + arr[1] - 1);
			int height = segmentTree.query(L, R, 1, N, 1) + arr[1];
			max = Math.max(max, height);
			res.add(max);
			segmentTree.update(L, R, height, 1, N, 1);
		}
		return res;
	}

}
```




