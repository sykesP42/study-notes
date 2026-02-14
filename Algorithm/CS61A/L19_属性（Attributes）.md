# 📘 Lecture 19：属性（Attributes）
> 在上一讲中，我们学习了如何定义类和创建实例。本节课将深入探讨属性的工作机制：什么是类属性？什么是实例属性？它们如何存储和查找？赋值语句对属性有何影响？理解这些概念是掌握面向对象编程的关键，也是避免“魔法”般错误的基础。

---

## 一、类属性

### 1. 类语句的执行

当 Python 执行一个 `class` 语句时，它会创建一个新的**类对象**，并在当前环境的第一个帧中将其绑定到指定的类名。


```python

class Account:
    """一个简单的银行账户类"""
    interest = 0.02        # 这是一个类属性
```
在类语句的套件（缩进块）中，我们可以编写任意代码，但最常见的两种是：

- **赋值语句**：创建类属性（如 `interest = 0.02`）。
    
- **`def` 语句**：定义方法，这些方法也是类属性（属性值是函数对象）。
    

**重要**：类语句中创建的变量（通过赋值或 `def`）都是**类的属性**，而不是局部变量。它们被存储在类对象上，而不是某个帧中。

### 2. 类属性的特点

- **共享性**：类属性被该类的**所有实例共享**。因为它是类的属性，而不是实例的属性。
    
- **存储效率**：类属性在内存中只存储一份，无论创建多少个实例，都不会重复存储。
    

#### 银行账户示例

```python

class Account:
    interest = 0.02          # 类属性：所有账户共享的利率
    def __init__(self, holder):
        self.balance = 0     # 实例属性：每个账户独立
        self.holder = holder # 实例属性
    def deposit(self, amount):
        self.balance += amount
        return self.balance
    def withdraw(self, amount):
        if amount > self.balance:
            return 'Insufficient funds'
        self.balance -= amount
        return self.balance
```
现在，所有 `Account` 实例都可以访问 `interest` 属性：

```python

>>> a = Account('John')
>>> b = Account('Mary')
>>> a.interest
0.02
>>> b.interest
0.02
```
注意，`a.interest` 和 `b.interest` 访问的是同一个类属性，而不是各自拥有的副本。

---

## 二、属性查找顺序

### 1. 点表达式的求值规则

点表达式 `obj.attr` 的求值过程如下：

1. 计算点左侧的表达式 `obj`，得到对象。
    
2. 在该对象的**实例属性**中查找名称为 `attr` 的属性。
    
3. 如果找到了，返回该属性的值。
    
4. 如果没找到，则在该对象的**类**中查找名称为 `attr` 的属性。
    
5. 如果类中找到了该属性，并且该属性是一个函数，则返回一个**绑定方法**（bound method），该函数已经绑定了 `self` 为 `obj`。
    
6. 如果类中找到了非函数的属性，直接返回该值。
    
7. 如果都没找到，抛出 `AttributeError`。
    

这个查找顺序意味着：**实例属性会遮蔽（shadow）同名的类属性**。

#### 示例

```python

>>> a.interest            # 实例 a 没有自己的 interest 属性，所以去类 Account 中查找
0.02
>>> a.interest = 0.05      # 给实例 a 创建自己的 interest 属性
>>> a.interest
0.05
>>> b.interest             # b 没有自己的 interest 属性，仍使用类属性
0.02
>>> Account.interest       # 类属性本身没有改变
0.02
```
### 2. 特殊情况：函数属性与绑定方法

如果类属性是一个函数（即方法），那么通过实例访问时，Python 会自动返回一个**绑定方法**对象，该对象将实例作为第一个参数（`self`）绑定到函数。

```python

>>> a.deposit              # 返回一个绑定方法
<bound method Account.deposit of <__main__.Account object at 0x...>>
>>> a.deposit(100)         # 等价于 Account.deposit(a, 100)
100
```
如果通过类访问方法，得到的是原始函数对象，需要手动传入实例：

```python

>>> Account.deposit        # 返回原始函数
<function Account.deposit at 0x...>
>>> Account.deposit(a, 100)  # 手动传入实例
200
```
---

