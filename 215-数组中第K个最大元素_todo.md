# 215-数组中的第K个最大元素

[215. 数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/) (中等)

<br />

## 解法一：快速选择 + 二分查找 (单轴快排序)

### 算法描述

采用快排思想，不单是因为快排本身速度快，还在于解决这个问题无需完成所有元素的排序。单轴快排每次确定一个轴，并检查这个轴的下标是否对应第 $k$ 个大的数的下标，如果对应，直接返回该轴的值。第 $k$ 大也就是从小往大数数第下标为 $n - k$ 的元素。「代码」中给出随机轴快速排序实现。

<br />

### 时空复杂度

时间复杂度：平均 $O(n)$。 

空间复杂度：$O(1)$

<br />

### 代码

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        int n = nums.length, l = 0, r = n - 1;
        while(l <= r){
            int rand = new Random().nextInt(r - l + 1) + l; // 随机轴
            swap(nums, l, rand);
            int p = partition(nums, l, r);
            if(p == n - k) return nums[p]; // 第 k 大也就是从小往大数数第下标为 n - k 的元素
            else if (p < n - k) l = p + 1;
            else r = p - 1;
        }
        return -1; // 若返回-1，说明k不在nums.length范围内
    }
    private int partition(int[] nums, int l, int r) {
        int p = l, index = p + 1;
        // 注意此时right是坐标，要执行到最后一个元素，所以是<=
        for (int i = index; i <= r; i++) {
            if (nums[i] < nums[p]) {
                swap(nums, index, i);
                index++;
            }
        }
        swap(nums, p, index - 1);
        return index - 1;
    }
    private void swap(int[] nums, int i, int j){
        int tmp = nums[i];
        nums[i] = nums[j];
        nums[j] = tmp;
    }
}
```

<br />

## 解法一：快速选择 (双轴快排)

### 算法描述

与单轴快排实现的单轴快速选择思想一致，但采用双轴实现。双轴快排一次可以得到两个轴，每次找轴都只在三个区间中的一个中寻找，其他两个区间无需排序所以嗖嗖快。平均时间复杂度为 $O(n)$ ，空间复杂度为 $O(1)$ 。

<br />

### 时空复杂度

时间复杂度：平均 $O(n)$ 。

空间复杂度：$O(1)$

<br />

### 代码

```java
class Solution {
    public int findKthLargest(int[] nums, int k) { // 1,2,3,4,5,6 5,4,3,2,1,0
        int left = 0, right = nums.length - 1, target = nums.length - k; // 注意我这个快排是递增排序，所以第k个大的数在排序后的数组中下标为target
        int[] pivots = dualPivotQuickSort(nums, 0, nums.length - 1);
        int pivot1 = pivots[0], pivot2 = pivots[1]; // 找到第一次双轴
        while(pivot1 != target && pivot2 != target) { 
            if(target < pivot1) right = pivot1 - 1; // 目标在区间1
            else if(target > pivot2) left = pivot2 + 1; // 目标在区间3
            else { // 目标在区间2
                left = pivot1 + 1;
                right = pivot2 - 1;
            }
            pivots = dualPivotQuickSort(nums, left, right);
            pivot1 = pivots[0];
            pivot2 = pivots[1];
        }
         return pivot1 == target ? nums[pivot1] : nums[pivot2];
    }
    // 双轴快排
    private int[] dualPivotQuickSort(int[] arr, int left, int right) {
        int[] res = new int[2];
        if(arr[left] > arr[right]) swap(arr, left, right); // 令左右两端元素中较小者居左
        int index = left + 1; // 当前考察元素下标
        int lower = left + 1; // 用于推进到pivot1最终位置的动态下标，总有[left, lower)确定在区间1中
        int upper = right - 1; // 用于推进到pivot2最终位置的动态下标，总有(upper, right]确定在区间3中
        while(index <= upper) { // upper以右（不含upper）的元素都是确定在区间3的元素，所以考察元素的右界是end
            if (arr[index] < arr[left]) { // 若arr[index] < arr[left]，即arr[index]小于左轴值，则arr[index]位于区间1
                swap(arr, index, lower); // 交换arr[index]和arr[lower]，配合后一条lower++，保证arr[index]位于区间1
                lower++; // lower++，扩展区间1，lower位置向右一位靠近pivot1的最终位置
            }
            else if(arr[index] > arr[right]) { // 若arr[index] > arr[right]，即arr[index]大于右轴值，则arr[index]位于区间3
                while(arr[upper] > arr[right] && index < upper) { // 先扩展区间3，使得while结束后upper以右（不含upper）的元素都位于区间3
                    upper--; // 区间3左扩不可使index == upper，否则之后的第二条upper--将导致upper为一个已经确定了区间归属的元素的位置（即index - 1）
                }
                swap(arr, index, upper); // 交换arr[index]和arr[upper]，配合后一条upper--，保证arr[index]位于区间3
                upper--;
                if(arr[index] < arr[left]) { // 上述交换后，index上的数字已经改变，arr[index]有可能在区间1或区间2
                    swap(arr, index, lower); // 交换arr[index]和arr[lower]，配合后一条lower++，保证arr[index]位于区间1
                    lower++; // lower++，扩展区间1
                }
            }
            index++; // 考察下一个数字
        }
        lower--; // 确定左轴
        upper++; // 确定右轴
        swap(arr, left, lower); // 左轴归位
        swap(arr, upper, right); // 右轴归位
        return new int[]{lower, upper};
    }
    private void swap(int[] nums, int i, int j) {
        int tmp = nums[i];
        nums[i] = nums[j];
        nums[j] = tmp;
    }
}
```

<br />



