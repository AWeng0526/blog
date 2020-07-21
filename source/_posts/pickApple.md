---
title: 一道字节面试题
date: 2020-07-08 14:08:45
tags:
---

朋友前两天的字节后端实习二面真题,本人回答链接:https://leetcode-cn.com/circle/article/NA6h7i/

<!-----more---->

### 题目描述

> 牛牛有一个苹果园.又到了一年一度的收获季,牛牛现在要去采摘苹果买给市场的摊贩们.
> 牛牛的果园里面有n棵苹果树,第i棵苹果树上有ai个果子.
> 牛牛为了保证果子的新鲜程度,每天都会去苹果树上采摘果子.
> 牛牛特意安排一个计划表： 
> 
> 计划m天去采摘果子.
> 对于第i天,它会去所有果树上轮流采摘bi个果子.
> 如果对于第i天,某棵果树上没有bi个果子,那么它只会把当前果树上的果子采摘完.
> 牛牛想知道它每天能供应多少个苹果给市场的摊贩们.

#### 示例

输入

> [10,20,10],[5,7,2]

输出

> [15,17,2]

说明

> 苹果树上的果子变化[10,20,10]-->[5,15,5]-->[0,8,0]-->[0,6,0]

### 题解

#### 法一:BF

暴力循环比较简单,此处不再赘述

#### 法二:预排序+二分

法二的思想在于做加法而非做减法.每天摘完果子后我们不去更新果树上的果子数目,而是考虑明天摘更多的果子.

以上文示例为例:

- 果树预排序后得到[10,10,20]

- 第一天采摘5颗苹果,使用二分查找.

[![UVm4sJ.png](https://s1.ax1x.com/2020/07/08/UVm4sJ.png)](https://imgchr.com/i/UVm4sJ)

第一天采摘的苹果数目=3*5=15.

我们还需要更新start(虽然这一步中更新没啥用),原因后文会解释.

- 第二天我们采摘5+7=12颗苹果,使用二分查找.

![UVmqJK.png](https://s1.ax1x.com/2020/07/08/UVmqJK.png)

查找结果i=2,代表左边的果树上苹果数目不足12颗,需要遍历每棵果树计算总和.而对于右边的果树,只需做一个乘法即可.

即10+10+(2-2+1)*12=32,但这32棵苹果里包括了之前摘的,需要把之前摘的所有苹果数目减去.最终得到32-15=17颗苹果.

还需将start更新为2,原因在于,第三天采摘数目大于等于12,我们可以缩小查找范围.

- 第三天采摘5+7+2=14颗苹果,二分查找结果如下:

[![UVnTXQ.png](https://s1.ax1x.com/2020/07/08/UVnTXQ.png)](https://imgchr.com/i/UVnTXQ)

剩余步骤同上,不再赘述.

### 代码

```java
import java.util.Arrays;

class Solution {
    public static void main(String[] args) {
        int[] ans = new Solution().pickApple(new int[] { 10, 20, 10 }, new int[] { 5, 7, 2 });
        System.out.println(Arrays.toString(ans));
    }

    public int[] pickApple(int[] appleNum, int[] pickNum) {
        Arrays.sort(appleNum);
        // 第n天要摘的果子数
        int pickSum = 0;
        // 前面n天的苹果数
        int prevSum = 0;
        int start = 0;
        int n = appleNum.length;
        int days = pickNum.length;
        int[] ans = new int[days];
        for (int i = 0; i < days; i++) {
            int todayAppleSum = 0;
            pickSum += pickNum[i];
            int idx = Arrays.binarySearch(appleNum, start, n - 1, pickSum);
            if (idx < 0) {
                idx = -idx - 1;
            }
            start = idx;
            todayAppleSum += (n - idx) * pickSum;
            for (int j = 0; j < idx; j++) {
                todayAppleSum += appleNum[j];
            }
            todayAppleSum -= prevSum;
            prevSum += todayAppleSum;
            ans[i] = todayAppleSum;
        }
        return ans;
    }
}
```

### 时间复杂度

这里我们假设有n棵树,k天,每次更新start都使得查找范围缩小一半(后面会解释).

- 第一天需要在n个果树中二分查找,时间复杂度为log n.需要计算n/2棵果树上的苹果总和,时间复杂度为 n/2.

- 第一天需要在n/2个果树中二分查找,时间复杂度为log n/2.需要计算n/4棵果树上的苹果总和,时间复杂度为 n/4.

- 第k天需要在n/(2^k-1)个果树中二分查找,时间复杂度为log n/(2^k-1).需要计算n/(2^k)棵果树上的苹果总和,时间复杂度为 n/(2^k).

最终的时间复杂度为 log n + n/2 + log n/2 + n/4 ...... + log n/(2^k-1) + n/(2^k)

当n>4时,log n < n/2,即 log n + n/2 < n/2 + n/2 = n.

由于时间复杂度公式中每项是递减的,故log n + n/2 + log n/2 + n/4 ...... + log n/(2^k-1) + n/(2^k) < kn

#### 补充说明

这里再解释一下为什么每次查找范围要缩小一半.

如果每次查找结果恰好位于正中间,那么二分查找的时间复杂度是O(1),远小于O(n).如果要想满足二分查找的时间复杂度为O(logn),那么无非是下面俩种情况:

- 黄色为第一次二分结果,蓝色为第二次,可以看到,第三次二分无论是落在5/7哪个索引上,余下的部分都小于n/2,也就是我们扫描的时间复杂度会更小.

[![UVQi7j.png](https://s1.ax1x.com/2020/07/08/UVQi7j.png)](https://imgchr.com/i/UVQi7j)

- 同上,无论是落在1/3哪个索引上,余下部分都大于n/2,也就是我们的扫描时间复杂度会更大.

[![UVQcgf.png](https://s1.ax1x.com/2020/07/08/UVQcgf.png)](https://imgchr.com/i/UVQcgf)

综上,情况1与情况2出现的概率相等,故取平均值,设定每次缩小一半.

### 优化

可以使用前缀和数组进行优化,这样无需去扫描苹果数不足的果树.

#### 代码

```java
import java.util.Arrays;

class Solution {
    public static void main(String[] args) {
        int[] ans = new Solution().pickApple(new int[] { 10, 20, 10 }, new int[] { 5, 7, 2 });
        System.out.println(Arrays.toString(ans));
    }

    public int[] pickApple(int[] appleNum, int[] pickNum) {
        Arrays.sort(appleNum);
        // 第n天要摘的果子数
        int pickSum = 0;
        // 前面n天的苹果数
        int prevSum = 0;
        int start = 0;
        int n = appleNum.length;
        int days = pickNum.length;
        int[] ans = new int[days];
        int[] prefix = new int[n + 1];
        prefix[0] = 0;
        for (int i = 1; i <= n; i++) {
            prefix[i] = prefix[i - 1] + appleNum[i - 1];
        }
        for (int i = 0; i < days; i++) {
            int todayAppleSum = 0;
            pickSum += pickNum[i];
            int idx = Arrays.binarySearch(appleNum, start, n - 1, pickSum);
            if (idx < 0) {
                idx = -idx - 1;
            }
            start = idx;
            todayAppleSum += (n - idx) * pickSum;
            todayAppleSum += prefix[idx];
            todayAppleSum -= prevSum;
            prevSum += todayAppleSum;
            ans[i] = todayAppleSum;
        }
        return ans;
    }
}
```

#### 时间复杂度

通过前面的分析很容易的可以得出时间复杂度为 log n + log n/2 + ... + log n/(2^k-1)