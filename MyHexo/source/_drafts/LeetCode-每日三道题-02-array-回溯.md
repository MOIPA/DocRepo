title: LeetCode 每日三道题 02 array + 回溯
author: tr
tags:
  - LeetCode
  - Array
categories:
  - LeetCode
date: 2021-04-08 15:44:00
---
# Combination Sum 39


<!--more-->

```java
package com.leetcode;

import java.util.Arrays;
import java.util.Comparator;
import java.util.LinkedList;
import java.util.List;
import java.util.stream.Collectors;

public class CombinationSum39 {
    public static void main(String[] args) {
        List<List<Integer>> lists = new Solution().combinationSum(new int[]{2, 7, 6, 3, 5, 1}, 9);
        lists.forEach(System.out::println);
        System.out.println(lists.size());

    }


    /**
     * 题目：用给的数组 拼成目标数，数组的数可以被使用多次
     * Input: candidates = [2,3,6,7], target = 7
     * Output: [[2,2,3],[7]]
     * <p>
     * 思路：留一个list用于记录使用过的数，递归，每次遍历数组的数，用target减去，如果为0说明路径可行，添加
     * 但是只这样的话，结果会有重复，为了不重复，
     */
    static class Solution {
        public List<List<Integer>> combinationSum(int[] candidates, int target) {
            List<List<Integer>> res = new LinkedList<>();
            generatePath(Arrays.stream(candidates).sorted().toArray(), 0, target, new LinkedList<>(), res);
//            res.stream().forEach(x -> x.sort(Comparator.comparingInt(x2 -> x2)));
//            return res.stream().distinct().collect(Collectors.toList());
            return res;
        }

        private void generatePath(int[] candidates, int start, int target, List<Integer> path, List<List<Integer>> allPath) {
            boolean deletedLastPath = false;
            for (int i = start; i < candidates.length; i++) {
                if (target - candidates[i] == 0) {
                    path.add(candidates[i]);
                    allPath.add(new LinkedList<>(path));
                    path.remove(path.size() - 1);
                    path.remove(path.size() - 1);
                    break;
                } else if (target - candidates[i] > 0) {
                    path.add(candidates[i]);
                    generatePath(candidates, i, target - candidates[i], path, allPath);
                }else{
                    if (!deletedLastPath) {
                        // 本层结束后删除本层添加的节点
                        path.remove(path.size() - 1);
                        deletedLastPath = true;
                    }
                }
            }
        }

    }

}

```

# Combination Sum II 40

```java
package com.leetcode;

import java.util.Arrays;
import java.util.LinkedList;
import java.util.List;

public class CombinationSumII40 {
    public static void main(String[] args) {
        List<List<Integer>> lists = new Solution().combinationSum(new int[]{1, 1}, 2);
        lists.forEach(System.out::println);
        System.out.println(lists.size());

    }


    /**
     * 题目：用给的数组 拼成目标数，数组的数可以使用一次  39题的加强版
     * Input: candidates = [2,3,6,7], target = 7
     * Output: [[2,2,3],[7]]
     * <p>
     * 思路：留一个list用于记录使用过的数，递归，每次遍历数组的数，用target减去，如果为0说明路径可行，添加
     * 但是只这样的话，结果会有重复，为了不重复，
     *
     * 和 39 题对比 ，为了防止重复 先排序，后再在循环中跳过重复字符。
     * 为了每个字符只使用一次，排序后下一层传递的开始值为当前值+1
     *
     * Runtime: 6 ms, faster than 28.62% of Java online submissions for Combination Sum II.
     * Memory Usage: 39.5 MB, less than 22.30% of Java online submissions for Combination Sum II.
     */
    static class Solution {
        public List<List<Integer>> combinationSum(int[] candidates, int target) {
            List<List<Integer>> res = new LinkedList<>();
            generatePath(Arrays.stream(candidates).sorted().toArray(), 0, target, new LinkedList<>(), res);
            return res;
        }

        private void generatePath(int[] candidates, int start, int target, List<Integer> path, List<List<Integer>> allPath) {
            for (int i = start; i < candidates.length; i++) {
                if (target - candidates[i] == 0) {
                    path.add(candidates[i]);
                    allPath.add(new LinkedList<>(path));
                    path.remove(path.size() - 1);
                    break;
                } else if (target - candidates[i] > 0) {
                    path.add(candidates[i]);
                    generatePath(candidates, i + 1, target - candidates[i], path, allPath);
                    path.remove(path.size() - 1);
                }
                // 为了防止重复 跳到最后一个不一致的
                while (i < candidates.length - 1 && candidates[i] == candidates[i + 1]) i++;
            }
        }

    }

}

```

# First Missing Positive 41

hard题

网上找的图
![upload successful](/images/pasted-47.png)

```java
 private static class Solution {
        public int firstMissingPositive(int[] nums) {
            for (int i = 0; i < nums.length; i++) {
                // 换
                if (nums[i] != i + 1 && nums[i] > 0 && nums[i] <= nums.length && nums[i] != nums[nums[i] - 1]) {
                    swap(nums, i, nums[i] - 1);
                    i--;
                }
            }
            // 桶排序后的数组顺序遍历不满足nums[i]= i+1的就是
            for (int i = 0; i < nums.length; i++) {
                if (nums[i] != i + 1) return i + 1;
            }
            return nums.length + 1;
        }

        private void swap(int[] nums, int a, int b) {
            int tmp = nums[a];
            nums[a] = nums[b];
            nums[b] = tmp;
        }
    }
```