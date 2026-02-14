# 📘 Lecture 28：Scheme

> Scheme 是一种简洁而优雅的函数式编程语言，是 Lisp 方言之一。通过学习 Scheme，你将发现许多编程理念可以轻松地从一种语言迁移到另一种语言。本节课将带你从零开始掌握 Scheme 的基础语法、特殊形式、递归编程以及海龟绘图，并通过经典的 Sierpinski 三角形示例展示递归的魅力。

---

## 一、Scheme 简介

### 1.1 什么是 Scheme？

Scheme 是一种较古老的编程语言，对 Python 的设计产生了深远影响。它是 Lisp 的一种方言，而 Lisp 是至今仍在使用的两种最古老的编程语言之一（另一种是 Fortran）。尽管历史悠久，Lisp 仍在不断发展和创新，吸引了众多开发者。

### 1.2 人们对 Lisp 的评价

- **Alan Kay**（面向对象编程之父）称 Lisp 为“有史以来设计的最伟大的单一编程语言”。
    
- **Neal Stephenson**（著名科幻作家）认为它是“唯一美丽的计算机语言”。
    
- **Brian Harvey**（伯克利 CS 教师）将其称为“上帝的编程语言”。
    
- 甚至 XKCD 漫画也表达了对 Lisp 的喜爱，展示了其持久的文化影响力。
    

Lisp 的美丽在于它的**简洁性**：整个语言可以在一天内学会，但它的表达能力极强，能够构建极其复杂的程序。

### 1.3 学习 Scheme 的意义

- 理解编程语言的核心理念可以跨语言迁移。
    
- 体验函数式编程的纯粹与优雅。
    
- 为后续学习解释器、编译器等高级主题打下基础。
    

---

## 二、Scheme 基础

### 2.1 程序由表达式组成

Scheme 程序完全由**表达式**构成。表达式分为两类：

- **原始表达式**（primitive expressions）：如数字、布尔值、基本运算符。
    
- **组合表达式**（combinations）：用括号括起来的表达式，第一个元素是操作符，后面是操作数。
    

#### 原始表达式

- 数字：`2`、`3.3` 等，求值为自身。
    
- 布尔值：`#t`（真）、`#f`（假）。
    
- 符号：如 `+`、`quotient` 等，绑定到相应的过程（函数）。
    

#### 组合表达式（调用表达式）

形式：`(operator operand1 operand2 ...)`

示例：

```scheme

> (quotient 10 2)
5
> (not #t)
#f
> (+ 1 2 3 4)
10
> (* 1 2 3 4)
24
```
**特殊情形**：

- `(+)` 求值为 `0`（加法的单位元）。
    
- `(*)` 求值为 `1`（乘法的单位元）。
    

#### 嵌套表达式

```scheme

> (quotient (+ 8 7) 5)
3
> (- (* 1 2 1 2 2 1 2 3 3 3 3) 1)
863
```
#### 求值规则

1. 数字自求值。
    
2. 符号在当前环境中查找其绑定的值。
    
3. 调用表达式先求值操作符和所有操作数，然后将过程应用到参数。
    

#### 谓词（Predicates）

以问号结尾的过程通常返回布尔值，用于判断：

```scheme

> (number? 3)
#t
> (number? +)
#f
> (zero? 2)
#f
> (zero? (- 2 2))
#t
> (integer? 2.2)
#f
> (integer? 2)
#t
```
### 2.2 REPL 环境

Scheme 解释器提供 Read-Eval-Print Loop（REPL）：

- 输入表达式，立即求值并打印结果。
    
- 支持任意深度嵌套。
    

错误处理示例：

```scheme

> (* 1 2 1 2 1 3 ...)  ; 若括号不匹配或符号未定义，会报错。
```
---

## 三、特殊形式（Special Forms）

特殊形式是那些求值规则不同于普通调用表达式的表达式。它们控制代码的执行流程或绑定符号。

### 3.1 `if` 表达式

语法：

```scheme

(if <predicate> <consequent> <alternative>)
```
求值规则：

1. 求值 `<predicate>`。
    
2. 如果结果为真（非 `#f`），则求值 `<consequent>` 并返回其值；否则求值 `<alternative>`。
    

示例：

```scheme

> (if (> 3 2) 'yes 'no)
yes
> (if #f 1 0)
0
```
### 3.2 `and` 和 `or`

- `(and <e1> ... <en>)`：从左到右求值，遇到假值即停止并返回该假值；若所有为真，返回最后一个表达式的值。
    
- `(or <e1> ... <en>)`：从左到右求值，遇到真值即停止并返回该真值；若所有为假，返回 `#f`。
    

示例：

```scheme

> (and 1 2 3)
3
> (and #f 2)
#f
> (or #f 2 3)
2
> (or #f #f)
#f
```
### 3.3 绑定符号：`define`

