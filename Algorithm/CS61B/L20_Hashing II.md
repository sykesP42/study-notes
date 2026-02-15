## 🧭 一、集合与映射 ADT 回顾

### 1.1 集合（Set）与映射（Map）

- **集合（Set）**：存储不重复的元素，核心操作：
    
    - `add(x)`：将元素 x 加入集合（如果已存在则无变化）
        
    - `contains(x)`：检查 x 是否在集合中
        
- **映射（Map）**：存储键值对（key-value），核心操作：
    
    - `put(key, value)`：存入键值对
        
    - `get(key)`：根据键取值
        

这两种抽象数据类型（ADT）是编程中最常用的数据容器。

### 1.2 已有实现的性能对比

|实现|contains 最坏|add 最坏|特点与限制|
|---|---|---|---|
|`ArraySet`|Θ(N)|Θ(N)|简单但线性扫描|
|BST（二叉搜索树）|Θ(N)|Θ(N)|随机数据平均 Θ(log N)，但可能退化为链表|
|2-3 树|Θ(log N)|Θ(log N)|完美平衡，但实现复杂|
|LLRB（左倾红黑树）|Θ(log N)|Θ(log N)|与 2-3 树一一对应，依然复杂|

**关键问题**：

- 基于比较的树结构要求元素必须可比较（如整数、字符串可以比较大小），但现实中有很多对象无法直接比较（比如“苹果”和“橙子”）。
    
- Θ(log N) 已经很快，但**有没有可能比 Θ(log N) 更快**？答案是**有**，那就是**平均 O(1)** 的哈希表。
    

---

## 🧠 二、哈希表的诞生 —— 从生活场景到数据结构

### 2.1 初始想法：WriteOnTheWallSet（写在墙上的集合）

想象一面墙，你可以在上面任意位置用铅笔写数字。

- **添加**：找个空位直接写 → **O(1)**
    
- **查找**：必须扫视整个墙面 → **O(N)**
    

✅ 优点：添加极快，且不关心数据类型（汉字、符号都能写）。  
❌ 缺点：查找太慢。

**效率判断原则**：对人类困难的操作对计算机也困难；对人类繁琐但简单的操作对计算机容易；对人类耗时的操作对计算机同样耗时。所以线性扫描对计算机也是 O(N)。

### 2.2 改进：BobaCounterSet（奶茶店分类法）

奶茶店取餐时，店员根据订单号的**末位数字**将做好的奶茶放在对应编号的格子里（0~9）。  
顾客取餐时只需检查自己订单号对应的格子，不用翻遍所有格子。

**应用到集合**：

- 将墙面分成 10 个区域（桶），每个区域对应一个数字（0~9）。
    
- **添加**：计算数字的末位，放入对应区域 → **O(1)**
    
- **查找**：只检查末位对应的区域 → 理想情况下 **Θ(N/10)**，但可能分布不均（比如所有数字都以 0 结尾）。
    

**暴露的问题**：

- 空间浪费：即使某个区域没有元素，也要预留空间。
    
- 分布不均：实际数据可能集中在某几个数字上，导致这些区域的查找变成线性。
    
- 无法直接处理字符串：如何将字符串映射到 0~9？
    

### 2.3 动态链表数组：DynamicArrayOfListsSet

为了解决空间浪费和分布不均，我们让每个区域**动态增长**，比如用链表来存储每个桶中的元素。

- **添加**：找到桶号，将元素插入链表（头插或尾插）。
    
- **查找**：只遍历对应桶内的链表。
    

此时，如果元素均匀分布，每个桶平均有 **N/M** 个元素（N 总元素数，M 桶数），查找时间 **Θ(N/M)**。  
如果 M 是常数，那么 N/M 仍会随着 N 增长而增长，最终退化为 Θ(N)。

**解决方案**：让 **M 随 N 增长**，保证 **N/M ≤ 常数**（即平均每个桶的元素个数不超过某个阈值），这样查找就是常数时间。

---

## 🧮 三、缩减函数（Reduction Function）与模运算

如何将任意对象映射到 0 ~ M-1 的桶号？最自然的选择：**模运算（mod）**。

- 对整数：`index = value % M`
    
- 对字符串：先转化为整数，再取模
    

