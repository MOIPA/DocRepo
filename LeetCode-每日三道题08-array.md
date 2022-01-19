title: LeetCode 每日三道题08 array
author: tr
tags:
  - LeetCode
categories:
  - LeetCode
date: 2021-05-17 11:07:00
---
#  Sort Colors


快排  看到leetcode有类似快排的算法，0换到左边，2换到右边 懒得实现了

```java
 private void doSort(int[] nums, int start, int end) {
            if (start >= end) return;
            int tmp = nums[start];
            int startPoint = start;
            int endPoint = end;
            while (startPoint < endPoint) {
                while (startPoint < endPoint && nums[endPoint] > tmp) endPoint--;
                if (startPoint < endPoint) nums[startPoint++] = nums[endPoint];
                while (startPoint < endPoint && nums[startPoint] < tmp) startPoint++;
                if (startPoint < endPoint) nums[endPoint--] = nums[startPoint];
            }
            nums[startPoint] = tmp;
            doSort(nums, start, startPoint - 1);
            doSort(nums, startPoint + 1, end);
        }
```
# Subsets

这个问题要好好看看的 新的套路或者说新的思想，两种回溯，一种是深度优先搜索


![upload successful](/images/pasted-73.png)

![upload successful](/images/pasted-74.png)

```java
package com.leetcode;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.LinkedList;
import java.util.List;
import java.util.stream.Collectors;

public class Subsets {
    public static void main(String[] args) {
        List<List<Integer>> subsets = new Solution().subsets(new int[]{0});
        subsets.forEach(x -> {
            x.forEach(System.out::print);
            System.out.println();
        });
    }

    /**
     * Given an integer array nums of unique elements, return all possible subsets (the power set).
     * <p>
     * The solution set must not contain duplicate subsets. Return the solution in any order.
     */
      /**
     * 回溯法
     */
    private static class Solution2 {
        public List<List<Integer>> subsets(int[] nums) {
            List<List<Integer>> ans = new ArrayList<>();
            getAns(nums, 0, new ArrayList<>(), ans);
            return ans;
        }

        private void getAns(int[] nums, int start, ArrayList<Integer> temp, List<List<Integer>> ans) {
            ans.add(new ArrayList<>(temp));
            for (int i = start; i < nums.length; i++) {
                temp.add(nums[i]);
                getAns(nums, i + 1, temp, ans);
                temp.remove(temp.size() - 1);
            }
        }
    }
    private static class Solution {
        public List<List<Integer>> subsets(int[] nums) {

            List<Integer> list = Arrays.stream(nums).boxed().collect(Collectors.toList());
            List<List<Integer>> lists = null;
            if (nums.length < 2) {
                lists = new ArrayList<>();
            } else {
                lists = doSplit(list);
            }
            for (int num : nums) {
                lists.add(Arrays.asList(num));
            }
            lists.add(new ArrayList<>());
            return lists;
        }

        List<List<Integer>> doSplit(List<Integer> nums) {
            List<List<Integer>> res = new LinkedList<>();
            if (nums.size() == 2) {
                res.add(Arrays.asList(nums.get(0), nums.get(1)));
                return res;
            }
            int startNum = nums.get(0);
            nums.remove(0);
            nums.forEach(x -> {
                res.add(Arrays.asList(startNum, x));
            });
            List<List<Integer>> subs = doSplit(nums);
            res.addAll(subs);
            subs.forEach(x -> {
                ArrayList<Integer> tmp = new ArrayList<>(x);
                tmp.add(0, startNum);
                res.add(tmp);
            });

            return res;
        }

    }
}

```

# Word Search

思路：首先定位首字母，然后查找周围匹配的下一个字母，注意的是这里需要回溯，一条路走不通，可能另一条就可以。且每个字母不能被重复用，所以加一个数组用来标记

![upload successful](/images/pasted-69.png)

