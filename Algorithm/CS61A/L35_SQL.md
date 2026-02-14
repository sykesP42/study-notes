# 📘 Lecture 35：SQL

> 数据库管理系统（DBMS）是现代软件基础设施的核心，而 SQL 是操作数据库的标准语言。与 Python、Scheme 等命令式语言不同，SQL 是一种**声明式**语言：你只需要描述**想要什么结果**，而不需要指定如何得到它。本节课将带你从零开始掌握 SQL 的基础语法，学会如何创建表、查询数据、过滤行、排序结果以及进行简单的算术运算。

---

## 一、数据库与 SQL 概述

### 1.1 数据库管理系统（DBMS）

数据库以结构化的**表格**形式存储数据。一个表由**行**（记录）和**列**（字段）组成，每一列有名称和数据类型。

**示例表：cities**

| Latitude | Longitude | Name        |
| -------- | --------- | ----------- |
| 38       | 122       | Berkeley    |
| 42       | 71        | Cambridge   |
| 45       | 93        | Minneapolis |

- 每一行是一条记录，描述一个城市。
    
- 每一列是一个属性（如纬度、经度、城市名）。
    

DBMS 负责高效地存储、检索和管理这些数据，支持复杂的查询和事务处理。企业和组织投入大量资金建设 DBMS，同时也有优秀的开源系统（如 SQLite、PostgreSQL）。

### 1.2 SQL 的声明式特性

SQL（Structured Query Language）是一种**声明式编程语言**。你只需要描述期望的结果，DBMS 会自己决定如何执行（例如是否使用索引、如何连接表等）。

**对比命令式语言（Python/Scheme）**：

- 命令式：描述**计算过程**，解释器严格按照指令执行。
    
- 声明式：描述**最终结果**，解释器（DBMS）优化执行路径。
    

#### 示例：获取西海岸城市

```sql

-- SQL 声明式
SELECT name FROM cities WHERE longitude >= 115;
```

```python

# Python 命令式
west_coast = []
for row in cities:
    if row['longitude'] >= 115:
        west_coast.append(row['name'])
```
SQL 让你专注于“要什么”，而不是“怎么找”。

---

## 二、SQL 基础语法

### 2.1 SELECT 语句：从字面量创建表

最简单的 `SELECT` 语句可以创建一个只包含字面量的单行表。

```sql

SELECT 38 AS latitude, 122 AS longitude, 'Berkeley' AS name;
```
**结果**：

|latitude|longitude|name|
|---|---|---|
|38|122|Berkeley|

- `SELECT` 后跟逗号分隔的列描述，每个描述可以是表达式，后跟可选的 `AS` 列名。
    
- 如果没有 `AS`，列名由系统自动生成（通常就是表达式本身）。
    
- 语句必须以分号 `;` 结束。
    

#### 多行表：使用 `UNION`

`UNION` 可以将多个 `SELECT` 的结果合并成一个表。要求：

- 每个 `SELECT` 必须有相同数量的列。
    
- 对应列的数据类型必须兼容。
    
- 列名由第一个 `SELECT` 决定。
    

```sql

SELECT 38 AS latitude, 122 AS longitude, 'Berkeley' AS name
UNION
SELECT 42, 71, 'Cambridge'
UNION
SELECT 45, 93, 'Minneapolis';
```
**结果**：

|latitude|longitude|name|
|---|---|---|
|38|122|Berkeley|
|42|71|Cambridge|
|45|93|Minneapolis|

> **注意**：`UNION` 默认会去除重复行。如果想保留所有行（包括重复），可以使用 `UNION ALL`。

### 2.2 命名表：`CREATE TABLE`

`SELECT` 产生的结果表是临时的，不会持久化。要给表命名以便后续使用，需要用 `CREATE TABLE`。

```sql

CREATE TABLE cities AS
  SELECT 38 AS latitude, 122 AS longitude, 'Berkeley' AS name
  UNION
  SELECT 42, 71, 'Cambridge'
  UNION
  SELECT 45, 93, 'Minneapolis';
```
之后就可以通过名称 `cities` 引用这个表。

---

## 三、查询现有表

### 3.1 基本投影：`SELECT ... FROM`

`SELECT` 语句可以从现有表中选取列，甚至可以添加新的计算列。

```sql

SELECT name, latitude FROM cities;
```
**结果**：

