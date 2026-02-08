# 📚 第14讲：不相交集合（Disjoint Sets / Union-Find）深度详解

## 一、课前概述与课程哲学

### 🎯 本单元核心目标

本课程进入**数据结构精讲单元**，我们将通过几周时间深入剖析经典数据结构：

-  ==**集合（Set）==的实现机制**
    
- ==**映射（Map）==的内部结构**
    
- ==**优先队列（Priority Queue）==的工作方式**
    

### 🔍 今日重点：不相交集合数据结构

我们将从零推导 **Disjoint Sets（并查集）** 数据结构，解决**动态连通性（Dynamic Connectivity）** 问题。重点展示：

- 数据结构如何从**基础实现**演变为**高效优化**
    
- **底层抽象选择**如何影响算法**渐近复杂度**（使用Big-Θ表示法）
    
- **设计决策**如何影响**代码复杂度**（10行vs100行实现差异）
    

### 🧠 教学哲学："洞穴寓言"方法


```text

用户视角（洞穴外的皮影戏）     vs     实现视角（洞穴内的操作）
只关心接口功能                需要处理底层复杂性
"这两个点连通吗？"             如何高效存储和查询连通关系
```
## 二、动态连通性问题详解

### 📋 问题定义

给定N个元素，支持两种操作：

1. **connect(p, q)**：连接元素p和q
    
2. **isConnected(p, q)**：查询p和q是否连通
    

### 🔗 关键特性：传递性

```text

如果 A 连接 B，B 连接 C，则 A 连接 C
即使 A 和 C 没有直接连接
```
### 🌐 真实世界例子：国家连接

```java

// 用户操作
connect(俄罗斯, 中国);
connect(俄罗斯, 蒙古);
// 查询结果
isConnected(中国, 蒙古);  // 返回 true（通过俄罗斯连接）
isConnected(美国, 蒙古);  // 返回 false（初始状态）
connect(俄罗斯, 美国);
isConnected(美国, 蒙古);  // 返回 true（现在连通了）
```
## 三、接口设计与简化

### 🎨 接口定义

```java

public interface DisjointSets {
    /** 连接两个元素 p 和 q */
    void connect(int p, int q);
    
    /** 检查两个元素是否连通 */
    boolean isConnected(int p, int q);
}
```
### 🔧 设计简化

为了简化实现，我们做出两个约定：

1. **元素标识**：只使用整数（0, 1, 2, ...）代表元素，而非任意对象
    
2. **提前声明**：创建时必须指定元素总数N，初始状态所有元素不连通
    

```java

// 使用示例
DisjointSets ds = new DisjointSets(7);  // 创建7个元素：0-6
ds.connect(0, 1);
ds.connect(1, 2);
ds.connect(0, 4);
ds.connect(3, 5);
ds.isConnected(2, 4);  // true（通过0-1-2和0-4路径）
ds.isConnected(3, 0);  // false（初始不连通）
ds.connect(4, 6);
ds.connect(3, 6);
ds.isConnected(3, 0);  // true（现在连通：3-6-4-0）
```
## 四、逐步改进的实现策略

### 第一阶段：朴素方法（Naive Approach）

#### 思路：记录所有连接边

```text

存储方式：记录每一条连接边 (0,1), (1,2), (0,4), (3,5), ...
查询方式：从p出发进行图搜索，看是否能到达q
```
#### ⚠️ 问题

- **存储开销大**：需要存储大量边
    
- **查询效率低**：每次查询都需要遍历相关路径
    
- **实现复杂**：需要图搜索算法
    

### 第二阶段：集合列表（List of Sets）

#### 思路：直接维护连通分量

```text

初始状态：{0}, {1}, {2}, {3}, {4}, {5}, {6}
连接(0,1)：合并{0}和{1} → {0,1}
连接(1,2)：合并{0,1}和{2} → {0,1,2}
最终状态：{0,1,2,4}, {3,5}, {6}
```
#### 实现方式

```java

List<Set<Integer>> components;
```
#### 性能分析

|操作|时间复杂度|原因|
|---|---|---|
|构造器|Θ(N)|需要创建N个单元素集合|
|`connect(p, q)`|O(N)|需要查找p和q所在集合，然后合并|
|`isConnected(p, q)`|O(N)|需要遍历所有集合查找p和q|

#### 💀 致命缺陷