**模运算的优点**：

- 适用于任意 M
    
- 对随机输入能均匀分布
    
- 改变 M 时，只需重新计算所有元素的桶号
    

### 为什么模运算是“缩减函数”？

因为它将一个很大的整数（哈希码）缩减到一个较小的范围内（桶数组大小）。这个函数必须是确定性的（同一个哈希码总是得到同一个桶号），并且尽量均匀。

### 其他缩减函数？

理论上也可以用其他函数，比如取最高位、取数字和等，但模运算是最简单且效果最好的。

---

## 📈 四、动态扩容策略

### 4.1 扩容时机

当**负载因子（load factor）** `N/M` 超过某个阈值（例如 0.75）时，就需要增加桶的数量。负载因子是哈希表满的程度，过大会导致链表变长，过小浪费空间。

### 4.2 如何增加 M

- **算术增长**：每次加一个固定常数 → 每次扩容都要 O(N) 时间，均摊后仍是 O(N)（因为扩容次数太多）。
    
- **几何增长**：每次乘以一个常数（如翻倍） → 均摊后能达到 **O(1)**。
    

✅ 几何增长（翻倍）是哈希表常用的扩容策略，类似 `ArrayList` 的扩容。为什么翻倍能保证均摊 O(1)？因为扩容次数大约是 log N，每次扩容的 O(N) 成本均摊到 N 次插入，每次插入均摊成本 O(1)。

### 4.3 扩容示例

设阈值 `N/M ≥ 1.5` 时翻倍 M。

初始 M=4，添加 7,16,3,11,20,13：

- 7%4=3 → 桶3
    
- 16%4=0 → 桶0
    
- 3%4=3 → 桶3
    
- 11%4=3 → 桶3
    
- 20%4=0 → 桶0
    
- 13%4=1 → 桶1
    

此时 N=6，M=4，负载因子 6/4=1.5 → 触发扩容，M 翻倍为 8。然后**重新哈希**所有元素：

- 7%8=7，16%8=0，3%8=3，11%8=3，20%8=4，13%8=5
    

现在元素分布更均匀了。

---

## 🧵 五、字符串如何变成整数？（哈希函数设计）

### 5.1 简单想法：字母位置求和

将 a=0, b=1, ..., z=25，把单词中每个字母的值加起来。  
例如 "cat" = 2+0+19=21  
❌ **问题**：

- 不同单词可能和相同（如 "act" 也是 21），碰撞率高。
    
- 短单词集中在小区间，长单词数值大但实际英语中极少有超长单词，导致分布不均匀。
    

### 5.2 更优：二十六进制

将单词看成以 26 为基数的数：

```text

"cat" = 2×26² + 0×26¹ + 19×26⁰ = 1371
```
- 注意：通常 a=1, b=2, ..., z=26（避免 "a" 和 "aa" 都映射到 0，因为 "a" 按公式 1×26⁰=1，"aa"=1×26¹+1×26⁰=27，不同）。
    
- 这种编码能保证每个小写单词得到唯一整数，只要基数 ≥ 26。实际上，基数为 26 时，不同单词必然得到不同整数（因为这是唯一表示法）。
    

### 5.3 扩展到所有 ASCII 字符

ASCII 中可打印字符范围 33~126，最大 126。  
因此可以用 **126 为基数**，将每个字符的 ASCII 码作为位值。  
例如 "bea" = 98×126² + 101×126¹ + 101×126⁰。

#### Java 实现示例

```java

public static int asciiToInt(String s) {
    int intRep = 0;
    for (int i = 0; i < s.length(); i++) {
        intRep = intRep * 126;
        intRep = intRep + s.charAt(i);
    }
    return intRep;
}
```
这个算法类似将字符串解析为 126 进制数。

### 5.4 扩展到 Unicode

如果要支持中文等更多字符，基数需要提高到最大码点。Java 中 `char` 是 16 位 Unicode，最大码点 0xFFFF（65535），但 Unicode 实际有超过这个范围的字符（用两个 `char` 表示）。对于常用汉字，码点范围大约 0x4E00~0x9FFF，最大约 40959。如果以 40959 为基数，一个 4 个汉字的字符串计算出的整数会巨大，远超 `int` 范围。例如 "横田誠司" 按 40959 进制计算得到约 1.86e18，远超 2^31-1。

