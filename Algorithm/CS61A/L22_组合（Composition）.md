# 📘 Lecture 22：组合（Composition）—— 链表与树

> 本节课我们将学习两种重要的递归数据结构：**链表**和**树**。它们通过对象组合的方式构建：一个对象包含对其他对象的引用，从而形成层次或线性结构。理解这些结构不仅能加深你对递归的认识，也是后续学习更复杂数据结构（如二叉树、图）的基础。

---

## 一、链表（Linked List）

### 1.1 链表的定义

**链表**是一种线性数据结构，由一系列节点组成，每个节点包含一个值（称为 **first**）和一个指向下一个节点的引用（称为 **rest**）。链表要么为空，要么是一个节点，其 rest 指向另一个链表。

这种定义是**递归**的：一个链表由它的第一个元素和剩余链表组成。

### 1.2 用类实现链表

我们将创建一个 `Link` 类来表示链表节点。空链表用类属性 `Link.empty` 表示（我们用一个空元组作为标记）。


```python
class Link:
    """链表类，每个实例代表一个节点。"""
    empty = ()   # 空链表的标记（一个空元组）
    def __init__(self, first, rest=empty):
        assert rest is Link.empty or isinstance(rest, Link)
        self.first = first
        self.rest = rest
```
- `first` 存储节点的值。
    
- `rest` 要么是 `Link.empty`（表示链表结束），要么是另一个 `Link` 实例。
    
- 构造函数中的 `assert` 确保 `rest` 的类型正确，同时也允许子类实例通过检查（因为 `isinstance` 支持继承）。
    

### 1.3 创建和访问链表

```python


>>> s = Link(3, Link(4, Link(5)))
>>> s.first
3
>>> s.rest
Link(4, Link(5))
>>> s.rest.first
4
>>> s.rest.rest.first
5
>>> s.rest.rest.rest is Link.empty
True
```
链表的嵌套结构清晰地通过 `Link` 实例的嵌套体现：  
`Link(3, Link(4, Link(5, Link.empty)))`。

### 1.4 链表的基本操作

#### 遍历链表（打印）

```python


def print_link(s):
    """打印链表元素，用空格分隔。"""
    if s is Link.empty:
        return
    print(s.first, end=' ')
    print_link(s.rest)
```
#### 获取长度

```python


def len_link(s):
    """返回链表的长度。"""
    if s is Link.empty:
        return 0
    return 1 + len_link(s.rest)
```
#### 获取元素（索引）

```python


def get_link(s, i):
    """返回链表中索引为 i 的元素。"""
    if i == 0:
        return s.first
    return get_link(s.rest, i - 1)
```
### 1.5 递归处理链表：`range_link`, `map_link`, `filter_link`

由于链表是递归结构，处理它的函数通常也是递归的。

#### `range_link(start, end)` —— 生成整数链表

```python


def range_link(start, end):
    """返回一个包含从 start 到 end-1 的整数的链表。"""
    if start >= end:
        return Link.empty
    return Link(start, range_link(start + 1, end))
```
示例：

```python


>>> range_link(3, 6)
Link(3, Link(4, Link(5)))
```
#### `map_link(f, s)` —— 对链表每个元素应用函数 f

```python


def map_link(f, s):
    """返回一个新链表，其中每个元素是 f(s 中对应元素) 的结果。"""
    if s is Link.empty:
        return Link.empty
    return Link(f(s.first), map_link(f, s.rest))
```
示例：

```python


>>> square = lambda x: x * x
>>> map_link(square, range_link(3, 6))
Link(9, Link(16, Link(25)))
```
#### `filter_link(pred, s)` —— 筛选满足谓词的元素

```python


def filter_link(pred, s):
    """返回一个新链表，包含 s 中所有使 pred(x) 为真的元素。"""
    if s is Link.empty:
        return Link.empty
    filtered_rest = filter_link(pred, s.rest)
    if pred(s.first):
        return Link(s.first, filtered_rest)
    else:
        return filtered_rest
```
示例：

```python


>>> odd = lambda x: x % 2 == 1
>>> filter_link(odd, range_link(3, 6))
Link(3, Link(5))
```
组合使用：

```python


>>> list(map(square, filter(odd, range(1, 6))))   # 内置列表版本
[1, 9, 25]
>>> map_link(square, filter_link(odd, range_link(1, 6)))
Link(1, Link(9, Link(25)))
```
### 1.6 链表的可变性

`Link` 的实例是可变的，我们可以直接修改 `first` 和 `rest` 属性。

#### 修改元素

```python


>>> s = Link(1, Link(2, Link(3)))
>>> s.first = 5
>>> s
Link(5, Link(2, Link(3)))
>>> s.rest.first = 7
>>> s
Link(5, Link(7, Link(3)))
```
#### 创建循环引用

由于 `rest` 可以指向任意链表，我们甚至可以让它指向自身或之前的节点，形成循环链表。

