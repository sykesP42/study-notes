# 🌳 前缀操作与 Trie（Prefix Operations and Tries）—— 从 DAG 到 Trie，每一处细节都讲透

> 本文是 CS61B Spring 2024 Lecture 28 的超详细笔记，目标是让你通过阅读彻底掌握：拓扑排序、Dijkstra 的负权局限、DAG 最短路径算法、最长路径问题及其归约解法、归约的思想、字符键数组实现、以及 Trie 数据结构（含三种子节点实现）的完整代码与性能分析。文中包含大量 Java 代码示例、步骤拆解、正确性证明、复杂度计算和思考题答案。

---

## 🔁 一、拓扑排序（Topological Sort）回顾

### 1.1 定义

**拓扑排序** 是对有向无环图（DAG）的所有顶点的一种线性排序，使得对于每条有向边 `u → v`，顶点 `u` 在排序中都出现在 `v` 之前。直观地，将所有顶点按水平线排列，所有边都指向右方。

### 1.2 基于 DFS 后序反转的算法

**核心思想**：深度优先搜索（DFS）在递归返回时记录顶点（后序），最后反转该顺序即得拓扑排序。因为后序中，一个顶点在其所有后继都被访问后才被记录，所以它的后继都出现在它前面，反转后它就在后继的前面。

#### 代码实现


```java

import java.util.*;
public class TopologicalSort {
    private boolean[] marked;
    private Stack<Integer> reversePost; // 用于存储后序结果（栈）
    public TopologicalSort(Digraph G) {
        reversePost = new Stack<>();
        marked = new boolean[G.V()];
        for (int v = 0; v < G.V(); v++) {
            if (!marked[v]) dfs(G, v);
        }
    }
    private void dfs(Digraph G, int v) {
        marked[v] = true;
        for (int w : G.adj(v)) {
            if (!marked[w]) dfs(G, w);
        }
        reversePost.push(v); // 后序：递归返回时记录
    }
    public Iterable<Integer> order() {
        // 返回拓扑排序（将栈弹出）
        List<Integer> topo = new ArrayList<>();
        while (!reversePost.isEmpty()) {
            topo.add(reversePost.pop());
        }
        return topo;
    }
}
```
**说明**：

- `Digraph` 是一个有向图类，提供 `V()` 返回顶点数，`adj(v)` 返回邻接表。
    
- `reversePost` 是栈，后序压栈，最后弹出顺序即为拓扑排序。
    
- 由于图可能不连通，需要对每个未访问顶点启动 DFS。
    

**时间复杂度**：O(V + E)，每个顶点和每条边都访问一次。

**正确性证明**：对于任意边 `v → w`，在 DFS 中，要么先访问 `v` 再访问 `w`，要么先访问 `w` 再访问 `v`。若先访问 `v`，则可能先完成 `w` 的 DFS 再完成 `v`，因此 `w` 会在 `v` 之前被压栈，反转后 `v` 在 `w` 前，符合要求。若先访问 `w`，由于存在 `v → w`，`w` 不可能在 `v` 之前完成（否则会形成环），所以不会出现这种情况。因此算法正确。

---

## ⚠️ 二、Dijkstra 算法与负权边

### 2.1 Dijkstra 算法回顾

Dijkstra 算法解决非负权图单源最短路径问题。它维护一个优先队列，按当前最短距离从小到大取出顶点，并松弛其出边。关键不变式：**一旦顶点被取出，其距离就不再改变**，因为所有未取出顶点的距离都 ≥ 它，且边权非负，不可能通过它们得到更短路径。

### 2.2 负权边的致命影响

当存在负权边时，上述不变式被破坏。考虑下图（来自讲义）：

```text

A --2--> B --3--> F
|        ↑        ↑
1        | -20    |
+--> C --+        |
     |            |
     +-----1-----> E
```
假设源点为 A。Dijkstra 过程：

- 初始 dist: A=0, B=∞, C=∞, E=∞, F=∞。
    