```java

// 查找元素p所在集合需要遍历所有集合
for (Set<Integer> set : components) {
    if (set.contains(p)) {
        // 找到了，但可能需要遍历大量集合
    }
}
// 最坏情况：检查p和q需要遍历所有集合两次
```
### 第三阶段：快速查找（Quick Find）

#### 💡 核心洞察

与其存储**集合的集合**，不如为每个元素分配一个**集合标识符（ID）**

#### 数据结构

```java

int[] id;  // id[i] = 元素i所属集合的ID
```
#### 初始状态（N=7）

```text

元素:  0  1  2  3  4  5  6
id[]:  0  1  2  3  4  5  6  // 每个元素自成一集合
```
#### 操作示例

```text

1. connect(0, 1)
   找到id[0]=0, id[1]=1
   将所有id=0的元素改为1
   结果: id[] = [1, 1, 2, 3, 4, 5, 6]
   
2. connect(1, 2)
   找到id[1]=1, id[2]=2
   将所有id=1的元素改为2
   结果: id[] = [2, 2, 2, 3, 4, 5, 6]
   
3. isConnected(0, 2)
   比较id[0]=2, id[2]=2 → 相等 → 返回true
```
#### 代码实现

```java

public class QuickFindDS implements DisjointSets {
    private int[] id;
    
    public QuickFindDS(int N) {
        id = new int[N];
        for (int i = 0; i < N; i++) {
            id[i] = i;  // 每个元素初始化为自己的ID
        }
    }
    
    @Override
    public boolean isConnected(int p, int q) {
        return id[p] == id[q];  // O(1)时间复杂度！
    }
    
    @Override
    public void connect(int p, int q) {
        int pid = id[p];
        int qid = id[q];
        // 如果已经连通，无需操作
        if (pid == qid) return;
        
        // 将所有属于pid集合的元素改为qid
        for (int i = 0; i < id.length; i++) {
            if (id[i] == pid) {
                id[i] = qid;  // 需要遍历整个数组！
            }
        }
    }
}
```
#### 性能分析

|操作|时间复杂度|解释|
|---|---|---|
|构造器|Θ(N)|需要初始化大小为N的数组|
|`isConnected(p, q)`|**Θ(1)**|只需要比较两个数组元素|
|`connect(p, q)`|**Θ(N)**|需要遍历整个数组修改值|

#### ⚖️ 优缺点

- ✅ **查询极快**：O(1)时间判断连通性
    
- ❌ **合并极慢**：每次合并需要O(N)时间，对于大量操作无法接受
    
- ❌ **无法处理大规模数据**：百万元素合并一次需要百万次操作
    

### 第四阶段：快速合并（Quick Union）

#### 💡 核心洞察

使用**树结构**表示集合，每个节点指向其父节点，根节点代表集合

#### 数据结构

java

int[] parent;  // parent[i] = 元素i的父节点
// 根节点的parent通常为-1或指向自己

#### 树形表示思想

```text

集合 {0,1,2,4} 可以表示为：
    0
   / \
  1   4
  |
  2

parent[] 表示：
索引: 0  1  2  3  4  5  6
值:  -1  0  1 -1  0  3 -1  // -1表示根节点
```
#### 操作原理

1. **查找根节点（find）**：不断向上查找，直到根节点
    
2. **判断连通性**：检查两个元素是否有相同的根节点
    
3. **连接操作**：将一个根节点指向另一个根节点
    

#### 代码实现

```java

public class QuickUnionDS implements DisjointSets {
    private int[] parent;
    
    public QuickUnionDS(int N) {
        parent = new int[N];
        for (int i = 0; i < N; i++) {
            parent[i] = -1;  // -1表示根节点
        }
    }
    
    // 私有辅助方法：查找元素i的根节点
    private int find(int i) {
        while (parent[i] >= 0) {  // 非根节点
            i = parent[i];  // 向上查找
        }
        return i;  // 返回根节点
    }
    
    @Override
    public boolean isConnected(int p, int q) {
        return find(p) == find(q);
    }
    
    @Override
    public void connect(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        if (rootP != rootQ) {
            parent[rootP] = rootQ;  // 将p的根节点指向q的根节点
        }
    }
}
```
#### 性能分析