#### 定义变量

```scheme

(define pi 3.14)
(* pi 2)  ; 求值为 6.28
```
#### 定义过程

```scheme

(define (square x) (* x x))
(square 16)  ; 256
(define (average a b) (/ (+ a b) 2))
(average 3 7)  ; 5
```
**环境模型**：Scheme 使用与 Python 相同的环境模型，`define` 在当前框架中创建绑定。

### 3.4 `lambda` 表达式（匿名过程）

语法：

```scheme

(lambda (<formal-parameters>) <body>)
```
示例：

```scheme

> (define double (lambda (x) (* x 2)))
> (double 5)
10
```
`lambda` 可以直接用在调用表达式中：

```scheme

> ((lambda (x) (* x x)) 4)
16
```
### 3.5 `cond` 表达式（多条件分支）

`cond` 类似于 Python 的 `if-elif-else` 链。

语法：

```scheme

(cond
  (<predicate1> <expression1>)
  (<predicate2> <expression2>)
  ...
  (else <expression>))  ; else 可选
```
示例：

```scheme

(define (sign x)
  (cond
    ((> x 0) 'positive)
    ((= x 0) 'zero)
    (else 'negative)))
(sign -5)  ; negative
```
**执行规则**：依次求值每个 `<predicate>`，遇到第一个真值时，求值对应的 `<expression>` 并返回；若所有谓词为假且无 `else`，返回未定义值（通常为 `#f` 或未指定）。

### 3.6 `begin` 顺序执行

`begin` 用于按顺序执行多个表达式，并返回最后一个表达式的值。

语法：

```scheme

(begin <e1> <e2> ... <en>)
```
示例：

```scheme

> (begin (display "hello") (newline) 42)
hello
42
```
### 3.7 `let` 临时绑定

`let` 用于创建局部变量，绑定只在 `let` 体内有效。

语法：

```scheme

(let ((<var1> <expr1>)
      (<var2> <expr2>)
      ...)
  <body>)
```
示例：

```scheme

> (let ((x 3)
        (y 4))
    (+ (* x x) (* y y)))
25
```
**作用域**：`let` 创建的绑定在 `<body>` 中可见，且不会污染外部环境。

**风格建议**：

- `define` 用于持久绑定（如过程或全局常量）。
    
- `let` 用于临时计算。
    

---

## 四、递归与过程抽象

### 4.1 递归求平方根（巴比伦方法）

巴比伦方法（牛顿迭代法）求平方根：不断改进猜测值，直到满足精度。

```scheme

(define (sqrt-iter guess x)
  (if (good-enough? guess x)
      guess
      (sqrt-iter (improve guess x) x)))
(define (improve guess x)
  (average guess (/ x guess)))
(define (average a b)
  (/ (+ a b) 2))
(define (good-enough? guess x)
  (< (abs (- (* guess guess) x)) 0.001))
(define (sqrt x)
  (sqrt-iter 1.0 x))
```
**测试**：

```scheme

> (sqrt 2)
1.4142156862745097
```
**说明**：

- `sqrt-iter` 是递归过程，不断调用自身直到满足条件。
    
- 词法作用域允许内部函数访问外部参数（如 `x`）。
    

### 4.2 定义重复执行过程：`repeat`

```scheme

(define (repeat fn n)
  (if (= n 0)
      'done
      (begin
        (fn)
        (repeat fn (- n 1)))))
```
示例：

```scheme

> (repeat (lambda () (display "hello\n")) 3)
hello
hello
hello
done
```
---

## 五、Scheme 绘图与 Sierpinski 三角形

Scheme 解释器（如 STk）内置了海龟绘图（turtle graphics）功能。

### 5.1 基础绘图命令

- `(forward 100)` 或 `(fd 100)`：向前移动 100 单位。
    
- `(right 90)` 或 `(rt 90)`：右转 90 度。
    
- `(left 90)` 或 `(lt 90)`：左转 90 度。
    
- `(backward 100)` 或 `(bk 100)`：后退 100 单位。
    

这些命令可与 Scheme 语言特性结合使用。

### 5.2 绘制三角形

定义绘制一个三角形的过程：

```scheme

(define (triangle side)
  (repeat (lambda () (fd side) (rt 120)) 3))
```
`repeat` 过程见 4.2 节。

### 5.3 通用三角形过程（支持自定义边绘制）

```scheme

(define (triangle fn)
  (repeat fn 3)
  (rt 120))  ; 每次画完一条边后右转 120 度
```
使用时传入一个绘制边的过程，例如：

```scheme

(triangle (lambda () (fd 100)))
```
### 5.4 Sierpinski 三角形

谢尔宾斯基三角形（Sierpinski triangle）是一种分形，由递归绘制。

**算法**：

