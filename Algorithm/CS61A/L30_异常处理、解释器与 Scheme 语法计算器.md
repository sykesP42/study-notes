# 📘 Lecture 30：计算器 —— 异常处理、解释器与 Scheme 语法计算器

> 编程语言是一种形式化的交流方式，它由语法和语义定义。解释器是一种程序，它能读取用该语言编写的代码并执行它。本节课我们将从 Python 的异常处理入手，学习如何构建一个简单的 Scheme 语法计算器，从而深入理解解释器的基本原理：解析（将文本转换为表达式树）和求值（按照语义规则计算结果）。我们将实现一个支持基本算术运算的 REPL 循环，并学会如何优雅地处理各种错误。

---

## 一、Python 异常处理

### 1.1 异常机制

异常是程序运行过程中发生的错误或特殊事件。Python 使用 **异常**（exception）对象来表示这些情况，并提供了一套处理机制，避免程序直接崩溃。

#### `raise` 语句

语法：`raise <expression>`

- `<expression>` 必须求值为一个 `BaseException` 的子类（异常类）或该类的实例。
    
- 内置异常类型：`TypeError`、`NameError`、`KeyError`、`RecursionError` 等。
    

示例：

```python

def divide(a, b):
    if b == 0:
        raise ZeroDivisionError("除数不能为0")
    return a / b
```
也可以引发自定义异常：

```python

class MyError(Exception):
    pass
raise MyError("自定义错误信息")
```
### 1.2 处理异常：`try` 语句

语法：

```python

try:
    <try suite>          # 可能引发异常的代码
except <exception class> as <name>:
    <except suite>       # 处理异常的代码
```
- 首先执行 `try` 块中的代码。
    
- 如果执行过程中未引发异常，则跳过 `except` 块，继续执行后续代码。
    
- 如果引发了异常，且异常类型匹配 `except` 后指定的类（或其子类），则执行 `except` 块中的代码。可以使用 `as` 将异常对象绑定到变量，以便获取详细信息。
    
- 如果异常未被捕获，程序将终止并打印回溯信息。
    

**示例：防止除零崩溃**

```python

def safe_divide(a, b):
    try:
        return a / b
    except ZeroDivisionError as e:
        print(f"错误: {e}")
        return float('inf')   # 返回无穷大
print(safe_divide(10, 2))   # 5.0
print(safe_divide(10, 0))   # 错误: division by zero  inf
```
**多个 `except` 子句**：可以捕获不同类型异常，按顺序匹配。

---

## 二、`reduce` 高阶函数

`reduce` 是一种将序列“归约”为单个值的强大工具。它通过一个二元函数反复组合元素，最终得到一个结果。

### 2.1 递归定义

```python

def reduce(f, s, initial):
    """将二元函数 f 依次应用到 initial 和 s 的元素上，归约成单个值。"""
    if not s:
        return initial
    else:
        first, rest = s[0], s[1:]
        return reduce(f, rest, f(initial, first))
```
- `f` 必须接收两个参数，且返回值类型应能作为下一次调用的第一个参数。
    
- `initial` 是初始值，必须与 `f` 的第一个参数类型兼容。
    
- 当 `s` 为空时，返回 `initial`。
    

**示例：乘法归约**

```python

from operator import mul
print(reduce(mul, [2, 4, 8], 1))   # 64
```
计算过程：

- 初始值 `1`，第一个元素 `2` → `1*2=2`
    
- 当前值 `2`，第二个元素 `4` → `2*4=8`
    
- 当前值 `8`，第三个元素 `8` → `8*8=64`
    

### 2.2 幂运算示例

```python

def pow_reduce(s):
    return reduce(pow, s, 2)
print(pow_reduce([1,2,3,4]))   # ((((2^1)^2)^3)^4) = 2^(1*2*3*4) = 2^24 = 16777216
```
### 2.3 除法与异常处理

我们可以利用 `try` 在归约过程中捕获异常，例如除零错误：

```python

def divide_all(total, divisors):
    try:
        return reduce(lambda x, y: x / y, divisors, total)
    except ZeroDivisionError:
        return float('inf')   # 除零时返回无穷大
```
这种设计实现了**关注点分离**：`reduce` 只负责归约逻辑，调用者负责处理异常。

---

## 三、编程语言与解释器

### 3.1 什么是解释器？

**解释器**（interpreter）是一个程序，它接收用某种编程语言编写的代码作为输入，并执行该代码，产生符合语言语义的行为。

- **机器语言**：由硬件直接执行，指令集固定，难以编程（缺乏抽象）。
    
- **高级语言**：由解释器或编译器翻译成机器语言，提供命名、函数、对象等抽象，便于人类使用。
    
- **元语言学抽象**：针对特定领域定义新语言，如 **Erlang**（并发编程）、**MediaWiki**（静态网页生成）。
    

