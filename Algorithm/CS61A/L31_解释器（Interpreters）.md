# 📘 Lecture 31：解释器（Interpreters）—— Scheme 求值器深度解析

> 解释器是一种程序，它能读取用某种编程语言编写的代码并执行它。本节课我们将深入 Scheme 解释器的内部，学习如何处理各种特殊形式（if、lambda、define、quote），理解环境模型如何支持词法作用域，并掌握 eval 和 apply 相互递归的核心机制。这是计算机科学中“元语言抽象”的经典体现，也是项目4的基础。

---

## 一、Scheme 表达式的求值

在 Scheme 中，所有表达式都可以分为三类：

1. **自求值表达式**：如数字、布尔值、空列表 `nil`，求值结果就是自身。
    
2. **符号**：如 `x`、`+`，求值结果为当前环境中该符号绑定的值。
    
3. **组合表达式**：以括号开头的列表，根据第一个元素（操作符）的不同，可能是**特殊形式**或**普通调用表达式**。
    

### 1.1 `eval` 的分发机制

Scheme 的 `eval` 函数（求值器）根据表达式的类型进行分发：

- 如果是**自求值表达式**，直接返回。
    
- 如果是**符号**，在当前环境中查找其绑定。
    
- 如果是**组合表达式**，则查看第一个元素：
    
    - 如果是**特殊形式**（如 `if`、`lambda`、`define`、`quote` 等），按特殊规则处理。
        
    - 否则视为**调用表达式**：先递归求值操作符和操作数，然后将操作符应用到操作数上。
        

---

## 二、特殊形式详解

特殊形式之所以“特殊”，是因为它们的求值规则不同于普通调用表达式——它们不会对所有子表达式求值，而是根据语义选择性地求值。

### 2.1 `if` 表达式

语法：`(if <predicate> <consequent> <alternative>)`

**求值规则**：

1. 求值 `<predicate>`。
    
2. 如果结果为真（即非 `#f`），则求值 `<consequent>` 并返回；否则求值 `<alternative>` 并返回。
    
3. **注意**：只有被选中的分支会被求值，另一个分支完全忽略（即使包含错误也不会触发）。
    

```scheme

> (if #t 1 (/ 1 0))   ; 1，因为 (/ 1 0) 从未被求值
1
> (if #f 1 (/ 1 0))   ; 错误：division by zero
```
### 2.2 `cond` 表达式

语法：

```scheme

(cond
  (<p1> <e1>)
  (<p2> <e2>)
  ...
  (else <e>))
```
**求值规则**：

- 依次求值每个 `<p>`，遇到第一个为真的，求值对应的 `<e>` 并返回。
    
- 如果所有 `<p>` 都为假，且存在 `else` 子句，则求值 `<e>`；否则返回未指定值（通常为 `#f` 或未定义）。
    

`cond` 是多重条件分支的惯用写法，相当于 Python 的 `if-elif-else` 链。

### 2.3 `and` 与 `or`

- `(and <e1> ... <en>)`：从左到右求值，遇到假值立即返回该假值（短路）；若所有为真，返回最后一个表达式的值。
    
- `(or <e1> ... <en>)`：从左到右求值，遇到真值立即返回该真值；若所有为假，返回 `#f`。
    

```scheme

> (and 1 2 3)          ; 3
> (and #f 2)           ; #f
> (or #f 2 3)          ; 2
> (or #f #f)           ; #f
```
### 2.4 `quote` 表达式

语法：`(quote <expression>)` 或简写 `'<expression>`

**核心作用**：阻止表达式求值，将其作为数据返回。

```scheme

> (quote (+ 1 2))       ; (+ 1 2) 列表，不是数字 3
> '(+ 1 2)             ; 同上
```
**实现机制**：Scheme 的读取器（reader）会将 `'x` 自动转换为 `(quote x)`，然后传递给求值器。求值器遇到 `quote` 时，直接返回其子表达式（不求值）。

**重要性**：`quote` 是 Lisp 元编程的基础，它使“代码即数据”成为可能。

### 2.5 `lambda` 表达式

语法：`(lambda (<formal-parameters>) <body>)`

**作用**：创建并返回一个用户定义的过程（函数）。

**求值结果**：一个过程对象，包含三个关键信息：

- 形式参数列表（如 `(x)`）
    
- 函数体表达式（如 `(* x x)`）
    
- **定义时的环境**（闭包环境）
    

**示例**：

