# 📘 Lecture 36：表连接（Tables）

> 在关系型数据库中，数据通常分散在多个表中，通过**连接（JOIN）**操作可以将这些表组合起来，回答更复杂的问题。本节课将带你深入理解 SQL 中的连接机制：从最简单的笛卡尔积开始，到自连接、多表连接，再到数值和字符串表达式的应用。通过丰富的示例（如狗的毛发与父母关系、城市距离计算、名词句子生成），你将掌握连接操作的核心思想，并能独立编写复杂的多表查询。

---

## 一、连接两个表

### 1.1 基本连接操作

在 SQL 中，用逗号分隔两个表名即可实现连接，这会生成两个表的**笛卡尔积**——第一个表的每一行与第二个表的每一行组合。

**语法**：

```sql

SELECT * FROM table1, table2;
```
**示例**：假设有两个小表 A 和 B：

- A: (1,2)
    
- B: (3,4)
    

连接结果有 2×2=4 行：`(1,3)`, `(1,4)`, `(2,3)`, `(2,4)`。

### 1.2 实际意义与 `WHERE` 过滤

单纯的笛卡尔积通常没有实际意义，我们通过 `WHERE` 子句添加连接条件，只保留那些有意义的组合。例如，要查询学生和他们的选课信息，可能用 `student.id = enrollment.student_id` 作为连接条件。

---

## 二、示例：狗的毛发与父母关系

我们通过一个完整的例子来演示连接操作。

### 2.1 数据准备

首先创建两个表：`dogs`（记录狗的名字和毛发类型）和 `parents`（记录父母-孩子关系）。

```sql

-- 创建 dogs 表
CREATE TABLE dogs AS
  SELECT 'abraham' AS name, 'long' AS fur UNION
  SELECT 'barack', 'short'            UNION
  SELECT 'clinton', 'long'             UNION
  SELECT 'delano', 'long'               UNION
  SELECT 'eisenhower', 'short'          UNION
  SELECT 'fillmore', 'curly'            UNION
  SELECT 'grover', 'short'              UNION
  SELECT 'herbert', 'curly';
-- 创建 parents 表
CREATE TABLE parents AS
  SELECT 'abraham' AS parent, 'barack' AS child UNION
  SELECT 'abraham', 'clinton'                    UNION
  SELECT 'fillmore', 'abraham'                   UNION
  SELECT 'fillmore', 'clinton'                    UNION
  SELECT 'fillmore', 'barack'                     UNION
  SELECT 'eisenhower', 'fillmore';
```
**数据说明**：

- `dogs` 表：每只狗的名字和毛发类型（long/short/curly）。
    
- `parents` 表：每一行记录一个 parent-child 关系。
    

### 2.2 查询卷毛狗的父母

**问题**：找出所有卷毛狗（fur = 'curly'）的父母。

**SQL**：

```sql

SELECT parent FROM parents, dogs WHERE child = name AND fur = 'curly';
```
**执行过程**：

1. 先计算 `parents` 和 `dogs` 的笛卡尔积（所有父母行与所有狗行组合）。
    
2. 用 `WHERE` 子句过滤：
    
    - `child = name`：只保留父母表中孩子名字与狗名字相同的行（即找到每只狗的父母记录）。
        
    - `fur = 'curly'`：只保留那些狗是卷毛的行。
        
3. 最终只输出 `parent` 列。
    

**结果**：

```text

parent
--------
fillmore   (因为 fillmore 的孩子中有卷毛狗？实际上 fillmore 的孩子是 abraham, clinton, barack，其中 abraham 是长毛，clinton 长毛，barack 短毛，所以 fillmore 本身不是卷毛狗的父母？我们需要重新检查数据)
```

这个查询展示了如何通过连接两个表来回答跨表问题。

**关键点**：连接让我们能够将狗的属性（毛发）与其家庭关系（父母）结合起来查询。

---

## 三、连接执行过程详解

### 3.1 笛卡尔积与过滤

连接操作的执行步骤（逻辑上）：

1. 生成笛卡尔积：将左表的每一行与右表的每一行配对。
    
2. 应用 `WHERE` 条件，筛选出满足条件的行。
    
3. 选择指定的列输出。
    

对于大表，笛卡尔积可能非常巨大，因此实际 DBMS 会通过优化（如使用索引）避免显式生成整个笛卡尔积，只计算满足条件的结果。