```python


>>> s = Link(1, Link(2, Link(3)))
>>> t = s.rest
>>> t.rest = s          # 让 t 的 rest 指向 s
```
现在链表变成了一个环，遍历它会无限循环。

```python


>>> s.rest.rest.rest.rest.first   # 不断沿着 rest 最终会回到 2
2
```
循环链表虽然有趣，但需要小心处理，否则容易导致无限循环或内存泄漏。

### 1.7 有序链表的插入（带去重）

假设我们有一个递增有序的链表（无重复元素），要实现一个函数 `add(s, v)`，将 `v` 插入到正确位置（若已存在则不插入），并返回新的链表（可能修改原链表或创建新节点）。

```python


def add(s, v):
    """将 v 插入有序链表 s（无重复），返回新链表（可能修改原链表）。"""
    assert s is not Link.empty, "链表不能为空"   # 实际上我们可以处理空链表，但这里为简化假设非空
    if v < s.first:
        return Link(v, s)
    elif v == s.first:
        return s
    elif s.rest is Link.empty:
        s.rest = Link(v)
        return s
    else:
        s.rest = add(s.rest, v)
        return s
```
**注意**：这个函数会修改原链表（当插入位置在中间或尾部时），但返回的仍然是原链表（头部可能改变）。若插入在头部，则返回一个新节点作为新头部。

**测试**：

```python


>>> s = Link(1, Link(3, Link(5)))
>>> add(s, 0)
Link(0, Link(1, Link(3, Link(5))))   # 新头部
>>> s   # 原 s 现在被修改了？不，add(s,0) 返回新链表，原 s 仍然指向原来的头部 (1)？注意 add 返回了新节点，所以需要赋值给变量
>>> s = Link(1, Link(3, Link(5)))
>>> s = add(s, 0)   # 重新赋值
>>> s
Link(0, Link(1, Link(3, Link(5))))
>>> add(s, 3)   # 已存在，返回原链表
Link(0, Link(1, Link(3, Link(5))))
>>> add(s, 4)   # 插入中间
Link(0, Link(1, Link(3, Link(4, Link(5)))))
>>> add(s, 6)   # 插入尾部
Link(0, Link(1, Link(3, Link(4, Link(5, Link(6))))))
```
---

## 二、树（Tree）

### 2.1 树的定义

树是一种递归数据结构，由以下组成：

- 一个**根标签**（root label）
    
- 一系列**分支**（branches），每个分支本身也是一棵树
    

如果一棵树没有分支，则称为**叶子**（leaf）。

### 2.2 用类实现树

```python


class Tree:
    """树类，每个实例代表一棵树。"""
    def __init__(self, label, branches=[]):
        for b in branches:
            assert isinstance(b, Tree)
        self.label = label
        self.branches = list(branches)   # 复制一份，避免共享
```
- `label` 存储根节点的值。
    
- `branches` 是一个列表，包含子树（`Tree` 实例）。
    
- 构造函数中验证每个分支都是 `Tree` 实例，并复制列表以确保独立性。
    

### 2.3 创建树

```python


>>> t = Tree(3, [Tree(1), Tree(2, [Tree(0)])])
>>> t.label
3
>>> t.branches
[Tree(1), Tree(2, [Tree(0)])]
>>> t.branches[1].branches[0].label
0
```
### 2.4 树的遍历与操作

#### 判断叶子

```python


def is_leaf(t):
    """返回 t 是否为叶子。"""
    return not t.branches
```
#### 收集所有叶子标签

```python


def leaves(t):
    """返回树 t 中所有叶子节点的标签列表。"""
    if is_leaf(t):
        return [t.label]
    else:
        result = []
        for b in t.branches:
            result.extend(leaves(b))
        return result
```
示例：

```python


>>> t = Tree(3, [Tree(1), Tree(2, [Tree(0)])])
>>> leaves(t)
[1, 0]
```
#### 计算树的高度

树的高度定义为从根到最远叶子的路径上的节点数（通常根高度为 1，或 0？有多种定义）。这里我们采用：叶子高度为 0，非叶子高度为 1 + max(分支高度)。

```python


def height(t):
    """返回树的高度（叶子高度为 0）。"""
    if is_leaf(t):
        return 0
    else:
        return 1 + max(height(b) for b in t.branches)
```
示例：

```python


>>> height(t)
2   # 路径 3 -> 2 -> 0 有两个分支，所以高度为 2
```
### 2.5 构建树：斐波那契树

类似之前用数据抽象构建的斐波那契树，现在用 `Tree` 类实现。

```python


def fib_tree(n):
    """返回一棵表示斐波那契计算过程的树。"""
    if n == 0 or n == 1:
        return Tree(n)
    else:
        left = fib_tree(n - 2)
        right = fib_tree(n - 1)
        return Tree(left.label + right.label, [left, right])
```
打印树（带缩进）：

```python


def print_tree(t, indent=0):
    print('  ' * indent + str(t.label))
    for b in t.branches:
        print_tree(b, indent + 1)
>>> print_tree(fib_tree(4))
3
  1
    0
    1
  2
    1
    1
      0
      1
```
### 2.6 树的修改：修剪（prune）