---

## ⚠️ 六、整数溢出与哈希码（hashCode）

### 6.1 Java 的 int 溢出

```java

int x = 2147483647;
System.out.println(x + 1); // 输出 -2147483648
```
超过最大值后会绕回最小值，产生负数。这是 Java 整数的循环特性。

### 6.2 哈希码的必然碰撞

由于 `int` 只有约 40 亿个可能值（-2^31 到 2^31-1），而字符串的可能种类无限，**碰撞不可避免**。  
因此哈希表必须能处理碰撞（如链表法）。

### 6.3 为什么 Java 的 String.hashCode() 用 31？

Java 的 `String` 哈希码计算公式为：

```text

s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
```
为什么是 31？

- 31 是质数，能减少碰撞。
    
- 乘法可以用移位减优化：`31 * i = (i << 5) - i`，提高计算效率。
    
- 实验表明 31 能产生较均匀的分布。
    

#### String.hashCode() 示例

```java

"横田誠司".hashCode(); // 可能得到 839611422
```
尽管有无数其他字符串也会映射到同一个哈希码，但只要分布均匀，哈希表依然高效。

### 6.4 负哈希码的处理

取模时要注意负数。`hashCode()` 可能返回负数，直接 `% M` 会得到负数索引。解决方法：

- `Math.floorMod(hash, M)` 返回 [0, M-1] 的正余数。
    
- 或者 `(hash & 0x7fffffff) % M` 先去掉符号位。
    

---

## 🧱 七、哈希表（Hash Table）最终结构

### 7.1 定义

哈希表是一个包含 **M 个桶（bucket）** 的数组，每个桶通常是一个链表（**分离链接法**）。  
通过 **哈希函数** 将键转换为整数（哈希码），再通过取模得到桶下标，将键值对存入对应桶。

### 7.2 操作流程

1. **添加**：
    
    - 计算键的哈希码 `h = key.hashCode()`
        
    - 桶号 `i = h % M`（考虑负数修正）
        
    - 遍历桶 `i` 的链表，如果找到键相同的节点（用 `equals` 比较），则更新值；否则在链表头部或尾部插入新节点
        
    - 如果添加后 `N/M` 超过阈值，则扩容（M 翻倍，重新哈希所有元素）
        
2. **查找**：
    
    - 计算桶号，遍历该桶链表，用 `equals` 比较找到目标，返回值或 null。
        
3. **删除**（HashMap 支持）：
    
    - 类似查找，找到后从链表中移除。
        

### 7.3 时间复杂度

- **理想情况**（均匀分布，每个桶大小 O(1)）：**平均 O(1)**
    
- **最坏情况**（所有键映射到同一桶）：**O(N)**（链表长度 N）
    

---

## ⚖️ 八、hashCode 与 equals 的一致性（核心重点）

### 8.1 黄金法则

> **如果两个对象通过 `equals()` 比较相等，那么它们的 `hashCode()` 必须相等。**  
> 反之则不必（不同对象可以有相同哈希码，即碰撞）。

**为什么？**  
哈希表先用哈希码定位桶，再用 `equals` 在桶内精确查找。如果两个相等对象哈希码不同，它们会进入不同桶，导致永远找不到。

### 8.2 反面示例：只重写 equals，不重写 hashCode

假设 `ColoredNumber` 类：

```java

public class ColoredNumber {
    private int num;
    private Color color;
    // constructor, getters...
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof ColoredNumber)) return false;
        ColoredNumber that = (ColoredNumber) o;
        return this.num == that.num;  // 只比较 num
    }
    
    // 没有重写 hashCode，所以继承 Object 的 hashCode，基于内存地址
}
```
#### 实验一：查找丢失

```java

HashSet<ColoredNumber> set = new HashSet<>();
for (int i = 0; i < 20; i++) {
    set.add(new ColoredNumber(i));  // 每个对象 num 从 0 到 19
}
ColoredNumber twelve = new ColoredNumber(12);
System.out.println(set.contains(twelve)); // 大概率 false
```
**可视化解释**：  
假设 M=6，默认哈希码基于内存地址，20 个对象可能分布在各个桶。新对象 `twelve` 的内存地址不同，哈希码不同，取模后可能进入另一个桶，而真正存储 `num=12` 的对象在别的桶，所以找不到。

