# 📘 Lecture 18：面向对象编程（Objects）

> 面向对象编程（Object-Oriented Programming, OOP）是当今组织大型软件的主导范式。它建立在数据抽象和可变状态的基础上，将信息与相关行为捆绑成对象，使程序更模块化、更易维护。本节课将带你从零掌握 Python 中的类和对象，理解其核心机制，并学会如何定义和使用自定义类。

---

## 一、面向对象编程概述

### 1. 从数据抽象到对象

在之前的学习中，我们了解了**数据抽象**：通过构造函数和选择器将数据的使用与表示分离。但数据抽象只处理了数据的“静态”部分。现实世界中的事物不仅拥有属性，还会表现出行为，并且这些行为往往依赖于自身的状态（并且状态会随时间变化）。

**对象**（Object）就是对此的自然扩展：它将**数据**（属性）和**行为**（方法）捆绑在一起，形成一个独立的单元。每个对象维护自己的局部状态，并对外提供接口（方法）来与外界交互。

### 2. 类与实例

- **类**（Class）是对象的蓝图或模板，定义了该类对象共有的属性和方法。
    
- **实例**（Instance）是根据类创建的具体对象，每个实例拥有自己独立的属性值。
    

例如，`list` 是一个内置类，而 `[1,2,3]` 是该类的一个实例。所有列表实例都支持 `append`、`pop` 等方法，但每个列表存储不同的数据。

### 3. 面向对象的核心优势

- **模块化**：将程序拆分为多个独立的、可重用的对象。
    
- **信息隐藏**：对象内部状态对外部不可见（通过方法访问），减少耦合。
    
- **可扩展性**：通过继承和多态，可以方便地扩展已有代码。
    
- **贴近现实**：用对象模拟现实世界的事物，思维更自然。
    

---

## 二、内置类示例：列表

Python 中一切都是对象，包括内置类型。让我们以列表为例，观察类与实例的关系。

```python

>>> list                      # list 是一个类
<class 'list'>
>>> s = list(range(3))        # 调用 list 类创建实例
>>> s
[0, 1, 2]
>>> type(s)
<class 'list'>

```
- 调用一个类（如 `list()`）会创建该类的实例，这个过程称为**实例化**。
    
- 实例拥有该类定义的所有行为：方法（如 `s.append(4)`）、运算符（如 `s + [5]`）、索引访问（如 `s[0]`）等。这些行为由类决定。
    

```python

>>> s.append(4)               # 调用方法
>>> s.pop()                   # 另一个方法
4
>>> s + [5,6]                 # 运算符也是方法（实际上是特殊方法）
[0, 1, 2, 5, 6]
>>> s[0] = 10                 # 索引赋值
```
**关键**：类定义了所有实例的行为，每个实例只保存自己的数据。

---

## 三、自定义类：银行账户

我们通过一个银行账户的例子来学习如何定义自己的类。

### 1. 定义类

使用 `class` 关键字定义一个类，类名通常采用驼峰命名法（如 `Account`）。

```python

class Account:
    """一个简单的银行账户类"""
    
    def __init__(self, holder):
        """构造方法，初始化账户持有人和余额"""
        self.holder = holder
        self.balance = 0
    def deposit(self, amount):
        """存款，增加余额，返回新余额"""
        self.balance += amount
        return self.balance
    def withdraw(self, amount):
        """取款，如果余额足够则扣除并返回余额，否则返回错误信息"""
        if amount > self.balance:
            return 'Insufficient funds'
        self.balance -= amount
        return self.balance
```
**解释**：

- `__init__` 是一个特殊方法，称为**构造方法**（或初始化方法）。当创建新实例时（如 `a = Account('John')`），Python 会自动调用 `__init__` 来初始化实例。
    
- 方法的第一个参数必须是 `self`，它代表当前实例本身。在方法内部，通过 `self.属性名` 访问或修改该实例的属性。
    
- 属性（如 `self.balance`）是存储在实例上的变量，每个实例拥有独立副本。
    

