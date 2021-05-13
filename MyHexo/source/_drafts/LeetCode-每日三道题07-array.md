title: LeetCode 每日三道题07 array
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