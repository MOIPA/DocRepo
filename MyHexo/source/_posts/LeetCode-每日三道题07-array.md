title: LeetCode 每日三道题07 array + matrix
author: tr
tags:
  - LeetCode
categories:
  - LeetCode
date: 2021-05-13 11:03:00
---
# Plus One

easy 


![upload successful](/images/pasted-68.png)

```java
package com.leetcode;

import java.util.Arrays;

public class PlusOne66 {
    public static void main(String[] args) {
        System.out.println(Arrays.toString(new Solution().plusOne(new int[]{0})));
    }

    private static class Solution {
        public int[] plusOne(int[] digits) {
            int carry = 0;
            for (int i = digits.length - 1; i >= 0; i--) {
                if (i == digits.length - 1) {
                    digits[i] += 1;
                    if (digits[i] == 10) {
                        digits[i] = 0;
                        carry++;
                    }
                } else {
                    digits[i] += carry;
                    if (digits[i] == 10) digits[i] = 0;
                    else carry = 0;
                }
            }
            if (carry == 1) {
                int[] newArr = new int[digits.length + 1];
                newArr[0] = 1;
                for (int i = 0; i < digits.length; i++) {
                    newArr[i + 1] = digits[i];
                }
                return newArr;
            }
            return digits;
        }
    }
}

```

# Set Matrix Zeroes

难点在空间复杂度 O(1)

```java
package com.leetcode;

import java.util.Arrays;
import java.util.HashSet;
import java.util.Set;

public class SetZeroes73 {
    public static void main(String[] args) {
        int[][] matrix = new int[][]{
                {0, 1, 2, 0}, {3, 4, 5, 2}, {1, 3, 1, 5}
        };
        new Solution().setZeroes2(matrix);
        for (int[] ints : matrix) {
            System.out.println(Arrays.toString(ints));
        }
    }

    private static class Solution {
        public void setZeroes(int[][] matrix) {
            Set<Integer> cols = new HashSet();
            Set<Integer> rows = new HashSet();
            for (int i = 0; i < matrix.length; i++) {
                for (int j = 0; j < matrix[i].length; j++) {
                    if (matrix[i][j] == 0) {
                        cols.add(j);
                        rows.add(i);
                    }
                }
            }
            for (Integer col : cols) {
                for (int i = 0; i < matrix.length; i++) {
                    matrix[i][col] = 0;
                }
            }
            for (Integer row : rows) {
                for (int i = 0; i < matrix[row].length; i++) {
                    matrix[row][i] = 0;
                }
            }
        }

        // O(1) space的算法 遍历 如果一个数为0 将对应第一行和第一列的值置为0a
        public void setZeroes2(int[][] matrix) {
            int m = matrix.length;
            int n = matrix[0].length;
            for (int i = 1; i < m; i++)
                for (int j = 1; j < n; j++)
                    if (matrix[i][j] == 0) {
                        matrix[i][0] = 0;
                        matrix[0][j] = 0;
                    }
            // set to zero
            for (int i = 1; i < m; i++)
                for (int j = 1; j < n; j++)
                    if (matrix[i][0] == 0 || matrix[0][j] == 0) matrix[i][j] = 0;
            // 看第一行 和 第一列是否需要置空
            boolean isRow = false;
            boolean isCol = false;
            for (int i = 0; i < m; i++) {
                if (matrix[i][0] == 0) isRow = true;
            }
            for (int i = 0; i < n; i++) {
                if (matrix[0][i] == 0) isCol = true;
            }
            if (isRow)
                for (int i = 0; i < m; i++) {
                    matrix[i][0] = 0;
                }
            if (isCol)
                for (int i = 0; i < n; i++) {
                    matrix[0][i] = 0;
                }
        }

    }
}

```

#  Search a 2D Matrix

需要注意的是 二分法找行的时候，如果行首数小于target，还要判断行尾数是否也小于target，都小于才能`startRow = middleRow + 1`

```java

package com.leetcode;

public class SearchMatrix74 {
    public static void main(String[] args) {
        System.out.println(new Solution().searchMatrix(new int[][]{
                {1, 3, 5, 7}, {10, 11, 16, 20}, {23, 30, 34, 60}
        }, 11));

    }

    /**
     * Write an efficient algorithm that searches for a value in an m x n matrix. This matrix has the following properties:
     * <p>
     * Integers in each row are sorted from left to right.
     * The first integer of each row is greater than the last integer of the previous row.
     */
    private static class Solution {
        public boolean searchMatrix(int[][] matrix, int target) {
            int startRow = 0;
            int endRow = matrix.length - 1;
            int startCol = 0;
            int endCol = matrix[0].length - 1;

            // 二分法查行
            while (startRow < endRow) {
                int middleRow = (startRow + endRow) / 2;
                if (matrix[middleRow][0] > target) {
                    endRow = middleRow - 1;
                } else if (matrix[middleRow][0] < target) {
                    if (matrix[middleRow][endCol] < target) startRow = middleRow + 1;
                    else startRow = endRow = middleRow;
                } else if (matrix[middleRow][0] == target) {
                    return true;
                }
            }
            // 二分查找列
            while (startCol <= endCol) {
                int middleCol = (startCol + endCol) / 2;
                if (matrix[startRow][middleCol] == target) {
                    return true;
                } else if (matrix[startRow][middleCol] > target) {
                    endCol = middleCol - 1;
                } else if (matrix[startRow][middleCol] < target) {
                    startCol = middleCol + 1;
                }
            }
            return false;
        }
    }
}

```