|操作|时间复杂度|解释|
|---|---|---|
|构造器|Θ(N)|初始化数组|
|`find`操作|O(H)|H是树的高度|
|`isConnected`|O(H)|需要两次find操作|
|`connect`|O(H)|需要两次find操作+一次赋值|

#### ⚠️ 最坏情况：树退化为链表

```text

连续执行：
connect(4, 3)   // 树：4←3
connect(3, 2)   // 树：4←3←2
connect(2, 1)   // 树：4←3←2←1
connect(1, 0)   // 树：4←3←2←1←0

此时树高=4，查找需要4步
如果有N个元素，最坏树高=N，操作时间=O(N)
```
### 第五阶段：加权快速合并（Weighted Quick Union）

#### 💡 核心改进

合并时总是将**较小的树**连接到**较大的树**下，避免树过高

#### 数据结构

```java

int[] parent;  // 根节点存储负的树大小，非根节点存储父节点索引
```
#### 存储方式创新

```text

传统：根节点 parent[i] = -1
加权：根节点 parent[i] = -size  // 负数，绝对值表示树的大小

例如：包含4个元素的树，根节点存储 -4
```
#### 代码实现

```java

public class WeightedQuickUnionDS implements DisjointSets {
    private int[] parent;
    
    public WeightedQuickUnionDS(int N) {
        parent = new int[N];
        for (int i = 0; i < N; i++) {
            parent[i] = -1;  // 每个树初始大小为1
        }
    }
    
    private int find(int i) {
        while (parent[i] >= 0) {
            i = parent[i];
        }
        return i;
    }
    
    @Override
    public boolean isConnected(int p, int q) {
        return find(p) == find(q);
    }
    
    @Override
    public void connect(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        
        if (rootP == rootQ) return;
        
        // 比较树的大小（注意：parent[root]存储的是负的大小）
        int sizeP = -parent[rootP];
        int sizeQ = -parent[rootQ];
        
        // 总是将较小的树连接到较大的树下
        if (sizeP < sizeQ) {
            parent[rootP] = rootQ;           // 小树根指向大树根
            parent[rootQ] = -(sizeP + sizeQ); // 更新大树大小
        } else {
            parent[rootQ] = rootP;           // 小树根指向大树根
            parent[rootP] = -(sizeP + sizeQ); // 更新大树大小
        }
    }
}
```
#### 性能分析的关键定理

**定理**：使用加权快速合并，包含k个节点的树的高度最多为⌊log₂k⌋

**证明思路**：

1. 树T的高度只在合并时可能增加
    
2. 当树T₁（大小k₁）合并到树T₂（大小k₂）时
    
    - 如果k₁ ≤ k₂，则新树高度为max(h₁+1, h₂)
        
    - 但更重要的是：合并后树的大小至少是原来的两倍
        
3. 每次节点所在的树大小至少翻倍，因此最多log₂N次合并后，树的大小达到N
    

**最坏情况示例**：

```text

N=1: 高度=0
N=2: 高度=1  (1+1)
N=4: 高度=2  (2+2)
N=8: 高度=3  (4+4)
N=16: 高度=4 (8+8)
规律：树高 = log₂(N)
```
#### 性能总结

|操作|时间复杂度|解释|
|---|---|---|
|构造器|Θ(N)|初始化数组|
|`isConnected`|**O(log N)**|需要两次find，树高为O(log N)|
|`connect`|**O(log N)**|需要两次find+更新，树高为O(log N)|

## 五、性能对比总览

### 📊 四种实现方式对比

|实现方式|构造器|connect|isConnected|空间|适用场景|
|---|---|---|---|---|---|
|**ListOfSetsDS**|Θ(N)|O(N)|O(N)|O(N)|教学演示，实际不使用|
|**QuickFindDS**|Θ(N)|**Θ(N)**|**Θ(1)**|O(N)|查询远多于合并的场景|
|**QuickUnionDS**|Θ(N)|O(N)|O(N)|O(N)|基础版本，有优化空间|
|**WeightedQuickUnionDS**|Θ(N)|**O(log N)**|**O(log N)**|O(N)|**实际应用首选**|

### 🎯 实际应用选择

对于大多数实际问题，**加权快速合并**是最佳选择：

- **可接受的理论保证**：对数时间对于大规模数据足够快
    
    - log₂(1,000,000) ≈ 20
        
    - log₂(1,000,000,000) ≈ 30
        