|name|latitude|
|---|---|
|Berkeley|38|
|Cambridge|42|
|Minneapolis|45|

- 使用 `*` 表示所有列：`SELECT * FROM cities;`
    

### 3.2 过滤行：`WHERE`

`WHERE` 子句用于筛选满足条件的行。

```sql

SELECT name FROM cities WHERE longitude >= 115;
```
返回西海岸城市（假设经度大于等于115为西海岸）。

### 3.3 排序：`ORDER BY`

`ORDER BY` 指定结果行的排序方式。默认升序（`ASC`），可指定降序（`DESC`）。

```sql

SELECT name, latitude FROM cities ORDER BY latitude DESC;
```
### 3.4 字符串比较

在 SQL 中，字符串比较默认按字典序（字母顺序）。

```sql

SELECT name FROM cities WHERE name < 'C';
```
会返回 `Berkeley`（因为 'B' < 'C'），而 'Cambridge' 和 'Minneapolis' 不会出现。

---

## 四、实战示例

### 4.1 创建并查询 `parents` 表

```sql

CREATE TABLE parents AS
  SELECT 'abraham' AS parent, 'barack' AS child UNION
  SELECT 'abraham', 'clinton'         UNION
  SELECT 'fillmore', 'abraham'        UNION
  SELECT 'fillmore', 'clinton'        UNION
  SELECT 'fillmore', 'barack'         UNION
  SELECT 'eisenhower', 'fillmore';
```
#### 查询所有列

```sql

SELECT * FROM parents;
```
#### 找出所有父辈为 'abraham' 的孩子

```sql

SELECT child FROM parents WHERE parent = 'abraham';
-- 结果: barack, clinton
```
#### 找出所有父辈名字大于孩子名字的记录（按字母序）

```sql

SELECT parent FROM parents WHERE parent > child;
-- 结果: fillmore 出现两次（因为 'fillmore' > 'abraham' 且 > 'clinton' 且 > 'barack'）
```
注意：`WHERE parent > child` 是基于字符串比较，'fillmore' > 'abraham' 为真。

### 4.2 算术运算

SELECT 中可以使用算术表达式。

#### 示例：缆车人数计算

假设有一个表 `rides`，记录缆车每次的乘客数，我们可以计算总人数、平均数等。但这里笔记中有一个简单示例：计算 `one + two + four + eight` 的值。

```sql

CREATE TABLE ints AS
  SELECT 'one' AS word, 1 AS one, 0 AS two, 0 AS four, 0 AS eight UNION
  SELECT 'two', 0, 2, 0, 0 UNION
  SELECT 'four', 0, 0, 4, 0 UNION
  SELECT 'eight', 0, 0, 0, 8;
```
现在，我们可以计算每个单词对应的数值（各列之和）：

```sql

SELECT word, one + two + four + eight AS value FROM ints;
```
**结果**：

| word  | value |
| ----- | ----- |
| one   | 1     |
| two   | 2     |
| four  | 4     |
| eight | 8     |

#### 进阶：选出幂次为 2 的单词

要求选出那些“恰好是 2 的幂”的单词，即数值为 1、2、4、8 等。我们可以用算术表达式检查：`one + two/2 + four/4 + eight/8 = 1`。

```sql

SELECT word FROM ints WHERE one + two/2 + four/4 + eight/8 = 1;
```
**原理**：每个幂次列的值除以对应的系数（1/2/4/8）后，只有真正的幂次列会贡献 1，其他列贡献 0。总和为 1 意味着该单词对应的列恰好是 1，其他为 0。所以结果正好是 `one`, `two`, `four`, `eight`。

---

## 五、SQL 标准与方言

SQL 是 ANSI 和 ISO 标准，但不同 DBMS（如 SQLite、MySQL、PostgreSQL）会有自己的方言和扩展功能。尽管如此，核心语法（如 `SELECT`、`CREATE TABLE`、`WHERE`、`ORDER BY`）基本通用。

本课程使用的 DBMS 是 **SQLite**，它轻量级、易于学习，非常适合教学。你可以通过以下方式实践：