- 取出 A：更新 B=2, C=1。
    
- 取出 C（距离 1）：更新 E=1+1=2。
    
- 取出 B（距离 2）：更新 F=2+3=5。
    
- 此时队列中剩下 E=2, F=5。取出 E（距离 2），发现 E 没有出边（或忽略）。
    
- 结果：dist[F]=5，但实际最短路径 A→C→E→F？E→F 是否存在？图中没有 E→F 边。实际最短路径是 A→C→B→F？但 C→B 是 -20，所以 A→C (1) + C→B (-20) = -19，然后 B→F (3) = -16，比 5 小得多！但在 Dijkstra 中，当处理 C 时，它更新了 B 为 -19，但 B 此时可能还在队列中（假设 B 距离为 2 已被取出？这里需要具体看顺序）。若 B 在 C 之前被取出（B=2 先于 C=1？不对，C 距离 1 更小，应先取 C）。所以顺序是 A→C→B？让我们详细模拟：
    

正确模拟（考虑负权）：

- 初始：PQ = [(A,0)]
    
- 取出 A：更新 B=2, C=1 → PQ 变为 [(C,1), (B,2)]
    
- 取出 C：松弛 C→B：1 + (-20) = -19 < 2，更新 B= -19，PQ 中 B 的优先级变为 -19 → PQ 变为 [(B,-19)]
    
- 取出 B：松弛 B→F：-19 + 3 = -16，更新 F= -16 → PQ 变为 [(F,-16)]
    
- 取出 F：结束。
    

咦，这居然得到了正确结果 -16？但实际上 B 被更新后再次取出，并没有问题。但关键在于，如果有一条边从已取出的顶点指向另一个已取出的顶点，且边权为负，则可能导致已取出顶点被更新。例如，假设图中有 A→B (5), A→C (1), C→B (-4)，B 已被取出（距离 5），然后从 C 更新 B 为 1-4=-3，B 已被取出，算法不会再次处理 B，因此 B 的最短距离会错误地保持 5。这正是 Dijkstra 失败的情形。因此，Dijkstra 不能处理负权边，因为无法保证已取出顶点不会被更短的路径更新。

**结论**：Dijkstra 算法仅适用于非负权图。

---

## 🚀 三、DAG 最短路径算法（可处理负权）

### 3.1 算法思想

利用 DAG 的拓扑顺序，按拓扑序依次处理每个顶点，松弛其所有出边。由于所有边都从先处理的顶点指向后处理的顶点，当处理到顶点 v 时，所有能到达 v 的路径都已经通过之前的顶点处理过，因此 `distTo[v]` 已经是最优的。后续顶点不会反向影响 v，所以只需一次遍历。

### 3.2 算法步骤

1. 对 DAG 进行拓扑排序，得到顺序 `topo[]`。
    
2. 初始化 `distTo` 数组，`distTo[s] = 0`，其余为 `∞`；`edgeTo` 为 `null`。
    
3. 按 `topo` 顺序遍历每个顶点 v：
    
    - 对于 v 的每条出边 `v → w`（权重 wgt）：
        
        - 如果 `distTo[v] + wgt < distTo[w]`，更新 `distTo[w]` 和 `edgeTo[w]`。
            
4. 最终 `distTo` 即为最短距离。
    

### 3.3 Java 实现