#### 实验二：重复添加

```java

HashSet<ColoredNumber> set = new HashSet<>();
for (int i = 0; i < 20000; i++) {
    set.add(new ColoredNumber(12));
}
System.out.println(set.size()); // 输出 20000，而不是 1
```
因为每个新对象都有不同内存地址，哈希码不同，被分配到不同桶，所以 `HashSet` 认为它们是不同元素。尽管 `equals` 认为它们相等，但哈希表根本没机会调用 `equals`，因为桶都不同。

### 8.3 正确做法：同时重写 hashCode

让 `hashCode` 也基于 `num` 计算：

```java

@Override
public int hashCode() {
    return Integer.hashCode(num);   // 或直接 return num;
}
```
现在所有 `num` 相等的对象哈希码相同，它们会被放入同一桶。当添加 20000 个 `num=12` 时，第一个进入桶，后续发现桶里已经有 `equals` 相等的对象，就不会重复添加，`size` 为 1。

### 8.4 为什么默认 hashCode 是好的，但还需要自定义？

默认 `hashCode` 基于内存地址，分布确实很好（近似随机）。  
但问题是：如果两个对象在逻辑上相等（`equals` 返回 true），但内存地址不同，它们的默认哈希码就不同，违反了黄金法则。  
因此**只要重写了 `equals`，就必须重写 `hashCode` 以保持一致性**。

---

## 🔒 九、不可变对象与可变键陷阱

### 9.1 不可变对象的定义

> 对象创建后，其状态（内部数据）在任何可观察的方面都不能改变。

### 9.2 判断不可变性练习（详细分析）

#### 例1：`Pebble`

```java

public class Pebble {
    public int weight;
}
```
❌ **可变**：`weight` 是 public 且非 final，外部代码可以直接 `pebble.weight = 10` 修改。

#### 例2：`Rock`

```java

public class Rock {
    private final int weight;
    public Rock(int weight) { this.weight = weight; }
}
```
✅ **不可变**：`weight` 是 private final，没有 setter，无法修改。即使有 getter，也只是返回基本类型副本，不影响内部状态。

#### 例3：`RocksBox`

```java

public class RocksBox {
    public final Rock[] rocks;
    public RocksBox(Rock[] rocks) { this.rocks = rocks; }
}
```
❌ **可变**：虽然 `rocks` 引用是 final，但数组内容可以被外部修改。例如：

```java

Rock[] rocks = new Rock[10];
RocksBox box = new RocksBox(rocks);
rocks[0] = new Rock(5);   // 修改了 box 内部数组的内容！
```
`final` 只保证引用不改变，不保证引用指向的对象内容不变。

#### 例4：`SecretRocksBox`（尝试封装）

```java

public class SecretRocksBox {
    private final Rock[] rocks;
    public SecretRocksBox(Rock[] rocks) { this.rocks = rocks; }
    public Rock[] getRocks() { return rocks; }
}
```
❌ **仍然可变**：问题有两个：

1. 构造函数直接保存外部传入的数组引用，外部如果保留了该数组引用，仍可修改数组内容。
    
2. `getRocks()` 直接返回内部数组引用，调用者可以修改数组内容，例如 `box.getRocks()[0] = null`。
    

### 9.3 如何真正实现不可变

```java

public class ImmutableRocksBox {
    private final Rock[] rocks;
    public ImmutableRocksBox(Rock[] rocks) {
        // 防御性拷贝：创建新数组，复制内容
        this.rocks = Arrays.copyOf(rocks, rocks.length);
    }
    public Rock[] getRocks() {
        // 返回拷贝，而不是原始数组
        return Arrays.copyOf(rocks, rocks.length);
    }
}
```
同时要求 `Rock` 本身也是不可变的，否则仍可通过修改 `Rock` 对象内容破坏不可变性。如果 `Rock` 可变，防御性拷贝只能保护数组结构，但 `Rock` 对象内部仍可能被修改。

### 9.4 不可变性的优缺点

✅ **优点**：

