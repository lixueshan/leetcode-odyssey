# 剑指Offer1-66-构建乘积数组

[剑指 Offer 66. 构建乘积数组](https://leetcode.cn/problems/gou-jian-cheng-ji-shu-zu-lcof/) (中等)

<br />

## 解法一：前缀和思想 (前后缀积)

### 算法描述

前缀和思想的应用。一个前缀乘积数组 + 一个后缀乘积数组即可。这两个数组的大小 比 $a$ 大 1，方便统一处理。也可以只计算前缀积或后缀积中的一个，另一个在计算结果的同时同步计算。

<br />

### 时空复杂度

时间复杂度：$O(n)$

空间复杂度：$O(n)$ 。若采用空间优化的方法，并且输出数组不算入空间复杂度的话，为 $O(1)$ 。 

<br />

### 代码

```java
class Solution {
    public int[] constructArr(int[] a) {
        if(a.length == 0) return new int[]{};
        int n = a.length;
        int[] preMul = new int[n + 1], postMul = new int[n + 1], b = new int[n];
        preMul[0] = 1; postMul[n] = 1;
        for(int i = 0; i < n; i++){ // 前缀乘 preMul[i] 表示 a[i] 之前(不含)元素乘积
            preMul[i + 1] = preMul[i] * a[i];
        }
        for(int i =  n - 1; i >= 0; i--){ // 后缀乘 postMul[i + 1] 表示 a[i] 之后(不含)元素乘积
            postMul[i] = postMul[i + 1] * a[i];
        }
        for(int i = 0; i < n; i++){
            b[i] = preMul[i] * postMul[i + 1];
        }
        return b;
    }
}

// 空间优化版本
class Solution {
    public int[] constructArr(int[] a) {
        if(a.length == 0) return new int[]{};
        int n = a.length;
        int[] postMul = new int[n + 1], b = new int[n];
        postMul[n] = 1;
        for(int i =  n - 1; i >= 0; i--){ // 后缀乘 postMul[i + 1] 表示 a[i] 之后(不含)元素乘积
            postMul[i] = postMul[i + 1] * a[i];
        }
        int preMul = 1;
        for(int i = 0; i < n; i++){ // 在构建 b[] 的同时计算前缀乘
            b[i] = preMul * postMul[i + 1];
            preMul *= a[i];
        }
        return b;
    }
}
```

<br />

