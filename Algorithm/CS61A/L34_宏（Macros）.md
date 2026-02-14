# 📘 Lecture 34：宏（Macros）—— 完全自学指南

> 宏是 Lisp 家族语言中最强大的特性之一。它允许你在代码求值之前对源代码进行转换，从而定义新的语言构造（特殊形式），扩展语言的核心语法。宏利用了 Lisp“代码即数据”的同像性，让我们能够以一种其他语言难以企及的方式进行元编程。本节课将带你从准引用复习开始，深入理解宏的工作原理，并通过多个经典示例掌握宏的定义和使用。

---

## 一、准引用复习

在深入宏之前，我们先复习一下准引用（quasiquote），因为它是编写宏的基石。

### 1.1 准引用与取消引用

- **准引用**：使用反引号 `` ` `` 标记一个表达式模板。模板中大部分内容被引用（即不求值），但可以通过**逗号 `,`** 标记的子表达式会被求值并插入到模板中。
    
- **取消引用**：`,expr` 表示 `expr` 会被求值，其结果嵌入到准引用模板中。
    

#### 示例

```scheme

> (define b 4)
> `(a ,(+ b 1) c)
(a 5 c)
```
### 1.2 讨论题：毕达哥拉斯定理

**题目**：完成以下代码填空，使得 `(list (square-expr 'a) (square-expr 'b))` 的求值结果为 `(+ (* a a) (* b b))`。

```scheme

(define (square-expr term)
  `(* ,term ,term))
`(+ ,(square-expr 'a) ,(square-expr 'b))
```
**关键点**：

- `square-expr` 需要返回一个 `(* term term)` 形式的列表，因此使用准引用模板 `(* ,term ,term)`，其中 `term` 被取消引用。
    
- 最终表达式也需要准引用，并取消引用两个 `square-expr` 的调用结果。
    

#### 常见错误分析

|错误代码|结果|原因|
|---|---|---|
|`(define (square-expr term) '(* term term))`|`(+ (* term term) (* term term))`|`term` 未被求值替换，保留为符号 `term`|
|`(define (square-expr term) (* term term))`|报错 `operand 0 (a) is not a number`|试图对符号 `'a` 进行乘法运算，不是构造列表|
|`` `(`+ ,(square-expr 'a) ,(square-expr 'b)) ``|`(quasiquote +) (* a a) (* b b)`|多了一层准引用，导致 `+` 前出现 `quasiquote` 符号|

---

## 二、宏的基本概念

### 2.1 什么是宏？

**宏**是一种在程序求值之前对源代码进行转换的机制。它接收一段代码（作为数据），返回另一段代码，然后由解释器对返回的代码进行求值。宏让我们能够定义新的**特殊形式**，扩展语言本身。

### 2.2 为什么需要宏？

- 普通过程（函数）的参数会在调用前被求值。
    
- 宏的参数是**未求值的源代码**，宏可以控制哪些部分被求值、何时求值，甚至完全改变代码结构。
    
- 这使得我们可以实现新的控制结构（如 `for` 循环、条件测试框架等），而这些在函数中无法实现。
    

### 2.3 宏与普通过程的区别

|特性|普通过程|宏|
|---|---|---|
|参数求值时机|调用前求值|传递源代码（未求值）|
|返回值|值|代码（表达式）|
|执行时机|运行时|编译/转换时（求值前）|
|用途|计算值|转换代码，扩展语法|

### 2.4 定义宏：`define-macro`

Scheme 中使用 `define-macro` 定义宏。语法类似于过程定义，但行为完全不同。

```scheme

(define-macro (twice expr)
  (list 'begin expr expr))
```
这个宏 `twice` 接收一个表达式 `expr`，返回一个 `begin` 表达式，其中 `expr` 被写两次。

#### 使用示例

```scheme

> (twice (print 2))
2
2
; 返回值不确定（print 返回空值）
```
**执行过程**：

1. 宏调用 `(twice (print 2))` 被识别。
    
2. 宏 `twice` 被调用，参数 `expr` 绑定到源代码 `(print 2)`（未求值）。
    
3. 宏体执行 `(list 'begin expr expr)`，生成代码 `(begin (print 2) (print 2))`。
    
4. 生成的代码被求值，因此 `print` 被执行两次。
    

如果 `twice` 是普通过程，`(print 2)` 会在过程调用前求值（打印一次），然后过程收到的是 `print` 的返回值（可能是空值），无法实现重复执行。

---

## 三、宏示例详解

### 3.1 `twice` 宏

```scheme

(define-macro (twice expr)
  `(begin ,expr ,expr))
