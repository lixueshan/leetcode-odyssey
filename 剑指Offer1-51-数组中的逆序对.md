# 剑指Offer1-51-数组中的逆序对

[剑指 Offer 51. 数组中的逆序对](https://leetcode.cn/problems/shu-zu-zhong-de-ni-xu-dui-lcof/) (困难)

<br />

## 解法一：归并排序

### 算法描述

归并排序的经典应用，$merge$ 方法与常规归并排序只有一行的差异。只需要理解如何 **在「归并」过程中累计「逆序数」** 即可。对两个排序好的左右数组进行「归并」时，比较左数组的当前数 $nums[l]$ 与右数组的当前数 $nums[r]$ ，若 $nums[r] < nums[l]$ ，说明 $nums[r]$ 与从 $nums[l]$ 开始的，左数组剩余的所有数都构成「逆序对」。于是累计此数量，每次比较左数组当前数与右数组当前数时，都执行此累计。不难发现该累计只发生在 $merge$ 方法的 $nums[r] < nums[l]$ 分支里，因此相比常规的归并排序 $merge$ 方法，仅有一行不同。代码中仅展示「自底向上归并」，其他方式的归并实现类似，仅需在 $merge$ 方法中添加一行逆序数累计语句即可。

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

## 解法二：树状数组

### 算法描述

本解法需要有「树状数组」的基础，运用树状数组求逆序对是简单的。我们先假定大小为 $n$ 的输入数组 $nums$ 的值为 $\{1,2,3,..,n\}$ 的一个排列。例如大小为 4 的 $nums = \{4,3,1,2\}$ ，逆序对为 4-3, 3-1, 4-1, 4-2, 3-2 共 5 对。计算方法为从左到右累计当前数字与其左侧数字组成的逆序对数，例如考察 4 时，左侧无数字，逆序对数为 0；接着考察 3，向左考察，逆序对数为 1；考察 1，逆序对数为 2；考察 2 ，逆序对数为 2。总逆序对数为 0+1+2+2=5。



在讨论树状数组解法之前，我们先给出最朴素两层循环做法 (该做法有许多不同的写法，这里选取一种与我们树状数组解法最为对应的写法)。

- 外层循环遍历 $nums$ ，对于当前元素 $nums[i]$ ，内层循环 **顺序遍历**  $nums[j](j>i)$ ，如果 $nums[i]>nums[j]$ ，则 $(nums[i],num[j])$ 是一个逆序对，令  $countGreater[j]++$ ， $countGreater[j]$ 表示 $nums[j]$ 与其左侧的数形成的逆序对的个数。

- 当程序结束时，每一个 $countGreater[i]$ 就是 $nums[i]$ 的逆序数 (逆序对形式为 $(x,nums[i]),x>nums[i]$ ，$x$ 是 $nums[i]$ 左侧的元素)，累计所有的 $countGreater[i]$ 就是所要求的 $nums$ 的逆序对总数。由于 $countGreater[i]$ 是在结束针对d $nums[i-1]$ 的内层循环时确定的，因此在开始 $nums[i]$ 内层循环前累计。

```java
class Solution {
    public int reversePairs(int[] nums) {
        int n = nums.length, ans = 0;
        int[] countGreater = new int[n];
        for(int i = 0; i < n; i++){
            ans += countGreater[i];
            for(int j = i + 1; j < n; j++){
                if(nums[i] > nums[j]) countGreater[j]++;
            }
        }
        return ans;
    }
}
```

请读者先查看「代码」中的松离散做法，后续对照讲解。

在主方法 $reversePairs$ 中，首先将 $nums$ 离散化 (后续的 $nums$ 均指离散化后的 $nums$ ) ，接着 **逆序遍历** $nums$ 元素，对每一个 $nums[i]$ ，依次执行 $preSum$ 和 $add$ 方法。我们知道 $preSum(k)$ 是求输入序列前 $k+1$ 项的和，$add(k,x)$ 是从首个包含下标为 $k$ 的元素的区间开始，为所有包含该元素的区间的区间和加上 $x$ 。那么代码中的 $add(nums[i]-1,1)$ 是什么意思呢？对比朴素做法，可知 $add(nums[i]-1,1)$ 相当于朴素做法的内层循环，用于计算「当前元素逆序数」，分析如下。

该方法使得 $tree$ 从 $tree[nums[i]-1]$ 开始，沿着父链上升，包含下标为 $nums[i]-1$ 的更大区间 $x$ 的 $tree[x]$ 加 1 。$tree$ 的下标范围 $[0,n-1]$ 与 (松离散化后的)  $nums$ 元素值的大小范围 $[1,n]$  **一一对应**  ，可以把 $add$ 方法 (以及 $preSum$ 方法) 中的 $nums[i]-1$ 等同于原数组中的 $nums[i]$ 方便理解。 $tree[i]$ 的意义也不再是区间和。

举例说明。假设松离散化后有 $nums = \{2,6,8,7,4,5,1,3\}$ ，逆序遍历 $nums[i]$ 并执行 $add(nums[i]-1,1)$ 。对第一个遍历到的 $nums[7]=3$ 执行 $add(2, 1)$ (操作 ① )，如下图。

![image.png](https://pic.leetcode-cn.com/1665827150-VTykUh-image.png)

$add$ 操作使得 $tree[2]$ , $tree[3]$ , $tree[7]$ 增加 1 ，其意义相当于为数组中 **大于等于** $nums[7]$ 的所有数 $nums[j]$ ，都执行 $countGreaterEqual[j]++$ 。注意，树状数组做法中不存在 $countGreaterEqual$ ，这里只是类比朴素方法中的 $CounterGreater$ 数组。那我们如何得到此时的 $countGreaterEqual[j]$ 呢？从代码中我们已经知道，通过执行 $preSum$ 方法求得。

由于我们从后往前求逆序对，因此我们考察左侧元素 $nums[j]$ 的 $countGreaterEqual[j]$ (不存在该数组，只是类比) 是否被正确更新了。由上图很容易看出，可通过 $preSum(nums[j]-1)$ 查询到。

```
preSum(nums[6] - 1) = preSum[0] = 0
preSum(nums[5] - 1) = preSum[4] = 1
preSum(nums[4] - 1) = preSum[3] = 1
preSum(nums[3] - 1) = preSum[6] = 1
preSum(nums[2] - 1) = preSum[7] = 1
preSum(nums[1] - 1) = preSum[5] = 1
preSum(nums[0] - 1) = preSum[1] = 0
```

所以本质上 $add(nums[i]-1,1)$ 操作相当于对 $nums$ 中的 **大于等于**  $nums[i]$ 的所有的 $nums[j]$ ，使其对应的 $countGreaterEqual[j]$ 加 1 。而 $counterGreaterEqual[j]$ 是通过 $preSum(nums[j]-1)$ 求得的。遍历 $nums[i]$ 时，以它为逆序对左元素 (即 $(nums[i], x)$ 形式，$x$ 是 $nums[i]$ 右侧的元素) 的逆序对数量在上一轮执行 $add[nums[i-1] - 1, 1]$ 后就完全确定了，因此先执行 $preSum$ 方法累计该逆序对数。

需要注意的是，当我们执行 $preSum(nums[i]-1)$ 时，求的是 $\{(nums[i],x), nums[i]≥x\}$  的数量 (对应 $counterGreaterEqual[i]$)，而逆序对要求的是 $\{(nums[i],x), nums[i]>x\}$ (对应 $counterGreater[i]$) ，即严格大于才算逆序。要如何处理呢？再次看向树状图，以 $preSum(6)$ 为例，其结果为 $tree[6]+tree[5]+tree[3]$ ，其中 $tree[6]$ 就代表了等于时带来的累计量，去掉这个累计量只需要向左侧移动一位即可，即求 $preSum(5)$ 。对应到题解代码中，即为 $preSum((nums[i]-1)-1)$ ，或写成 $preSum(nums[i]-2)$ ，对应了朴素解法中的 $counterGreater[i]$ 。



现在我们将树状数组解法与朴素解法对比如下。

| 操作                 | 朴素解法                                                     | 树状数组解法                                                 |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 累计当前元素的逆序数 | `ans += countGreater[i];`<br />$(x,nums[i]),x>nums[i]$ 形式逆序对 | `ans += bit.preSum(nums[i] - 2);`<br />$(nums[i],x),nums[i]>x$ 形式逆序对 |
| 计算当前元素的逆序数 | 内层循环<br />$(nums[i],x),nums[i]>x$ 形式逆序对             | `bit.add(nums[i] - 1, 1);`<br />$(x,nums[i]),x>nums[i]$ 形式逆序对 |

我们惊讶地发现，$add$ 方法本质上完成了朴素做法的内层循环所做的事，并且是以 $O(logn)$ 时间复杂度完成的。而且实际上完成得更多，因为朴素做法更新的是 $nums[i]$ 右侧的小于 $nums[i]$ 的 $nums[j] (j>i)$ 的 $countGreater[j]$ ，而树状数组 $add$ 方法是对所有大于等于 $nums[i]$ 的 $nums[j]$ 更新其 $countGreaterEqual[j]$ (类比，实际通过 $preSum(num[j]-1)$ 求出)，并不限制在 $nums[i]$「左侧」，只不过我们按逆序方向执行 (向左处理)，即便 $nums[i]$ 右侧某个元素大于 $nums[i]$ ，经由 $add$ 方法使该数对应的逆序数增加 1，它也没有机会再被累加了，况且，它在 $nums[i]$ 右侧，与 $nums[i]$ 构成的实际上是正序对。



总之，我们将一个 $nums$ 离散化后，逆序遍历它，先执行 $preSum(nums[i]-2)$ ，这个操作即为获取 $\{(nums[i],x),nums[i]>x\}$ 形式的逆序对对数。再执行 $add(nums[i]-1, 1)$ 找到 $\{(x,nums[i]),x>nums[i]\}$ 形式的逆序对，更新相应的 $tree[]$ 值。

实际上 $preSum(([nums[i]-1)-1])$ 就是本小节标题「指定区间内指定取值范围的元素数」 的体现，区间指 $[0,i-1]$ 区间，给定取值范围指 $>nums[i]$ 。



对于前面的例子 $nums = \{2,6,8,7,4,5,1,3\}$ ，我们给出执行 ① ~ ⑧ 后得到的树形图，供读者仔细验证。

```
① add(2, 1)
② add(0, 1)
③ add(4, 1)
④ add(3, 1)
⑤ add(6, 1)
⑥ add(7, 1)
⑦ add(5, 1)
⑧ add(1, 1)
```

![image.png](https://pic.leetcode-cn.com/1665830781-SDAcPL-image.png)

<br />

#### 松离散与紧离散

在前面的分析中，我们提到了「离散化」的概念，即在题目只关心输入数组 $nums$ 元素的大小关系而不关心具体的值时，为了压缩空间，我们先将 $nums$ 离散化。通常你可能会见到两种离散化方式，两种都基于排序，但其中一种借助了 $set$ 去重，使得离散化后的有效数字更少，取值范围更小，我们把这种方式称为「紧离散」；另一种则没有去重，因此离散化后的有效数字更多 (存在相同的数字)，取值范围也更大，我们把这种方式称之为「松离散」。以下是两种离散化方式的实现。

```java
// 松离散
private void discrete(int[] nums){ 
    int n = nums.length;
    int[] tmp = new int[n];
    System.arraycopy(nums, 0, tmp, 0, n);
    Arrays.sort(tmp);
    for (int i = 0; i < n; ++i) {
        nums[i] = Arrays.binarySearch(tmp, nums[i]) + 1;
    }
}

// 紧离散
private Map<Integer, Integer> discrete(int[] nums){ 
    Map<Integer, Integer> map = new HashMap<>();
    Set<Integer> set = new HashSet<>();
    for(int num : nums) set.add(num);
    List<Integer> list = new ArrayList<>(set);
    Collections.sort(list);
    int idx = 0;
    for(int num : list) map.put(num, ++idx);
    return map;
}
```

 例如对于 $nums=\{2,4,4,6\}$ ，松离散得到 $nums=\{1,2,2,4\}$，离散化后的 $nums$ 大小与原来相同；紧离散得到 $map=\{(2,1),(4,2),(6,3)\}$ ，其 $value$ 一定是从 1 开始的没有重复的连续正整数。两种方式离散化后虽然有效数字不同，取值范围也不同，但求解结果都是正确的 (读者可以思考一下为什么)。松离散无需哈希计算，通常速度更快，但有的题目可能更适合返回 $map$ 的紧离散 (例如 [327. 区间和的个数](https://leetcode.cn/problems/count-of-range-sum/) )。

<br />

### 时空复杂度

时间复杂度：离散化 $O(nlogn)$，树状数组求逆序对 $O(nlogn)$ ，综合为 $O(nlogn)$ 。渐进时间复杂度与归并排序做法相同，但显然常系数要更大一些。

空间复杂度：$O(n)$

<br />

### 代码

松离散写法。

```java
class Solution {
    public int reversePairs(int[] nums) {
        discrete(nums);// 离散化
        PURQBIT bit = new PURQBIT(nums);
        int n = nums.length, ans = 0;
        for(int i = n - 1; i >= 0; --i) {
            ans += bit.preSum((nums[i] - 1) - 1);
            bit.add(nums[i] - 1, 1);
        }
        return ans;
    }
    private void discrete(int[] nums){ // 离散化
        int n = nums.length;
        int[] tmp = new int[n];
        System.arraycopy(nums, 0, tmp, 0, n);
        Arrays.sort(tmp);
        for (int i = 0; i < n; ++i) {
            nums[i] = Arrays.binarySearch(tmp, nums[i]) + 1;
        }
    }
}
// 单点修改区间查询
class PURQBIT {
    int[] tree; // tree 为对应 nums 的区间和树状数组
    int n; // nums大小
    public PURQBIT(int[] nums){
        this.n = nums.length;
        this.tree = new int[n];
    }
    // 单点修改: 令 nums[k] += x
    public void add(int k, int x){
        for(int i = k + 1; i <= n; i += lowbit(i)){
            tree[i - 1] += x; // 包含第 k 项的区间都加上 x
        }
    }
    // 求前缀和: 求 nums[0] 到 nums[k] 的区间和 (前 k+1 项和)
    public int preSum(int k){
        int ans = 0;
        for(int i = k + 1; i > 0; i -= lowbit(i)){
            ans += tree[i - 1];
        }
        return ans;
    }
    private int lowbit(int i){
        return i & -i;
    }
}
```

紧离散写法。

```java
class Solution {
    public int reversePairs(int[] nums) {
        Map<Integer, Integer> map = discrete1(nums);// 离散化
        PURQBIT bit = new PURQBIT(nums);
        int n = nums.length, ans = 0;
        for(int i = n - 1; i >= 0; --i) {
            int pos = map.get(nums[i]);
            ans += bit.preSum((pos - 1) - 1);
            bit.add(pos - 1, 1);
        }
        return ans;
    }
    private Map<Integer, Integer> discrete1(int[] nums){ // 离散化
        Map<Integer, Integer> map = new HashMap<>();
        Set<Integer> set = new HashSet<>();
        for(int num : nums) set.add(num);
        List<Integer> list = new ArrayList<>(set);
        Collections.sort(list);
        int idx = 0;
        for(int num : list) map.put(num, ++idx);
        return map;
    }
}
// 单点修改区间查询
class PURQBIT {
    int[] tree; // tree 为对应 nums 的区间和树状数组
    int n; // nums大小
    public PURQBIT(int[] nums){
        this.n = nums.length;
        this.tree = new int[n];
    }
    // 单点修改: 令 nums[k] += x
    public void add(int k, int x){
        for(int i = k + 1; i <= n; i += lowbit(i)){
            tree[i - 1] += x; // 包含第 k 项的区间都加上 x
        }
    }
    // 求前缀和: 求 nums[0] 到 nums[k] 的区间和 (前 k+1 项和)
    public int preSum(int k){
        int ans = 0;
        for(int i = k + 1; i > 0; i -= lowbit(i)){
            ans += tree[i - 1];
        }
        return ans;
    }
    private int lowbit(int i){
        return i & -i;
    }
}
```

<br />