```scheme

(define square (lambda (x) (* x x)))   ; 定义过程
(square 5)                              ; 25
```
在解释器内部，`lambda` 表达式求值后会生成一个 `LambdaProcedure` 实例，保存上述三要素。

### 2.6 `define` 表达式

语法有两种形式：

**1. 变量定义**：`(define <name> <expression>)`

- 先求值 `<expression>`，然后将 `<name>` 绑定到该值在当前环境的第一帧中。
    

```scheme

(define pi 3.14)        ; pi 绑定到 3.14
```
**2. 过程定义**：`(define (<name> <params>) <body>)`

- 这是**语法糖**，实际等价于：
    
    ```scheme
    
    (define <name> (lambda (<params>) <body>))
    ```

```scheme

(define (square x) (* x x))   ; 等价于 (define square (lambda (x) (* x x)))
```
---

## 三、环境模型

### 3.1 帧（Frame）与环境

环境由一系列**帧**（frame）组成，每个帧是一个存储名称-值绑定的字典。帧之间有父子关系，形成链。最顶层的帧是**全局帧**。

在 Scheme 解释器中，我们用一个 `Frame` 类来表示：

- `frame.lookup(name)`：在当前帧查找 `name`，若未找到则递归到父帧。
    
- `frame.define(name, value)`：在当前帧创建或更新绑定。
    

### 3.2 词法作用域（Lexical Scoping）

**核心规则**：一个过程的**定义环境**决定了它执行时查找变量的范围。也就是说，函数的父环境是它被**定义**时的环境，而不是被**调用**时的环境。

**示例**：

```scheme

(define y 3)                 ; 全局帧中 y=3
(define (f x)
  (define g (lambda () x))   ; g 定义时环境是 f 的局部帧（包含 x）
  g)
(define h (f 2))             ; h 是 g，其定义环境是 f 的局部帧（x=2）
(h)                          ; 2，因为 g 的父环境中有 x=2
```
### 3.3 动态作用域（Dynamic Scoping）—— 对比

动态作用域中，函数的父环境是**调用时**的环境。Scheme 和 Python 都使用词法作用域，但了解动态作用域有助于理解历史语言（如早期 Lisp）和一些特殊情况。

```scheme

(define y 3)
(define (f x)
  (g))
(define (g)
  y)
(f 2)   ; 在词法作用域中，g 的父环境是全局，y=3
        ; 在动态作用域中，g 的父环境是调用者 f 的帧（包含 x=2），但 f 中没有 y，继续向上到全局，结果仍是 3。但如果 f 中定义了 y，动态作用域就会找到 f 的 y。
```
**关键差异**：词法作用域“记住”定义位置，动态作用域“跟随”调用链。

---

## 四、`eval` 与 `apply` 的相互递归

这是解释器最核心的机制。`eval` 负责求值表达式，`apply` 负责应用过程。

### 4.1 `eval` 的职责

输入：一个表达式 `expr` 和一个环境 `env`。  
输出：表达式的求值结果。

伪代码：

```python

def eval(expr, env):
    if is_self_evaluating(expr):
        return expr
    elif is_symbol(expr):
        return env.lookup(expr)
    elif is_combination(expr):
        if is_special_form(expr):
            return eval_special_form(expr, env)
        else:
            operator = eval(expr.first, env)
            operands = [eval(arg, env) for arg in expr.rest]
            return apply(operator, operands)
```
### 4.2 `apply` 的职责

输入：一个过程 `procedure` 和参数列表 `args`。  
输出：过程应用的结果。

```python

def apply(procedure, args):
    if is_primitive_procedure(procedure):
        return apply_primitive(procedure, args)
    elif is_lambda_procedure(procedure):
        # 创建新环境：新帧，父帧为 procedure 的定义环境
        new_env = Frame(procedure.env)
        # 将形参与实参绑定
        for param, arg in zip(procedure.params, args):
            new_env.define(param, arg)
        # 在新环境中求值函数体
        return eval(procedure.body, new_env)
    else:
        raise TypeError("Unknown procedure type")
```
### 4.3 相互递归示例：递归函数 `demo`

考虑之前定义的 `demo` 过程：

```scheme

(define (demo s)
  (if (null? s)
      '(3)
      (cons (car s) (demo (cdr s)))))
```
调用 `(demo '(1 2))` 的过程：

1. `eval` 检测到调用表达式，求值操作符 `demo`（得到过程对象），求值操作数 `'(1 2)`（得到列表 `(1 2)`）。
    
