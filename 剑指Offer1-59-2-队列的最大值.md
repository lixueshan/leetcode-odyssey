# 剑指Offer1-59-2-队列的最大值

[剑指 Offer 59 - II. 队列的最大值](https://leetcode.cn/problems/dui-lie-de-zui-da-zhi-lcof/) (中等)

<br />

## 解法一：单调队列

### 算法描述

经典的单调队列应用。设置两个双端队列，$q$ 为普通双端队列，用于元素的出队入队；$maxQ$ 为一单调队列，用于维护当前最大值。元素入队出队都通过 $q$ 来完成，并同时调整 $maxQ$ 。具体来说，$maxQ$ 按照单调队列那样在元素入队时调整队列内的元素，即从队尾开始将小于入队元素的元素舍弃，维持队列内的单调性。于是 $max\_value$ 方法直接操作 $maxQ$ 即可。问题在于随着 $pop\_front$ 的执行，当前最大元素正常出队时，$maxQ$ 也要相应地将队首出队。

容易想到的解决办法是设置两个下标变量，$inIdx$ 维护下一个入队元素下标，$outIdx$ 维护下一个出队元素下标。$maxQ$ 中除了保存元素值，也同时保存该元素对应的 $inIdx$ ，这样一来每次元素从 $q$ 出队时，都询问 $outIdx$ 是否等于当前 $maxQ$ 队首元素的 $inIdx$ ，若是，则说明该队首元素应当出队。这样我们就能给出「代码」中的第一版实现。

实际上稍加思考，我们会发现通过跟踪出入队的元素序号来决定 $maxQ$ 队首是否该出队并不是必须的，只需要在 $q$ 元素出队时，比较该元素是否与 $maxQ$ 队首元素相等，如相等，说明当前出队的元素就是 $maxQ$ 的队首元素，$maxQ$ 队首也应出队。不难想到，这一操作也涵盖了有相等元素的情形。于是我们可以轻松给出「代码」中的第二版实现。

<br />

### 时空复杂度

时间复杂度：$O(1)$ ，单次操作的平均复杂度。 删除和求最大值操作为 $O(1)$ 。对于插入操作，由于每个数字只会出队一次，出队的时间平均到每次插入操作上，为 $O(1)$ 。

空间复杂度：$O(n)$ ，$n$ 为插入元素的个数。

<br />

### 代码

```java
// 跟踪入队和出队元素的序号
class MaxQueue {
    Deque<Integer> q; // 普通双端队列，保存元素
    Deque<int[]> maxQ; // 单调队列，队首维护最大值
    int inIdx, outIdx; // inIdx为进入队列的元素序号，outIdx为出队序号
    public MaxQueue() {
        this.q = new ArrayDeque<>();
        this.maxQ = new ArrayDeque<>();
    }
    public int max_value() {
        if(maxQ.isEmpty()) return -1; // 单调队列为空，返回 -1
        return maxQ.peekFirst()[0];
    }
    public void push_back(int value) {
        q.offerLast(value); // 直接进入普通队列
        while(!maxQ.isEmpty() && maxQ.peekLast()[0] < value){ // 调整单调队列
            maxQ.pollLast();
        }
        maxQ.offerLast(new int[]{value, inIdx}); // 在单调队列中同时维护元素值与该元素序号
        inIdx++;
    }
    public int pop_front() {
        if(q.isEmpty()) return -1; // 普通队列空，返回 -1
        if(outIdx == maxQ.peekFirst()[1]) maxQ.pollFirst(); // 当前出队元素序号与单调队列队首元素需要相同时，单调队列也需出队
        outIdx++;
        return q.pollFirst(); // 普通队列出队并返回该值
    }
}

// 优化：不必跟踪序号
class MaxQueue {
    Deque<Integer> q; // 普通双端队列，保存元素
    Deque<Integer> maxQ; // 单调队列，队首维护最大值
    public MaxQueue() {
        this.q = new ArrayDeque<>();
        this.maxQ = new ArrayDeque<>();
    }
    public int max_value() {
        if(maxQ.isEmpty()) return -1; // 单调队列为空，返回 -1
        return maxQ.peekFirst();
    }
    public void push_back(int value) {
        q.offerLast(value); // 直接进入普通队列
        while(!maxQ.isEmpty() && maxQ.peekLast() < value){ // 调整单调队列
            maxQ.pollLast();
        }
        maxQ.offerLast(value); // 在单调队列中同时维护元素值与该元素序号
    }
    public int pop_front() {
        if(q.isEmpty()) return -1; // 普通队列空，返回 -1
        int head = q.pollFirst(); 
        if(head == maxQ.peekFirst()) maxQ.pollFirst(); // 当前出队元素序号与单调队列队首元素需要相同时，单调队列也需出队
        return head;
    }
}
```

<br />

