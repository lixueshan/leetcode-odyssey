若正在阅读本篇题解的你还未掌握线段树，推荐阅读我创作的 [线段树 (树ADT连载 11/13)](https://leetcode.cn/circle/discuss/H4aMOn/) ，全文两万余字，从最基本的概念讲起，全面并详细涉及以下内容，给出八种完整的线段树类代码实现。读完并理解后可解决力扣上几乎所有线段树题目，欢迎大家阅读指正。

> 基本静态线段树 / 完全二叉树下标性质 / 分治算法 / 懒惰标记 / 延迟修改 / 离散化 / 强制在线 / 动态开点线段树 / 结点数组法动态开点 (预估结点数) / 结点指针法动态开点 

***

# 715-Range模块

[715. Range 模块](https://leetcode.cn/problems/range-module/) (困难)

<br />

## 解法一：动态开点线段树 (指针法)

### 算法描述

容易按如下理解题意，因此本题可用支持覆盖式「区间修改」和「区间求和」的线段树实现。

- $addRange$ 方法即覆盖式「区间修改」，我们可以视作将区间 $[left,right-1]$ 的每个元素修改为 1。
- $queryRange$ 方法即「区间查询」，对区间 $[left,right-1]$ 求和，若 $sum = right-left$  返回 $true$ ，否则返回 $false$ 。
- $removeRange$ 方法也是覆盖式「区间修改」，将区间 $[left,right-1]$ 的每个元素修改为 0。

根据题目给出的取值范围，我们看到「区间」范围为 $[1,10^9]$ ，因此不能直接用堆式线段树。当我们打算通过「离散化」来缩小区间时，发现本题是强制在线的，也就是并未提前告诉我们所有涉及询问和修改的区间范围，因此无法离散化。

所以我们只能采用「动态开点线段树」，本解法给出指针法动态开点线段树的做法。

<br />

### 时空复杂度

时间复杂度：叶子结点数为 $n$ (即区间最大值，本题为 $10^9$) ，单次区间查询或单次区间修改复杂度为 $O(logn)$ 。查询和修改的次数为 $m$ ，总时间复杂度为 $mO(logn)$ 。

空间复杂度：单次查询将创建 $O(logn)$ 个结点，总空间复杂度为 $mO(logn)$ 。

<br />

### 代码

```java
// 跟踪[l,r): 区间修改，将[l,r)改为1
// 停止跟踪 [l,r): 区间修改，将[l,r)改为0
// query [l,r): 区间求和，sum = (r - l) 表示跟踪了每一个数，返回 true，否则返回false
class RangeModule {
    DynamicSegmentTreePointerUpdate st;
    static final int N = 1_000_000_000;
    public RangeModule() {
        this.st = new DynamicSegmentTreePointerUpdate(N);
    }
    public void addRange(int left, int right) { // 区间修改
        st.update(left, right - 1, 1);
    }
    public boolean queryRange(int left, int right) { // 区间求和并判断
        return st.sum(left, right - 1) == right - left;
    }
    public void removeRange(int left, int right) { // 区间修改
        st.update(left, right - 1, 0);
    }
}
class DynamicSegmentTreePointerUpdate{
    private class Node{
        int lazy, val;
        boolean updated;
        Node lChild, rChild;
    }
    int n;
    Node root;
    public DynamicSegmentTreePointerUpdate(int n){
        this.n = n;
        this.root = new Node();
    }
    public void update(int l, int r, int x){ // 区间修改(驱动): 增量式 [l,r] 区间所有元素加上x
        update(l, r, x, 0, n - 1, root);
    }
    public int sum(int l, int r){ // 区间查询(驱动): nums[l]~nums[r]之和
        return sum(l, r, 0, n - 1, root);
    }
    private void update(int l, int r, int x, int s, int t, Node cur){ // 区间修改: 覆盖式 [l,r] 区间所有元素改为x
        if(l <= s && t <= r){ // 当前结点代表的区间在所求区间之内
            cur.val = (t - s + 1) * x; // 结点i的区间和等于t-s+1个x
            if(s != t) {
                cur.lazy = x; // 结点i不是叶子结点，懒标记值等于x
                cur.updated = true;
            }
            return;
        }
        addNode(cur); // 动态开点
        int c = s + (t - s) / 2;
        if(cur.updated) pushDown(s, c, t, cur); // 当前结点懒惰标记不为0，推送标记
        if(l <= c) update(l, r, x, s, c, cur.lChild);
        if(r > c) update(l, r, x, c + 1, t, cur.rChild);
        pushUp(cur); // 后序动作，自底向上更新结点区间和 tree[i]
    }
    private int sum(int l, int r, int s, int t, Node cur){ // 区间查询: 求 nums[l]~nums[r]之和
        if(l <= s && t <= r) return cur.val; // 当前结点代表的区间在所求区间之内
        addNode(cur); // 动态开点
        int c = s + (t - s) / 2, sum = 0;
        if(cur.updated) pushDown(s, c, t, cur); // 当前结点懒惰标记不为0，推送标记
        if(l <= c) sum += sum(l, r, s, c, cur.lChild);
        if(r > c) sum += sum(l, r, c + 1, t, cur.rChild);
        return sum;
    }
    private void pushUp(Node cur){ // 更新 tree[i].val
        Node lChild = cur.lChild, rChild = cur.rChild;
        cur.val = lChild.val + rChild.val;
    }
    private void pushDown(int s, int c, int t, Node cur){ // 更新结点i左右子结点的区间和以及懒惰标记值，最后重置结点i的懒惰标记值
        Node lChild = cur.lChild, rChild = cur.rChild;
        lChild.val = (c - s + 1) * cur.lazy; // 更新其左子结点的区间和
        lChild.lazy = cur.lazy; // 传递懒标记(增量标记)
        lChild.updated = true;
        rChild.val = (t - c) * cur.lazy;
        rChild.lazy = cur.lazy;
        rChild.updated = true;
        cur.lazy = 0; // 重置当前结点懒惰标记值（增量标记置0）
        cur.updated = false;
    }
    private void addNode(Node node){ // 动态开点
        if(node.lChild == null) node.lChild = new Node();
        if(node.rChild == null) node.rChild = new Node();
    }
}
```

<br />

## 解法二：动态开点线段树 (结点数组法)

### 算法描述

本题提示给出了单个测试用例调用的查询/修改次数 (操作次数) 上限为 $10^4$  次，根据这个信息也可以提前估计树的大小，从而可以采用结点数组法来动态开点。

$tree[]$ 的大小 $m$ 需要预估。每次查询或修改操作的「开点」次数与树高有关，我们知道树高 $h$ 与叶子结点数 $n$ (即我们必须知道的区间大小) 的关系，因此只要能够提前知道操作总次数 (查询和修改) ，并假设查询或修改的区间 **不太大**，我们就能估计经过所有操作后创建的总的结点数，即线段树的大小 $m$。。 如下：

1. 整棵线段树的结点数不会超过 $4*n$ 。
2. 每次查询从根结点往下，在假设查询或修改的区间 **只有一个值** ，则每次操作从根结点到表示该值的叶子结点的路径上，每一层只会经过一个区间，开两个点。于是开点次数为树高 $h = \lceil log_{2}(4*n) \rceil -1=log_{2}(4*n)$  (根结点高 0)，每次开两个点，一次查询最多新建 $2*h$ 个结点。但平均而言操作的区间当然不会只有一个值，因此这里我们要适当放大倍数，根据经验可放大到 $6*h$ 。
3. 假设有 $k$ 次操作，则有估计 $m=6*k*log_{2}(4*n)$ ，省略 $6*k*2$ 后得到 $m = 6*k*logn$  。

 

<br />

### 时空复杂度

时间复杂度：叶子结点数为 $n$，单次查询时间复杂度为 $O(logn)$ 。查询次数为 $k$ ，总时间复杂度为 $O(klogn)$ 。

空间复杂度：根据前面的分析，为 $O(klogn)$ 。

<br />

### 代码

```java
class RangeModule {
    DynamicSegmentTreeArrayUpdate st;
    int N, M;
    public RangeModule() {
        this.N = 1_000_000_000;
        this.M = 10000 * 2 * (int) ((2 + Math.log(N)) / Math.log(2) + 1);
        this.st = new DynamicSegmentTreeArrayUpdate(N, M);
    }
    public void addRange(int left, int right) { // 区间修改
        st.update(left, right - 1, 1);
    }
    public boolean queryRange(int left, int right) { // 区间求和并判断
        return st.sum(left, right - 1) == right - left;
    }
    public void removeRange(int left, int right) { // 区间修改
        st.update(left, right - 1, 0);
    }
}
class DynamicSegmentTreeArrayUpdate{
    private class Node{
        int lIdx, rIdx, lazy, val;
        boolean updated;
    }
    Node[] tree;
    int n, count;
    public DynamicSegmentTreeArrayUpdate(int n, int m){
        this.n = n;
        this.count = 1;
        this.tree = new Node[m];
    }
    public void update(int l, int r, int x){ // 区间修改(驱动): 覆盖式 [l,r] 区间所有元素改为x
        update(l, r, x, 0, n - 1, 1);
    }
    public int sum(int l, int r){ // 区间查询(驱动): nums[l]~nums[r]之和
        return sum(l, r, 0, n - 1, 1);
    }
    private void update(int l, int r, int x, int s, int t, int i){ // 区间修改: 覆盖式 [l,r] 区间所有元素改为x
        if(l <= s && t <= r){ // 当前结点代表的区间在所求区间之内
            tree[i].val = (t - s + 1) * x; // 结点i的区间和加上t-s+1个x
            if(s != t) {
                tree[i].lazy = x; // 结点i不是叶子结点，懒标记值加上x
                tree[i].updated = true;
            }
            return;
        }
        addNode(i); // 动态开点
        int c = s + (t - s) / 2;
        if(tree[i].updated) pushDown(s, c, t, i); // 当前结点懒惰标记不为0，推送标记
        if(l <= c) update(l, r, x, s, c, tree[i].lIdx);
        if(r > c) update(l, r, x, c + 1, t, tree[i].rIdx);
        pushUp(i); // 后序动作，自底向上更新结点区间和 tree[i]
    }
    private int sum(int l, int r, int s, int t, int i){ // 区间查询: 求 nums[l]~nums[r]之和
        if(l <= s && t <= r) return tree[i].val; // 当前结点代表的区间在所求区间之内
        addNode(i); // 动态开点
        int c = s + (t - s) / 2, sum = 0;
        if(tree[i].updated) pushDown(s, c, t, i); // 当前结点懒惰标记不为0，推送标记
        if(l <= c) sum += sum(l, r, s, c, tree[i].lIdx);
        if(r > c) sum += sum(l, r, c + 1, t, tree[i].rIdx);
        return sum;
    }
    private void pushUp(int i){ // 更新 tree[i].val
        Node cur = tree[i], lChild = tree[cur.lIdx], rChild = tree[cur.rIdx];
        cur.val = lChild.val + rChild.val;
    }
    private void pushDown(int s, int c, int t, int i){ // 更新结点i左右子结点的区间和以及懒惰标记值/updated，最后重置结点i的懒惰标记值/updated
        Node cur = tree[i], lChild = tree[cur.lIdx], rChild = tree[cur.rIdx];
        lChild.val = (c - s + 1) * cur.lazy; // 更新其左子结点的区间和
        lChild.lazy = cur.lazy; // 传递懒标记(增量标记)
        lChild.updated = true;
        rChild.val = (t - c) * cur.lazy;
        rChild.lazy = cur.lazy;
        rChild.updated = true;
        cur.lazy = 0; // 重置当前结点懒惰标记值（增量标记置0）
        cur.updated = false; // 将updated置为false
    }
    private void addNode(int i){ // 动态开点
        if(tree[i] == null) tree[i] = new Node(); // add首次被调用时为 add(1) ，即tree[1] 为根结点
        if(tree[i].lIdx == 0) { // 若 tree[i] 结点无左孩子，添加之
            tree[i].lIdx = ++count; // 赋予结点标号
            tree[tree[i].lIdx] = new Node(); // 开辟实例
        }
        if(tree[i].rIdx == 0) { // 若 tree[i] 结点无右孩子，添加之
            tree[i].rIdx = ++count;
            tree[tree[i].rIdx] = new Node();
        }
    }
}
```

<br />