```java
package com.leetcode;

import java.util.Arrays;

public class WordSearch79 {
    public static void main(String[] args) {
        System.out.println(new Solution().exist(new char[][]{
                {'a', 'a', 'a', 'a'}, {'a', 'a', 'a', 'a'}, {'a', 'a', 'a', 'a'}
        }, "aaaaaaaaaaaaa"));
    }

    /**
     * Given an m x n grid of characters board and a string word, return true if word exists in the grid.
     * <p>
     * The word can be constructed from letters of sequentially adjacent cells, where adjacent cells are horizontally or vertically neighboring. The same letter cell may not be used more than once.
     */
    private static class Solution {
        public boolean exist(char[][] board, String word) {
            // 1. 定位到字母开头
            int colLen = board[0].length;
            char character = word.charAt(0);
            boolean[][] usage = new boolean[board.length][colLen];
            for (int i = 0; i < board.length; i++) {
                for (int j = 0; j < colLen; j++) {
                    if (board[i][j] == character) {
                        // 2. 判断是否能找完
                        usage[i][j] = true;
                        if (doFind(board, i, j, -1, 1, word, usage)) return true;
                        else usage[i][j] = false;
                    }
                }
            }
            return false;
        }

        /**
         * @param board          矩阵
         * @param i              当前位置
         * @param j
         * @param previousDirect 从哪来
         * @param chPosi         要查找的字符位置
         * @return
         */
        private boolean doFind(char[][] board, int i, int j, int previousDirect, int chPosi, String word, boolean[][] usage) {
            if (chPosi == word.length()) return true;
            // 0 1 2 3  上 下 左 右
            if (previousDirect != 0 && i > 0 && !usage[i - 1][j] && board[i - 1][j] == word.charAt(chPosi)) {
                usage[i - 1][j] = true;
                if (doFind(board, i - 1, j, 1, chPosi + 1, word, usage)) return true;
                else usage[i - 1][j] = false;
            }
            if (previousDirect != 1 && i < board.length - 1 && !usage[i + 1][j] && board[i + 1][j] == word.charAt(chPosi)) {
                usage[i + 1][j] = true;
                if (doFind(board, i + 1, j, 0, chPosi + 1, word, usage)) return true;
                else usage[i + 1][j] = false;
            }
            if (previousDirect != 2 && j > 0 && !usage[i][j - 1] && board[i][j - 1] == word.charAt(chPosi)) {
                usage[i][j - 1] = true;
                if (doFind(board, i, j - 1, 3, chPosi + 1, word, usage)) return true;
                else usage[i][j - 1] = false;
            }
            if (previousDirect != 3 && j < board[0].length - 1 && !usage[i][j + 1] && board[i][j + 1] == word.charAt(chPosi)) {
                usage[i][j + 1] = true;
                if (doFind(board, i, j + 1, 2, chPosi + 1, word, usage)) return true;
                else usage[i][j + 1] = false;
            }
            return false;
        }
    }
}

```

# Remove Duplicates

```java
package com.leetcode;

import java.util.Arrays;

public class RemoveDuplicates80 {
    public static void main(String[] args) {
        int[] tmp = new int[]{1, 2, 2, 2, 3, 3};
        System.out.println(new Solution().removeDuplicatesEasy(tmp));
        System.out.println(Arrays.toString(tmp));
    }

    private static class Solution {
        public int removeDuplicates(int[] nums) {

            int point = 1;
            int same = nums[0];
            int sameTimes = 0;
            int len = nums.length;
            while (point < len) {
                if (nums[point] == same) sameTimes++;
                else {
                    same = nums[point];
                    sameTimes = 0;
                }
                if (sameTimes >= 2) {
                    for (int i = point + 1; i < len; i++) {
                        nums[i - 1] = nums[i];
                    }
                    len--;
                    point--;
                }
                point++;
            }
            return len;
        }

        /**
         * Success
         * Details
         * Runtime: 1 ms, faster than 16.69% of Java online submissions for Remove Duplicates from Sorted Array II.
         * Memory Usage: 38.6 MB, less than 99.46% of Java online submissions for Remove Duplicates from Sorted Array II.
         * Next challenges:
         *
         * @param nums
         * @return
         */
        public int removeDuplicatesFaster(int[] nums) {
            // 差距 默认为-1，如果这一次和上一次相等 差值为0 如果这一次和上一次再相等差值++ 如果这一次和上一次不等，判断差距，移动，然后重设为-1；
            int range = -1;
            int point = nums.length - 2;
            int len = nums.length;
            while (point >= 0) {
                if (nums[point] == nums[point + 1]) range++;
                if (nums[point] != nums[point + 1] || point == 0) {
                    if (range >= 1) {
                        len -= range;
                        if (point == 0 && nums[0] == nums[1]) point--;
                        for (int i = point + 1; i < len; i++) {
                            nums[i] = nums[i + range];
                        }
                    }
                    range = -1;
                }
                point--;
            }
            return len;
        }

        public int removeDuplicatesEasy(int[] nums) {
                int i = 0;
                for (int n : nums)
                    if (i < 2 || n > nums[i-2])
                        nums[i++] = n;
                return i;
        }
    }
}

```

# SearchinRotatedSortedArrayII