### 2. 创建实例

```python

>>> a = Account('John')
>>> b = Account('Mary')
```
此时内存中有两个不同的 `Account` 实例：

- `a` 的 `holder` 为 `'John'`，`balance` 为 `0`。
    
- `b` 的 `holder` 为 `'Mary'`，`balance` 为 `0`。
    

### 3. 调用方法

```python

>>> a.deposit(100)        # 相当于 Account.deposit(a, 100)
100
>>> a.balance
100
>>> a.withdraw(30)
70
>>> a.withdraw(80)
'Insufficient funds'
>>> b.balance             # b 不受 a 的影响
0
```
**注意**：点表达式 `a.deposit(100)` 会自动将调用对象 `a` 作为第一个参数传递给 `deposit` 方法，因此 `deposit` 中的 `self` 就指向 `a`。这称为**方法绑定**。

### 4. 动态属性

Python 允许在运行时给实例添加新属性：

```python

>>> a.backup = b          # 给 a 添加一个 backup 属性，指向另一个账户
>>> a.backup.deposit(50)  # 通过 a 的 backup 调用 b 的方法
50
>>> b.balance             # 确实修改了 b
50
```
也可以添加任意属性，甚至与类中定义的方法同名（但通常不建议，会覆盖方法）。

```python

>>> a.interest = 0.02     # 添加一个新属性 interest
```
这种动态性提供了极大的灵活性，但也可能带来意外，因此使用时需谨慎。

---

## 四、对象标识

### 1. 唯一身份

每个对象（实例）都有一个唯一的身份标识，可以用 `id()` 函数查看，或用 `is` 运算符判断两个引用是否指向同一个对象。

```python

>>> a = Account('John')
>>> b = Account('John')   # 持有人相同，但是不同对象
>>> a is b
False
>>> c = a                 # c 成为 a 的别名
>>> a is c
True
```
### 2. 相等性 vs 同一性

- `==` 运算符比较两个对象的值是否相等。对于自定义类，默认行为是比较对象身份（与 `is` 相同），但我们可以通过定义 `__eq__` 方法来改变这一行为。
    
- `is` 比较对象身份，即是否是同一个对象。
    

```python

>>> a == b                # 默认调用 a is b
False
>>> a == c                # a is c 为 True，所以相等
True
```
如果我们希望两个 `holder` 相同的账户视为相等，可以在类中定义 `__eq__` 方法：

```python

    def __eq__(self, other):
        if not isinstance(other, Account):
            return False
        return self.holder == other.holder   # 仅比较持有人
```
但注意，这可能会掩盖对象的身份区别。

---

## 五、方法详解

### 1. 方法与普通函数的区别

- 方法是定义在类内部的函数。
    
- 方法的第一个参数必须是 `self`，代表调用该方法的实例。
    
- 调用方法时，使用点表达式 `实例.方法名(参数)`，Python 自动将实例传递给 `self`。
    

也可以直接通过类调用方法，并手动传递实例：

```python

>>> Account.deposit(a, 100)   # 等价于 a.deposit(100)
```
这种形式展示了方法本质上就是类的一个属性，该属性是一个函数。

### 2. 属性查找顺序

当通过实例访问一个名称时（如 `a.balance`），Python 首先在实例自身的属性字典中查找（`a.__dict__`），如果找不到，再到类的属性中查找。方法通常存储在类中，因此所有实例共享同一方法代码。

### 3. 不同类的同名方法

不同类可以定义同名方法，调用时根据实例的类自动选择正确的方法。这是多态的基础。

```python

class InvestmentFund:
    def deposit(self, amount):
        # 投资基金的存款可能有一些额外规则
        self.balance += amount * 1.05   # 假设有 5% 奖励
        return self.balance
fund = InvestmentFund()
fund.deposit(100)          # 调用的是 InvestmentFund 的 deposit
```
---

## 六、知识点总结表

