# 剑指 Offer 45. 把数组排成最小的数

[剑指 Offer 45. 把数组排成最小的数](https://leetcode.cn/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/)

<br />

本题解提供一种易懂却不失严谨的证明：「**为什么本题的比较规则之下从小到大排序后的结果拼接的数字是最小的？**」。



关于排序，推荐阅读这篇我创作的两万余字的热门文章：[十大排序从入门到入赘](https://leetcode.cn/circle/discuss/eBo9UB/) 

<br />

## 解法一：快速排序

### 算法描述

本题本质为排序问题，即按本题要求所约束的 **「比较规则」** 比较 $nums$ 中各数字的大小，从小到大排列后拼接结果即为所求。这里的比较规则不再是数值比较，而是如下 。

首先我们约定如下:

- $<$ 符号表示数值比较的「小于」，$<'$ 表示本题比较规则的「小于」。
- $x,y$ 为数字字符串。$|x|,|y|$ 为其数值大小。
- $xy$ 表示数字字符串 $x$ 与 $y$ 拼接后的数字字符串，$|xy|$ 表示其数值大小。

> 【本题比较规则】
>
> 将 $nums$ 中的两个数 $x,y$ 视作字符串后拼接得到 $xy$ 和 $yx$ 。
>
> - 若 $|xy|<|yx|$ ，则称 $x$ 小于 $y$ ，记作 $x<'y$。
> - 若 $|xy|>|yx|$ ，则称 $y$ 小于 $x$ ，记作 $y<'x$。
> - 若 $|xy|=|yx|$ ，则称 $x$ 等于 $y$，记作 $x='y$ 。
>
> 例如 $|x|=3,|y|=30$ ，$|xy|=330,|yx|=303$ ，因为 $330>303$ 更大，则在这个比较规则下 $y<'x$ 。

以 $nums = [3,30,34,5,9]$ 为例，可以两两比较得到在本题比较规则小的大小  `"30" <' "3" <' "34" <' "5" <' "9"`，因此答案为 `"3033459"` 。



现在来思考两个问题：

- 上述规则为何正确？

  上述规则之所以正确，是因为它具有传递性。就像数值比较中 $1<2,2<3$ 的情况下，一定有 $1<3$ ，否则比较规则不能成立。要证明有传递性，即证明对于数字 $x,y,z$ 在上述规则下有 $x<'y,y<'z$ ，则在同一规则下必有 $x<'z$ 。只要根据已知的两个不等式，转换成数值比较进行算式变换推导即可。具体过程可参考 krahets 在他的 [题解](https://leetcode.cn/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/solution/mian-shi-ti-45-ba-shu-zu-pai-cheng-zui-xiao-de-s-4/) 下的评论 。

- 更关键的问题是， **为什么本题规则之下从小到大排序后的结果拼接的数字是最小的？** 

  采用反证法证明，即令 $A$ 是按照本题比较规则得到的升序排列构成的字符串，**假设** 存在排列 $B$ ，使得 $|B|<|A|$ ，下面我们来推翻此假设。再次强调，后续说明中的「序」、「大小」、「最大」等都是本题比较规则下的表述。

  推论1: $B$ 可通过有限次相邻「逆序对」的交换得到 $A$ ，且交换次数是「逆序对」的数量 (逆序数)。

  - 可以这么思考，在 $B$ 中找到当前最大数 $m$ ，$m$ 右侧的数都与 $m$ 构成一个逆序对，通过「冒泡」的方式将 $m$ 交换到最右侧。然后将剩下的数中最大者按照同样的方式交换到最右侧 ($m$ 的前一位) ，重复此操作直到无逆序对，则 $B$ 变为 $A$ 。交换次数显然为逆序数。

  推论2: 对于 $A$ ，交换任意两个相邻元素 $a,b$  ($a<'b$) 的位置得到排列 $C$ ，则必有 $|A|<|C|$ 。

  - 将相邻的 $a$ 与 $b$ 交换，$ab$ 变为 $ba$ ，其左右侧的「数字字符串」对应的「数值」不变，但 $|ab|<|ba|$ ，因此 $|A|<|C|$ 。

  推论3: 对于 $A$ ，交换任意两个元素 $a,b$  ($a<'b$) 位置得到排列 $C$ ，则必有 $|A|<|C|$。

  - 当这两个元素相邻时，为推论2。若 $a,b$ 不相邻 ，例如 $axyzb$ ，则交换 $a,b$ 相当于依次交换 $ax,ay,az$ ，接着交换 $zb,yb,xb$ 。根据推论2，每一次相邻交换都使得交换后的值更大，因此 $|A|<|C|$ 。

  推论4: $|A|<|B|$ 。

  - 根据推论1， $B$ 通过交换相邻元素得到 $A$ 的逆过程，就是 $A$ 通过交换相邻元素得到 $B$ 的过程，且在这个过程中交换的 $x$ 和 $y$ ( $x$ 在 $y$ 的左侧) 均满足 $x<'y$ ，因此从 $A$ 到 $B$ 的过程的每一次交换，都使得交换后的排列对应的数值更大，因此 $|B|>|A|$ 。

  假设不成立，即不存在数值更小的 $B$ 排列，所以 $A$ 一定是数值最小的排列。



我们可以采用任何一种数值排序方法对 $nums$ 进行排序，只需要在所有「比较 $nums$ 中的元素大小」的地方，应用本题的比较规则即可。可以将该规则封装为一个辅助方法。本解法我们采用随机轴快速排序。

```java
private int compare(int a, int b){ // 本题元素大小比较规则
    long x = a * (long)Math.pow(10, Integer.toString(b).length()) + b;
    long y = b * (long)Math.pow(10, Integer.toString(a).length()) + a;
    if(x > y) return 1;
    else if(x < y) return -1;
    else return 0;
}
```

<br />

### 时空复杂度

时间复杂度：$O(nlogn)$

空间复杂度：$O(n)$ ，用于输出的 $StringBuilder$ 的大小。

<br />

### 代码

```java
// 随机轴快排
class Solution {
    public String minNumber(int[] nums) {
        StringBuilder sb = new StringBuilder();
        quickSortRandom(nums, 0, nums.length - 1); 
        for(int num : nums) sb.append(num);
        return sb.toString();
    }
    private void quickSortRandom(int[] arr, int left, int right) {
        if (left < right) {
            int randomIndex = new Random().nextInt(right - left) + left + 1; // 在[left + 1, right]范围内的随机值
            swap(arr, left, randomIndex); // arr[left]与它之后的某个数交换
            int pivot = partition(arr, left, right);
            quickSortRandom(arr, left, pivot - 1);
            quickSortRandom(arr, pivot + 1, right);
        }
    }
    private int partition(int[] arr, int left, int right) {
        int pivot = left, index = pivot + 1;
        for (int i = index; i <= right; i++) {
            if (compare(arr[i], arr[pivot]) == -1) { // 注意此处的比较应用的是本题的规则
                swap(arr, index, i);
                index++;
            }
        }
        swap(arr, pivot, index - 1);
        return index - 1;
    }
    private int compare(int a, int b){ // 本题元素大小比较规则
        long x = a * (long)Math.pow(10, Integer.toString(b).length()) + b;
        long y = b * (long)Math.pow(10, Integer.toString(a).length()) + a;
        if(x > y) return 1;
        else if(x < y) return -1;
        else return 0;
    }
    private void swap(int[] arr, int i, int j){
        int tmp = arr[i];
        arr[i] = arr[j];
        arr[j] = tmp;
    }
}
```

<br />

