title: LeetCode 每日三道题 03 动规 指针
author: tr
tags:
  - LeetCode
  - Array
  - 动态规划
categories:
  - LeetCode
date: 2021-04-09 09:23:00
---


# Trapping rain water   -hard

三种解题方法和思路，最好的80%

```java
package com.leetcode;

public class TrappingRainWater42 {
    public static void main(String[] args) {
        System.out.println(new Solution3().trap(new int[]{9, 8, 9, 5, 8, 8, 8, 0, 4}));
    }

    /**
     * Given n non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it can trap after raining.
     * <p>
     * 涉及数组 动规 双指针
     * <p>
     * 思路：首先找到最高点，分别向左右两边找第二高的点，找到第二高的点，将第二高的值减去包裹的所有高度，
     * <p>
     * Runtime: 2 ms, faster than 21.28% of Java online submissions for Trapping Rain Water.
     * Memory Usage: 38.3 MB, less than 86.97% of Java online submissions for Trapping Rain Water.
     */
    private static class Solution {
        public int trap(int[] height) {
            return doTrap(height, 0, height.length - 1);
        }

        private int doTrap(int[] height, int start, int end) {
            if (start >= end - 1) return 0;
            int heighestIndex = findMaxIndex(height, start, end, 0);
            int totalWater = 0;
            int leftSecondHeightIndex = findMaxIndex(height, start, heighestIndex - 1, 0);
            int rightSecondHeightIndex = findMaxIndex(height, heighestIndex + 1, end, 1);
            // 找到左边包含水源
            if (leftSecondHeightIndex != -1) {
                for (int i = leftSecondHeightIndex + 1; i <= heighestIndex - 1; i++) {
                    totalWater += height[leftSecondHeightIndex] - height[i];
                }
                totalWater += doTrap(height, start, leftSecondHeightIndex);
            }
            // 找到右边包含水源
            if (rightSecondHeightIndex != -1) {
                for (int i = heighestIndex + 1; i <= rightSecondHeightIndex - 1; i++) {
                    totalWater += height[rightSecondHeightIndex] - height[i];
                }
                totalWater += doTrap(height, rightSecondHeightIndex, end);
            }
            return totalWater;
        }

        /**
         * 找到最高点的下标
         *
         * @param direction 靠近哪边的最高点，因为可能有高度相同， 1：表示靠近右边的最大值，0表示靠近左边的最大值
         */
        private int findMaxIndex(int[] height, int start, int end, int direction) {
            // 相邻的两个墙壁找最高无意义
            if (start >= end) return -1;
            int tmp;
            if (direction == 0) {
                tmp = start;
                for (int i = start; i <= end; i++) {
                    if (height[tmp] < height[i]) tmp = i;
                }
            } else {
                tmp = end;
                for (int i = end; i >= start; i--) {
                    if (height[tmp] < height[i]) tmp = i;
                }
            }
            return tmp;
        }
    }

    /**
     * 上一种太慢，换种，两个指针，一个头，一个尾 都不为0，左指针又走到比自身大的，右指针左走到比自己大的，分别计算两个指针刚刚和现在的位置包含的水量。
     * 直到左右指针相遇结束
     * <p>
     * Runtime: 2 ms, faster than 21.28% of Java online submissions for Trapping Rain Water.
     * Memory Usage: 38.3 MB, less than 86.97% of Java online submissions for Trapping Rain Water.
     */

    private static class Solution2 {
        public int trap(int[] height) {
            int totalWater = 0;
            int startPoint = 0, endPoint = height.length - 1;
            int recordStart, recordEnd;
            while (startPoint < endPoint) {
                // 直到比自己大
                recordStart = startPoint;
                recordEnd = endPoint;
                // 指针走的时候会有递减的情况 如果指针一次走到底了 那么需要重置指针 让另一个指针走
                while (startPoint < endPoint - 1 && height[recordStart] >= height[startPoint + 1]) startPoint++;
                startPoint++;
                // reset
                if (startPoint >= endPoint && height[recordStart] > height[startPoint]) startPoint = recordStart;
                while (startPoint < endPoint - 1 && height[recordEnd] >= height[endPoint - 1]) endPoint--;
                endPoint--;
                if (startPoint >= endPoint && height[endPoint] < height[recordEnd]) endPoint = recordEnd;
                // 分别计算水量
                for (int i = recordStart + 1; i < startPoint; i++) {
                    totalWater += height[recordStart] - height[i];
                }
                for (int i = endPoint + 1; i < recordEnd; i++) {
                    totalWater += height[recordEnd] - height[i];
                }
            }
            return totalWater;
        }
    }

    /**
     * 上面那个思路稍微修改下即可 80%  不重置指针，而是先找到最大值，左指针和右指针最高只能找到最大值
     *
     * Runtime: 1 ms, faster than 81.80% of Java online submissions for Trapping Rain Water.
     * Memory Usage: 38 MB, less than 99.27% of Java online submissions for Trapping Rain Water.
     *
     */
    private static class Solution3 {
        public int trap(int[] height) {
            int totalWater = 0;
            int startPoint = 0, endPoint = height.length - 1;
            int recordStart, recordEnd;
            // 找到最大值 左右指针只能走到最大值
            int maxIndex = 0;
            for (int i = 1; i < height.length; i++) {
                if (height[maxIndex] < height[i]) maxIndex = i;
            }
            while (startPoint < endPoint) {
                // 直到比自己大
                recordStart = startPoint;
                recordEnd = endPoint;
                // 指针走的时候会有递减的情况 如果指针一次走到底了 那么需要重置指针 让另一个指针走
                while (startPoint < maxIndex && height[recordStart] >= height[startPoint + 1]) startPoint++;
                if (startPoint < maxIndex) startPoint++;
                while (endPoint > maxIndex && height[recordEnd] >= height[endPoint - 1]) endPoint--;
                if (endPoint > maxIndex) endPoint--;
                // 分别计算水量
                for (int i = recordStart + 1; i < startPoint; i++) {
                    totalWater += height[recordStart] - height[i];
                }
                for (int i = endPoint + 1; i < recordEnd; i++) {
                    totalWater += height[recordEnd] - height[i];
                }
            }
            return totalWater;
        }
    }

}

```

