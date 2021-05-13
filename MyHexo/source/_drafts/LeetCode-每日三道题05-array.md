title: LeetCode 每日三道题05 array
author: tr
tags:
  - LeetCode
categories:
  - LeetCode
date: 2021-04-12 14:30:00
---
# Merge intervals

<!--more-->
medium

```java
 /**
     * 题目：合并有交集的集合
     * Input: intervals = [[1,3],[2,6],[8,10],[15,18]]
     * Output: [[1,6],[8,10],[15,18]]
     * Explanation: Since intervals [1,3] and [2,6] overlaps, merge them into [1,6].
     */
    private static class Solution {
        public int[][] merge(int[][] intervals) {
            if (intervals.length == 1) return intervals;
//            Arrays.sort(intervals, Comparator.comparingInt(x -> x[0]));
            List<int[]> sorted = Arrays.stream(intervals).sorted(Comparator.comparingInt(x -> x[0])).collect(Collectors.toList());
            for (int i = 0; i < sorted.size() - 1; i++) {
                if (sorted.get(i)[1] >= sorted.get(i + 1)[0]) {
                    int[] ints = {sorted.get(i)[0], Math.max(sorted.get(i + 1)[1], sorted.get(i)[1])};
                    sorted.set(i, ints);
                    sorted.remove(i + 1);
                    i--;
                }
            }
            return sorted.toArray(new int[0][]);
        }
    }
```

# Insert intervals

medium 思路：先确定左端点，在确定右端点，在端点之外的都添加到结果。

```java
 public int[][] insert2(int[][] intervals, int[] newInterval) {
            List<int[]> res = new LinkedList<>();
            if (intervals.length == 0) {
                res.add(newInterval);
                return res.toArray(new int[0][]);
            }
            // 找到开始合并的数组
            int newS = newInterval[0];
            int newE = newInterval[1];
            int posi = 0;
            while (posi < intervals.length && newS >= intervals[posi][0]) posi++;
            posi--;
            for (int i = 0; i < posi; i++) res.add(intervals[i]);
            if (posi >= intervals.length) res.add(newInterval);
            else {
                if (posi >= 0) {
                    if (intervals[posi][1] < newS) {
                        posi++;
                        res.add(intervals[posi - 1]);
                    } else {
                        newS = Math.min(newS, intervals[posi][0]);
                    }
                } else posi++;
                // 确定右端点
                while (posi < intervals.length && intervals[posi][1] < newE) posi++;
                if (posi < intervals.length && intervals[posi][0] <= newE) newE = intervals[posi++][1];

                res.add(new int[]{newS, newE});
                for (int i = posi; i < intervals.length; i++) {
                    res.add(intervals[i]);
                }
            }
            return res.toArray(new int[0][]);
        }
```

# GenerateMatrix 

medium 生成矩阵 一次AC，思路是每次都填充好外边一圈，然后调用函数反复填充外边一圈

```java
package com.leetcode;

import java.util.Arrays;

public class GenerateMatrix59 {
    public static void main(String[] args) {
        int[][] ints = new Solution().generateMatrix(100);
        for (int[] anInt : ints) {
            System.out.println(Arrays.toString(anInt));
        }
    }

    /**
     * Given a positive integer n, generate an n x n matrix filled with elements from 1 to n2 in spiral order.
     * <p>
     * Input: n = 3
     * Output: [[1,2,3],[8,9,4],[7,6,5]]
     */
    private static class Solution {
        public int[][] generateMatrix(int n) {
            int[][] matrix = new int[n][n];
            doGenerate(matrix, 0, n, 1);
            return matrix;
        }

        /**
         * @param matrix 要生产的矩阵
         * @param start  起始位置
         * @param nums   填充数量
         * @param num    填充的数字
         */
        public void doGenerate(int[][] matrix, int start, int nums, int num) {
            // nums <=0 结束
            if (nums <= 0) return;
            // 填充第一行
            for (int i = 0; i < nums; i++) {
                matrix[start][start + i] = num++;
            }
            for (int i = 0; i < nums - 1; i++) {
                matrix[start + i + 1][start + nums - 1] = num++;
            }
            for (int i = 0; i < nums - 1; i++) {
                matrix[start + nums - 1][start + nums - 2 - i] = num++;
            }
            for (int i = 0; i < nums - 2; i++) {
                matrix[start + nums - 2 - i][start] = num++;
            }
            doGenerate(matrix, ++start, nums - 2, num);
        }
    }
}

```