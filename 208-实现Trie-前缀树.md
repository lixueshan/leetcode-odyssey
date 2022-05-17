# 208-实现Trie(前缀树)

[208. 实现 Trie (前缀树)](https://leetcode-cn.com/problems/implement-trie-prefix-tree/) (中等)

<br />

## 解法一：基于哈希表的前缀树

### 算法描述

本题考查前缀树的实现。在了解前缀树的结构和作用后，其实现是简单而直接的。前缀树问题一般都是在同一棵树上求解。树的根为非前缀，前缀从根的儿子开始。树结点类通常写作Trie。每一个结点的儿子结点数量，与「构成前缀的基本元素的种类数」相关。本题的插入和搜索方法也是一般前缀树的基本方法。

1. insert：从root开始按输入的word的字符依次向下延伸，若无代表次字符的结点，创建之。
2. search：向下寻找word，若找到，则末尾字符处判断是否为单词。因此，Trie类还需维护一个isEnd属性，用以标记到该节点为止的前缀是否为单词。
3. startsWith：向下寻找prefix。某一字符找不到则立即返回false，否则返回true。

其中，2，3动作类似，可以另外给出一个searchPrefix方法，寻找输入的字符串是否在前缀树上，找到则返回最后一个结点，否则返回null。于是2和3就可以通过调用searchPrefix，以一条返回语句完成方法。

<br />

### 时空复杂度

时间复杂度：初始化为O(1)，每次操作为O(N)，N为插入或查找的字符串的长度。

空间复杂度：O(N)，N表示Trie结点数量。N基本上等于所有插入字符的长度之和。（说基本上是因为如果插入单词的有部分前缀相同，那么结点数量要减去这些前缀的长度）。

<br />

### 代码

```java
class Trie {
    Map<Character, Trie> next;
    boolean isEnd;
    public Trie() {
        this.next = new HashMap<>();
        this.isEnd = false;
    }
    public void insert(String word) {
        Trie cur = this; // 得到根结点
        for(char c : word.toCharArray()){
            if(cur.next.get(c) == null){ // 若当前无此字符，添加之
                cur.next.put(c, new Trie());
            }
            cur = cur.next.get(c); // 向下考察
        }
        cur.isEnd = true; // 置末尾字符节点isEnd为true
    }
    public boolean search(String word) {
        Trie end = searchPrefix(word);
        return end != null && end.isEnd;
    }
    public boolean startsWith(String prefix) {
        return searchPrefix(prefix) != null;
    }
    private Trie searchPrefix(String prefix){
        Trie cur = this;
        for(char c : prefix.toCharArray()){
            if(cur.next.get(c) == null){
                return null; // 无此前缀，返回null
            }
            cur = cur.next.get(c);
        }
        return cur;
    }
}
```

<br />

## 解法二：基于数组化哈希的前缀树

### 算法描述

前缀树通常被用来解决字符相关的问题（因此也称为字典树），多数典型问题中，字符为英文字母。若题目声明字符集为英文小写字母，那么每一个结点儿子数量为26。于是每一个结点中维护的子结点哈希表可以替换为大小为26的Trie数组。实际上，这是我们更常看到的做法。

<br />

### 时空复杂度

时间复杂度：初始化为O(1)，每次操作为O(N)，N为插入或查找的字符串的长度。

空间复杂度：O(26*N)，N表示Trie结点数量。

<br />

### 代码

```java
class Trie {
    Trie[] next;
    boolean isEnd;
    public Trie() {
        this.next = new Trie[26];
        this.isEnd = false;
    }
    public void insert(String word) {
        Trie cur = this; // 得到根结点
        for(char c : word.toCharArray()){
            int idx = c - 'a';
            if(cur.next[idx] == null){ // 若当前无此字符，添加之
                cur.next[idx] = new Trie();
            }
            cur = cur.next[idx]; // 向下考察
        }
        cur.isEnd = true; // 置末尾字符节点isEnd为true
    }
    public boolean search(String word) {
        Trie end = searchPrefix(word);
        return end != null && end.isEnd;
    }
    public boolean startsWith(String prefix) {
        return searchPrefix(prefix) != null;
    }
    private Trie searchPrefix(String prefix){
        Trie cur = this;
        for(char c : prefix.toCharArray()){
            int idx = c - 'a';
            if(cur.next[idx] == null){
                return null; // 无此前缀，返回null
            }
            cur = cur.next[idx];
        }
        return cur;
    }
}
```

<br /> 