- 行为可预测，无副作用
    
- 调试简单，避免因状态改变引起的 bug
    
- 天然线程安全，适合并发
    

❌ **缺点**：

- 每次“修改”都需要创建新对象，可能带来性能开销（例如 `String` 拼接）
    
- 需要仔细设计，防御性拷贝可能增加内存占用
    

### 9.5 可变键的致命陷阱

**绝对不要将可变对象用作哈希表的键！**  
因为一旦键被修改，其 `hashCode` 可能改变，导致它原来所在的桶位置失效，再也无法被找到。

#### 示例：用 List 作为键

```java

List<Integer> items = new ArrayList<>();
items.add(0); items.add(1);
HashSet<List<Integer>> set = new HashSet<>();
set.add(items);                     // 此时 items 的 hashCode 为 962
set.add(List.of(2, 3));             // 另一个列表
items.add(7);                        // 修改原列表
System.out.println(set.contains(items)); // 大概率 false
```
**步骤分析**：

1. `[0,1]` 的哈希码为 962，假设桶数 M=4，则桶索引 = 962 % 4 = 2。
    
2. 修改列表后，`[0,1,7]` 的哈希码变为 29829，新索引 = 29829 % 4 = 1。
    
3. 但该对象物理上仍然在桶 2 中，因此 `contains` 去桶 1 查找，必然找不到。
    

**后果**：

- 对象在哈希表中“丢失”，无法被访问、更新或删除。
    
- 直到哈希表扩容并重新哈希所有元素，该对象才可能被重新分配到正确位置（但在此之前一直占用内存，造成内存泄漏）。
    

**更严重的例子**：

```java

set.contains(items);  // false，但 items 实际上还在集合中吗？是的，但找不到了。
set.remove(items);    // 无法删除
```
### 9.6 为什么编译器不阻止我们？

判断一个类是否可变在理论上等价于**停机问题**，不可判定。  
例如，一个类可能看起来可变，但实际使用时从未修改；或者看起来不可变，但通过反射可以修改。因此 Java 语言设计者无法在编译期禁止使用可变类型作为键，只能靠开发者自觉遵守规范。

> “如果你能自动检测可变键，你就摧毁了数学。” —— Josh Hug

### 9.7 正确做法

- 使用不可变类型作为键，如 `String`、`Integer`、`List.of()` 返回的不可变列表（Java 9+）。
    
- 如果必须用可变对象，确保它**在作为键期间不被修改**（例如先克隆一份不可变的副本，或者用快照模式）。
    
- 如果对象已经作为键放入集合，绝对不要修改它。
    

---

## 🧰 十、Java 中的 HashSet / HashMap 实现细节

### 10.1 HashSet 底层就是 HashMap

`HashSet` 内部持有一个 `HashMap<E, Object>`，所有元素作为键存入，值用一个固定的虚对象 `PRESENT` 占位。  
[查看 OpenJDK 源码](https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/HashSet.java)

```java

public class HashSet<E> {
    private transient HashMap<E,Object> map;
    private static final Object PRESENT = new Object();
    public boolean add(E e) {
        return map.put(e, PRESENT) == null;
    }
    // ...
}
```
### 10.2 HashMap 的核心参数

- **初始容量**：16（桶数组大小）
    
- **负载因子**：默认 0.75
    
- **扩容倍数**：翻倍（新容量 = 旧容量 << 1）
    
- **缩减函数**：不是简单的取模，而是  
    `(h = key.hashCode()) ^ (h >>> 16) & (capacity - 1)`
    

**为什么用这个扰动函数？**  
因为 `capacity` 是 2 的幂，`capacity - 1` 是一个低位掩码（例如 16-1=15 二进制 1111）。直接 `hash & (capacity-1)` 相当于取模，但只用了哈希码的低位。为了让高位也参与索引计算，将哈希码的高 16 位与低 16 位异或，混合高低位，减少碰撞。

### 10.3 树化优化（Java 8+）

当某个桶的链表长度超过阈值（默认 8）时，链表会被转换为**红黑树**，使最坏查找时间从 O(N) 降到 O(log N)。  
当桶内元素减少到 6 时，又会转回链表（避免频繁转换）。