2. `apply` 被调用，过程对象是 `demo`，参数为 `(1 2)`。
    
    - 创建新帧 `f1`，父环境为全局帧，绑定 `s = (1 2)`。
        
    - 在新环境 `f1` 中求值函数体 `(if ...)`。
        
3. `eval` 在 `f1` 中遇到 `if`，谓词 `(null? s)` 为假，因此求值 `(cons (car s) (demo (cdr s)))`。
    
    - 分别求值 `(car s)` 得 `1`，求值 `(demo (cdr s))`。
        
4. 对 `(demo (cdr s))` 再次调用 `eval` 和 `apply`，创建新帧 `f2`，父环境仍为全局（因为 `demo` 定义在全局），绑定 `s = (cdr (1 2)) = (2)`。
    
    - 在 `f2` 中求值 `(if ...)`，谓词 `(null? s)` 为假，求值 `(cons (car s) (demo (cdr s)))`。
        
    - `(car s)` 得 `2`，再次递归调用 `(demo (cdr s))`，`(cdr s)` 为 `()`。
        
5. 第三次调用 `demo`，创建帧 `f3`，绑定 `s = ()`。
    
    - 在 `f3` 中求值 `(if (null? s) '(3) ...)`，谓词为真，求值 `'(3)`，返回 `(3)`。
        
6. 回到 `f2`，`(demo (cdr s))` 得到 `(3)`，然后 `(cons 2 '(3))` 得到 `(2 3)`，返回。
    
7. 回到 `f1`，`(demo (cdr s))` 得到 `(2 3)`，`(cons 1 '(2 3))` 得到 `(1 2 3)`，返回最终结果。
    

**环境链**：`f1`（父=全局）→ `f2`（父=全局）→ `f3`（父=全局）。每次调用都基于定义环境（全局）创建新帧，这正是词法作用域的体现。

---

## 五、引用（Quote）的读取与求值

### 5.1 读取器的预处理

当你在 REPL 中输入 `'(1 2)` 时，读取器会将其转换为 `(quote (1 2))` 这个列表，然后传递给 `eval`。因此，`eval` 实际看到的是 `(quote (1 2))`。

### 5.2 `eval` 处理 `quote`

`eval` 识别出第一个元素是 `quote` 这种特殊形式后，会直接返回其第二个子表达式（不求值）。所以 `(quote (1 2))` 返回列表 `(1 2)`。

---

## 六、定义（Define）的实现

### 6.1 变量定义

`(define x 42)` 的处理：

1. `eval` 识别出这是 `define` 特殊形式。
    
2. 求值第二个子表达式 `42`，得到数值 `42`。
    
3. 在当前环境的**第一帧**中，将符号 `x` 绑定到 `42`。
    

### 6.2 过程定义

`(define (f x) (* x x))` 的处理：

1. `eval` 识别出 `define`，并发现第一个子表达式是列表 `(f x)`。
    
2. 将其转换为等价形式：`(define f (lambda (x) (* x x)))`。
    
3. 按变量定义处理：求值 `(lambda (x) (* x x))`，得到过程对象，然后绑定 `f` 到该对象。
    

---

## 七、历史渊源：Lisp 1.5 中的 eval/apply

`eval` 和 `apply` 的递归模式最早出现在 **Lisp 1.5**（约1960年）中。那时的语法与现代不同，但核心思想完全一致：

- **原子**（atom）：数字或符号，符号需在关联列表（环境）中查找。
    
- **特殊形式**：如 `QUOTE`、`COND`、`LAMBDA`、`LABEL`（用于定义递归函数）。
    
- **`apply`**：处理函数应用，包括内置函数（如 `CAR`、`CDR`、`CONS`）和用户定义函数（以 `LAMBDA` 或 `LABEL` 形式出现）。
    
- **`eval`**：根据表达式类型分发，对组合表达式调用 `apply`。
    

这种架构至今仍是大多数 Lisp 方言的基础，体现了其设计的优雅和永恒性。

---

## 八、知识点总结表

