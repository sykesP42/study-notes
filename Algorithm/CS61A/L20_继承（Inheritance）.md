# 📘 Lecture 20：继承（Inheritance）

> 继承是面向对象编程中实现代码复用和层次化设计的关键机制。通过继承，我们可以定义一个新的类（子类）来扩展现有类（父类）的行为，而无需重复编写相同的代码。本节课将带你深入理解继承的工作原理、属性查找规则、如何设计继承体系，以及多重继承的复杂性。

---

## 一、继承的定义

### 1. 什么是继承？

**继承**（Inheritance）是一种将多个类关联起来的方法。它允许我们基于一个已有的类（称为**基类**、**父类**或**超类**）定义一个新的类（称为**子类**或**派生类**），子类会自动获得父类的所有属性和方法，并且可以添加新的属性或覆盖（override）原有的属性和方法。

**适用场景**：当两个类相似但存在专业化程度差异时，使用继承。专业化的类（子类）拥有通用类（父类）的所有特性，同时添加或修改某些行为。

### 2. 语法


```python

class <子类名>(<父类名>):
    <类体>
```
- 子类定义时在类名后的括号中指定父类。
    
- 子类的类体中只需定义与父类不同的部分（新增或覆盖的属性和方法），相同的部分会自动从父类继承。
    

**概念理解**：子类“共享”父类的所有属性。这种共享不是复制，而是通过属性查找机制动态实现的。

---

## 二、继承示例：银行账户体系

我们从一个通用的 `Account` 类出发，然后派生出专门的 `CheckingAccount`（支票账户）。

### 1. 基类 `Account`

```python

class Account:
    """一个通用的银行账户"""
    interest = 0.02          # 类属性：利率
    def __init__(self, holder):
        self.holder = holder
        self.balance = 0
    def deposit(self, amount):
        """存款"""
        self.balance += amount
        return self.balance
    def withdraw(self, amount):
        """取款（余额不足时返回错误信息）"""
        if amount > self.balance:
            return 'Insufficient funds'
        self.balance -= amount
        return self.balance
```
### 2. 子类 `CheckingAccount`

支票账户与普通账户的区别：

- 利率更低（`interest = 0.01`）
    
- 每次取款收取 $1 手续费
    

```python

class CheckingAccount(Account):
    """支票账户：取款收取 1 美元手续费"""
    interest = 0.01               # 覆盖父类利率
    withdraw_fee = 1              # 新增类属性
    def withdraw(self, amount):
        # 调用父类的 withdraw 方法，但金额需加上手续费
        return Account.withdraw(self, amount + self.withdraw_fee)
        # 注意：不能直接调用 self.withdraw，否则会递归调用自己
```
**关键点**：

- 子类没有定义 `__init__`，因此会自动调用父类的 `__init__` 来初始化 `holder` 和 `balance`。
    
- 子类定义了同名的 `interest` 类属性，覆盖了父类的属性。
    
- 子类新增了 `withdraw_fee` 类属性。
    
- 子类重写了 `withdraw` 方法，在调用父类方法前将手续费加到取款金额中。这里显式使用 `Account.withdraw(self, ...)` 来调用父类方法，避免了递归。
    

#### 测试

```python

>>> ch = CheckingAccount('Tom')
>>> ch.interest
0.01
>>> ch.deposit(20)
20
>>> ch.withdraw(5)
14          # 20 - 5 - 1 = 14
```
### 3. 继承的好处

- **代码复用**：`deposit` 方法完全继承自 `Account`，无需重写。
    
- **一致性**：修改父类的 `deposit` 方法会自动影响所有子类。
    
- **可扩展性**：子类可以轻松添加新属性或覆盖方法以实现特殊行为。
    

---

## 三、属性查找规则

继承是通过属性查找的动态机制实现的，而不是将父类属性复制到子类中。

### 1. 查找流程

当通过实例访问一个属性时（如 `ch.interest`），Python 按照以下顺序查找：

1. **实例属性**：先查找实例对象自身是否有该属性（`ch.__dict__`）。
    
2. **类属性**：若未找到，则查找实例的类（`type(ch)`）中是否有该属性。
    
3. **父类属性**：若类中未找到，则沿着继承链向上查找（先查父类，再查父类的父类，直到 `object`）。
    
4. 如果最终未找到，抛出 `AttributeError`。
    

对于方法调用，查找过程相同，但如果找到的是函数，则返回一个**绑定方法**（已绑定到实例）。

### 2. 示例分析