源码注释（来自 HashMap）：

> “This map usually acts as a binned (bucketed) hash table, but when bins get too large, they are transformed into bins of TreeNodes, each structured similarly to those in java.util.TreeMap. Most methods try to use normal bins, but relay to TreeNode methods when applicable (simply by checking instanceof a node).”

**为什么用 8 作为阈值？**  
基于泊松分布，在理想随机哈希码下，桶中链表长度达到 8 的概率极低（约 0.00000006），所以触发树化的情况很少，不影响平均性能。而一旦触发，说明分布极差，需要树化来保证最坏情况性能。

### 10.4 负载因子 0.75 的模拟

- 保持 `N/M ≤ 0.75`，每个桶的平均长度 ≤ 0.75。
    
- 根据“球与箱子”概率模型（将 N 个球随机投入 M 个箱子），最长箱子中的球数约为 `log M / log log M`，当 M=16 时，最长桶长度约 5 左右。
    
- 因此查找时间接近常数。
    

**为什么负载因子不能太高？**  
太高会导致链表过长，查找变慢；太低会浪费空间，且扩容频繁。0.75 是时间和空间的折中。

---

## 📊 十一、哈希表性能分析与优化

### 11.1 影响性能的因素

1. **哈希函数的质量**：是否均匀分布。
    
2. **负载因子**：影响链表长度和扩容频率。
    
3. **冲突解决策略**：链表法 vs 开放寻址法（Java 用链表法，Java 8+ 加树化）。
    
4. **扩容策略**：翻倍扩容保证均摊 O(1)。
    

### 11.2 常见优化技巧

- **使用扰动函数混合高位**：如 Java 的 `(h ^ (h >>> 16))`。
    
- **选择质数作为模数**（如果容量不是 2 的幂），但 Java 用 2 的幂是为了快速位运算。
    
- **对于自定义类，组合多个字段的哈希码**：例如 `Objects.hash(field1, field2)` 或 `31 * result + (field == null ? 0 : field.hashCode())`。
    

### 11.3 最坏情况分析与应对

最坏情况是所有键映射到同一桶，退化为链表。Java 8+ 通过树化将最坏情况降到 O(log N)。此外，还可以通过改进哈希函数（如加密哈希）来避免恶意攻击导致的 DoS（但通常不必要）。

---

## 📌 十二、核心知识点总结（颜色标记）

|知识点|核心内容|<span style="color:red">重点/易错点</span>|
|---|---|---|
|哈希表工作流程|数据 → hashCode → 缩减 → 桶 → 存储|<span style="color:red">缩减通常用取模；扩容翻倍保证均摊 O(1)</span>|
|哈希函数设计|目标是均匀分布；整数返回自身即可；默认 hashCode 基于内存地址分布良好|<span style="color:red">不要用固定值（如 0）作为哈希码；不要用差的分布（如数字和）</span>|
|equals 与 hashCode|若 a.equals(b) 为 true，则 a.hashCode() 必须等于 b.hashCode()|<span style="color:red">只重写 equals 不重写 hashCode 会导致对象找不到或重复添加</span>|
|不可变对象|创建后状态不可变；需防御性拷贝并保证所有成员不可变|<span style="color:red">final 引用不能保证内部对象不可变；判断不可变很困难</span>|
|可变键风险|键修改后哈希码改变，对象丢失|<span style="color:red">绝对不要用可变对象作为哈希表键；使用不可变类型或确保键不被修改</span>|
|Java 实现优化|初始容量 16，负载因子 0.75，扩容翻倍；链表过长时转为红黑树|<span style="color:red">缩减函数用扰动混合高低位；树化阈值 8，退化阈值 6</span>|

---

## 🧠 十三、思考题与答案

### 1. 为什么说默认的 `hashCode` 分布是好的？它可能有什么缺点？

**答案**：

- 默认 `hashCode` 通常基于对象的内存地址，而内存地址在 JVM 中分配时具有一定随机性，因此不同对象的哈希码可以看作近似随机分布，能均匀分散到各个桶，所以“分布好”。
    
- 缺点：如果两个对象在逻辑上相等（`equals` 返回 true），但它们是不同实例，内存地址不同，哈希码就不同，违反了 `equals` 与 `hashCode` 的一致性。因此，一旦重写 `equals`，就必须重写 `hashCode` 来覆盖默认行为。
    