### 3.2 解释器的层次

Python 本身也是一种高级语言，它的实现（CPython）通常包含：

- **编译器**：将 Python 源码编译为**字节码**（一种低级中间语言）。
    
- **虚拟机**：解释执行字节码。
    

这种分层使得 Python 能跨平台运行。

### 3.3 元语言抽象：定义新语言

有时我们需要针对特定问题设计一种**领域特定语言**（DSL）。例如，本课将要实现的**Scheme语法计算器**就是一种极简的 DSL，只支持算术运算。

---

## 四、解析（Parsing）

解析是将文本转换为内部数据结构（通常是**抽象语法树**）的过程。它包含两个阶段：

### 4.1 词法分析（Lexical Analysis）

将输入字符串拆分成**标记**（token）序列，丢弃无关空格，识别数字、符号、括号等。

例如输入 `"(+ 1 (* 2 3))"` 会生成标记：  
`'(' , '+', '1', '(', '*', '2', '3', ')', ')'`

### 4.2 语法分析（Syntactic Analysis）

根据语言的语法规则，将标记序列转换为**树形结构**。对于 Scheme 风格的表达式，树结构通常用嵌套的 Pair 表示：

```python

class Pair:
    def __init__(self, first, rest):
        self.first = first
        self.rest = rest
    def __repr__(self):
        return f"Pair({self.first}, {self.rest})"
```
列表 `(1 2 3)` 可表示为 `Pair(1, Pair(2, Pair(3, nil)))`，其中 `nil` 是一个特殊标记。

**解析函数**（如 `scheme_read`）递归地消费标记，当遇到 `'('` 时开始构建列表，直到匹配的 `')'`。

---

## 五、Scheme 语法计算器

我们构建一个简化版的计算器，只支持 Scheme 风格的算术表达式，包括四种运算符 `+`、`-`、`*`、`/`，且支持嵌套。

### 5.1 语言语义

- **原始表达式**：数字，求值为自身（浮点数或整数）。
    
- **调用表达式**：`(<operator> <expr1> ... <exprn>)`，其中 `<operator>` 为 `+`、`-`、`*`、`/`。
    
    - `+`：对所有参数求和（至少 0 个参数，无参数返回 0）。
        
    - `*`：对所有参数求积（至少 0 个参数，无参数返回 1）。
        
    - `-`：若只有一个参数，返回其相反数；否则用第一个参数减去其余参数（至少 1 个参数）。
        
    - `/`：若只有一个参数，返回其倒数；否则用第一个参数除以其余参数（至少 1 个参数，除数不能为 0）。
        

### 5.2 实现求值函数 `calc_eval`

```python

def calc_eval(expr):
    """计算 Scheme 算术表达式，expr 为 Pair 表示的列表或数字"""
    if isinstance(expr, (int, float)):
        return expr
    elif isinstance(expr, Pair):
        operator = expr.first
        operands = expr.rest
        args = [calc_eval(arg) for arg in operands]   # 递归求值所有操作数
        return calc_apply(operator, args)
    else:
        raise TypeError(f"未知表达式类型: {expr}")
```
### 5.3 实现应用函数 `calc_apply`

```python

def calc_apply(operator, args):
    if operator == '+':
        return sum(args)
    elif operator == '*':
        product = 1
        for x in args:
            product *= x
        return product
    elif operator == '-':
        if len(args) == 0:
            raise TypeError("'-' 需要至少一个参数")
        if len(args) == 1:
            return -args[0]
        else:
            result = args[0]
            for x in args[1:]:
                result -= x
            return result
    elif operator == '/':
        if len(args) == 0:
            raise TypeError("'/' 需要至少一个参数")
        if len(args) == 1:
            return 1 / args[0]
        else:
            result = args[0]
            for x in args[1:]:
                result /= x
            return result
    else:
        raise TypeError(f"未知运算符: {operator}")
```
### 5.4 示例运行

假设我们已经有了解析函数 `scheme_read` 可以将字符串转换为 `Pair` 结构，那么：

```scheme

scm> (+ 1 2 3)
6
scm> (* 2 3 4)
24
scm> (- 10 3 2)
5
scm> (/ 8 2 2)
2.0
scm> (+ 5 (* 2 3) (/ 10 2))   ; 5 + 6 + 5 = 16
16
```
---

## 六、读取-求值-打印循环（REPL）

解释器的核心是一个无限循环，不断执行以下步骤：

1. **打印提示符**（如 `scm>`）。
    
2. **读取**用户输入的一行文本。
    
3. **解析**文本为表达式（`Pair` 结构）。
    
4. **求值**表达式（调用 `calc_eval`）。
    
5. **打印**结果。
    
6. 如果发生异常，打印错误信息并继续循环（除非是退出指令）。
    

### 6.1 简单的 REPL 实现