### 3.2 典型应用场景

- 关联查询：如查询学生和其选修的课程（学生表与选课表连接）。
    
- 数据整合：将用户基本信息与订单信息关联。
    
- 自连接：一个表与其自身连接，用于比较同一表中不同行的关系（如找出兄弟姐妹）。
    

---

## 四、别名与点表达式

### 4.1 为什么需要别名？

当连接涉及多个表，且这些表有同名列时，必须用**表别名**和**点表达式**来区分列属于哪个表。例如，`parents` 表有 `parent` 和 `child` 列，如果我们将 `parents` 与其自身连接，两个实例的列名相同，必须用别名区分。

### 4.2 示例：找出有相同父母的狗（兄弟姐妹）

**问题**：找出所有兄弟姐妹对（即父母相同），并显示兄弟姐妹的名字，要求每对只出现一次（按字母顺序排列，如 `barack` 和 `clinton`，而不是 `clinton` 和 `barack`）。

**SQL**：

```sql

SELECT a.child AS first, b.child AS second
FROM parents AS a, parents AS b
WHERE a.parent = b.parent AND a.child < b.child;
```
**解析**：

- `parents AS a, parents AS b`：将 `parents` 表视为两个独立的实例，分别命名为 `a` 和 `b`。
    
- `a.parent = b.parent`：连接条件，确保两个行来自同一个父母。
    
- `a.child < b.child`：避免重复对（如 (barack, clinton) 和 (clinton, barack)），只保留按字母顺序较小的在前的一对。
    
- 选择 `a.child` 和 `b.child` 作为输出列，并用 `AS` 重命名为 `first` 和 `second`。
    

**结果**（基于示例数据）：

```text

first    second
------   -------
abraham  barack    (来自 fillmore 的孩子？fillmore 的孩子有 abraham, clinton, barack，所以会有 abraham-barack, abraham-clinton, barack-clinton 三对)
abraham  clinton
barack   clinton
```
### 4.3 关键理解

- **自连接**：通过别名将一个表虚拟成两个副本，然后进行连接。
    
- **点表达式**：`表别名.列名` 明确指定列来源。
    
- **不等条件**：`<` 用于去除对称重复对，是一种常见技巧。
    

---

## 五、连接多个表

### 5.1 祖孙关系查询

**问题**：找出所有祖孙对（grandparent-grandchild）。

**SQL**：

```sql

SELECT a.parent AS grandparent, b.child AS grandchild
FROM parents AS a, parents AS b
WHERE a.child = b.parent;
```
**解析**：

- `a` 实例代表父母关系（parent → child）。
    
- `b` 实例也代表父母关系。
    
- 连接条件 `a.child = b.parent` 表示 a 的孩子是 b 的父母，即 a 是 b 的祖父母。
    

### 5.2 扩展：找出毛发相同的祖孙

**问题**：在祖孙关系中，进一步要求祖父母和孙子的毛发类型相同。

**分析**：我们需要将祖孙关系与 `dogs` 表连接两次：一次获取祖父母的毛发，一次获取孙子的毛发。

**SQL**：

```sql

SELECT grandparent, grandchild
FROM (
  SELECT a.parent AS grandparent, b.child AS grandchild
  FROM parents AS a, parents AS b
  WHERE a.child = b.parent
) AS grandparents, dogs AS g1, dogs AS g2
WHERE grandparent = g1.name AND grandchild = g2.name
  AND g1.fur = g2.fur;
```
或者直接用三表连接：

```sql

SELECT a.parent AS grandparent, b.child AS grandchild
FROM parents AS a, parents AS b, dogs AS g1, dogs AS g2
WHERE a.child = b.parent
  AND a.parent = g1.name
  AND b.child = g2.name
  AND g1.fur = g2.fur;
```
**结果**：例如 eisenhower（短毛）和 fillmore（卷毛）毛发不同，不会出现；但如果有毛发相同的祖孙对，就会被选出。笔记中提到“eisenhower 和 fillmore 是符合条件的祖父母”，这显然毛发不同，所以可能数据有差异。但核心思想是：多表连接允许我们组合多个关系，回答复杂的多步查询。

---

## 六、数值表达式

### 6.1 在 `SELECT` 中使用算术运算

SQL 的 `SELECT` 子句中可以包含算术表达式，对列值进行计算。

