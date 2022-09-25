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

从左向右依次考察一个数的逆序对，对应树状数组中从 1 到 $n$ 初始化 $tree[1]\sim tree[n]$ 的操作。在基本树状数组中通过 $add(k, nums[k - 1])$ 来为第 $k$ 个区间和结点 $tree[k]$ 加上 $nums[k - 1]$ ，且 $add$ 方法中的循环会使其到根的每一个结点都加上该数。求逆序对时，执行的是 $add(nums[i], 1)$ ，表示在 $tree[]$ 的下标为 $nums[i]$ 的结点位置上出现了「1」个 $nums[i]$ ，其结果使得 $tree[nums[i]] += 1$ ，然后 $add$ 方法中的循环会「告知」到根结点路径上的结点 $nums[i]$ 的存在，即告诉之后更大的数，出现了一个比它小的数。

一开始我们假设 $nums$ 的值为 $\{1,2,3,..,n\}$ 的一个排列，这是因为 $tree$ 以 $nums$ 中的元素作为下标，其大小取决于 $nums$ 中的最大数，但介于 $nums$ 最大数和最小数之间且未在 $nums$ 中出现的数所占用的 $tree$ 是无用的。由于求逆序对只关心数的大小关系，为了节约空间，一方面也为了使代码简单易写，需要对 $nums$ 做「离散化」处理。所谓「离散化」即将大小为 $n$ 的 $nums$ 的元素映射为 $\{1,2,3,..,n\}$ 的一个排列，这个排列中的任意两个数的大小关系，与原数组对应位置上的两个数的大小关系相同。



如果你是初次用树状数组解决 $nums$ 中当前 $nums[i]$ 之前有多少个大于/大于等于/小于/小于等于它的数这类问题，我想你可能对上面的解释仍有疑惑？没关系，下面我继续详细说明，树状数组究竟是如何解决这一问题的。

在代码中，可以看到，树状数组类就是常规的单点修改区间查询的 BIT 实现代码。但我们注意到，在主方法 ($reversePairs$) 中，调用 $add$ 时，传入的增量 $x$ 是固定的 1。我们知道对于一般的 BIT，  $nums[k -1]$ 增加 $x$ 后，调用 $add(k,x)$ ，从第一个包含 $nums[k-1]$ 的区间的区间和开始，都增加 $x$ ，即从 $tree[k]+=x$ 开始，沿着父链更新所有包含 $nums[k-1]$ 的区间和。那么 $add(nums[i], 1)$ 是什么意思呢？

先看 $nums$ ，在执行 $add(nums[i],1)$ 之前，原大小为 $n$ 的输入数组 $nums$ 已由 **「离散化」** 操作变为了这样一个数组:

> 离散化后的 $nums$ 的所有元素的取值范围变为 $[1,n]$ ，且任意一对 $nums[i]$ 与 $nums[j]$ 的大小关系与原 $nums$ 中 $nums[i]$ 与 $nums[j]$ 的大小关系一致。

例如 $\{-2,6,-10\}$ 离散化后为 $\{2,3,1\}$  。

接着分析 $add(nums[i], 1)$ 的含义。这个操作是在遍历 $nums$ 过程中进行的，实际动作是对于 $nums[i]$ ，从 $tree[nums[i]]$ 开始，沿着父链上升，使得包含下标 $nums[i]$ 的代表更大区间的 $tree[x]+= 1$ 。$tree$ 的 $[1,n]$ 下标与 $nums$ 元素值的大小范围一致。 $add$ 的这一操作，相当于加入 $nums[i]$  时，告知 $nums$ 中在 $i$ 之后的比 $nums[i]$ 更大的数，在它前面新增了一个小于它的数。有点难以理解，没关系，结合下图，我们实际操作一下。

```
按照上述方式对 nums = {3,1,5,4,7,8,6,2} 执行 add(nums[i],1)
① add(3, 1)
② add(1, 1)
③ add(5, 1)
④ add(4, 1)
⑤ add(7, 1)
⑥ add(8, 1)
⑦ add(6, 1)
⑧ add(2, 1)
```

