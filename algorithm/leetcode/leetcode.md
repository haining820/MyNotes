---
title: leetcode记录
date: 2022-08-14
tags: [算法]
toc: true
---

# 字符串类

<!--more-->

## 8、[字符串转换整数 (atoi)](https://leetcode-cn.com/problems/string-to-integer-atoi/)

> 请你来实现一个 myAtoi(string s) 函数，使其能将字符串转换成一个 32 位有符号整数（类似 C/C++ 中的 atoi 函数）。
>
> 函数 myAtoi(string s) 的算法如下：
>
> 读入字符串并丢弃无用的前导空格
> 检查下一个字符（假设还未到字符末尾）为正还是负号，读取该字符（如果有）。 确定最终结果是负数还是正数。 如果两者都不存在，则假定结果为正。
> 读入下一个字符，直到到达下一个非数字字符或到达输入的结尾。字符串的其余部分将被忽略。
> 将前面步骤读入的这些数字转换为整数（即，"123" -> 123， "0032" -> 32）。如果没有读入数字，则整数为 0 。必要时更改符号（从步骤 2 开始）。
> 如果整数数超过 32 位有符号整数范围 [−231,  231 − 1] ，需要截断这个整数，使其保持在这个范围内。具体来说，小于 −231 的整数应该被固定为 −231 ，大于 231 − 1 的整数应该被固定为 231 − 1 。
> 返回整数作为最终结果。
> 注意：
>
> 本题中的空白字符只包括空格字符 ' ' 。
> 除前导空格或数字后的其余字符串外，请勿忽略 任何其他字符。

这道题没有用到什么算法，需要注意的是边界情况的处理。

- 第10行的如果将前导空格处理完毕后可能会到达数组末端。
- 在判断数字累加和时可能会超出 int 的最大值，所以将乘10和加上当前为数字的操作都移动到了等号右边进行。

用到的比较少用的方法如下。

```java
String s;
char[] chars = s.toCharArray();	// 将字符串转化为字符数组
Character.isDigit(char ch);		// 判断指定字符是不是数字
```

思路：从到到尾遍历字符数组，进行判断运算。

```java
class Solution {
    public int myAtoi(String s) {
        char[] chars = s.toCharArray();
        int len = chars.length;
        int pos = 0;
        boolean negative = false;
        while (pos < len && chars[pos] == ' ') {
            pos++;
        }	// 处理前导空格
        if(pos == len) {
            return 0;
        }	// 前导空格处理完毕后如果pos走到字符串尽头直接返回0
        if (chars[pos] == '-') {
            negative = true;
            pos++;
        } else if (chars[pos] == '+') {
            negative = false;
            pos++;
        } else if (!Character.isDigit(chars[pos])) {
            return 0;
        }	// 判断正负号与其他非数字字符
        int res = 0;
        while (pos < len && Character.isDigit(chars[pos])) {
            int digit = chars[pos] - '0';
            if (res > (Integer.MAX_VALUE - digit) / 10) {
                // *边界处理，比较巧妙的操作，将运算的元素都移动到等号右边
                return negative ? Integer.MIN_VALUE : Integer.MAX_VALUE;
            }
            res = 10 * res + digit;
            pos++;
        }
        return negative ? -res : res;	// 使用三目运算符来判断最终结果
    }
}
```



# 动态规划

## 300、[最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)

>给你一个整数数组 `nums` ，找到其中最长严格递增子序列的长度。
>
>子序列是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。
>
>例如，`[3,6,2,7]` 是数组 `[0,3,1,6,2,2,7]` 的子序列。