```python

>>> ch = CheckingAccount('Tom')
>>> ch.deposit(20)           # 1. 实例无 deposit；2. 类 CheckingAccount 无 deposit；3. 父类 Account 有，调用 Account.deposit
20
>>> ch.withdraw(5)            # 1. 实例无；2. 类 CheckingAccount 有 withdraw，调用它
```
### 3. 覆盖与访问父类属性

子类可以覆盖父类的属性，但父类的原始属性仍然可以通过父类名直接访问（如 `Account.withdraw`）。这为我们提供了在子类方法中调用父类实现的能力。

---

## 四、面向对象设计指南

### 1. 避免重复（DRY 原则）

**DRY**（Don't Repeat Yourself）是软件工程的基本原则。继承正是实现 DRY 的强大工具。例如，`CheckingAccount` 无需重写 `deposit` 方法，直接复用父类代码。

### 2. 访问被覆盖的属性

即使子类覆盖了父类的属性，仍可通过父类名显式调用原始实现。这在子类方法中扩展父类行为时非常常见。

```python

class CheckingAccount(Account):
    def withdraw(self, amount):
        # 先执行一些额外逻辑，再调用父类方法
        print(f"Withdrawing {amount} plus fee {self.withdraw_fee}")
        return Account.withdraw(self, amount + self.withdraw_fee)
```
### 3. 实例属性优先

在设计类时，应优先使用实例属性来存储每个实例特有的值，类属性用于所有实例共享的值。例如，`withdraw_fee` 作为类属性是合理的，因为所有支票账户手续费相同；但如果未来某些账户手续费不同，则应改为实例属性。

```python

class CheckingAccount(Account):
    def __init__(self, holder, fee=1):
        super().__init__(holder)      # 调用父类 __init__
        self.withdraw_fee = fee       # 实例属性，可自定义
```
### 4. 继承与组合的选择

- **继承**（is-a 关系）：子类是一种特殊的父类。例如，`CheckingAccount` **是一种** `Account`。
    
- **组合**（has-a 关系）：一个对象包含另一个对象作为其一部分。例如，`Bank` **拥有多个** `Account` 对象。
    

**设计原则**：优先使用组合，因为它更灵活，耦合度更低。只有在明确的 is-a 关系且需要多态行为时才使用继承。

#### 组合示例：`Bank` 类

```python

class Bank:
    def __init__(self):
        self.accounts = []
    def open_account(self, holder, amount, kind=Account):
        """开户，kind 指定账户类型（默认普通账户）"""
        account = kind(holder)
        account.deposit(amount)
        self.accounts.append(account)
        return account
    def pay_interest(self):
        """给所有账户支付利息"""
        for acc in self.accounts:
            acc.deposit(acc.balance * acc.interest)
    def too_big_to_fail(self, account):
        """保护某个账户（特殊处理）"""
        # ...
```
**测试**：

```python

>>> bank = Bank()
>>> john = bank.open_account('John', 1000, Account)
>>> jack = bank.open_account('Jack', 500, CheckingAccount)
>>> bank.pay_interest()
>>> john.balance   # 普通账户利率 0.02
1020.0
>>> jack.balance   # 支票账户利率 0.01
505.0
```
这里 `Bank` 通过组合管理多个账户，`pay_interest` 方法利用多态性调用每个账户自己的 `deposit` 和 `interest` 属性，无需关心具体账户类型。

---

## 五、继承与属性查找实践

我们通过一个复杂示例来深入理解属性查找规则。

### 1. 类结构

```python

class A:
    z = -1
    def f(self, x):
        return B(x - 1)
class B(A):
    n = 4
    def __init__(self, y):
        if y % 2 == 0:
            self.z = y
        else:
            self.z = self.f(y)   # self.f 从 A 继承
class C(B):
    def f(self, x):
        return x
```
### 2. 实例化与调用

```python

a = A()          # A 没有 __init__，创建空白实例
b = B(1)         # B(1) 触发递归链
c = C(2)
```
逐步分析 `b = B(1)` 的过程：

1. 调用 `B.__init__(b, 1)`。
    
    - `y = 1` 为奇数，执行 `self.z = self.f(1)`。
        
2. 查找 `self.f`：实例 `b` 无 `f`，类 `B` 无 `f`，父类 `A` 有 `f`，返回 `A.f` 方法（绑定到 `b`）。
    
    - 调用 `b.f(1)`，即 `A.f(b, 1)`。
        