```
使用准引用让代码更清晰：`` `(begin ,expr ,expr) ``。

### 3.2 `check` 宏：测试表达式并报告失败

我们希望创建一个测试宏 `check`，它接收一个表达式，如果表达式求值为真，则返回 `'pass`；如果为假，则返回包含原始表达式的失败信息。

```scheme

(define-macro (check expr)
  `(if ,expr
       'pass
       (list 'failed: ',expr)))
```
**关键点**：

- `,expr` 被求值（在生成的代码中，它是实际测试的表达式）。
    
- `',expr` 通过引用捕获原始表达式的源代码，用于失败时显示。
    
- 宏返回的代码是一个 `if` 表达式，在运行时执行测试。
    

**使用示例**：

```scheme

> (define x -3)
> (check (> x 0))
(failed: (> x 0))
> (check (number? x))
pass
```
如果使用普通过程，我们无法在失败时显示原始表达式，因为过程只能看到表达式的值（如 `#f`），看不到源代码。

### 3.3 `for` 宏：模拟循环

我们希望创建一个 `for` 宏，使得：

```scheme

(for x in (2 3 4) (* x x))
```
能够展开为对每个元素应用 `(* x x)` 的列表，相当于：

```scheme

(map (lambda (x) (* x x)) '(2 3 4))
```
**宏定义**：

```scheme

(define-macro (for var in lst expr)
  `(map (lambda (,var) ,expr) ,lst))
```
**使用示例**：

```scheme