# JumpGame II 45 medium

动规和贪心的解决方案


贪心算法：

![upload successful](/images/pasted-48.png)

```java
package com.leetcode;

import java.util.Arrays;

public class JumpGameII45 {
    public static void main(String[] args) {
        System.out.println(new Solution2().jump(new int[]{2, 3, 1, 1, 4}));
    }

    /**
     * 问题：
     * Given an array of non-negative integers nums, you are initially positioned at the first index of the array.
     * Each element in the array represents your maximum jump length at that position.
     * Your goal is to reach the last index in the minimum number of jumps.
     * You can assume that you can always reach the last index.
     * <p>
     * 当前位置值为最大跳跃步数，如何移动最小次数跳到最后
     * <p>
     * 思路：先用动态规划的思想，从低到高，从最后开始算最优跳跃次数 -1表示无法到达终点
     * <p>
     * Runtime: 0 ms, faster than 100.00% of Java online submissions for Jump Game II.
     * Memory Usage: 36.3 MB, less than 85.37% of Java online submissions for Jump Game II.
     */
    private static class Solution {
        public int jump(int[] nums) {
            int[] jumpTimes = new int[nums.length];
            Arrays.fill(jumpTimes, -1);
            for (int i = nums.length - 1; i >= 0; i--) {
                if (i == nums.length - 1) {
                    jumpTimes[i] = 0;
                    continue;
                }
                // 每位再遍历后面的次数 选最小的
                int min = jumpTimes[i + 1];
                for (int j = 1; j <= nums[i]; j++) {
                    if (i + j < nums.length && min > jumpTimes[i + j] && jumpTimes[i + j] != -1) min = jumpTimes[i + j];
                }
                jumpTimes[i] = min + 1;
            }
            return jumpTimes[0];
        }
    }

    /**
     * 改造成贪心算法
     * <p>
     * 思路：选择能跳到的范围内的，最远下一次跳跃距离
     * 当前的currentJumpEnd记录能跳到的最远位置，在当前位置和最远位置里面挑出跳的最远的方案：max，i==currentPosi表明已经找到最大跳远距离了
     */
    private static class Solution2 {
        public int jump(int[] nums) {
            int jumpTimes = 0;
            int max = 0;
            int currentJumpEnd = 0;
            // 记录能跳的最远位置
            for (int i = 0; i < nums.length - 1; i++) {
                max = Math.max(max, i + nums[i]);
                if (i == currentJumpEnd) {
                    jumpTimes++;
                    currentJumpEnd = max;
                }
            }
            return jumpTimes;
        }
    }
}

```

# Rotate image
medium
```java
package com.leetcode;

import java.lang.reflect.Array;
import java.util.Arrays;

public class RotateImage48 {
    public static void main(String[] args) {
        new Solution().rotater2(new int[][]{
                {1, 2, 3}, {4, 5, 6}, {7, 8, 9}
        });
    }

    /**
     * Runtime: 3 ms, faster than 100.00% of Java online submissions for Rotate Image.
     * Memory Usage: 39 MB, less than 48.87% of Java online submissions for Rotate Image.
     */
    private static class Solution {
        public void rotate(int[][] matrix) {
            // 从第一列开始
            for (int i = 0; i < matrix.length; i++) {
                for (int j = i; j < matrix.length; j++) {
                    swap(matrix, i, j, j, i);
                }
                for (int j = 0; j < matrix.length / 2; j++) {
                    swap(matrix, i, j, i, matrix.length - 1 - j);
                }
            }
            Arrays.stream(matrix).forEach(x -> System.out.println(Arrays.toString(x)));
        }

        private void swap(int[][] matrix, int i, int j, int newI, int newJ) {
            int tmp = matrix[newI][newJ];
            matrix[newI][newJ] = matrix[i][j];
            matrix[i][j] = tmp;
        }

        public void rotater2(int[][] matrix) {
            // 从第一列开始
            StringBuffer out = new StringBuffer();
            out.append("[");
            // 列
            for (int i = 0; i < matrix.length; i++) {
                out.append("[");
                for (int j = matrix.length - 1; j >= 0; j--) {
                    out.append(matrix[j][i]);
                    if (j!=0) out.append(",");
                }
                out.append("]");
                if (i != matrix.length-1) out.append(",");
            }
            out.append("]");
            System.out.println(out.toString());
        }
    }
}

```