3. `A.f` 返回 `B(x - 1)`，即 `B(0)`。
    
    - 创建新实例 `B(0)`，记为 `b2`。
        
4. 初始化 `b2`：`y = 0` 为偶数，执行 `self.z = 0`（设置实例属性 `z = 0`）。
    
    - `b2` 没有 `n`，但类 `B` 有 `n = 4`。
        
5. `A.f` 返回 `b2`，所以 `b.z = b2`。
    

因此 `b.z` 是一个 `B` 实例（`b2`），其 `z` 属性为 0，`n` 类属性为 4。

接下来 `c = C(2)`：

- `C` 没有定义 `__init__`，继承自 `B`。
    
- 调用 `B.__init__(c, 2)`：`y = 2` 偶数，设置 `c.z = 2`（实例属性）。
    
- `c` 有自己的 `f` 方法（来自 `C`）。
    

### 3. 属性访问测试

```python

>>> C(2).n           # 4，从 B 继承的类属性
4
>>> a.z == C.z       # a.z 是 -1（类属性），C.z 也是 -1（继承自 A），True
True
>>> b.z              # 一个 B 实例 (b2)
<__main__.B object at ...>
>>> b.z.z            # b2 的实例属性 z，值为 0
0
>>> b.z.z.z          # 0 是整数，没有 z 属性，AttributeError
```
**关键点**：

- 实例属性优先于类属性。
    
- 属性查找沿着继承链向上。
    
- 类属性可以被实例属性覆盖。
    

---

## 六、多重继承

### 1. 概念

一个类可以有多个直接父类，这称为**多重继承**（Multiple Inheritance）。子类会继承所有父类的属性和方法。

#### 示例：电视购物账户 `AsSeenOnTVAccount`

假设我们有一个支票账户 `CheckingAccount`（取款手续费 $1）和一个储蓄账户 `SavingsAccount`（存款手续费 $2）。现在想创建一个电视购物账户，同时具备取款手续费和存款手续费，并且开户时额外赠送 $1。

```python

class SavingsAccount(Account):
    deposit_fee = 2
    def deposit(self, amount):
        return Account.deposit(self, amount - self.deposit_fee)
class AsSeenOnTVAccount(CheckingAccount, SavingsAccount):
    """电视购物账户：开户送 $1，取款手续费 $1，存款手续费 $2"""
    def __init__(self, holder):
        self.holder = holder
        self.balance = 1          # 开户赠送 1 美元
```
### 2. 方法解析顺序（MRO）

当子类继承多个父类，且这些父类有同名方法时，Python 需要决定优先使用哪个父类的方法。这个顺序称为**方法解析顺序**（Method Resolution Order，MRO）。

Python 使用 **C3 线性化算法**（C3 Linearization）来确定 MRO。可以通过 `类名.__mro__` 查看。

```python

>>> AsSeenOnTVAccount.__mro__
(<class '__main__.AsSeenOnTVAccount'>, <class '__main__.CheckingAccount'>, <class '__main__.SavingsAccount'>, <class '__main__.Account'>, <class 'object'>)
```
**规则**：在查找属性时，Python 按照 MRO 的顺序从左到右搜索，直到找到第一个匹配的属性。

因此：

- `withdraw` 方法：优先使用 `CheckingAccount` 中的（因为它在 MRO 中先于 `SavingsAccount`）。
    
- `deposit` 方法：优先使用 `SavingsAccount` 中的（因为它先于 `CheckingAccount` 出现在 MRO 中？实际上，`CheckingAccount` 没有定义 `deposit`，所以会继续往后找，找到 `SavingsAccount` 的 `deposit`）。
    
- `interest`：`CheckingAccount` 和 `SavingsAccount` 都未定义，最终找到 `Account` 的 `interest`。
    

#### 测试

```python

>>> tv = AsSeenOnTVAccount('Alice')
>>> tv.balance
1
>>> tv.deposit(20)      # 存款 20，扣除 2 手续费，实际 +18，余额 19
19
>>> tv.withdraw(5)      # 取款 5，扣除 1 手续费，实际 -6，余额 13
13
```
### 3. 多重继承的注意事项

- **复杂性**：多重继承容易导致混乱，特别是当多个父类有同名方法时，需要清楚 MRO 规则。
    
- **钻石问题**（Diamond Problem）：当两个父类继承自同一个祖先时，可能会产生重复继承。Python 的 MRO 算法会确保祖先类只被处理一次。
    