```java

public class AcyclicSP {
    private double[] distTo;
    private DirectedEdge[] edgeTo;
    public AcyclicSP(EdgeWeightedDigraph G, int s) {
        distTo = new double[G.V()];
        edgeTo = new DirectedEdge[G.V()];
        for (int v = 0; v < G.V(); v++)
            distTo[v] = Double.POSITIVE_INFINITY;
        distTo[s] = 0.0;
        Topological topological = new Topological(G); // 拓扑排序类
        if (!topological.hasOrder()) throw new IllegalArgumentException("Graph is not acyclic");
        for (int v : topological.order()) {
            for (DirectedEdge e : G.adj(v))
                relax(e);
        }
    }
    private void relax(DirectedEdge e) {
        int v = e.from(), w = e.to();
        if (distTo[v] + e.weight() < distTo[w]) {
            distTo[w] = distTo[v] + e.weight();
            edgeTo[w] = e;
        }
    }
    public double distTo(int v) { return distTo[v]; }
    public boolean hasPathTo(int v) { return distTo[v] < Double.POSITIVE_INFINITY; }
    public Iterable<DirectedEdge> pathTo(int v) {
        if (!hasPathTo(v)) return null;
        Stack<DirectedEdge> path = new Stack<>();
        for (DirectedEdge e = edgeTo[v]; e != null; e = edgeTo[e.from()])
            path.push(e);
        return path;
    }
}
```
**说明**：

- `EdgeWeightedDigraph` 是带权有向图，提供 `adj(v)` 返回出边迭代器。
    
- `Topological` 类基于 DFS 后序反转实现拓扑排序，并提供 `order()` 返回迭代顺序。
    
- 松弛操作与 Dijkstra 相同，但无需优先队列。
    

### 3.4 时间复杂度

- 拓扑排序：O(V+E)
    
- 遍历每个顶点并松弛每条边：O(V+E)
    
- 总时间：**O(V+E)**
    

### 3.5 正确性证明

**归纳法**：设拓扑顺序为 `v1, v2, ..., vn`。假设在处理 `vk` 之前，所有 `vi (i < k)` 的 `distTo` 已经是最短距离（归纳假设）。处理 `vk` 时，对于每条边 `vk → w`，由于 `w` 在拓扑序中位于 `vk` 之后，任何从源点到 `w` 的路径如果经过 `vk`，则其前缀到 `vk` 的距离已经是最优，因此这次松弛是正确的。如果路径不经过 `vk`，则它由更早的顶点处理时已经考虑过。因此处理完所有顶点后，所有 `distTo` 都是最优的。

---

## ⏱️ 四、最长路径问题与归约

### 4.1 一般图上的最长路径

**最长路径问题**：在给定图中，寻找从源点 `s` 到每个其他顶点的**最长简单路径**（无环）。这个问题在一般图中是 NP 难的，已知最佳算法是指数级时间。

### 4.2 DAG 上的最长路径

由于 DAG 无环，最长路径一定存在且有限。我们可以通过**归约**到最短路径来解决。

**归约方法**：

1. 创建原图 `G` 的副本 `G'`，将每条边的权重取反（`w' = -w`）。
    
2. 在 `G'` 上运行 DAG 最短路径算法，得到 `distTo'` 和 `edgeTo'`。
    
3. 将 `distTo'` 中的每个值取反，得到原图的最长距离。
    
4. `edgeTo'` 直接作为最长路径树（因为路径结构相同，只是权重符号不同）。
    

**为什么正确？**  
因为对于任何一条路径 `p`，其原图总权 `sum(w)` 与取反后的总权 `sum(-w) = -sum(w)` 成反比。因此 `G'` 上的最短路径（最小负值）对应原图上的最长路径（最大正值）。

### 4.3 Java 实现

```java

public class AcyclicLP {
    private double[] distTo;
    private DirectedEdge[] edgeTo;
    public AcyclicLP(EdgeWeightedDigraph G, int s) {
        // 创建取反权重的图
        EdgeWeightedDigraph G2 = new EdgeWeightedDigraph(G.V());
        for (int v = 0; v < G.V(); v++) {
            for (DirectedEdge e : G.adj(v)) {
                G2.addEdge(new DirectedEdge(v, e.to(), -e.weight())); // 权重取反
            }
        }
        // 在 G2 上运行 DAG 最短路径
        AcyclicSP sp = new AcyclicSP(G2, s);
        distTo = new double[G.V()];
        edgeTo = new DirectedEdge[G.V()];
        for (int v = 0; v < G.V(); v++) {
            distTo[v] = -sp.distTo(v); // 结果取反
            edgeTo[v] = sp.edgeTo(v);   // 边对象复用（注意边权重已取反，但 edgeTo 记录的是边本身，不影响路径）
        }
    }
    // 其他方法类似 AcyclicSP
}
```
**注意**：`edgeTo` 中存储的边是取反后的图上的边，其权重为负。但路径结构正确，因为边指向关系相同。如果需要在原图中使用，可以存储原边，或者通过 `edgeTo` 反向查找时忽略权重。