- **实现相对简单**：只需要稍微修改QuickUnion的合并策略
    
- **内存效率高**：只使用一个数组
    

## 六、高级优化：路径压缩

### 💡 进一步优化思想

在查找根节点过程中，将路径上的所有节点直接指向根节点，压平树结构

### 代码实现（简单版本）

```java

private int findWithCompression(int i) {
    if (parent[i] < 0) {
        return i;  // 根节点
    } else {
        // 递归查找并压缩路径
        parent[i] = findWithCompression(parent[i]);
        return parent[i];
    }
}
```
### 路径压缩效果

```text

压缩前：      压缩后：
  0             0
 / \           /|\
1   2         1 2 3
    |
    3
查找3的根节点时，将3直接连接到0
```
### 性能提升

- **均摊复杂度**：接近O(α(N))，其中α是反阿克曼函数
    
- **实际效果**：对于任何实际规模的N，α(N) < 5，几乎是常数时间
    

## 七、应用场景与扩展

### 🌐 实际应用领域

1. **网络连通性**：判断两台计算机是否在同一网络
    
2. **社交网络**：判断两个人是否有共同好友链
    
3. **图像处理**：连通区域标记
    
4. **游戏开发**：判断两个游戏对象是否在同一个物理空间
    
5. **编译器**：变量名等价性分析
    

### 🔄 支持任意对象类型

虽然我们的实现只支持整数，但可以通过映射扩展到任意对象：

```java

public class GenericDisjointSets<T> {
    private Map<T, Integer> itemToId;
    private WeightedQuickUnionDS uf;
    
    public void connect(T a, T b) {
        int idA = getOrCreateId(a);
        int idB = getOrCreateId(b);
        uf.connect(idA, idB);
    }
    
    public boolean isConnected(T a, T b) {
        // ... 类似实现
    }
}
```
## 八、关键概念总结

### 🔑 核心要点

1. **连通性传递**：连接关系具有传递性，这是数据结构设计的基础
    
2. **抽象层次**：用户只关心接口，实现可以不断优化而不改变API
    
3. **权衡设计**：需要在查询速度、合并速度、实现复杂度之间平衡
    
4. **渐进改进**：从朴素方法到高效算法是逐步优化的过程
    

### 🧠 学习收获

通过本讲学习，你应该能够：

- 理解并查集解决**动态连通性问题**的基本原理
    
- 分析不同实现方式的**时间复杂度**
    
- 实现**加权快速合并**并理解其性能保证
    
- 理解**路径压缩**的优化思想
    
- 在实际问题中选择合适的并查集实现
    

## 九、练习题与思考

### 📝 思考题

1. 如果元素数量N很大，但连接操作很少，哪种实现方式最好？
    
2. 加权快速合并中，为什么按大小合并比按高度合并实现更简单？
    
3. 路径压缩为什么不能直接在connect操作中实现？
    

### 💻 编码练习

```java

// 实现完整的WeightedQuickUnionDS，并测试以下场景：
DisjointSets ds = new WeightedQuickUnionDS(10);
ds.connect(0, 1);
ds.connect(2, 3);
ds.connect(4, 5);
ds.connect(6, 7);
ds.connect(8, 9);
ds.connect(0, 2);
ds.connect(4, 6);
ds.connect(0, 4);
// 现在检查：
// ds.isConnected(1, 7) 应该返回什么？
// ds.isConnected(3, 9) 应该返回什么？
```
## 十、进一步学习资源

### 📚 推荐阅读

1. **算法导论**：第21章 用于不相交集合的数据结构
    
2. **Algorithms, 4th Edition**：Chapter 1.5 Union-Find
    
3. **原始论文**：Bernard A. Galler and Michael J. Fischer, "An improved equivalence algorithm"
    

### 🎥 可视化工具

- [USFCA可视化](https://www.cs.usfca.edu/~galles/visualization/UnionFind.html)
    
- [VisuAlgo可视化](https://visualgo.net/en/ufds)
    

---

通过本讲深度学习，你已经掌握了并查集这一重要数据结构的**核心原理**、**实现方法**和**优化策略**。记住：并查集不仅是解决连通性问题的工具，更是**算法设计思想**的绝佳示例——通过**逐步优化**和**巧妙设计**，可以将看似复杂的操作降低到可接受的时间复杂度。