- **最佳实践**：尽量使用单一继承，或使用**混入类**（Mixin）来组织可复用的功能，避免复杂的多重继承。
    

---

## 七、知识点总结表

|知识点|核心内容|考试重点 / 易混淆点|难度|
|---|---|---|---|
|**继承的基本概念**|子类继承父类属性和方法，实现代码复用|语法 `class Sub(Base):`；子类可覆盖父类方法|⭐⭐|
|**属性查找规则**|实例 → 类 → 父类 → ... 沿 MRO 查找|区分实例属性与类属性；覆盖后的父类属性仍可通过父类名访问|⭐⭐⭐|
|**方法重写**|子类定义与父类同名方法，扩展或修改行为|重写时调用父类方法：`Parent.method(self, ...)`|⭐⭐|
|**继承设计指南**|DRY 原则；优先使用实例属性；明确 is-a 关系|选择继承 vs 组合；self 的使用|⭐⭐⭐|
|**组合**|对象包含其他对象，表示 has-a 关系|与继承对比：`Bank` 包含 `Account` 列表|⭐⭐|
|**多重继承**|一个子类有多个直接父类|MRO 顺序；同名方法的冲突解决|⭐⭐⭐⭐|
|**MRO**|方法解析顺序（`类.__mro__`）|C3 线性化算法；钻石继承问题|⭐⭐⭐⭐|

---

## 八、自测题

### 1. 基础继承

```python

class Parent:
    x = 10
    def __init__(self):
        self.y = 20
class Child(Parent):
    x = 5
    def __init__(self):
        self.z = 30
c = Child()
print(c.x)
print(c.y)
```
输出是什么？为什么？


```text

5
AttributeError: 'Child' object has no attribute 'y'
```
`c.x` 查找：实例无 `x`，类 `Child` 有 `x=5`，输出 5。`c.y` 查找：实例无 `y`（因为 `Child.__init__` 没有调用 `Parent.__init__`，所以 `y` 未创建），类 `Child` 无 `y`，父类 `Parent` 无实例属性 `y`（`y` 是实例属性，不是类属性），因此抛出 `AttributeError`。若想继承 `y`，需在 `Child.__init__` 中调用 `super().__init__()`。


### 2. 方法覆盖

```python

class Base:
    def greet(self):
        return "Hello from Base"
class Derived(Base):
    def greet(self):
        return "Hello from Derived"
d = Derived()
print(d.greet())
print(Base.greet(d))
```

```text

Hello from Derived
Hello from Base
```
`d.greet()` 调用子类覆盖后的方法；`Base.greet(d)` 显式调用父类方法，传入实例 `d`，因此输出父类的问候语。

### 3. 多重继承 MRO

```python

class A:
    def f(self):
        return 'A'
class B(A):
    def f(self):
        return 'B'
class C(A):
    def f(self):
        return 'C'
class D(B, C):
    pass
d = D()
print(d.f())
print(D.__mro__)
```

```text

B
(<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>, <class '__main__.A'>, <class 'object'>)
```
MRO 为 D → B → C → A，因此 `d.f()` 先找到 B 的 `f`，返回 'B'。



### 4. 组合设计

设计一个 `Library` 类，包含多个 `Book` 对象。`Book` 有标题和作者。提供 `add_book` 方法添加书籍，`find_by_author` 方法返回指定作者的所有书籍列表。

```python

class Book:
    def __init__(self, title, author):
        self.title = title
        self.author = author
class Library:
    def __init__(self):
        self.books = []
    def add_book(self, book):
        self.books.append(book)
    def find_by_author(self, author):
        return [book for book in self.books if book.author == author]
```

### 5. 思考题

为什么 Python 中多重继承的 MRO 要采用 C3 线性化，而不是简单的深度优先？请简述其优势。

```
C3 线性化保证了单调性（子类的 MRO 不会改变父类的顺序），并解决了钻石继承中基类被重复访问的问题。它确保每个父类在 MRO 中只出现一次，且子类优先于父类，避免了二义性。
```

---

## 九、总结与展望

本节课我们深入学习了继承这一面向对象的核心机制。你学会了：

- 如何定义子类并利用继承复用代码。
    
- 属性查找规则和方法重写。
    
- 设计指南：DRY 原则、继承 vs 组合。
    
- 多重继承及其方法解析顺序（MRO）。
    

继承是构建可扩展、可维护系统的基石。在后续课程中，我们将学习更多高级特性，如特殊方法、抽象基类等，进一步完善你的面向对象编程能力。