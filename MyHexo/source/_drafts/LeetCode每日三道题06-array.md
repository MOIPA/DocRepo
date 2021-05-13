title: LeetCode每日三道题06 动态规划
author: tr
date: 2021-05-13 09:49:44
tags:
---
# Unique Paths

medium 动态规划题
一次ac
![upload successful](/images/pasted-62.png)


![upload successful](/images/pasted-59.png)

机器人只能向右和向下走，问走到终点几种方案，比较简单的经典动规

思路：在finish上方和左边到finish点个只有一种，记录下来，finish点左上方到达下面一个点是1种，到达右边也是一种，所以左上角到终点是`1*1 + 1*1 = 2` 种。所以对于每个点只要算上右边和下边的和就够了。

![upload successful](/images/pasted-60.png)

```java
package com.leetcode;

public class UniquePaths62 {
    public static void main(String[] args) {
        System.out.println(new Solution().uniquePaths(7, 3));
    }

    /**
     * A robot is located at the top-left corner of a m x n grid (marked 'Start' in the diagram below).
     * <p>
     * The robot can only move either down or right at any point in time. The robot is trying to reach the bottom-right corner of the grid (marked 'Finish' in the diagram below).
     * <p>
     * How many possible unique paths are there?
     * <p>
     * Input: m = 3, n = 2
     * Output: 3
     * Explanation:
     * From the top-left corner, there are a total of 3 ways to reach the bottom-right corner:
     * 1. Right -> Down -> Down
     * 2. Down -> Down -> Right
     * 3. Down -> Right -> Down
     */
    private static class Solution {
        public int uniquePaths(int m, int n) {
            int[][] matrix = new int[m][n];
            for (int i = 0; i < n; i++) {
                matrix[m - 1][i] = 1;
            }
            for (int i = 0; i < m; i++) {
                matrix[i][n - 1] = 1;
            }
            for (int i = m - 2; i >= 0; i--) {
                for (int j = n - 2; j >= 0; j--) {
                    matrix[i][j] = matrix[i + 1][j] + matrix[i][j + 1];
                }
            }
            return matrix[0][0];
        }
    }
}

```

# Unique Paths II

在上题基础上加了障碍物

![upload successful](/images/pasted-63.png)

思路：还是和上题一样，只不过每个节点算下和右和的时候如果有障碍物不算进去

leetcode 的eg真是佛了，还有输入{{0}}和{{1}}，{{0，0}，{0，1}}即起点在终点，终点被堵住这种情况。


![upload successful](/images/pasted-64.png)
```java
package com.leetcode;

public class UniquePathsII63 {
    public static void main(String[] args) {
        System.out.println(new Solution().uniquePathsWithObstacles(new int[][]{
                {0}
        }));
    }


    /**
     * A robot is located at the top-left corner of a m x n grid (marked 'Start' in the diagram below).
     * <p>
     * The robot can only move either down or right at any point in time. The robot is trying to reach the bottom-right corner of the grid (marked 'Finish' in the diagram below).
     * <p>
     * Now consider if some obstacles are added to the grids. How many unique paths would there be?
     * <p>
     * An obstacle and space is marked as 1 and 0 respectively in the grid.
     */
    private static class Solution {
        public int uniquePathsWithObstacles(int[][] obstacleGrid) {
            int m = obstacleGrid.length;
            int n = obstacleGrid[0].length;
            if (m == 1 && n == 1) {
                if (obstacleGrid[0][0] == 0) return 1;
                else return 0;
            }
            if (obstacleGrid[m - 1][n - 1] == 1) return 0;
            for (int i = n - 2; i >= 0; i--) {
                if (obstacleGrid[m - 1][i] == 1) obstacleGrid[m - 1][i] = 0;
                else {
                    if (i == n - 2) obstacleGrid[m - 1][i] = 1;
                    else obstacleGrid[m - 1][i] = obstacleGrid[m - 1][i + 1];
                }
            }
            for (int i = m - 2; i >= 0; i--) {
                if (obstacleGrid[i][n - 1] == 1) obstacleGrid[i][n - 1] = 0;
                else {
                    if (i == m - 2) obstacleGrid[i][n - 1] = 1;
                    else obstacleGrid[i][n - 1] = obstacleGrid[i + 1][n - 1];
                }
            }
            for (int i = m - 2; i >= 0; i--) {
                for (int j = n - 2; j >= 0; j--) {
                    if (obstacleGrid[i][j] == 1) obstacleGrid[i][j] = 0;
                    else {
                        obstacleGrid[i][j] = obstacleGrid[i + 1][j] + obstacleGrid[i][j + 1];
                    }
                }
            }
            return obstacleGrid[0][0];
        }
    }
}

```

# Minimum Path Sum


![upload successful](/images/pasted-67.png)

思路和上面两题一样 动规

```
private static class Solution {
        public int minPathSum(int[][] grid) {
            int m = grid.length;
            int n = grid[0].length;
            for (int i = m - 2; i >= 0; i--) grid[i][n - 1] +=  grid[i + 1][n - 1];
            for (int i = n - 2; i >= 0; i--) grid[m - 1][i] += grid[m - 1][i + 1];
            for (int i = m - 2; i >= 0; i--) {
                for (int j = n - 2; j >= 0; j--) {
                    grid[i][j] += Math.min(grid[i + 1][j], grid[i][j + 1]);
                }
            }
            return grid[0][0];
        }
    }
```