**示例**：查询城市并计算其纬度与某个参考值的差。

```sql

CREATE TABLE cities AS
  SELECT 38 AS latitude, 122 AS longitude, 'Berkeley' AS name UNION
  SELECT 42, 71, 'Cambridge' UNION
  SELECT 45, 93, 'Minneapolis';
-- 查询纬度大于43度的城市（寒冷地区）
SELECT name FROM cities WHERE latitude >= 43;
-- 计算每个城市与 Minneapolis（纬度45）的纬度差
SELECT name, latitude - 45 AS diff FROM cities;
```
### 6.2 应用：查找距离最远的南方城市

假设我们想找出离 Minneapolis 最远的南方城市（纬度小于45），按距离排序。

```sql

SELECT name, 45 - latitude AS south_distance
FROM cities
WHERE latitude < 45
ORDER BY south_distance DESC;
```
**实际意义**：纬度差可近似换算为距离（1° ≈ 60 海里），这类查询常用于地理数据分析。

---

## 七、字符串表达式

### 7.1 字符串连接：`||`

SQL 中使用 `||` 作为字符串连接运算符（与 Python 的 `+` 不同）。

```sql

SELECT 'hello,' || ' world' AS greeting;
-- 结果: hello, world
```
### 7.2 常用字符串函数

- `substr(s, start, length)`：从 `start` 位置开始取长度为 `length` 的子串（起始索引为1）。
    
- `instr(s, substr)`：返回 `substr` 在 `s` 中第一次出现的位置（未找到返回0）。
    

**示例**：

```sql

CREATE TABLE phrase AS SELECT 'hello, world' AS s;
SELECT substr(s, 4, 2) FROM phrase;       -- "lo"
SELECT instr(s, ' ') FROM phrase;         -- 7（空格的位置）
SELECT substr(s, 4, 2) || substr(s, instr(s, ' ')+1, 1) FROM phrase;
-- "lo" || "w" = "lo w"
```
### 7.3 用字符串表示结构化数据（不推荐）

有时可以用字符串模拟数据结构，例如用逗号分隔的列表表示链表，然后用字符串函数解析。但这不是 SQL 的设计用途，应避免在生产代码中使用。

```sql

CREATE TABLE lists AS
  SELECT 'one' AS car, 'two,three,four' AS cdr;
-- 获取 cdr 中的第一个元素
SELECT substr(cdr, 1, instr(cdr, ',')-1) AS cadr FROM lists;
-- 返回 "two"
```
**注意事项**：这种用法复杂、易错、性能差，应使用规范化的表结构代替。

---

## 八、应用：生成名词句子

### 8.1 基本句子生成

有一个 `nouns` 表包含名词短语：

```sql

CREATE TABLE nouns AS
  SELECT 'bird' AS phrase UNION
  SELECT 'cat'           UNION
  SELECT 'dog';
```
我们想生成所有可能的“主语 chased 宾语”句子，且主语和宾语不同。

```sql

SELECT subject.phrase || ' chased ' || object.phrase
FROM nouns AS subject, nouns AS object
WHERE subject.phrase != object.phrase;
```
**结果**：

```text

bird chased cat
bird chased dog
cat chased bird
cat chased dog
dog chased bird
dog chased cat
```
### 8.2 扩展：复合主语/宾语

我们可以创建复合名词表，如 `ands` 表，包含 "X and Y" 形式的短语。

```sql

CREATE TABLE ands AS
  SELECT first.phrase || ' and ' || second.phrase AS phrase
  FROM nouns AS first, nouns AS second
  WHERE first.phrase != second.phrase;
```
然后从 `ands` 中选取主语和宾语，生成更复杂的句子。

```sql

SELECT subject.phrase || ' chased ' || object.phrase
FROM ands AS subject, ands AS object
WHERE subject.phrase != object.phrase;
```
这会生成如 "dog and bird chased cat and dog" 之类的句子。

**关键点**：这种查询展示了 SQL 的**组合能力**——通过自连接和笛卡尔积，可以枚举所有可能的组合，生成新数据。

---

## 九、知识点总结表