- 基础情况：画一个实心三角形（或仅画边框）。
    
- 递归情况：在三个角分别绘制更小的谢尔宾斯基三角形。
    

```scheme

(define (sierpinski d)
  (if (= d 0)
      (fd 100)                 ; 基础情况：画一条边（实际需要闭合，这里简化）
      (begin
        (sierpinski (- d 1))
        (lt 60) (fd 100) (rt 60)
        (sierpinski (- d 1))
        (rt 60) (fd 100) (lt 60)
        (sierpinski (- d 1)))))
;; 更标准的实现（带三角形闭合）：
(define (sierpinski d)
  (if (= d 0)
      (fd 100)
      (begin
        (sierpinski (- d 1))
        (rt 60) (fd 100) (lt 60)
        (sierpinski (- d 1))
        (lt 60) (fd 100) (rt 60)
        (sierpinski (- d 1)))))
```
**实际绘制**：需要初始化位置和方向，例如：

```scheme

(define (draw)
  (pen-up) (backward 200) (right 90) (forward 100) (left 90) (pen-down)
  (sierpinski 4))
```
**特点**：

- 每个递归深度都会产生大量重复绘制。
    
- 完成后海龟通常返回起始位置。
    

---

## 六、知识点总结表

|知识点|核心内容|考试重点 / 易混淆点|难度|
|---|---|---|---|
|**Scheme 表达式**|原始表达式（自求值）与组合表达式（括号包围）|数字、布尔值、符号；调用表达式求值规则|⭐⭐|
|**特殊形式**|`if`, `and`, `or`, `define`, `lambda`, `cond`, `begin`, `let`|`if` 必须三个子表达式；`cond` 的 `else` 子句；`let` 作用域|⭐⭐⭐|
|**谓词命名约定**|以 `?` 结尾的过程返回布尔值（如 `number?`）|与 Python 的 `is_*` 风格类比|⭐|
|**递归**|巴比伦方法求平方根，`repeat` 过程|终止条件（`good-enough?`）；递归调用自身|⭐⭐⭐|
|**Lambda**|匿名过程，可作为值传递|与 `define` 定义的过程等价|⭐⭐|
|**海龟绘图**|`fd`, `rt`, `lt`, `bk` 等命令|结合递归绘制分形（Sierpinski 三角形）|⭐⭐⭐|
|**环境模型**|符号在环境中绑定，与 Python 类似|`define` 在全局帧绑定，`let` 创建局部帧|⭐⭐|

---

## 七、自测题

### 1. 基础表达式求值

写出以下 Scheme 表达式的求值结果：

- `(+ 1 2 3)`
    
- `(*)`
    
- `(quotient 10 3)`
    
- `(number? #t)`
    
- `(zero? (- 5 5))`
    

ans:
- `6`
    
- `1`
    
- `3`（整数除法）
    
- `#f`（`#t` 不是数字）
    
- `#t`
    


### 2. `if` 表达式

以下表达式的结果是什么？

```scheme

(if (and #t (> 3 2)) 'good 'bad)
```
 
`good`，因为 `and` 返回最后一个真值 `#t`，`if` 取 consequent。

 

### 3. `cond` 与 `else`

编写一个过程 `(classify n)`，根据整数 `n` 返回：

- 正数：`'positive`
    
- 负数：`'negative`
    
- 零：`'zero`
    

```scheme

(define (classify n)
  (cond
    ((> n 0) 'positive)
    ((< n 0) 'negative)
    (else 'zero)))
```

### 4. `let` 作用域

以下表达式求值结果是什么？

```scheme

(let ((x 5))
  (let ((x 10))
    x))
```

`10`。内层 `let` 绑定遮蔽了外层绑定。


### 5. 递归过程

定义递归过程 `(power b n)` 计算 `b^n`（`n` 为非负整数）。

```scheme

(define (power b n)
  (if (= n 0)
      1
      (* b (power b (- n 1)))))
```

### 6. 绘图思考题

在 Sierpinski 三角形递归中，如果深度为 0，应该画什么？为什么？

<details> <summary>提示</summary>

深度为 0 时通常画一条边或一个实心三角形。这代表了分形的最小单元。在递归中，它是终止条件。

</details>

---

## 八、总结与展望

本节课我们快速领略了 Scheme 语言的魅力。你学会了：

- Scheme 的基本语法和求值规则。
    
- 特殊形式如何控制流程和绑定。
    
- 递归编程的技巧（平方根计算、重复执行）。
    
- 使用 `lambda` 创建匿名过程。
    
- 海龟绘图的基础命令和递归绘制 Sierpinski 三角形。
    

Scheme 的简洁性和表达能力使其成为学习函数式编程和编程语言原理的理想选择。在后续课程中，你将用 Scheme 实现一个解释器，进一步理解编程语言的运行机制。