# 剑指Offer1-51-数组中的逆序对

[剑指 Offer 51. 数组中的逆序对](https://leetcode.cn/problems/shu-zu-zhong-de-ni-xu-dui-lcof/) (困难)

<br />

## 解法一：归并排序

### 算法描述

归并排序的经典应用，merge方法与常规归并排序只有一行的差异。只需要理解如何在「归并」过程中累计「逆序数」即可。对两个排序好的左右数组进行「归并」时，比较左数组的当前数nums[l]与右数组的当前数nums[r]，若nums[r] < nums[l]，说明nums[r]对从nums[l]开始的，左数组剩余的所有数都「逆序」，于是累计此数量，每次比较左数组当前数与右数组当前数时，都执行此累计。不难发现该累计只发生在merge方法的nums[r] < nums[l]分支里，因此相比常规的归并排序merge方法，仅有一行不同。代码中仅展示「自底向上归并」，其他方式的归并实现类似，仅需在merge方法中添加一行逆序数累计语句即可。

<br />

### 时空复杂度

时间复杂度：$O(nlogn)$

空间复杂度：辅助数组空间 $O(n)$

<br />

### 代码

```java
// 自底向上归并
class Solution {
    int count = 0;
    public int reversePairs(int[] nums) {
        if(nums.length < 2) return 0;
        int n = nums.length;
        int[] tmp = new int[n];
        for(int gap = 1; gap < n; gap *= 2){
            for(int l = 0; l < n - gap; l += 2 * gap) {
                // 调用非原地合并方法。leftEnd = left+gap-1; rightEnd = left+2*gap-1;
                merge(nums, tmp, l, l + gap - 1, Math.min(l + 2 * gap - 1, n - 1));
            }
        }
        return count;
    }
    // 非原地合并方法
    private void merge(int[] nums, int[] tmp, int l, int lEnd, int rEnd) {
        int r = lEnd + 1, start = l, cur = l;
        while (l <= lEnd && r <= rEnd) {
            if (nums[l] <= nums[r]) tmp[cur++] = nums[l++];
            else {
                count += lEnd - l + 1; // 与归并排序merge方法仅有此行的差异
                tmp[cur++] = nums[r++];
            }
        }
        while (l <= lEnd) tmp[cur++] = nums[l++]; // 左数组仍有剩余
        while (r <= rEnd) tmp[cur++] = nums[r++]; // 右数组仍有剩余
        for(; start <= rEnd; start++) { // 拷回nums
            nums[start] = tmp[start];
        }
    }
}
```

<br />

