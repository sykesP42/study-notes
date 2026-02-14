# 📘 Lecture 37：聚合（Aggregation）—— 完全自学指南

> 到目前为止，我们学习的 SQL 查询都是基于单行操作的：每一行独立地参与计算，输出也是每行一个结果。但现实中我们常常需要对一组数据进行汇总计算，比如求总和、平均值、最大值等。这就是**聚合**（Aggregation）的功能。本节课将带你掌握 SQL 中的聚合函数、分组聚合以及如何筛选分组结果，使你能够从数据中提取更高层次的统计信息。

---

## 一、聚合函数基础

### 1.1 什么是聚合函数？

**聚合函数**（Aggregate Function）接收多行的一个表达式（通常是某列），并返回一个单一的汇总值。常见的聚合函数有：

- `MAX(expr)`：返回组内 expr 的最大值。
    
- `MIN(expr)`：返回组内 expr 的最小值。
    
- `SUM(expr)`：返回组内 expr 的总和（expr 必须是数值类型）。
    
- `AVG(expr)`：返回组内 expr 的平均值。
    
- `COUNT(expr)`：返回组内 expr 非 NULL 值的行数。`COUNT(*)` 返回总行数（包括 NULL）。
    

### 1.2 示例数据：动物表

我们先创建一个示例表，用于后续所有查询。

```sql

CREATE TABLE animals AS
  SELECT 'dog' AS kind, 4 AS legs, 20 AS weight UNION
  SELECT 'cat', 4, 10 UNION
  SELECT 'ferret', 4, 10 UNION
  SELECT 'parrot', 2, 6 UNION
  SELECT 'penguin', 2, 10 UNION
  SELECT 't-rex', 2, 12000;
```
表内容：

|kind|legs|weight|
|---|---|---|
|dog|4|20|
|cat|4|10|
|ferret|4|10|
|parrot|2|6|
|penguin|2|10|
|t-rex|2|12000|

### 1.3 基本聚合查询

#### 求最大腿数

```sql

SELECT MAX(legs) FROM animals;
-- 结果: 4
```
#### 求所有动物总重量

```sql

SELECT SUM(weight) FROM animals;
-- 结果: 20+10+10+6+10+12000 = 12056
```
#### 聚合表达式可以是计算列

```sql

SELECT MAX(legs - weight) + 5 FROM animals;
```
先计算每行的 `legs - weight`：

- dog: 4-20 = -16
    
- cat: 4-10 = -6
    
- ferret: 4-10 = -6
    
- parrot: 2-6 = -4
    
- penguin: 2-10 = -8
    
- t-rex: 2-12000 = -11998  
    最大值是 -4，加 5 得 1。
    

### 1.4 聚合与 WHERE 结合

`WHERE` 子句在聚合之前执行，只对过滤后的行进行聚合。

```sql

SELECT MAX(weight) FROM animals WHERE kind != 't-rex';
-- 过滤掉 t-rex 后，最大重量是 20
```
### 1.5 COUNT 与 DISTINCT

`COUNT(*)` 统计所有行数，包括重复行和 NULL。

```sql

SELECT COUNT(*) FROM animals;   -- 6
```
`COUNT(列名)` 统计该列非 NULL 值的个数。若需统计不同值的个数，使用 `COUNT(DISTINCT 列名)`。

```sql

SELECT COUNT(DISTINCT legs) FROM animals;   -- 2 (4 和 2)
SELECT COUNT(DISTINCT weight) FROM animals; -- 4 (20,10,6,12000 有重复 10，去重后四个不同值)
```
---

## 二、混合聚合函数与普通列

### 2.1 问题描述

有时我们希望在输出中同时包含聚合结果和某个普通列的值。例如：“找出最重的动物及其名字”。

直觉上可能写成：

```sql

SELECT MAX(weight), kind FROM animals;
```
这个查询在某些数据库中是合法的，但结果中的 `kind` 取哪一行的值？标准 SQL 规定：如果 `SELECT` 中既有聚合函数又有非聚合列，且没有 `GROUP BY`，则非聚合列的值来自**聚合计算时选中的某一行**，但这个行为是不确定的（取决于数据库实现）。在 SQLite 中，它会返回某一行（通常是第一行或最后一行），但具体哪一行不可预测。

**示例**：

```sql

SELECT MAX(weight), kind FROM animals;
-- 可能返回 (12000, 't-rex')，因为 t-rex 是最大重量对应的行。
-- 但严格来说，这种行为不保证，其他数据库可能报错。
```
### 2.2 有意义的情况