思路：如果 `nums[i]>nums[j]`，那么 `dp[i] = Math.max(dp[i],dp[j]+1)`。时间复杂度 **O(n^2)**，计算状态 dp[i] 时，需要 O(n)的时间遍历 dp[0…i−1] 的所有状态。空间复杂度 **O(n)**，需要额外使用长度为 n 的 dp 数组。

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        int n = nums.length;
        int[] dp = new int[n];
        int res = 1;
        Arrays.fill(dp,1);
        for(int i=1;i<n;i++){
            for(int j=0;j<i;j++){
                if(nums[j]<nums[i]){
                    dp[i]=Math.max(dp[i],dp[j]+1);
                }
            }
            res = Math.max(res,dp[i]);
        }
        return res;
    }
}
```

## 221、[最大正方形](https://leetcode-cn.com/problems/maximal-square/)

> 在一个由 `'0'` 和 `'1'` 组成的二维矩阵内，找到只包含 `'1'` 的最大正方形，并返回其面积。
>
> 示例 1：
>
> ![img](leetcode/max1grid.jpg)
>
> ```
> 输入：matrix = [["1","0","1","0","0"],["1","0","1","1","1"],["1","1","1","1","1"],["1","0","0","1","0"]]
> 输出：4
> ```
>
> **示例 2：**
>
> ![img](leetcode/max2grid.jpg)
>
> ```
> 输入：matrix = [["0","1"],["1","0"]]
> 输出：1
> ```
>
> **示例 3：**
>
> ```
> 输入：matrix = [["0"]]
> 输出：0
> ```

【方法一】暴力解法

**复杂度分析**

时间复杂度：O(mnmin(m,n)^2)

- 需要遍历整个矩阵寻找每个 1，遍历矩阵的时间复杂度是 O(mn)。
- 对于每个可能的正方形，其边长不超过 m 和 n 中的最小值，需要遍历该正方形中的每个元素判断是不是只包含 1，遍历正方形时间复杂度是O(min(m,n)^2)。


空间复杂度：O(1)

```java
class Solution {
    public int maximalSquare(char[][] matrix) {
        int maxSide = 0;
        int row = matrix.length;	// 行数
        int col = matrix[0].length;	// 列数
        if(row == 0 || col == 0) {
            return maxSide;
        }	// 处理特殊情况
        for(int i = 0;i < row;i++) {
            for(int j = 0;j < col;j++) {
                if(matrix[i][j] == '1') {	// 搜索整个矩阵，如果有1的格子就以这个格子为左上角继续向下搜索
                    maxSide = Math.max(maxSide,1);
                    // 假设剩下所有的格子都为1的情况下最大的继续向下走的步数
                    int curMaxStep = Math.min(row - i,col - j);
                    // 对该步数进行遍历，每遍历一次就是在左上角格子外边新增一圈
                    for(int k = 1;k < curMaxStep;k++) {
                        boolean flag = true;
                        if(matrix[i + k][j + k] == '0') {
                            break;	// 先判断对角线上的端点是不是符合要求
                        }
                        for(int p = 0;p < k;p++) {
                            if(matrix[i + p][j + k] == '0' || matrix[i + k][j + p] == '0') {
                                flag = false;
                                break;	// 继续判断外圈上的除了端点的两条边是否符合要求
                            }
                        }
                        if(flag) {	// 如果符合要求则更新最大边值
                            maxSide = Math.max(k + 1,maxSide);
                        } else {
                            break;	// 不符合要求立即结束对当前左上角的搜索，搜索其他的格子作为新的左上角
                        }
                    }
                }
            }
        }
        return maxSide * maxSide;
    }
}
```

【方法二】动态规划

状态转移公式：`dp(i,j) = min(dp(i−1,j),dp(i−1,j−1),dp(i,j−1))+1`

**复杂度分析**

时间复杂度：O(mn)，其中 m 和 n 是矩阵的行数和列数。需要遍历原始矩阵中的每个元素计算 dp 的值。

空间复杂度：O(mn)，其中 m 和 n 是矩阵的行数和列数。创建了一个和原始矩阵大小相同的矩阵 dp 。

```java
class Solution {
    public int myMin (int a,int b,int c) {
        return a<b?a<c?a:c:b<c?b:c;
    }
    public int maximalSquare(char[][] matrix) {
        int row = matrix.length;
        int col = matrix[0].length;
        if(row == 0 || col == 0) {
            return 0;
        }
        // 首先明确dp数组的含义，dp[i][j]代表以该点为右下角的最大正方形边长
        int res = 0;
        int[][] dp = new int[row][col];
        // 在遍历过程中只需根据状态转移公式对矩阵进行判断即可
        // dp(i,j) = min(dp(i−1,j),dp(i−1,j−1),dp(i,j−1))+1
        for(int i = 0;i < row;i++) {
            for(int j = 0;j < col;j++) {
                if(matrix[i][j] == '1') {
                    if(i == 0 || j == 0) {
                        dp[i][j] = 1;
                    } else {
                        dp[i][j] = myMin(dp[i - 1][j],dp[i][j - 1],dp[i - 1][j - 1]) + 1;
                    }
                    res = Math.max(dp[i][j],res);
                }
            }
        }
        return res * res;
    }
}
```

#### 322、[零钱兑换](https://leetcode-cn.com/problems/coin-change/)

> 给你一个整数数组 coins ，表示不同面额的硬币；以及一个整数 amount ，表示总金额。
>
> 计算并返回可以凑成总金额所需的 最少的硬币个数 。如果没有任何一种硬币组合能组成总金额，返回 -1 。
>
> 你可以认为每种硬币的数量是无限的。
>
> 示例 1：
>
> ```
> 输入：coins = [1, 2, 5], amount = 11
> 输出：3 
> 解释：11 = 5 + 5 + 1
> ```
>
> 示例 2：
>
> ```
> 输入：coins = [2], amount = 3
> 输出：-1
> ```
>
> 示例 3：
>
> ```
> 输入：coins = [1], amount = 0
> 输出：0
> ```
>
> 示例 4：
>
> ```
> 输入：coins = [1], amount = 1
> 输出：1
> ```
>
> 示例 5：
>
> ```
> 输入：coins = [1], amount = 2
> 输出：2
> ```
>
>
> 提示：
>
> ```
> 1 <= coins.length <= 12
> 1 <= coins[i] <= 231 - 1
> 0 <= amount <= 104
> ```

【方法一】动态规划

状态转移公式：`dp[i] = Math.min(dp[i], dp[i - coins[j]] + 1)`

```java
class Solution {
    public int coinChange(int[] coins, int amount) {
        int[] dp = new int[amount + 1];
        Arrays.fill(dp,amount + 1);
        int len = coins.length;
        dp[0] = 0;
        for(int i = 1;i < amount + 1;i++) {
            for(int j = 0;j < len;j++) {
                if(i - coins[j] >= 0) {
                    dp[i] = Math.min(dp[i],dp[i - coins[j]] + 1);
                }
            }
        }
        return dp[amount] > amount?-1:dp[amount];
    }
} 
```



