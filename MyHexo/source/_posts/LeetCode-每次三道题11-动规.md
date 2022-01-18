title: LeetCode 每日三道题11 动规
author: tr
tags:
  - LeetCode
  - Dp
  - ''
categories:
  - LeetCode
date: 2021-06-23 10:20:00
---
# Triangle

medium 经典入门dp问题 一次ac

  
     * 计算三角形中每行最小值组成的路径 实际也是个经典的dp问题，从低到上
     *    2
     *   3 4
     *  6 5 7
     * 4 1 8 3
     *
     * 从最底层开始：4，1，8，3 往上分别为 （10，7），（6，13），（15，10） 选出最小的7，6，10
     * 继续：9，10  最后：11
<!--more-->

```java

    /**
     * 计算三角形中每行最小值组成的路径 实际也是个经典的dp问题，从低到上
     *    2
     *   3 4
     *  6 5 7
     * 4 1 8 3
     *
     * 从最底层开始：4，1，8，3 往上分别为 （10，7），（6，13），（15，10） 选出最小的7，6，10
     * 继续：9，10  最后：11
     */
    private static class Solution {
        public int minimumTotal(List<List<Integer>> triangle) {
//            return triangle.stream().map(x -> x.stream().min(Comparator.comparingInt(n -> n))).mapToInt(Optional::get).sum();
            int[] costs = new int[triangle.get(triangle.size() - 1).size()];
            for (int i = 0; i < triangle.get(triangle.size() - 1).size(); i++)
                costs[i] = triangle.get(triangle.size() - 1).get(i);
            for (int i = triangle.size() - 2; i >= 0; i--)
                for (int j = 0; j < triangle.get(i).size(); j++)
                    costs[j] = Math.min(costs[j] + triangle.get(i).get(j), costs[j + 1] + triangle.get(i).get(j));
            return costs[0];
        }
    }
```

# Best Time to Buy and Sell Stock

	
![upload successful](/images/pasted-77.png)

![upload successful](/images/pasted-78.png)
easy 差值最大 o（n)复杂度的算法不太容易想得到，具体问题具体分析


```java
 private static class Solution {
        /**
         * 暴力算法  o(n^2)
         * @param prices
         * @return
         */
        public int maxProfit(int[] prices) {
            int max = 0;
            for (int i = 0; i < prices.length; i++) {
                for (int j = i; j <prices.length; j++) {
                    max = Math.max(prices[j] - prices[i], max);
                }
            }
            return max;
        }
        /**
         * 牛逼算法  o(n)
         */
        public int maxProfit2(int[] prices) {
            int max = 0;
            int lowest = prices[0];
            for (int i = 1; i < prices.length; i++) {
                if (prices[i]>lowest) max = Math.max(max, prices[i] - lowest);
                else lowest = prices[i];
            }
            return max;
        }
    }
```

# Best Time to Buy and Sell Stock II 122

上题的变种，这次可以多次买入卖出，这题真……，想得到的话简单的不行，想不到的话……，最重要的是画成图看规律

![upload successful](/images/pasted-79.png)

```java

 /**
     * 这题不应该是easy  很难想到这个算法和规律
     * <p>
     * 第一个解法： 实际有个规律，就是所有的邻近高点-邻近低点的和 总是最大的！
     */
    private static class Solution {
        public int maxProfit(int[] prices) {
            int maxProfit = 0;
            int lowest;
            int highest;
            int i = 0;
            while (i < prices.length - 1) {
                // 找到最低点
                while (i < prices.length - 1 && prices[i] >= prices[i + 1]) i++;
                lowest = prices[i];
                while (i < prices.length - 1 && prices[i] <= prices[i + 1]) i++;
                highest = prices[i];
                maxProfit += highest - lowest;
            }
            return maxProfit;
        }

        /**
         * 牛逼算法  画成图实际所有的收益 不过是所有的上坡的和
         * @param prices
         * @return
         */
        public int maxProfit2(int[] prices) {
            int max = 0;
            for (int i = 0; i < prices.length - 1; i++) {
                if (prices[i]<prices[i+1]) max += prices[i + 1] - prices[i];
            }
            return max;
        }
    }
```

# Best Time to Buy and Sell Stock III 123

佛了 改改题目还能变成dp问题，在上一题的基础上添加限制条件：只能有两次买卖操作

第一种思路：

![upload successful](/images/pasted-80.png)

从最后一天开始：0，0.
倒数第二天：第一次最佳买入能赚3元
倒数第三天：第一次最佳买入赚1元，还没3元多，变为3元
倒数第四天：0-3一次+3的最大值=3，两次操作最大值=6，单次操作最大值4
.....

```java
 public int maxProfit2(int[] prices) {
            int[][] record = new int[2][prices.length];
            for (int i = prices.length - 2; i >= 0; i--) {
                int j = i + 1;
                while (j < prices.length) {
                    while (j < prices.length - 1 && prices[j] < prices[j + 1]) j++;
                    if (prices[j] - prices[i] > record[0][i]) {
                        record[0][i] = prices[j] - prices[i];
                        // 算第二次操作
                        if (j < prices.length - 1) {
                            record[1][i] = Math.max(record[1][i], record[0][j + 1] + record[0][i]);
                        }
                    }
                    j++;
                }
                record[0][i] = Math.max(record[0][i], record[0][i + 1]);
                record[1][i] = Math.max(record[1][i], record[1][i + 1]);
            }
            return Math.max(record[0][0], record[1][0]);
        }
```


![upload successful](/images/pasted-81.png)

```java
int maxProfit(vector<int>& prices) {
    if(prices.empty()) return 0;
    //进行初始化，第一天 s1 将股票买入，其他状态全部初始化为最小值
    int s1=-prices[0],s2=INT_MIN,s3=INT_MIN,s4=INT_MIN;

    for(int i=1;i<prices.size();++i) {            
        s1 = max(s1, -prices[i]); //买入价格更低的股
        s2 = max(s2, s1+prices[i]); //卖出当前股，或者不操作
        s3 = max(s3, s2-prices[i]); //第二次买入，或者不操作
        s4 = max(s4, s3+prices[i]); //第二次卖出，或者不操作
    }
    return max(0,s4);
}
```