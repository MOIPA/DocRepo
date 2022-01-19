title: LeetCode 每日三道题 04 动规
author: tr
tags:
  - LeetCode
  - 动态规划
categories:
  - LeetCode
date: 2021-04-09 14:13:00
---
# Maxium subarray 
easy

<!--more-->
 感觉动态规划切割问题的时候要将问题切割到每一个或者第一个开始。
```java
package com.leetcode;

public class MaximumSubarray53 {
    public static void main(String[] args) {
        System.out.println(new Solution().maxSubArray(new int[]{1}));
    }

    /**
     * 题目：
     * Given an integer array nums, find the contiguous subarray (containing at least one number) which has the largest sum and return its sum.
     * <p>
     * Input: nums = [-2,1,-3,4,-1,2,1,-5,4]
     * Output: 6
     * Explanation: [4,-1,2,1] has the largest sum = 6.
     * <p>
     * 动态规划：将问题切割到小问题上，从第一个元素开始，dp[i]表示以i结尾的最大连续子序和大小
     * dp[i] = max{nums[i],dp[i-1]+nums[i]}
     *
     * Runtime: 0 ms, faster than 100.00% of Java online submissions for Maximum Subarray.
     * Memory Usage: 38.4 MB, less than 99.75% of Java online submissions for Maximum Subarray.
     *
     */
    private static class Solution {
        public int maxSubArray(int[] nums) {
            // 记录上一个最大子序和
            int lastMaxSubArray = nums[0];
            // 全部最大子序和
            int max = lastMaxSubArray;
            for (int i = 1; i < nums.length; i++) {
                lastMaxSubArray = Math.max(lastMaxSubArray + nums[i], nums[i]);
                max = Math.max(max, lastMaxSubArray);
            }
            return max;
        }
    }
}

```
# Spiral Matrix
medium
```java
package com.leetcode;

import java.util.LinkedList;
import java.util.List;

public class SpiralMatrix54 {
    public static void main(String[] args) {
        System.out.println(new Solution().spiralOrder(new int[][]{{2, 5, 8}, {4, 0, -1}}));
    }

    /**
     * 问题：
     * Given an m x n matrix, return all elements of the matrix in spiral order.
     * <p>
     * Input: matrix = [[1,2,3],[4,5,6],[7,8,9]]
     * Output: [1,2,3,6,9,8,7,4,5]
     *
     * Runtime: 0 ms, faster than 100.00% of Java online submissions for Spiral Matrix.
     * Memory Usage: 37 MB, less than 66.66% of Java online submissions for Spiral Matrix.
     */
    private static class Solution {
        public List<Integer> spiralOrder(int[][] matrix) {
            List<Integer> list = new LinkedList<>();
            doSpiralARound(matrix, 0, 0, matrix[0].length, matrix.length, list);
            return list;
        }

        private void doSpiralARound(int[][] matrix, int x, int y, int length, int height, List<Integer> res) {
            if (length <= 0 || height <= 0 || x > matrix.length - 1 || y > matrix[0].length - 1) return;
            // 上行
            for (int i = 0; i < length; i++) {
                res.add(matrix[x][y + i]);
            }
            // 右列
            for (int i = 1; i < height; i++) {
                res.add(matrix[x + i][y + length - 1]);
            }
            if (height > 1) {
                // 下行
                for (int i = length - 2; i >= 0; i--) {
                    res.add(matrix[x + height - 1][y + i]);
                }
                // 左列
                if (length > 1) for (int i = height - 2; i > 0; i--) {
                    res.add(matrix[x + i][y]);
                }
            }
            doSpiralARound(matrix, x + 1, y + 1, length - 2, height - 2, res);
        }
    }
}

```

# Jump Game 
medium 

```java
package com.leetcode;

public class JumpGame55 {
    public static void main(String[] args) {
        System.out.println(new Solution().canJump(new int[]{2, 1, 1, 1, 4}));
    }

    /**
     * 题目：判断是否能到终点，和JumpGameII54差不多 这个是简单版本 都用贪心算法即可
     * <p>
     * Runtime: 1 ms, faster than 83.87% of Java online submissions for Jump Game.
     * Memory Usage: 40.8 MB, less than 75.58% of Java online submissions for Jump Game.
     */
    private static class Solution {
        public boolean canJump(int[] nums) {
            // 当前跳跃的最远点
            int currentMaxJump = 0;
            // 可以跳的最远步数 对比取最大值
            int canMaxJump = 0;
            for (int i = 0; i < nums.length - 1; i++) {
                canMaxJump = Math.max(nums[i] + i, canMaxJump);
                // 遍历完成 开始跳
                if (i == currentMaxJump) {
                    currentMaxJump = canMaxJump;
                    canMaxJump = 0;
                }
            }
            return currentMaxJump >= nums.length - 1;

        }
    }
}

```