- 在线工具：[sql.js](http://kripken.github.io/sql.js/GUI/)
    
- 本地安装：从 [SQLite 官网](https://sqlite.org/download.html) 下载命令行工具。
    

---

## 六、知识点总结表

|知识点|核心内容|考试重点 / 易混淆点|难度|
|---|---|---|---|
|**数据库表结构**|表由行（记录）和列（字段）组成，每列有名称和数据类型|区分行和列；列定义的数据类型约束|⭐|
|**声明式编程**|描述“想要什么”，DBMS 决定“如何做”|与命令式语言的对比|⭐⭐|
|**SELECT 字面量**|`SELECT expr AS name` 创建单行表，`UNION` 合并多行|列数、类型必须一致；列名取自第一个 SELECT|⭐⭐|
|**CREATE TABLE**|`CREATE TABLE name AS select` 持久化表|与 `SELECT` 的临时结果区别|⭐|
|**查询投影**|`SELECT 列名 FROM 表名` 选择指定列|`*` 表示所有列；可包含算术表达式|⭐|
|**WHERE 过滤**|`WHERE 条件` 筛选行|字符串按字典序比较；条件可以是任意布尔表达式|⭐⭐|
|**ORDER BY 排序**|`ORDER BY 列 [ASC\|DESC]`|默认升序；可指定多列|⭐|
|**算术运算**|在 SELECT 中可以使用 `+ - * /` 等|注意整数除法的行为（SQLite 中整数除法结果为整数）|⭐⭐|
|**UNION 与 UNION ALL**|`UNION` 去重，`UNION ALL` 保留所有|性能差异；去重可能改变结果顺序|⭐⭐|

---

## 七、自测题

### 1. 创建表与合并

创建一个名为 `fruits` 的表，包含两列：`name` 和 `price`。添加三行数据：苹果 3.5 元，香蕉 2.0 元，樱桃 5.0 元。写出 SQL 语句。

```sql

CREATE TABLE fruits AS
  SELECT 'apple' AS name, 3.5 AS price
  UNION
  SELECT 'banana', 2.0
  UNION
  SELECT 'cherry', 5.0;
```

### 2. 查询练习

基于上面的 `fruits` 表，查询所有价格大于 3 元的水果名称，并按价格降序排列。

```sql

SELECT name FROM fruits WHERE price > 3 ORDER BY price DESC;

```

### 3. 字符串比较

给定表 `words` 包含列 `word`，值为 `'apple'`、`'banana'`、`'cherry'`。写出查询，返回所有按字母序排在 `'banana'` 之前的单词。

```sql

SELECT word FROM words WHERE word < 'banana';
-- 结果: apple

```

### 4. 算术与列计算

创建表 `numbers`，包含列 `a` 和 `b`，插入两行 (2, 3) 和 (5, 7)。查询每行的 `a`、`b` 以及它们的和与乘积。

```sql

CREATE TABLE numbers AS
  SELECT 2 AS a, 3 AS b
  UNION
  SELECT 5, 7;
SELECT a, b, a+b AS sum, a*b AS product FROM numbers;
```

### 5. UNION 与 UNION ALL

给定两个单列表 `s1` 包含值 1, 2，`s2` 包含值 2, 3。写出使用 `UNION` 和 `UNION ALL` 的查询，并说明结果差异。

```sql

SELECT * FROM s1
UNION
SELECT * FROM s2;
-- 结果: 1, 2, 3 (去重)
SELECT * FROM s1
UNION ALL
SELECT * FROM s2;
-- 结果: 1, 2, 2, 3 (保留重复)
```

### 6. 思考题

为什么 SQL 被称为“声明式”语言？它与 Python 的函数式编程（如 `map`、`filter`）有何异同？


- SQL 中你只需描述结果的结构和条件，DBMS 负责生成执行计划。
    
- Python 的 `map`/`filter` 仍然是命令式的，你需要组合这些函数并控制数据流。但函数式编程的一些思想（如避免副作用）与声明式有相通之处。
    
- SQL 更接近关系代数，而 Python 更接近通用计算。
    



---

## 八、总结与展望

本节课我们迈入了数据库的世界，学习了 SQL 的核心基础：

- 数据库表的结构和 DBMS 的作用。
    
- 声明式编程思想及其与命令式语言的区别。
    
- `SELECT` 语句从字面量创建表，`CREATE TABLE` 命名表。
    
- 查询现有表：投影、过滤、排序。
    
- 算术运算和字符串比较。
    
- 通过多个示例实践了上述知识。
    

SQL 是数据领域的通用语言，掌握它将为你后续学习数据分析、后端开发打下坚实基础。下一节课我们将深入更复杂的查询，如多表连接、聚合函数和子查询。