```java
package com.leetcode;

public class SearchinRotatedSortedArrayII81 {
    public static void main(String[] args) {
        System.out.println(new Solution().search2(new int[]{2, 5, 6, 0, 0, 1, 2}, 3));
    }

    /**
     * There is an integer array nums sorted in non-decreasing order (not necessarily with distinct values).
     * <p>
     * Before being passed to your function, nums is rotated at an unknown pivot index k (0 <= k < nums.length) such that the resulting array is [nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]] (0-indexed). For example, [0,1,2,4,4,4,5,6,6,7] might be rotated at pivot index 5 and become [4,5,6,6,7,0,1,2,4,4].
     * <p>
     * Given the array nums after the rotation and an integer target, return true if target is in nums, or false if it is not in nums.
     */
    private static class Solution {
        public boolean search(int[] nums, int target) {
            int posi = findSplitPosi(nums, 0, nums.length - 1);
            System.out.println(posi);
            return false;
        }

        int findSplitPosi(int[] nums, int start, int end) {
            if (start > end - 1) return 0;
            int mid = (start + end) / 2;
            if (nums[mid] > nums[mid + 1]) return mid + 1;
            else {
                if (nums[mid] > nums[nums.length - 1]) return findSplitPosi(nums, mid + 1, end);
                else return findSplitPosi(nums, start, mid);
            }
        }

        /**
         * 一次性找出 就不再像上一个一样先找出两个顺序数组了
         *
         * @param nums
         * @param target
         * @return
         */
        public boolean search2(int[] nums, int target) {
            return doSearch(nums, target, 0, nums.length - 1);
        }

        private boolean doSearch(int[] nums, int target, int start, int end) {
            if (start > end) return false;
            boolean isSorted = nums[start] < nums[end];
            if (isSorted && target > nums[start] && target > nums[end]) return false;
            int mid = (start + end) / 2;
            if (nums[mid] == target) return true;
                // 在左区间？ 如果区间非顺序，那么也可能在右区间
            else if (nums[mid] > target) {
                if (isSorted) return doSearch(nums, target, start, mid - 1);
                else return doSearch(nums, target, start, mid - 1) || doSearch(nums, target, mid + 1, end);
            }
            // 右区间？
            else {
                if (isSorted) return doSearch(nums, target, mid + 1, end);
                else return doSearch(nums, target, start, mid - 1) || doSearch(nums, target, mid + 1, end);
            }
        }
    }
}

```

# LargestRectangleArea

hard 用了dp的思想，用数组存储了面积最大值，这个算法24ms，67%，有待优化

![upload successful](/images/pasted-70.png)



![upload successful](/images/pasted-71.png)

```java
package com.leetcode;

import java.util.Arrays;
import java.util.Map;

public class LargestRectangleArea {
    public static void main(String[] args) {
        System.out.println(new Solution().largestRectangleArea(new int[]{
               3,6,5,7,4,8,1,0
        }));
    }

    /**
     * Given an array of integers heights representing the histogram's bar height where the width of each bar is 1, return the area of the largest rectangle in the histogram.
     * <p>
     * 思路：很简单，先准备一个数组用来存放每个数的最大面积，然后从第一个开始，先看数组有没有没有再动，如果右边的小于左边，那么第一个的最大面积为右边的数的最大面积，
     * 去算右边的最大面积且存储在数组中，
     */
    private static class Solution {
        public int largestRectangleArea(int[] heights) {
            int[] areas = new int[heights.length];
            Arrays.fill(areas, 0);
            int max = 0;
            for (int i = 0; i < heights.length; i++) {
                max = Math.max(max, doFindArea(heights, i, areas));
            }
            return max;
        }

        /**
         * 找到这个位置的最大面积
         *
         * @param heights
         * @param posi
         * @param areas
         * @return
         */
        private int doFindArea(int[] heights, int posi, int[] areas) {
            // 当前最大面积不存在，开始查
            if (areas[posi] == 0) {
                areas[posi] = heights[posi];
                int i = posi, j = posi;
                int leftMax = 0;
                int rightMax = 0;
                // 往左走，找到左边比自己小的最大值，本身区域也增加
                while (i > 0) {
                    i--;
                    // 左边的比当前的高
                    if (heights[i] > heights[posi]) areas[posi] += heights[posi];
                    // 往左走的时候如果左边和自己一样 那么值也一样
                    else if (heights[i] == heights[posi]) {
                        areas[posi] = areas[i];
                        return areas[posi];
                    }
                    else {
                        // 左边的比当前低
                        // 先看左边是否有最大面积，没有就算
                        if (areas[i] == 0) doFindArea(heights, i, areas);
                        leftMax = areas[i];
                        break;
                    }
                }
                // 往右走，找到右边比自己小的最大值，本身区域也增加
                while (j < heights.length - 1) {
                    j++;
                    if (heights[j] >= heights[posi]) areas[posi] += heights[posi];
                    else {
                        if (areas[j] == 0) doFindArea(heights, j, areas);
                        rightMax = areas[j];
                        break;
                    }
                }
                // 现在有了本身的大小，有了遇到的两个比自身小的数的区域最大值
                areas[posi] = Math.max(Math.max(areas[posi], leftMax), rightMax);
            }
            return areas[posi];
        }
    }
}

```