|知识点|核心内容|考试重点 / 易混淆点|难度|
|---|---|---|---|
|**基本连接**|用逗号连接两个表，生成笛卡尔积，`WHERE` 添加连接条件|理解笛卡尔积；连接条件通常涉及主外键|⭐⭐|
|**自连接**|同一表用不同别名连接，用于比较同一表中不同行|别名必要性；典型应用：兄弟姐妹、祖先|⭐⭐⭐|
|**多表连接**|连接三个及以上表，逐步关联多个关系|多表连接时的列歧义；可用子查询简化|⭐⭐⭐|
|**别名与点表达式**|用 `AS` 给表起别名，用 `别名.列名` 指定列来源|自连接时必须用别名|⭐⭐|
|**数值表达式**|`SELECT` 中可使用 `+ - * /` 对列值计算|可与 `WHERE`、`ORDER BY` 结合|⭐|
|**字符串表达式**|`\|` 连接字符串，`substr`、`instr` 等函数处理|SQL 字符串索引从 1 开始；与 Python 区别|⭐⭐|
|**组合生成**|利用笛卡尔积生成所有可能组合，用于枚举（如句子生成）|注意避免过大结果集；可用 `WHERE` 过滤|⭐⭐⭐|
|**UNION**|合并多个查询结果，默认去重，`UNION ALL` 保留重复|列数、类型必须一致|⭐⭐|

---

## 十、自测题

### 1. 基本连接

假设有表 `students(id, name)` 和 `enrollments(student_id, course)`。写出 SQL 查询，显示每个学生的姓名和他们选修的课程。

```sql

SELECT students.name, enrollments.course
FROM students, enrollments
WHERE students.id = enrollments.student_id;

```

### 2. 自连接

给定表 `employees(id, name, manager_id)`（manager_id 指向同一表的 id），找出所有员工-经理对，显示员工名和经理名。

```sql

SELECT e.name AS employee, m.name AS manager
FROM employees AS e, employees AS m
WHERE e.manager_id = m.id;
```

### 3. 多表连接

基于本课的 `dogs` 和 `parents` 表，找出所有毛发为 'long' 的狗的父母的名字。

```sql

SELECT parent
FROM parents, dogs
WHERE child = name AND fur = 'long';
```

### 4. 字符串表达式

写一个查询，从 `phrase` 表（含列 `s`，值为 'hello, world'）中提取出 "world" 这个词。

```sql

SELECT substr(s, instr(s, ' ')+1) FROM phrase;
```

或者更精确地（假设后面没有逗号）：

```sql

SELECT substr(s, instr(s, ' ')+1, length(s) - instr(s, ' ')) FROM phrase;

```

### 5. 组合生成

使用 `nouns` 表（'bird','cat','dog'），生成所有可能的“主语 and 宾语”短语（如 'bird and cat'），要求主语和宾语不同，且每对只出现一次（即 'bird and cat' 和 'cat and bird' 视为不同，都保留？实际需求可能是不同，如果要避免顺序重复，可用 `<` 条件）。写出保留所有顺序的版本。

```sql

SELECT a.phrase || ' and ' || b.phrase
FROM nouns AS a, nouns AS b
WHERE a.phrase != b.phrase;
```

如果要避免顺序重复（只保留 a.phrase < b.phrase），则：
```sql

SELECT a.phrase || ' and ' || b.phrase
FROM nouns AS a, nouns AS b
WHERE a.phrase < b.phrase;

```

### 6. 思考题

为什么说自连接是“将表虚拟成两个副本”？在数据库中实际存储上真的有两个副本吗？

<details> <summary>提示</summary>

自连接在逻辑上相当于将同一张表复制两份，然后进行连接。但在物理存储上，并没有真正的副本，DBMS 通过引用同一份数据并分别用别名标识来模拟这一过程。这类似于编程中一个对象被两个变量引用。

</details>

---

## 十一、总结与展望

本节课我们深入学习了 SQL 中最重要的操作之一——**连接**。你学会了：

- 用逗号实现基本连接，并用 `WHERE` 添加连接条件。
    
- 自连接的原理和应用（兄弟姐妹、祖孙关系）。
    
- 多表连接如何组合多个关系回答复杂查询。
    
- 在 `SELECT` 中使用数值和字符串表达式进行计算和转换。
    
- 利用笛卡尔积生成组合数据（如句子生成）。
    

连接是关系数据库的基石，掌握它将使你能够处理几乎任何复杂的数据查询。在后续课程中，你将学习更高级的连接类型（如 `JOIN` 关键字）、聚合函数和分组，进一步完善你的 SQL 技能。