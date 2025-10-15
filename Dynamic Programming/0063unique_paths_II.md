# 63.不同路径 II

题目链接：[63.不同路径 II](https://leetcode.cn/problems/unique-paths-ii/)

## 题目大意

给定一个 `m x n` 的整数数组 `grid`，一个机器人初始位于 左上角（即 `grid[0][0]`）。机器人尝试移动到 右下角（即 `grid[m - 1][n - 1]`），机器人每次只能向下或者向右移动一步

网格中的障碍物和空位置分别用 `1` 和 `0` 来表示，机器人的移动路径中不能包含任何有障碍物的方格

返回机器人能够到达右下角的不同路径数量

测试用例保证答案小于等于 `2 * 10^9`

![alt text](https://github.com/donnapersonal/picx-images-hosting/raw/master/image.5q7kqfashp.png)

限制：
- m == obstacleGrid.length, n == obstacleGrid[i].length
- 1 <= m, n <= 100
- obstacleGrid[i][j] is 0 or 1.

## 解题

动态规划的题目分为两大类
- 一种是`求最优解类`，典型问题是背包问题
- 另一种就是计数类，如这里的统计方案数的问题，它们都存在一定的递推性质

> 前者的递推性质还有一个名字，叫做「最优子结构」 — 即当前问题的最优解取决于子问题的最优解；后者类似，当前问题的方案数取决于子问题的方案数。所以在遇到求方案数的问题时，可以往动态规划的方向考虑

这道题分解问题规模的关键是：到达 `(i, j)` 的路径条数等于到达 `(i-1, j)` 和到达 `(i, j-1)` 的路径条数之和

### 思路 1

状态定义：`dp[i][j]` 表示从 `(0,0)` 走到 `(i,j)` 的不同路径数

状态转移方程
- 遇到障碍物 (`grid[i][j] == 1`)：`dp[i][j] = 0`（无法通行）
- 否则，路径数等于来自上方和左方的路径数之和：`dp[i][j] = dp[i-1][j] + dp[i][j-1]`（若无障碍）

```js
var uniquePathsWithObstacles = function(obstacleGrid) {
    let m = obstacleGrid.length, n = obstacleGrid[0].length;
    if (obstacleGrid[0][0] === 1) return 0;
    let dp = Array.from({ length: m }, () => Array(n).fill(0));
    dp[0][0] = 1;
    for (let i = 1; i < m; i++) {
        dp[i][0] = obstacleGrid[i][0] === 1 ? 0 : dp[i-1][0];
    }

    for (let j = 1; j < n; j++) {
        dp[0][j] = obstacleGrid[0][j] === 1 ? 0 : dp[0][j-1];
    }

    for (let i = 1; i < m; i++) {
        for (let j = 1; j < n; j++) {
            if (obstacleGrid[i][j] === 1) {
                dp[i][j] = 0;
            } else {
                dp[i][j] = dp[i-1][j] + dp[i][j-1];
            }
        }
    }

    return dp[m-1][n-1]; 
};
```
```python
# 写法 1
class Solution:
    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        m, n = len(obstacleGrid), len(obstacleGrid[0])
        if obstacleGrid[0][0] == 1:
            return 0
        
        dp = [[0] * n for _ in range(m)]
        dp[0][0] = 1

        for i in range(1, m):
            dp[i][0] = 0 if obstacleGrid[i][0] == 1 else dp[i-1][0]
        
        for j in range(1, n):
            dp[0][j] = 0 if obstacleGrid[0][j] == 1 else dp[0][j-1]
        
        for i in range(1, m):
            for j in range(1, n):
                if obstacleGrid[i][j] == 1:
                    dp[i][j] = 0
                else:
                    dp[i][j] = dp[i-1][j] + dp[i][j-1]
            
        return dp[m-1][n-1]  

# 写法 2
class Solution:
    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        m, n = len(obstacleGrid), len(obstacleGrid[0])
        # 数组索引偏移一位，dp[0][..] dp[..][0] 代表 obstacleGrid 之外
        # 定义：到达 obstacleGrid[i][j] 的路径条数为 dp[i-1][j-1]
        dp = [[0] * (n + 1) for _ in range(m + 1)]
        # base case：如果没有障碍物，起点到起点的路径条数就是 1
        dp[1][1] = 0 if obstacleGrid[0][0] == 1 else 1

        for i in range(1, m + 1):
            for j in range(1, n + 1):
                # 跳过 dp[1][1] 的计算，因为 dp[1][1] 已经在初始化时赋值
                if i == 1 and j == 1:
                    # 跳过 base case
                    continue
                
                if obstacleGrid[i - 1][j - 1] == 1:
                    # 跳过障碍物的格子
                    continue
                
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1]
        
        # 返回到达 obstacleGrid[m-1][n-1] 的路径数量
        return dp[m][n]
```

- 时间复杂度：`O(mn)`，其中 `m` 为网格的行数，`m` 为网格的列数，只需要遍历所有网格一次即可
- 空间复杂度：`O(mn)`

### 思路 2: 空间优化

使用滚动数组思想优化二维 `dp` 数组为一维 `dp` 数组

```js
var uniquePathsWithObstacles = function(obstacleGrid) {
    let m = obstacleGrid.length, n = obstacleGrid[0].length;
    let dp = Array(n + 1).fill(0);
    dp[1] = obstacleGrid[0][0] == 1 ? 0 : 1;
    for (let i = 1; i <= m; i++) {
        for (let j = 1; j <= n; j++) {
            if (i == 1 && j == 1) continue;
            if (obstacleGrid[i - 1][j - 1] == 1) {
                dp[j] = 0;
                continue;
            }
            dp[j] = dp[j] + dp[j - 1];
        }
    }
    return dp[n];
};
```
```python
# 写法 1
class Solution:
    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        m, n = len(obstacleGrid), len(obstacleGrid[0])
        # 根据二维 dp 数组优化为一维 dp 数组
        dp = [0] * (n + 1)
        dp[1] = 0 if obstacleGrid[0][0] == 1 else 1
        

        for i in range(1, m + 1):
            for j in range(1, n + 1):
                # 跳过 base case
                if i == 1 and j == 1:
                    continue

                # 跳过障碍物的格子
                if obstacleGrid[i - 1][j - 1] == 1:
                    dp[j] = 0
                    continue
                
                dp[j] = dp[j] + dp[j - 1]
        
        # 返回到达 obstacleGrid[m-1][n-1] 的路径数量
        return dp[n]

# 写法 2
class Solution:
    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        m, n = len(obstacleGrid), len(obstacleGrid[0])
        dp = [0] * n
        dp[0] = 1 if obstacleGrid[0][0] == 0 else 0
        for i in range(m):
            for j in range(n):
                if obstacleGrid[i][j] == 1:
                    dp[j] = 0
                elif j > 0 and obstacleGrid[i][j - 1] == 0:
                    dp[j] += dp[j - 1]
            
        return dp[-1]
```

- 时间复杂度：`O(mn)`，其中 `m` 为网格的行数，`n` 为网格的列数，只需要遍历所有网格一次即可
- 空间复杂度：`O(n)`，利用滚动数组优化，可只用 `O(n)` 大小的空间来记录当前行的 `dp` 值