如果非聚合列的值与聚合结果来自同一行（即该行就是产生聚合值的行），那么结果是有意义的。例如 `MAX(weight)` 对应的 `kind` 确实是最大重量动物的种类。但在更复杂的情况下（如 `AVG(weight)` 搭配 `kind`），平均值并不对应任何特定行，此时的 `kind` 值就是随机的、无意义的。

```sql

SELECT AVG(weight), kind FROM animals;
-- 平均重量 2009.33，kind 可能取 't-rex' 或其他，毫无意义。
```
**最佳实践**：除非你确切知道结果中非聚合列的值与聚合结果对应（如与 `MAX`/`MIN` 搭配且该列来自同一行），否则**不要混合使用聚合函数和非聚合列**。若想获取最大值对应的其他属性，通常用子查询或排序+限制行数更安全。

### 2.3 正确获取最大值对应行的其他列

方法一：子查询

```sql

SELECT kind, weight FROM animals
WHERE weight = (SELECT MAX(weight) FROM animals);
```
方法二：排序 + LIMIT

```sql

SELECT kind, weight FROM animals ORDER BY weight DESC LIMIT 1;
```
---

## 三、分组聚合：GROUP BY

### 3.1 为什么需要分组？

有时候我们不仅要对整个表进行聚合，还想按照某个维度（如腿数）分别统计。例如：“每种腿数的动物各有多少只？”或“每种腿数的最大重量是多少？”

这就需要**分组聚合**：将行按某个表达式（通常是列）的值分组，每组分别进行聚合计算。

### 3.2 基本语法

```sql

SELECT 分组列, 聚合函数(列) FROM 表 GROUP BY 分组列;
```
- 分组列可以是列名、多个列、或表达式。
    
- `SELECT` 中的列要么是分组列，要么是聚合函数列。
    

### 3.3 示例

#### 按腿数分组，计算每组的最大重量

```sql

SELECT legs, MAX(weight) FROM animals GROUP BY legs;
```
**结果**：

|legs|MAX(weight)|
|---|---|
|4|20|
|2|12000|

#### 按腿数和重量分组，相当于去重

```sql

SELECT legs, weight FROM animals GROUP BY legs, weight;
```
这实际上会返回所有不重复的 (legs, weight) 组合，共 5 行（因为 (4,10) 出现了两次，去重后一次）。

### 3.4 多列分组与表达式分组

分组列也可以是表达式，例如按“每条腿的平均负重”（weight/legs）分组：

```sql

SELECT weight/legs, COUNT(*) FROM animals GROUP BY weight/legs;
```
结果：

|weight/legs|COUNT(*)|
|---|---|
|5|1|(20/4=5, dog)|
|2.5|2|(10/4=2.5, cat & ferret)|
|3|1|(6/2=3, parrot)|
|5|1|(10/2=5, penguin) 注意这里也有 5，但与前面的 5 不同组？实际上 5 是浮点数，两个 5 是否被视为同一组？取决于浮点精度，在 SQLite 中可能被视为相同。我们稍后处理。|
|6000|1|(12000/2=6000, t-rex)|

---

## 四、筛选分组：HAVING

### 4.1 为什么需要 HAVING？

`WHERE` 子句在分组之前过滤行，而分组之后我们可能还需要过滤掉某些分组（例如只保留出现次数大于 1 的分组）。这时就需要 `HAVING` 子句。

### 4.2 语法

```sql

SELECT 分组列, 聚合函数
FROM 表
GROUP BY 分组列
HAVING 条件;
```
`HAVING` 中的条件可以使用聚合函数，而 `WHERE` 中不能直接使用聚合函数（除非在子查询中）。

### 4.3 示例

#### 找出出现次数超过 1 的 (weight/legs) 比值

```sql

SELECT weight/legs, COUNT(*) FROM animals
GROUP BY weight/legs
HAVING COUNT(*) > 1;
```
结果（假设浮点数精度足够）：

|weight/legs|COUNT(*)|
|---|---|
|2.5|2|

因为只有 (4,10) 的 cat 和 ferret 的比值是 2.5，出现两次。

#### 如果比值 5 出现了两次（dog 和 penguin），也会被选出。

但注意，dog 的 20/4=5，penguin 的 10/2=5，所以 5 也出现了两次。但实际上 dog 和 penguin 的比值都是 5，所以应该有两个 5。但根据浮点数，它们可能被识别为同一值。所以结果应该包含 (5,2) 和 (2.5,2)。

---