|知识点|核心内容|考试重点 / 易混淆点|难度|
|---|---|---|---|
|**特殊形式**|`if`, `cond`, `and`, `or`, `quote`, `lambda`, `define` 等，求值规则与普通调用不同|短路求值；`quote` 阻止求值；`lambda` 创建闭包|⭐⭐⭐|
|**环境模型**|帧链结构，`lookup` 从当前帧向上查找，`define` 在当前帧创建绑定|区分词法作用域与动态作用域|⭐⭐⭐⭐|
|**词法作用域**|函数的父环境是定义时的环境|与动态作用域的对比；闭包的形成|⭐⭐⭐⭐|
|**eval-apply**|`eval` 根据类型分派，`apply` 处理过程应用，二者相互递归|理解递归过程的环境创建|⭐⭐⭐⭐⭐|
|**引用（quote）**|`'x` 等价于 `(quote x)`，读取器负责转换|引用与求值的区别；代码即数据|⭐⭐⭐|
|**定义（define）**|变量定义和过程定义（语法糖）|过程定义如何转换为 `lambda`|⭐⭐|
|**历史渊源**|Lisp 1.5 中的 `eval`/`apply` 架构|了解即可|⭐|

---

## 九、自测题

### 1. 特殊形式识别

以下 Scheme 表达式中，哪些是特殊形式？哪些是普通调用表达式？

- `(if (> x 0) x (- x))`
    
- `(lambda (x) (* x x))`
    
- `(define y 5)`
    
- `(quote (a b c))`
    
- `(cons 1 '(2 3))`
    


特殊形式：`if`、`lambda`、`define`、`quote`。普通调用：`cons`。


### 2. 短路求值

写出以下表达式的结果：

- `(and 1 #f 2)`
    
- `(or #f #f 3)`
    
- `(if #f 1 (/ 1 0))`
    


- `#f`
    
- `3`
    
- 错误：`division by zero`，因为 `#f` 导致 `alternative` 被求值。
    

### 3. 词法作用域 vs 动态作用域

假设有以下 Scheme 代码（语法合法，但若按动态作用域解释）：

```scheme

(define x 1)
(define (f) x)
(define (g) (define x 2) (f))
(g)
```
- 在词法作用域下，结果是多少？
    
- 如果改为动态作用域，结果是多少？
    


- 词法作用域：`f` 定义时环境是全局，`x=1`，所以 `(f)` 返回 `1`。
    
- 动态作用域：调用 `(f)` 时，父环境是调用者 `g` 的局部帧，其中 `x=2`，所以返回 `2`。
    


### 4. `lambda` 与环境

写出以下代码的输出：

```scheme

(define (make-adder n)
  (lambda (x) (+ x n)))
(define add5 (make-adder 5))
(add5 3)
```
<details> <summary>答案</summary>

8。`add5` 是一个闭包，其定义环境（来自 `make-adder` 的帧）中保存了 `n=5`。

</details>

### 5. `eval` 与 `apply` 模拟

假设我们有如下简化版 `eval` 和 `apply`，请手动模拟求值过程：

```scheme

(define (square x) (* x x))
(square 4)
```
描述每一步中 `eval` 和 `apply` 的调用。


1. `eval` 遇到调用表达式 `(square 4)`：
    
    - 求值操作符 `square`：在全局环境中找到过程对象（参数 `(x)`，体 `(* x x)`，定义环境=全局）。
        
    - 求值操作数 `4`：得到 `4`。
        
    - 调用 `apply`，传入过程对象和 `[4]`。
        
2. `apply` 创建新帧，父环境为过程定义环境（全局），绑定 `x=4`。
    
3. 在新环境中求值函数体 `(* x x)`：
    
    - 求值 `*` 得到内置乘法过程。
        
    - 求值 `x` 两次，各得 `4`。
        
    - 调用 `apply` 到乘法过程，参数 `[4,4]`，得到 `16`。
        
4. 返回 `16`。
    


### 6. 思考题

为什么 `eval` 和 `apply` 需要相互递归？如果只用 `eval` 能实现解释器吗？


- 不能。因为 `eval` 负责求值表达式，但遇到过程应用时，需要将参数求值后传递给过程，并管理新环境的创建和函数体的求值——这正是 `apply` 的职责。`apply` 在求值函数体时又会调用 `eval`，形成相互递归。这是处理嵌套函数调用的自然方式。


---

## 十、总结与展望

本节课我们深入剖析了 Scheme 解释器的内部工作原理：

- 特殊形式的求值规则（`if`、`lambda`、`define`、`quote` 等）。
    
- 环境模型如何支持词法作用域和闭包。
    
- `eval` 和 `apply` 的相互递归是解释器的核心引擎。
    
- 历史渊源：Lisp 1.5 早已奠定这一架构。
    

这些知识不仅是项目4的基础，更让你理解编程语言是如何“自举”的——我们正在用 Python 实现一个 Scheme 解释器，而 Python 解释器本身也是用 C 语言编写的。这种元循环求值器（metacircular evaluator）是计算机科学中最优雅的思想之一。