> (for x in (2 3 4) (* x x))
(4 9 16)
```
**分析**：

- 宏参数：`var` 是符号 `x`，`lst` 是列表 `(2 3 4)`，`expr` 是 `(* x x)`。
    
- 生成的代码：`(map (lambda (x) (* x x)) '(2 3 4))`。
    
- 注意：`lst` 在生成的代码中直接使用，因此它必须是字面列表或能求值为列表的表达式（但这里 `lst` 是未求值的源代码 `(2 3 4)`，在生成的代码中它被直接使用，求值后成为列表）。
    

### 3.4 `trace` 宏：追踪递归调用

我们希望创建一个 `trace` 宏，它可以临时追踪一个递归函数的调用，打印每次调用的参数，执行完后恢复原函数。

#### Python 装饰器版（对比）

```python

def trace(fn):
    def traced(n):
        print(f"{fn.__name__}({n})")
        return fn(n)
    return traced
@trace
def fact(n):
    return 1 if n == 0 else n * fact(n-1)
```
#### Scheme 宏版

```scheme

(define-macro (trace call)
  (let ((op (car call))
        (args (cdr call)))
    `(begin
       (define original ,op)
       (define ,op
         (lambda ,(map (lambda (arg) (gensym)) args)  ; 为避免变量捕获，使用 gensym
           (begin
             (display (list ',op ,@args))
             (newline)
             (apply original (list ,@args)))))
       (let ((result ,call))
         (set! ,op original)
         result))))
```
**简化版本**（假设只有一个参数且不考虑变量捕获）：

```scheme

(define-macro (trace call)
  (let ((op (car call))
        (arg (cadr call)))
    `(begin
       (define original ,op)
       (define ,op
         (lambda (x)
           (display (list ',op x))
           (newline)
           (original x)))
       (let ((result (,op ,arg)))
         (set! ,op original)
         result))))
```
**使用示例**：

```scheme

> (trace (fact 5))
(fact 5)
(fact 4)
(fact 3)
(fact 2)
(fact 1)
(fact 0)
120
```
**关键点**：

- 宏接收一个完整的调用表达式，例如 `(fact 5)`。
    
- 提取操作符 `fact` 和参数 `5`。
    
- 生成代码：临时重定义 `fact` 为打印版本，执行调用后恢复原定义。
    
- 使用 `gensym` 可以避免参数名与外部变量冲突（但简化版假设只有一个参数 `x`）。
    

---

## 四、宏的注意事项

### 4.1 变量捕获（Variable Capture）

在宏生成的代码中，如果不小心使用了与用户代码中相同的变量名，可能导致意外地捕获（遮蔽）用户变量。使用 `gensym` 生成唯一符号可以避免这个问题。

例如，在 `trace` 宏中，我们使用了 `x` 作为内部 lambda 的参数，如果用户代码中也使用了 `x` 作为全局变量，可能会产生混淆。更好的做法是用 `(gensym)` 生成一个唯一的符号。

### 4.2 宏的调试

宏在展开阶段执行，错误可能出现在展开后的代码中。可以使用 `macroexpand` 查看宏展开的结果，帮助调试。

```scheme

> (macroexpand '(twice (print 2)))
(begin (print 2) (print 2))
```
### 4.3 宏与过程的权衡

- 能用过程解决的问题，尽量用过程。
    
- 只有需要控制参数求值时机、或需要操作源代码本身时，才使用宏。
    
- 宏会增加代码复杂性，应谨慎使用。
    

---

## 五、知识点总结表

|知识点|核心内容|考试重点 / 易混淆点|难度|
|---|---|---|---|
|**准引用复习**|`` ` `` 引用模板，`,` 插入求值结果 \| 与普通引用的区别；嵌套准引用 \| ⭐⭐⭐ \||||
|**宏的定义**|`define-macro` 定义宏，接收源代码，返回代码|宏在求值前执行，参数未求值|⭐⭐⭐|
|**`twice` 宏**|重复执行表达式|宏如何避免参数提前求值|⭐⭐|
|**`check` 宏**|测试表达式，失败时显示源代码|宏捕获源代码的能力|⭐⭐⭐|
|**`for` 宏**|模拟循环，展开为 `map`|生成 `lambda` 和调用|⭐⭐⭐|
|**`trace` 宏**|临时追踪递归调用|临时重定义函数，恢复原定义|⭐⭐⭐⭐|
|**变量捕获**|宏生成的代码中意外使用用户变量名|用 `gensym` 生成唯一符号|⭐⭐⭐⭐|
|**宏与过程区别**|参数求值时机、返回值、执行阶段|何时用宏，何时用过程|⭐⭐⭐|

---

## 六、自测题

### 1. 准引用练习

已知 `x = 10`，写出以下表达式的结果：

```scheme

`(a ,(+ x 1) b ,(list 'c 'd))
```
<details> <summary>答案</summary> `(a 11 b (c d))` </details>

### 2. 宏 `twice` 变体

定义一个宏 `thrice`，使其能重复执行表达式三次。

```scheme 
(define-macro (thrice expr) `(begin ,expr ,expr ,expr)) 
``` 
### 3. `check` 宏改进

修改 `check` 宏，使其在失败时不仅显示表达式，还显示期望的结果（例如 `(check (= 2 3))` 输出 `(failed: (= 2 3) expected #t)`）。

```scheme 
(define-macro (check expr) `(if ,expr 'pass (list 'failed: ',expr 'expected #t))) 
``` 

### 4. 宏 `for` 的缺陷

如果 `lst` 不是一个字面列表，而是一个会返回列表的表达式，例如 `(for x in (range 5) (* x x))`，我们的 `for` 宏能正确工作吗？为什么？


- 不能。因为 `lst` 在宏展开时是未求值的源代码，如果 `(range 5)` 是一个函数调用，宏会原样放入生成的代码中，这是正确的，因为生成的代码是 `(map (lambda (x) (* x x)) (range 5))`，这样 `(range 5)` 会在运行时求值。所以实际上可以工作！但注意如果 `lst` 是 `'(2 3 4)` 这样的字面列表，它也会被原样放入，最终也是正确的。所以 `for` 宏是通用的。



### 5. `trace` 宏的简化

为什么简化版 `trace` 宏假设只有一个参数是危险的？如果用于有两个参数的函数会怎样？


简化版硬编码了 `(lambda (x) ...)`，如果函数有两个参数，会导致生成的代码参数数量不匹配，运行时出错。应该用 `(map (lambda (arg) (gensym)) args)` 生成与参数数量相同的唯一符号，并用 `apply` 处理。


### 6. 思考题

为什么 Lisp 语言特别适合定义宏？其他语言（如 Python）的类似机制（如装饰器）有何不同？

<details> <summary>提示</summary>

Lisp 的“代码即数据”特性让宏可以直接操作源代码列表，生成新代码的过程非常自然。Python 的装饰器是在函数定义后对其进行包装，只能操作函数对象，不能控制参数求值时机或任意代码转换。Python 没有真正的宏系统，因为它的语法不是以数据结构表示的。

</details>

---

## 七、总结与展望

本节课我们深入探索了 Scheme 中宏的强大能力：

- 复习了准引用，它是构造代码模板的基础。
    
- 理解了宏与普通过程的本质区别：宏操作源代码，过程操作值。
    
- 通过多个示例（`twice`、`check`、`for`、`trace`）展示了宏的实际应用，包括控制重复执行、构建测试框架、模拟循环、临时追踪递归。
    
- 讨论了变量捕获问题及其解决方案。
    

宏是元编程的终极工具之一，它让我们能够扩展语言本身，实现领域特定语言（DSL）和新的控制结构。在后续的课程或项目中，你将有机会自己设计和实现宏，体会这种“编程语言的编程”的魅力。