假设我们要从树中删除所有标签为某个值的子树。注意：删除一个节点意味着连同它的所有后代一起删除。

```python


def prune(t, label):
    """从树 t 中删除所有标签为 label 的子树（原地修改）。"""
    t.branches = [b for b in t.branches if b.label != label]
    for b in t.branches:
        prune(b, label)
```
**注意**：

- 我们先过滤直接分支，再递归修剪剩余分支。这样避免处理即将被删除的分支。
    
- 此函数不返回值，直接修改原树。
    

**示例**：

```python


>>> t = Tree(1, [Tree(2), Tree(3, [Tree(2), Tree(4)]), Tree(2, [Tree(5)])])
>>> prune(t, 2)
>>> print_tree(t)
1
  3
    4
```
原树中所有标签为 2 的节点（包括它们的子树）都被删除了。

---

## 三、知识点总结表

|知识点|核心内容|关键实现 / 特性|难度|
|---|---|---|---|
|**链表定义**|递归结构：`first` + `rest`（链表）|`Link` 类，`Link.empty` 表示空|⭐⭐|
|**链表访问**|`s.first`, `s.rest`, `s.rest.first` 等|通过属性链式访问|⭐|
|**链表递归操作**|`range_link`, `map_link`, `filter_link`|递归构建新链表|⭐⭐⭐|
|**链表可变性**|直接修改 `first` 和 `rest`|可创建循环引用|⭐⭐|
|**有序链表插入**|维护有序无重复链表|递归插入，注意头部和尾部|⭐⭐⭐|
|**树定义**|根标签 + 分支列表（递归）|`Tree` 类，分支为 `Tree` 实例列表|⭐⭐|
|**树遍历**|叶子收集、高度计算|递归遍历分支|⭐⭐|
|**树构建**|斐波那契树|递归构造，根标签为子树标签和|⭐⭐|
|**树修剪**|删除指定标签的子树|原地修改，先过滤再递归|⭐⭐⭐|

---

## 四、自测题

### 1. 链表基本操作

写出以下代码的输出：

```python


s = Link(1, Link(2, Link(3)))
print(s.first)
print(s.rest.first)
print(s.rest.rest.first)
print(s.rest.rest.rest is Link.empty)
```


```text

1
2
3
True
```

### 2. 实现 `extend_link`

实现函数 `extend_link(s, t)`，将链表 `t` 的所有元素追加到链表 `s` 的末尾，并返回新链表（不修改原链表）。

```python

def extend_link(s, t):
    if s is Link.empty:
        return t
    else:
        return Link(s.first, extend_link(s.rest, t))

```

### 3. 链表反转

实现 `reverse_link(s)`，返回原链表 `s` 的反转链表（不修改原链表）。

<details> <summary>提示</summary>

可以使用递归：反转剩余部分，然后将原第一个元素插入到反转链表的末尾。

</details>
```python

def reverse_link(s):
    def reverse_to(s, acc):
        if s is Link.empty:
            return acc
        else:
            return reverse_to(s.rest, Link(s.first, acc))
    return reverse_to(s, Link.empty)
```

### 4. 树的高度（另一种定义）

有时树的高度定义为从根到最远叶子的**边数**（即叶子高度为 0）。用这个定义，上面 `height` 函数已符合。如果定义为节点数（叶子高度为 1），如何修改？

- 将叶子情况返回 1，非叶子返回 1 + max(分支高度)：

```python

def height_nodes(t):
    if is_leaf(t):
        return 1
    else:
        return 1 + max(height_nodes(b) for b in t.branches)

```

### 5. 树的映射

实现 `map_tree(f, t)`，返回一棵新树，其中每个节点的标签都应用了函数 `f`（不修改原树）。

```python

def map_tree(f, t):
    return Tree(f(t.label), [map_tree(f, b) for b in t.branches])
```
### 6. 思考题

为什么链表和树都适合用递归处理？请从它们的定义角度解释。

<details> <summary>提示</summary>

因为它们本身就是递归定义的：链表要么为空，要么由一个值和另一个链表组成；树要么是叶子，要么由一个值和一组树组成。递归函数可以直接对应这种结构，处理一个节点时，递归处理其子部分。

</details>

---

## 五、总结与展望

本节课我们学习了两种通过对象组合构建的递归数据结构：链表和树。你学会了：

- 用 `Link` 类实现链表，掌握基本的递归操作（映射、过滤、生成）。
    
- 理解链表的可变性及其潜在副作用（循环引用）。
    
- 用 `Tree` 类实现树，实现遍历、高度计算、修剪等操作。
    
- 认识到递归在处理递归数据结构时的自然与强大。
    

链表是线性结构的基础，树则是层次结构的基础。后续课程中，你将学习二叉树、二叉搜索树、堆等更专业的树结构，以及如何利用它们高效地解决实际问题。