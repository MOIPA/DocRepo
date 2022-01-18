title: LeetCode 每日三道题10 tree
author: tr
tags:
  - LeetCode
categories:
  - LeetCode
date: 2021-06-21 11:38:00
---
#  Construct Binary Tree from Inorder and Postorder Traversal


![upload successful](/images/pasted-76.png)
106 同 105一样 同样规则构建树，只不过计算左右集合时需要注意下！ 55%

<!--more-->
```java
package com.leetcode;

public class ConstructBinaryTreefromPostorderandInorderTraversal106 {
    public static void main(String[] args) {
        TreeNode root = new Solution().buildTree(
                new int[]{9, 3, 15, 20, 7}, new int[]{9, 15, 7, 20, 3}
        );

    }

    /**
     * 同105 用中序和后序构造树
     */
    private static class Solution {
        public TreeNode buildTree(int[] inorder, int[] postorder) {
            return constructTree(postorder, 0, postorder.length - 1, inorder, 0, inorder.length - 1);
        }

        public TreeNode constructTree(int[] posOrder, int posS, int posE, int[] inOrder, int inS, int inE) {
            // 无遍历结果
            if (posS > posE || inS > inE) return null;
            else if (posS == posE) return new TreeNode(posOrder[posE]);
            // 找到根节点
            TreeNode root = new TreeNode(posOrder[posE]);
            // 找到中序遍历的左边和右边两个集合
            int rootPosi;
            for (rootPosi = inS; rootPosi <= inE; rootPosi++) {
                if (root.val == inOrder[rootPosi]) break;
            }
            int leftTreeLen = rootPosi - inS;
            root.left = constructTree(posOrder, posS, posS + leftTreeLen - 1, inOrder, inS, rootPosi - 1);
            root.right = constructTree(posOrder, posS + leftTreeLen, posE - 1, inOrder, rootPosi + 1, inE);
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

# Pascal's Triangle 118

easy 帕斯卡三角形

```java
package com.leetcode;

import java.util.Arrays;
import java.util.LinkedList;
import java.util.List;

public class PascalTriangle118 {
    public static void main(String[] args){
        System.out.println(new Solution().generate(5));
    }

    /**
     * 帕斯卡三角形
     *      1
     *    1   1
     *   1  2  1
     *  1 3   3  1
     * 1 4  6   4  1
     */
    private static class Solution{
        public List<List<Integer>> generate(int numRows) {
            List<List<Integer>> res = new LinkedList<>();
            List<Integer> tmp = Arrays.asList(1);
            res.add(tmp);
            for (int i = 1; i < numRows; i++) {
                List<Integer> curr = new LinkedList();
                curr.add(1);
                for (int j = 0; j < tmp.size() - 1; j++) {
                    curr.add(tmp.get(j)+tmp.get(j+1));
                }
                curr.add(1);
                tmp = curr;
                res.add(tmp);
            }
            return res;
        }
    }
}

```

# Pascal's TriangleII 119

和上题一样 只不过只需要输出最上层，这个是o(k)的c++算法

```c++
class Solution {
public:
    vector<int> getRow(int rowIndex) {
        vector<int> A(rowIndex+1, 0);
        A[0] = 1;
        for(int i=1; i<rowIndex+1; i++)
            for(int j=i; j>=1; j--)
                A[j] += A[j-1];
        return A;
    }
};
```