## 三、属性赋值规则

点表达式**出现在赋值语句左侧**时，行为与查找完全不同。

### 1. 赋值规则

- 如果点表达式的左侧对象是一个**实例**，那么赋值语句**总是创建或修改该实例的属性**，不会影响类属性。
    
- 如果点表达式的左侧对象是一个**类**，那么赋值语句**总是创建或修改该类的属性**，不会影响实例。
    

**关键**：赋值语句**不会沿着继承链查找**，它直接作用于指定的对象。

#### 示例

```python

>>> a = Account('John')
>>> a.interest = 0.05       # 在实例 a 上创建实例属性 interest
>>> Account.interest = 0.03 # 修改类属性 interest
>>> a.interest              # 实例属性遮蔽类属性
0.05
>>> b.interest              # b 没有实例属性，使用类属性
0.03
```
### 2. 实例属性创建

实例属性可以在任何时候创建，不仅仅在 `__init__` 中。

```python

>>> a.new_attr = 'hello'    # 给 a 动态添加新属性
>>> a.new_attr
'hello'
```
### 3. 类属性修改的影响

修改类属性会立即影响所有**没有同名实例属性**的实例。

```python

>>> Account.interest = 0.04
>>> b.interest              # b 受影响
0.04
>>> a.interest              # a 不受影响（因为有自己的实例属性）
0.05
```
---

## 四、内置函数 `getattr` 和 `hasattr`

Python 提供了两个内置函数，用于通过字符串名称动态访问属性。

- `getattr(obj, name)`：返回对象 `obj` 的属性名为 `name` 的值。如果属性不存在，可以指定默认值（否则抛出 `AttributeError`）。
    
- `hasattr(obj, name)`：检查对象 `obj` 是否有属性 `name`（返回 `True` 或 `False`）。
    

#### 示例

```python

>>> getattr(a, 'balance')      # 相当于 a.balance
0
>>> getattr(a, 'interest')     # 先查找实例，再查找类
0.05
>>> hasattr(a, 'deposit')      # 类属性方法也算
True
>>> hasattr(a, 'nonexist')     # 不存在的属性
False
>>> getattr(a, 'nonexist', 'default')  # 提供默认值
'default'
```
这些函数在需要动态处理属性名时非常有用（例如从用户输入或配置文件中读取属性名）。

---

## 五、方法调用机制

### 1. 点表达式调用方法

当点表达式 `obj.method` 返回一个绑定方法后，紧接着的括号 `()` 会调用该方法。调用时，绑定方法会自动将 `obj` 作为第一个参数（`self`）传入，括号内只需提供其余参数。

```python

>>> a.deposit(100)   # self = a, amount = 100
```
### 2. 绑定方法作为一等公民

绑定方法本身是一个对象，可以赋值给变量、作为参数传递等。

```python

>>> f = a.deposit     # f 是一个绑定方法
>>> f(50)             # 调用它，仍然作用于 a
250
```
甚至可以将绑定方法用于高阶函数，如 `map`：

```python

>>> amounts = [10, 20, 30]
>>> list(map(a.deposit, amounts))  # 对 a 连续存款
[260, 280, 310]                     # 注意顺序：10, 20, 30 依次存入
```
这里 `map(a.deposit, amounts)` 会依次调用 `a.deposit(10)`, `a.deposit(20)`, `a.deposit(30)`，返回每次存款后的余额列表。

**注意**：绑定方法是有状态的，因为它操作的是同一个实例 `a`，所以后续调用会基于前一次的结果。

---

## 六、术语澄清

- **对象**（Object）：任何 Python 值都是对象，包括整数、字符串、列表、类的实例、类本身等。
    
- **属性**（Attribute）：对象上的名称-值对。可以通过点表达式访问。
    
- **类**（Class）：对象的类型/类别，定义了该类实例的属性和行为。类本身也是对象（`type` 的实例）。
    
- **实例**（Instance）：通过调用类创建的具体对象。
    