![image.png](https://pic.leetcode-cn.com/1658400428-eNpbln-image.png)

我们发现，每次加入一个 $nums[i]$ 之后，我们总是能够通过前缀和查询当前 **「小于等于」** 某个数的元素个数。例如，第 ④ 步过后，「小于等于」3 的个数，即 $query(3) = 2$ 。第 ⑥ 步过后「小于等于」7 的个数，即 $query(4)=5$ 。而当我们问介于 $[lower, upper]$ 之间的数有多少个时，实际上就是求 $query(upper) - query(lower - 1)$ 。 还需注意的是， $lower, upper$ 的取值范围要在 $[1,n]$ 之间，否则查询越界 (在 [327. 区间和的个数](https://leetcode.cn/problems/count-of-range-sum/) 中我们将看到如何查询范围超过 $[1,n]$ 时该如何处理)。



总之，我们将一个 $nums$ 离散化后，遍历它，先执行 $query(x)$ ，再执行 $add(nums[i], 1)$，就是在每次添加 $nums[i]$ 前询问当前 $nums$ 中小于等于 $x$ 的元素个数。又因为我们总是先询问然后再插入第 $i$ 个元素，那么，在插入 $nums[i]$ 之前:

- 若执行 $query(upper)$ 时，是询问 $nums[0]$ 到 $nums[i-1]$ ，小于等于 $upper$ 的个数；
- 若执行 $query(lower-1)$ 时，是询问 $nums[0]$ 到 $nums[i-1]$ ，小于等于 $lower - 1$ 的个数； 
- 若执行 $query(upper)-query(lower-1)$ 时，是询问 $nums[0]$ 到 $nums[i-1]$ ，取值范围为 $[lower,upper]$  的元素个数。



现在分析「逆序对」问题就简单多了。由于离散化不改变任意一对元素的大小关系，因此对原数组求逆序数等同于对离散化后的数组求逆序数。我们要做的是询问 $nums[i]$ 的前面，有多少个大于它的数，就相当于在 $nums$ 中插入 $nums[i]$ 之前， 求 $i - query(nums[i])$ 。即 $[0,i-1]$ 范围内，有 $i$ 个数，其中「小于等于」$nums[i]$ 的有 $query(nums[i])$ 个，因此  $i - query(nums[i])$ 即为所求。

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
        BIT bit = new BIT(nums);
        int n = nums.length, ans = 0;
        for(int i = 0; i < n; i++) {
            ans += (i -bit.query(nums[i]));
            bit.add(nums[i], 1);
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
class BIT {
    int n;
    int[] tree;
    public BIT(int[] nums) {
        this.n = nums.length;
        this.tree = new int[n + 1];
    }
    public int query(int n){ // 查询前n项和
        int ans = 0;
        for(int i = n; i > 0; i -= lowbit(i)){ // 下一个左邻区间和结点下标为i -= lowbit(i)
            ans += tree[i];
        }
        return ans;
    }
    public void add(int k, int x){ // 为第k项加上x
        for(int i = k; i <= n; i += lowbit(i)){ // 下一个区间和结点下标为i += lowbit(i)
            tree[i] += x; // 包含第t项的区间都加上x
        }
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
        Map<Integer, Integer> map = discrete(nums);// 离散化
        BIT bit = new BIT(nums);
        int n = nums.length, ans = 0;
        for(int i = 0; i < n; i++) {
            int pos = map.get(nums[i]);
            ans += (i - bit.query(pos));
            bit.add(pos, 1);
        }
        return ans;
    }
    private Map<Integer, Integer> discrete(int[] nums){ // 紧离散
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
class BIT {
    int n;
    int[] tree;
    public BIT(int[] nums) {
        this.n = nums.length;
        this.tree = new int[n + 1];
    }
    public int query(int n){ // 查询前n项和
        int ans = 0;
        for(int i = n; i > 0; i -= lowbit(i)){ // 下一个左邻区间和结点下标为i -= lowbit(i)
            ans += tree[i];
        }
        return ans;
    }
    public void add(int k, int x){ // 为第k项加上x
        for(int i = k; i <= n; i += lowbit(i)){ // 下一个区间和结点下标为i += lowbit(i)
            tree[i] += x; // 包含第t项的区间都加上x
        }
    }
    private int lowbit(int i){
        return i & -i;
    }
}
```

<br />