|知识点|核心内容|考试重点 / 易混淆点|难度|
|---|---|---|---|
|**面向对象编程思想**|将数据和行为捆绑成对象，每个对象维护独立状态|对象与类的区别；数据抽象 vs OOP|⭐⭐|
|**类与实例**|类是模板，实例是具体对象，通过 `ClassName()` 创建|`__init__` 方法的作用；`self` 的含义|⭐⭐|
|**属性与方法**|属性是实例变量，方法是类中定义的函数，通过点表达式调用|实例属性在 `__init__` 中初始化；方法第一个参数必须是 `self`|⭐⭐|
|**对象标识**|`is` 比较对象身份，`==` 比较值（可自定义）|区分 `is` 和 `==`；别名效应|⭐⭐|
|**动态属性**|可随时给实例添加新属性|动态性的利弊；可能意外覆盖方法|⭐⭐|
|**方法绑定**|点表达式自动将实例传递给 `self`|理解隐式传参机制|⭐⭐⭐|

---

## 七、自测题

### 1. 基础概念

判断下列说法是否正确：

- 类定义了对象的属性，而实例定义了对象的行为。（ ）
    
- 同一个类的不同实例拥有各自独立的属性副本。（ ）
    
- 在方法内部，可以通过 `self` 访问当前实例的属性。（ ）
    
- 两个不同的实例即使属性完全相同，`a is b` 也为 `False`。（ ）
    

```
- 错。类定义了属性和行为（方法），实例存储具体的属性值。
    
- 对。
    
- 对。
    
- 对（除非有特殊实现）。
    
```

### 2. 补全代码

定义 `Dog` 类，包含 `__init__` 方法（接受名字参数，并初始化年龄为 0），以及 `bark` 方法（打印 `'Woof!'`）和 `have_birthday` 方法（年龄加 1）。

```python

class Dog:
    def __init__(self, name):
        self.name = name
        self.age = 0
    def bark(self):
        print('Woof!')
    def have_birthday(self):
        self.age += 1

```

### 3. 方法调用

```python

class Counter:
    def __init__(self):
        self.count = 0
    def increment(self):
        self.count += 1
        return self.count
c = Counter()
print(c.increment())
print(Counter.increment(c))
```
输出是什么？

```text

1
2

第一次调用 `c.increment()` 返回 1；第二次通过类调用，手动传入 `c`，再次增加，返回 2。
```

### 4. 对象标识

```python

a = [1, 2]
b = [1, 2]
c = a
print(a == b, a is b, a is c)
```


`True False True`。因为 `a` 和 `b` 值相等但不同对象，`c` 是 `a` 的别名。


### 5. 动态属性

以下代码会输出什么？

```python

class Person:
    def __init__(self, name):
        self.name = name
p = Person('Alice')
p.age = 25
p.name = 'Bob'
print(p.name, p.age)
```

```
Bob 25`。可以动态添加 `age` 属性，也可以修改已有属性 `name`。

```

### 6. 思考题

为什么方法需要 `self` 参数？如果忘记写 `self` 会怎样？

`self` 是实例的引用，通过它方法才能访问实例的属性。如果忘记写 `self`，调用时 Python 会将第一个实参作为普通参数传入，导致参数个数不匹配或逻辑错误。例如 `def deposit(amount):` 然后调用 `a.deposit(100)` 会引发 `TypeError`，因为 Python 自动将 `a` 作为第一个参数传入，但方法只定义了一个参数 `amount`，导致 `TypeError: deposit() takes 1 positional argument but 2 were given`。


---

## 八、总结与展望

本节课我们迈入了面向对象编程的大门。你学会了：

- 面向对象的基本思想：将数据和行为封装成对象。
    
- 如何定义类、创建实例，以及使用 `__init__` 初始化属性。
    
- 方法定义和 `self` 参数的作用。
    
- 对象标识：`is` vs `==`。
    
- Python 的动态属性特性。
    

面向对象编程是构建大型系统的基石，后续课程将深入探讨继承、多态、特殊方法等高级主题。建议你多动手实践，尝试用类模拟现实世界的事物（如汽车、学生、购物车等），加深理解。