### 4.4 时间复杂度

- 创建新图：O(E)
    
- 运行 DAG 最短路径：O(V+E)
    
- 总时间：O(V+E)
    

---

## 🔗 五、归约（Reduction）的思想

### 5.1 定义

> 如果问题 P 可以通过调用问题 Q 的解法来解决，则称 **P 归约到 Q**（P reduces to Q）。

归约是一种重要的算法设计策略：将未知问题转化为已知问题，从而直接使用已知解法。

### 5.2 示例：3SAT 归约到独立集

**3SAT 问题**：给定一个合取范式（CNF），每个子句有三个文字，问是否存在真值赋值使公式为真。

**独立集问题**：给定一个图，是否存在大小为 k 的独立集（顶点子集，任意两点不相邻）。

**归约方法**（简略）：

- 对于每个子句，构造一个三角形（三个顶点，两两相连），每个顶点对应子句中的一个文字。
    
- 对于每个变量 `x`，在所有出现 `x` 的顶点和所有出现 `¬x` 的顶点之间添加边，防止它们同时被选。
    
- 设置 `k = 子句数`。则存在大小为 k 的独立集当且仅当 3SAT 公式可满足。
    

这个归约说明独立集问题和 3SAT 一样难（NP 完全）。

### 5.3 归约 vs 分解

- **归约**：将整个问题转化为另一个问题的实例，直接使用其解法。
    
- **分解**：将复杂任务拆分成多个子任务，分别解决后组合。
    

分解是计算机科学的本质，通过抽象分层降低复杂度。

---

## 📦 六、特殊情况一：字符键映射（数组实现）

### 6.1 场景

当键是**单个 ASCII 字符**时，我们可以用数组实现集合或映射，比哈希表和平衡树更简单、更快。

### 6.2 集合实现（CharacterSet）

```java

public class CharSet {
    private boolean[] present = new boolean[128]; // ASCII 0-127
    public void add(char c) {
        present[c] = true; // c 自动转为 int
    }
    public boolean contains(char c) {
        return present[c];
    }
    public void remove(char c) {
        present[c] = false;
    }
}
```
**复杂度**：所有操作 O(1)，绝对常数，无哈希冲突。

### 6.3 映射实现（CharMap）

```java

public class CharMap<V> {
    private Object[] values = new Object[128]; // 存储值
    public void put(char c, V val) {
        values[c] = val;
    }
    public V get(char c) {
        return (V) values[c];
    }
}
```
### 6.4 扩展

如果键是 `a`-`z` 小写字母，可用数组大小 26，通过 `c - 'a'` 索引。如果键是 Unicode，数组会非常大（约 143,000），但若内存充足仍可行。

**缺点**：键空间必须有限且较小，否则浪费内存。

---

## 🌲 七、Trie（前缀树）

### 7.1 动机

当键是**字符串**时，我们希望数据结构能：

- 支持前缀查询（如自动补全）
    
- 操作时间与键长 L 相关，与集合大小 N 无关
    
- 避免哈希冲突
    

**Trie** 满足这些需求。

### 7.2 Trie 的基本结构

- 每个节点代表一个字符（根节点不存储字符）。
    
- 从根到某个节点的路径上的字符序列构成一个字符串的前缀。
    
- 节点中有一个布尔标志 `isKey`，表示从根到该节点的路径是否构成一个完整键。
    
- 每个节点有指向子节点的链接，通常用数组或哈希表存储。
    

### 7.3 示例

