title: LeetCode 每日三道题09 array
author: tr
tags:
  - LeetCode
categories:
  - LeetCode
date: 2021-06-20 13:12:00
---
# MaximalRectangle

dp思路：和上一题一样 84， 转换下就好

上一题的84题我实际用了递归和dp和这题不一样，这里的思路是：每个柱子的面积都是`长度*高度`，这个长度就是`柱子右边的第一个小于柱子高度的柱子  -  柱子左边的第一个小于柱子高度的柱子 - 1`，因此主要点在算长度。

这题的dp思想体现在算宽度，每层宽度可以通过上层算出。	


![upload successful](/images/pasted-72.png)

```java

        public int maximalRectangle2(char[][] matrix) {
            // 思路： 和84题一样
            if (matrix.length == 0) return 0;
            int m = matrix.length;
            int n = matrix[0].length;
            int[] left = new int[n];
            int[] right = new int[n];
            int[] height = new int[n];
            Arrays.fill(left, 0);
            Arrays.fill(right, 0);
            Arrays.fill(height, 0);
            int maxA = 0;
            for (int i = 0; i < m; i++) {
                int cur_left = 0, cur_right = n;
                for (int j = 0; j < n; j++) { // compute height (can do this from either side)
                    if (matrix[i][j] == '1') height[j]++;
                    else height[j] = 0;
                }
                for (int j = 0; j < n; j++) { // compute left (from left to right)
                    if (matrix[i][j] == '1') left[j] = Math.max(left[j], cur_left);
                    else {
                        left[j] = 0;
                        cur_left = j + 1;
                    }
                }
                // compute right (from right to left)
                for (int j = n - 1; j >= 0; j--) {
                    if (matrix[i][j] == '1') right[j] = Math.min(right[j], cur_right);
                    else {
                        right[j] = n;
                        cur_right = j;
                    }
                }
                // compute the area of rectangle (can do this from either side)
                for (int j = 0; j < n; j++)
                    maxA = Math.max(maxA, (right[j] - left[j]) * height[j]);
            }
            return maxA;
        }
    }
```

# merge sorted array  o(m+n)

```java
package com.leetcode;

import java.util.Arrays;

public class MergeSortedArray88 {
    public static void main(String[] args) {
        int[] a = new int[]{1, 2, 4, 5, 6, 0};
        int[] b = new int[]{3};
        new Solution().merge(a, 5, b, 1);
        System.out.println(Arrays.toString(a));
    }

    /**
     * 思路：把前m个数放到nums1后面m个上，前面m个当作空的用来存放结果
     */
    private static class Solution {
        public void merge(int[] nums1, int m, int[] nums2, int n) {
            if (n == 0) return;
            if (m == 0) {
                for (int i = 0; i < nums1.length; i++) {
                    nums1[i] = nums2[i];
                }
                nums1[0] = nums2[0];
                return;
            }
            // 不开辟多余空间的写法且速度为 o(m+n)
            for (int i = m - 1; i >= 0; i--) {
                nums1[i + nums1.length - m] = nums1[i];
            }
            int p1 = nums1.length - m;
            int p2 = 0;
            int pres = 0;
            while (pres < nums1.length) {
                if (p1 == nums1.length) {
                    for (; p2 < n; p2++) {
                        nums1[pres++] = nums2[p2];
                    }
                } else if (p2 == n) {
                    return;
                } else {
                    if (nums1[p1] < nums2[p2]) nums1[pres++] = nums1[p1++];
                    else nums1[pres++] = nums2[p2++];
                }
            }
        }

        public void merge2(int[] nums1, int m, int[] nums2, int n) {
            int p1 = 0;
            int p2 = 0;
            int pres = 0;
            int[] res = new int[nums1.length];
            while (p1 < m || p2 < n) {
                if (p1 == m) {
                    for (; p2 < n; p2++) {
                        res[pres++] = nums2[p2];
                    }
                } else if (p2 == n) {
                    for (; p1 < m; p1++) {
                        res[pres++] = nums1[p1];
                    }
                } else {
                    if (nums1[p1] < nums2[p2]) res[pres++] = nums1[p1++];
                    else res[pres++] = nums2[p2++];
                }
            }
            nums1 = res;
        }
    }
}

```

# Construct BinaryTree from Preorder and Inorder Traversal

不难
![upload successful](/images/pasted-75.png)

```java

package com.leetcode;

import java.util.Arrays;

public class ConstructBinaryTreefromPreorderandInorderTraversal105 {
    public static void main(String[] args) {
        TreeNode root = new Solution().buildTree(
                new int[]{3,9,20,15,7}, new int[]{9,3,15,20,7}
        );
//        System.out.println(Arrays.toString());

    }

    private static class Solution {
        public TreeNode buildTree(int[] preorder, int[] inorder) {
            return constructTree(preorder, 0, preorder.length - 1, inorder, 0, inorder.length - 1);
        }

        /**
         * @param preOrder 前序遍历结果
         * @param preS     结果开始下标
         * @param preE     结果结束下标
         * @param inOrder
         * @param inS
         * @param inE
         * @return
         */
        public TreeNode constructTree(int[] preOrder, int preS, int preE, int[] inOrder, int inS, int inE) {
            // 无遍历结果
            if (preS > preE || inS > inE) return null;
            else if (preS == preE) return new TreeNode(preOrder[preS]);
            // 找到根节点
            TreeNode root = new TreeNode(preOrder[preS]);
            // 找到中序遍历的左边和右边两个集合
            int rootPosi;
            for (rootPosi = inS; rootPosi <= inE; rootPosi++) {
                if (root.val == inOrder[rootPosi]) break;
            }
            // 找到前序遍历的左右两个集合
//            int preEnd;
//            for (preEnd = preS + 1; preEnd <= preE; preEnd++) {
//                // 判断是否在左集合内
//                boolean ifContains = false;
//                for (int i = inS; i <= rootPosi - 1; i++) {
//                    if (inOrder[i] == preOrder[preEnd]) ifContains = true;
//                }
//                if (!ifContains) {
//                    break;
//                }
//            }
//            preEnd--;
            int leftTreeLen = rootPosi-inS;
            root.left = constructTree(preOrder, preS + 1, preS+leftTreeLen, inOrder, inS, rootPosi - 1);
            root.right = constructTree(preOrder, preS+leftTreeLen + 1, preE, inOrder, rootPosi + 1, inE);
            return root;
        }
    }

    private static class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;

        TreeNode() {
        }

        TreeNode(int val) {
            this.val = val;
        }

        TreeNode(int val, TreeNode left, TreeNode right) {
            this.val = val;
            this.left = left;
            this.right = right;
        }
    }
}

```