```python

def repl():
    while True:
        try:
            text = input('scm> ')
            if not text:
                continue
            expr = scheme_read(text)   # 假设已实现
            result = calc_eval(expr)
            print(result)
        except (EOFError, KeyboardInterrupt):
            print(' 退出')
            break
        except Exception as e:
            print(f"错误: {e}")
```
### 6.2 异常处理策略

在 REPL 中，我们捕获所有异常，避免程序崩溃，让用户能继续输入。

- **词法/语法错误**：如无效数字 `2.3.4`，会引发 `ValueError` 或 `SyntaxError`，我们打印后继续。
    
- **求值错误**：如除零、参数个数错误等，由 `calc_apply` 中的 `TypeError` 捕获。
    
- **退出条件**：用户输入 `Ctrl+D`（EOF）或 `Ctrl+C` 时，优雅退出。
    

---

## 七、知识点总结表

|知识点|核心内容|考试重点 / 易混淆点|难度|
|---|---|---|---|
|**异常处理**|`raise` 引发异常，`try-except` 捕获异常，内置异常类型|自定义异常；多个 `except` 顺序；`as` 获取异常对象|⭐⭐|
|**`reduce` 函数**|二元函数逐步归约序列，需提供初始值|初始值的选取；递归实现与迭代实现|⭐⭐⭐|
|**解释器概念**|高级语言由解释器/编译器执行，元语言抽象|编译与解释的区别；字节码的作用|⭐⭐|
|**解析过程**|词法分析（标记化）与语法分析（构建树）|`Pair` 结构表示 Scheme 列表；括号匹配|⭐⭐⭐|
|**计算器语义**|四种算术运算符的求值规则|运算符的 arity（参数个数）和默认行为|⭐⭐|
|**求值函数**|`calc_eval` 递归求值，`calc_apply` 分派|递归对操作数求值；类型检查|⭐⭐⭐|
|**REPL 循环**|读取-求值-打印，异常处理保证持续运行|捕获所有异常避免崩溃；退出条件|⭐⭐|

---

## 八、自测题

### 1. 异常处理

写出以下代码的输出：

```python

def test(x):
    try:
        return 10 / x
    except ZeroDivisionError:
        return 'inf'
    finally:
        print('清理')
print(test(2))
print(test(0))
```
<details> <summary>答案</summary>

text

清理
5.0
清理
inf

`finally` 块无论是否发生异常都会执行。

</details>

### 2. `reduce` 应用

实现一个函数 `factorial(n)`，使用 `reduce` 计算 `n!`（提示：初始值应为 1，序列为 `range(1, n+1)`）。

```python

from operator import mul
def factorial(n):
    return reduce(mul, range(1, n+1), 1)
```

### 3. 解析与求值

假设我们有如下 `Pair` 结构：

```python

expr = Pair('+', Pair(1, Pair(2, Pair(3, nil))))
```
调用 `calc_eval(expr)` 的结果是什么？


6。`calc_eval` 递归求值，先对操作数 `1,2,3` 求值得自身，然后调用 `calc_apply('+', [1,2,3])` 求和。

### 4. 除法异常处理

在 `calc_apply` 中，如果 `operator` 为 `'/'` 且除数为 0，应如何处理？为什么？

<details> <summary>答案</summary>

应引发 `TypeError` 或 `ZeroDivisionError`，因为除法不满足语义。捕获该异常后，REPL 会打印错误信息，但不会崩溃。也可以返回特殊值，但通常错误应显式报告。

</details>

### 5. 扩展计算器

如何修改计算器使其支持 `expt`（幂运算）操作符？


- 在 `calc_apply` 中添加分支 `elif operator == 'expt'`，然后使用 `pow` 函数或 `**` 运算符。注意参数个数应为 2（左参数为底，右参数为指数）。


### 6. 思考题

为什么在 REPL 中捕获所有异常通常是一个好主意？有没有应该让程序崩溃的情况？

<details> <summary>提示</summary>

对于交互式环境，我们希望用户能不断尝试，即使输入错误也不应退出。但对于脚本模式，未捕获的异常应让程序终止，以便于调试。REPL 的设计目标是易用性和容错性。

</details>

---

## 九、总结与展望

本节课我们深入探讨了异常处理、高阶函数 `reduce`，以及解释器的基本构造。通过实现一个 Scheme 语法计算器，我们亲身体验了：

- 如何用 `Pair` 表示嵌套表达式（抽象语法树）。
    
- 如何递归求值表达式（`eval` 与 `apply` 模式）。
    
- 如何构建一个健壮的 REPL 循环，优雅地处理各种错误。
    

这些知识是理解更复杂编程语言（如 Python、Scheme）解释器的基础。在后续课程中，我们将扩展计算器，支持变量定义、条件表达式等，最终构建一个完整的 Scheme 解释器。