## 五、知识点总结表

|知识点|核心内容|考试重点 / 易混淆点|难度|
|---|---|---|---|
|**聚合函数**|`MAX`, `MIN`, `SUM`, `AVG`, `COUNT` 对多行计算单一结果|`COUNT(*)` 统计所有行，`COUNT(列)` 统计非 NULL 行|⭐|
|**DISTINCT**|`COUNT(DISTINCT 列)` 统计唯一值|也可用于其他聚合函数？某些 DB 支持 `SUM(DISTINCT col)`|⭐⭐|
|**混合聚合与非聚合列**|无 `GROUP BY` 时混用可能产生无意义结果|应避免，或用子查询、`ORDER BY + LIMIT` 替代|⭐⭐|
|**GROUP BY**|按指定列分组，每组分别聚合|`SELECT` 中只能出现分组列或聚合列|⭐⭐|
|**HAVING**|对分组结果进行过滤，可使用聚合函数|`WHERE` 在分组前过滤行，`HAVING` 在分组后过滤组|⭐⭐|
|**多列分组**|可指定多个列或表达式分组|分组列的顺序影响结果排序，但不影响分组逻辑|⭐|

---

## 六、自测题

### 1. 聚合基础

基于 `animals` 表，写出以下查询的结果：

- `SELECT MIN(weight) FROM animals;`
    
- `SELECT SUM(legs) FROM animals WHERE kind != 't-rex';`
    
- `SELECT AVG(weight) FROM animals;`
    
- `SELECT COUNT(DISTINCT weight) FROM animals;`
    

ANS:
- `6`（parrot 的体重）
    
- 所有非 t-rex 的腿数之和：dog(4)+cat(4)+ferret(4)+parrot(2)+penguin(2) = 16
    
- 总重量 12056 / 6 ≈ 2009.333...
    
- 不同体重值：20,10,6,12000 共 4 个
    


### 2. 混合列问题

假设运行 `SELECT MAX(weight), kind FROM animals;`，在 SQLite 中可能得到什么结果？为什么这不安全？


- 可能得到 (12000, 't-rex')，因为 t-rex 是最大体重对应的行。但标准 SQL 中这是非法的，因为 `kind` 没有出现在 `GROUP BY` 中，也不是聚合列。SQLite 会返回某一行，但不保证总是正确。应改用子查询或 `ORDER BY LIMIT` 来获取最大值对应的其他列。


### 3. 分组查询

写出查询：按腿数分组，计算每组的平均体重和动物数量。

```sql

SELECT legs, AVG(weight), COUNT(*) FROM animals GROUP BY legs;
```
结果：

- legs=4: 平均体重 (20+10+10)/3 ≈ 13.33, 数量 3
    
- legs=2: 平均体重 (6+10+12000)/3 ≈ 4005.33, 数量 3
    


### 4. HAVING 练习

基于 `animals` 表，找出平均体重大于 10 的腿数组。

```sql

SELECT legs, AVG(weight) FROM animals GROUP BY legs HAVING AVG(weight) > 10;
```
结果：

- legs=2 组平均体重 4005.33 >10，所以返回 legs=2 及其平均体重。
    


### 5. 复杂分组

我们希望找出所有出现次数超过 1 次的 (legs, weight) 组合。写出 SQL。

```sql

SELECT legs, weight, COUNT(*) FROM animals GROUP BY legs, weight HAVING COUNT(*) > 1;
```
结果：

- (4,10) 出现两次（cat 和 ferret），返回 (4,10,2)
    



### 6. 思考题

为什么 `WHERE` 不能直接使用聚合函数？例如 `WHERE SUM(weight) > 100` 为什么不允许？


`WHERE` 子句是在分组之前对每一行进行过滤的。此时聚合函数还没有计算，因此无法引用。`HAVING` 是在分组之后对每组进行过滤，此时每组已计算出聚合值，所以可以用。



---

## 七、总结与展望

本节课我们学习了 SQL 中至关重要的**聚合**操作。你掌握了：

- 使用 `MAX`, `MIN`, `SUM`, `AVG`, `COUNT` 对整表或分组进行汇总。
    
- 理解并避免混合聚合与非聚合列时的陷阱。
    
- 用 `GROUP BY` 实现分组统计。
    
- 用 `HAVING` 筛选分组结果。
    

聚合操作是数据分析的基石，能够帮助我们快速从大量数据中提取洞察。在后续的课程中，你将学习更高级的 SQL 特性，如子查询、窗口函数等，进一步提升数据处理能力。