- **实例属性**（Instance Attribute）：存储在实例上的属性，每个实例独立拥有。
    
- **类属性**（Class Attribute）：存储在类上的属性，被所有实例共享。
    
- **方法**（Method）：作为类属性的函数。通过实例访问时，会返回绑定方法。
    
- **绑定方法**（Bound Method）：一个可调用对象，它包装了原始函数和实例，调用时自动传入实例作为第一个参数。
    

---

## 七、知识点总结表

|知识点|核心内容|考试重点 / 易混淆点|难度|
|---|---|---|---|
|**类属性**|在类语句中通过赋值或 `def` 创建，存储于类对象上，被所有实例共享|类属性 vs 实例属性的区别；类属性修改对所有实例的影响|⭐⭐|
|**实例属性**|通过 `self.attr = value` 或在实例上直接赋值创建，每个实例独立|实例属性会遮蔽同名类属性|⭐⭐|
|**属性查找顺序**|先实例，后类；若类属性是函数则返回绑定方法|区分查找与赋值；绑定方法的自动 `self` 传递|⭐⭐⭐|
|**属性赋值规则**|点表达式在赋值左侧时，直接作用于指定对象（实例或类），不查找|赋值不会修改类属性，除非左侧是类|⭐⭐⭐|
|**`getattr`/`hasattr`**|通过字符串名称动态访问属性|与点表达式的等价关系；默认值的使用|⭐⭐|
|**绑定方法**|通过实例访问类中的方法时得到的可调用对象，已绑定 `self`|可作为高阶函数参数；注意状态累积|⭐⭐⭐|

---

## 八、自测题

### 1. 属性查找

```python

class C:
    x = 1
    def __init__(self):
        self.x = 2
c = C()
print(c.x)
print(C.x)
```
输出是什么？

```text

2
1
```
`c.x` 首先在实例中查找，找到 `x=2`，因此返回 2。`C.x` 直接在类中查找，返回 1。



### 2. 类属性修改

```python

class D:
    y = 10
d1 = D()
d2 = D()
D.y = 20
print(d1.y)
d1.y = 30
print(d1.y)
print(d2.y)
```

```text

20
30
20
```
修改 `D.y` 后，`d1` 和 `d2` 都访问到新的类属性 20。`d1.y = 30` 为 `d1` 创建了实例属性，遮蔽了类属性，所以 `d1.y` 变成 30，而 `d2` 仍然使用类属性 20。



### 3. 绑定方法

```python

class E:
    def __init__(self, val):
        self.val = val
    def add(self, x):
        self.val += x
        return self.val
e = E(5)
m = e.add
print(m(3))
print(m(3))
```
<details> <summary>答案</summary>

text

8
11

`m` 是绑定方法，每次调用都作用于同一个实例 `e`，因此第一次加 3 得 8，第二次再加 3 得 11。

</details>

### 4. `getattr` 应用

```python

class F:
    a = 100
f = F()
f.b = 200
print(getattr(f, 'a'))
print(getattr(f, 'b'))
print(getattr(f, 'c', 300))
```

```text

100
200
300
```
`getattr` 会遵循属性查找规则，所以 `'a'` 在类中找到，`'b'` 在实例中找到，`'c'` 不存在但提供了默认值 300。


### 5. 思考题

为什么说“类也是对象”？类对象有什么属性？


类本身是 `type` 类的实例。类对象有自己的属性，例如 `__name__`（类名）、`__dict__`（存储类属性的字典）、`__bases__`（基类元组）等。通过类名可以访问这些属性，例如 `Account.__name__`。


---

## 九、总结与展望

本节课深入剖析了属性的工作机制。你学会了：

- 类属性与实例属性的区别及共享性。
    
- 点表达式的查找顺序（先实例后类）及赋值规则（直接作用于对象）。
    
- 绑定方法的产生原理及其应用。
    
- 动态访问属性的工具函数 `getattr`/`hasattr`。
    

这些知识是理解面向对象高级特性（如继承、多态、特殊方法）的基础。在下一讲中，我们将探讨继承——如何通过类之间的关系复用代码，并实现多态行为。