### 2. 如果一个类只重写 `equals` 而不重写 `hashCode`，将它放入 `HashSet` 会发生什么？

**答案**：

- 对象可能无法被正确查找。因为 `HashSet` 先用 `hashCode` 定位桶，如果两个相等对象的哈希码不同，它们会进入不同桶，导致 `contains` 返回 false（除非恰好落入同一桶）。
    
- 添加时可能会产生重复元素。因为每次添加新对象，即使 `equals` 认为相等，但哈希码不同，就会被视为不同元素插入不同桶，造成集合中出现“逻辑上相等”的多个副本，`size` 不正确。
    

### 3. 如何实现一个真正不可变的类？给出代码示例。

**答案**：

- 所有字段声明为 `private final`。
    
- 不提供任何修改字段的方法（setter）。
    
- 如果字段是可变对象（如数组、集合），在构造函数和 getter 中进行防御性拷贝。
    
- 确保类不能被继承（通常用 `final` 修饰类），防止子类破坏不可变性。
    

**示例**（不可变的 `Person` 类，包含不可变字段和可变字段的防御性拷贝）：

```java

public final class Person {
    private final String name;               // String 不可变
    private final int age;                    // 基本类型
    private final List<String> hobbies;       // 可变类型，需防御性拷贝
    public Person(String name, int age, List<String> hobbies) {
        this.name = name;
        this.age = age;
        // 防御性拷贝：创建新的 ArrayList 并复制内容
        this.hobbies = new ArrayList<>(hobbies);
    }
    public String getName() { return name; }
    public int getAge() { return age; }
    public List<String> getHobbies() {
        // 返回不可修改的视图，或防御性拷贝
        return Collections.unmodifiableList(hobbies);
    }
}
```
### 4. 为什么 `String` 适合作为哈希表键？

**答案**：

- `String` 是**不可变**的，一旦创建就不能修改，因此作为键时哈希码不会改变，保证了键的稳定性。
    
- `String` 正确重写了 `equals` 和 `hashCode`，并且遵循一致性：内容相同的 `String` 对象 `equals` 为 true，且 `hashCode` 相同。
    
- `String` 的哈希码分布较好（使用 31 为乘数），能减少碰撞。
    

### 5. 假设你有一个可变对象，但你能保证在作为键期间不修改它，这样做安全吗？为什么？

**答案**：

- 不安全。因为即使你自己保证不修改，但代码的其他部分可能无意中修改它（例如通过其他引用），或者你可能忘记约束。更重要的是，编译器无法强制保证这一点，一旦未来有人修改代码，就会引入难以调试的 bug。
    
- 因此，最佳实践是**永远不要将可变对象用作键**，即使你认为可以控制。如果必须用，应创建该对象的不可变副本作为键（例如用 `List.copyOf` 创建不可变列表）。
    

### 6. 解释 Java 8 中 HashMap 的树化优化及其好处。

**答案**：

- 树化优化指当某个桶的链表长度超过阈值（默认 8）时，将链表转换为红黑树存储。
    
- **好处**：
    
    - 最坏情况下查找时间复杂度从 O(N) 降为 O(log N)，防止了哈希碰撞攻击或极端分布导致的性能退化。
        
    - 大多数情况下链表很短，仍保持 O(1) 平均性能；只有极少数桶过长时才触发树化，不影响平均性能。
        
- 阈值 8 是基于统计选择的，在理想随机哈希下，链表长度达到 8 的概率极低，所以树化很少发生，而一旦发生说明分布极差，需要树化来保证性能。
    

### 7. 负载因子为什么通常选 0.75？过高或过低会有什么影响？

**答案**：

- 负载因子 0.75 是时间和空间的折中。
    
- **过低**（如 0.5）：桶很多，空间浪费大，但链表短，查找快；扩容频繁，插入均摊成本略高。
    
- **过高**（如 1.0）：空间利用率高，但链表长，查找变慢；扩容少，但插入和查找时间增加。
    
- 0.75 在大多数场景下能提供较好的性能平衡，是 Java 设计者经过实验选择的默认值。