插入 `"sam"`, `"sad"`, `"sap"`, `"same"`, `"a"`, `"aws"` 后的 Trie 结构（见讲义图）。蓝色节点表示 `isKey = true`。

### 7.4 基本操作实现（以固定数组为例）

#### 节点定义

```java

private static class TrieNode {
    private boolean isKey;
    private TrieNode[] next = new TrieNode[R]; // R 为字母表大小
}
```
#### 查找

```java

public boolean contains(String key) {
    TrieNode node = root;
    for (int i = 0; i < key.length(); i++) {
        int c = key.charAt(i);
        if (node.next[c] == null) return false;
        node = node.next[c];
    }
    return node.isKey;
}
```
#### 插入

```java

public void add(String key) {
    root = put(root, key, 0);
}
private TrieNode put(TrieNode node, String key, int d) {
    if (node == null) node = new TrieNode();
    if (d == key.length()) {
        node.isKey = true;
        return node;
    }
    int c = key.charAt(d);
    node.next[c] = put(node.next[c], key, d+1);
    return node;
}
```
### 7.5 性能分析

- **查找/插入时间复杂度**：O(L)，L 为键长，与 N 无关。
    
- **空间复杂度**：每个节点有 R 个链接，可能大量空指针。对于英文小写字母（R=26），空间尚可；对于 ASCII（R=128）或 Unicode（R=65536），内存爆炸。
    

### 7.6 三种子节点实现方式对比

|实现方式|查找子节点时间复杂度|空间复杂度|适用场景|
|---|---|---|---|
|固定大小数组|O(1)|O(R × 节点数)，R 为字母表大小|字母表小（如英文小写）|
|哈希表|平均 O(1)|O(实际子节点数)|字母表大，但子节点稀疏|
|平衡 BST|O(log R)|O(实际子节点数)|需要有序遍历或字母表极大|

#### 7.6.1 固定数组实现（最简）

```java

class TrieNodeArray {
    boolean isKey;
    TrieNodeArray[] next = new TrieNodeArray[128]; // ASCII
}
```
#### 7.6.2 哈希表实现

```java

class TrieNodeHash {
    boolean isKey;
    Map<Character, TrieNodeHash> next = new HashMap<>();
}
```
#### 7.6.3 BST 实现（用 TreeMap）

```java

class TrieNodeBST {
    boolean isKey;
    TreeMap<Character, TrieNodeBST> next = new TreeMap<>();
}
```
### 7.7 完整示例：基于哈希表的 Trie

```java

public class TrieSet {
    private static class Node {
        boolean isKey;
        Map<Character, Node> next = new HashMap<>();
    }
    private Node root = new Node();
    public void add(String key) {
        Node node = root;
        for (char c : key.toCharArray()) {
            node = node.next.computeIfAbsent(c, k -> new Node());
        }
        node.isKey = true;
    }
    public boolean contains(String key) {
        Node node = root;
        for (char c : key.toCharArray()) {
            node = node.next.get(c);
            if (node == null) return false;
        }
        return node.isKey;
    }
    // 前缀查询：返回所有以 prefix 开头的键
    public List<String> keysWithPrefix(String prefix) {
        List<String> results = new ArrayList<>();
        Node node = root;
        for (char c : prefix.toCharArray()) {
            node = node.next.get(c);
            if (node == null) return results;
        }
        collect(node, new StringBuilder(prefix), results);
        return results;
    }
    private void collect(Node node, StringBuilder prefix, List<String> results) {
        if (node.isKey) results.add(prefix.toString());
        for (Map.Entry<Character, Node> entry : node.next.entrySet()) {
            prefix.append(entry.getKey());
            collect(entry.getValue(), prefix, results);
            prefix.deleteCharAt(prefix.length() - 1);
        }
    }
}
```
---

## 📊 八、知识点总结（彩色标记）

|知识点|核心内容|<span style="color:red">重点/易错点</span>|
|---|---|---|
|拓扑排序|有向无环图的线性排序，所有边从左到右|<span style="color:red">后序遍历反转得到拓扑序；必须是无环图</span>|
|Dijkstra 负权问题|负权边会导致已访问顶点被更新，破坏算法正确性|<span style="color:red">Dijkstra 不适用于负权图</span>|
|DAG 最短路径|按拓扑顺序松弛所有边，可处理负权|<span style="color:red">一次遍历即可，无需优先队列；时间复杂度 O(V+E)</span>|
|最长路径问题|一般图无高效解；DAG 上可通过取负权归约到最短路径|<span style="color:red">归约思想：将新问题转化为已知问题</span>|
|归约（Reduction）|用问题 Q 的解法解决问题 P，称 P 归约到 Q|<span style="color:red">3SAT 可归约到独立集，证明 NP 完全性</span>|
|字符键数组实现|用 ASCII 值作为索引，直接数组访问|<span style="color:red">严格 O(1)，但键空间必须有限</span>|
|Trie 结构|字符多叉树，节点标记是否为键结尾|<span style="color:red">查找时间 O(L)，与集合大小无关；支持前缀匹配</span>|
|Trie 子节点实现|固定数组（快但耗内存）、哈希表（平衡）、BST（有序）|<span style="color:red">根据字母表大小和空间需求选择</span>|

---

## 🧪 九、思考题与答案

### 1. 为什么 DAG 最短路径算法不需要优先队列？

**答案**：因为 DAG 的拓扑顺序保证了当处理一个顶点时，所有能到达它的路径都已经通过之前的顶点处理过，因此当前已知距离已是最优。无需动态选择最小顶点。

### 2. 如果 DAG 中有负权边，但存在负权环（不可能，因为 DAG 无环），算法还能工作吗？

**答案**：DAG 定义无环，所以不存在负权环。算法在负权边上仍然正确，因为拓扑顺序仍然有效，且路径简单。

### 3. 最长路径问题在 DAG 上的归约解法中，为什么要取负两次？

**答案**：取负一次将最长路径问题转化为最短路径问题（找最小值），得到的结果是负的最长距离，再取负一次得到正的最长距离。

### 4. 独立集问题中的“独立”是什么意思？

**答案**：独立集指图中顶点子集，其中任意两个顶点都不相邻（没有边直接连接）。即集合内所有顶点之间没有边。

### 5. 用固定数组实现 Trie 子节点有什么优缺点？

**优点**：查找子节点 O(1)，实现简单。  
**缺点**：每个节点都要分配大小为 R 的数组，即使大多数位置为空，浪费大量内存。R 较大时不可行。

### 6. 哈希表实现的 Trie 如何节省内存？

**答案**：只存储实际存在的子节点映射，每个节点只占用与子节点数量成正比的空间，而不是固定大小。

### 7. 为什么 Trie 特别适合前缀匹配（如自动补全）？

**答案**：Trie 将具有相同前缀的字符串存储在同一条路径上，因此只需沿着前缀路径走到底，就能快速找到所有以该前缀开头的字符串（通过遍历子树）。

### 8. 如果一个字符串集合很大，但所有字符串都很长，Trie 和哈希表哪个更优？

**答案**：Trie 查找时间与字符串长度 L 有关，哈希表平均 O(1) 但需要计算哈希并可能冲突。如果 L 很长但 N 也很大，Trie 可能更快（因为 O(L) 与 N 无关，而哈希表可能因冲突退化）。另外，Trie 支持前缀查询，哈希表不支持。具体选择取决于应用场景。

### 9. 如何判断一个有向图是否有环？

**答案**：可以用 DFS，维护递归栈标记。如果在遍历过程中遇到已在递归栈中的顶点，则存在环。也可以用拓扑排序：如果无法对所有顶点排序（即存在环），则算法会失败。

### 10. 归约在算法设计中有什么重要意义？

**答案**：归约让我们可以复用已知算法，避免从零开始。它也是理论计算机科学中证明问题难度（如 NP 完全性）的基本工具。