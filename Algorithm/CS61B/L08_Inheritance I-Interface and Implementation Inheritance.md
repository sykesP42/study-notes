# Java继承：从原理到实践的深度指南
## 一、继承概述

### 1. 课前回顾

继承是一种面向对象编程的核心机制，允许类（子类/hyponym）共享其他类（父类/hypernym）的属性和方法。本单元将用四节课深入讲解继承机制。

**现有基础**：

- `Alist`类包含`insert()`, `addFirst()`, `addLast()`, `getFirst()`, `getLast()`, `removeLast()`, `size()`等方法
    
- `SLList`类包含几乎相同的方法签名，包括`insert()`, `addFirst()`, `addLast()`等
    

### 2. 应用案例：寻找最长单词

**问题场景**：编写一个函数，从单词列表中找出最长的单词。

**初始实现**：


```java

public static String longest(SLList<String> list) {
    int maxDex = 0;
    for (int i = 0; i < list.size(); i++) {
        String longestString = list.get(maxDex);
        String thisString = list.get(i);
        if (thisString.length() > longestString.length()) {
            maxDex = i;
        }
    }
    return list.get(maxDex);
}
```
**问题发现**：当用户使用`Alist`而非`SLList`时，方法无法编译，因为类型不匹配。

**初步解决方案**：使用方法重载（overloading）

```java

public static String longest(SLList<String> list) { ... }
public static String longest(Alist<String> list) { ... }
```
**重载的缺点**：

1. **代码重复**：相同逻辑需要维护多份
    
2. **维护困难**：修改算法或修复bug时需要同步所有副本
    
3. **扩展性问题**：每新增一种列表类型都需要添加对应方法，代码量呈线性增长
    

**改进方向**：需要寻找更通用的解决方案，避免代码重复，支持未来扩展，便于维护。

---

## 二、超类与子类概念

### 1. 超类和子类的概念

**自然语言中的类比**：

- **超类（Hypernym）**：更一般的概念，如"狗"
    
- **子类（Hyponym）**：更具体的概念，如"贵宾犬"、"阿拉斯加雪橇犬"
    

**层级示例**：

```text

动物（Animal）
  ↓
食肉动物（Carnivore）
  ↓
犬科动物（Canine）
  ↓
狗（Dog）
  ↓
贵宾犬（Poodle）、阿拉斯加雪橇犬（Malamute）等
```
**应用到编程**：

- `SLList`和`Alist`都是某种"列表"
    
- "列表"是`SLList`和`Alist`的超类
    
- `SLList`和`Alist`都是"列表"的子类
    

### 2. Java中的实现步骤

**第一步**：为超类定义引用类型（接口）

```java

public interface List61B<Item> {
    public void addFirst(Item x);
    public void addLast(Item y);
    public Item getFirst();
    public Item getLast();
    // 其他方法...
}
```
**第二步**：指定子类实现该接口

```java

public class SLList<Item> implements List61B<Item> {
    // 实现所有接口方法...
}

public class Alist<Item> implements List61B<Item> {
    // 实现所有接口方法...
}
```
### 3. 接口（Interface）概念

**接口定义**：

- 使用`interface`关键字定义
    
- 是规范类行为的契约
    
- 只声明方法签名，不提供具体实现
    

**接口特点**：

1. 只规定"能做什么"（what），不规定"如何做"（how）
    
2. 方法默认为`public`，实现类中必须显式声明为`public`
    
3. 不能包含具体实现细节（如数组、变量等）
    

**实现要求**：

- 实现接口的类必须提供所有声明方法的具体实现
    
- 未完全实现将导致编译失败
    
- 必须建立"is-a"关系而非"has-a"关系
    

---

## 三、方法覆盖与方法重载

### 1. 方法覆盖（Override）

**定义**：当子类包含与父类/接口完全相同的签名方法时，称为方法覆盖。

**特点**：

- 方法签名必须完全相同（名称+参数列表）
    
- 发生在继承/实现关系的类之间
    
- 子类方法"取代"父类方法
    

**示例**：

```java

public interface Animal {
    void makeNoise();
}

public class Pig implements Animal {
    @Override
    public void makeNoise() {
        System.out.println("Oink!");
    }
}
```
### 2. 方法重载（Overload）

**定义**：同一类中包含多个同名但参数不同的方法。

**特点**：

- 方法名相同但参数不同（签名不同）
    
- 可以发生在同一个类内
    
- 多个版本共存，根据调用参数选择
    

**示例**：

```java

public class Dog {
    public void makeNoise() {
        System.out.println("Bark!");
    }
    
    public void makeNoise(Dog other) {
        System.out.println("Bark at another dog!");
    }
}
```
### 3. @Override注解

**作用**：

1. **编译检查**：确保被标记的方法确实覆盖了父类方法
    
2. **代码组织**：明确标识方法覆盖关系
    
3. **提醒功能**：提醒程序员该方法是从继承层次结构中覆盖而来
    

**使用示例**：

```java

public class Alist<Item> implements List61B<Item> {
    @Override
    public void addLast(Item x) {
        // 具体实现...
    }
    
    @Override
    public Item getLast() {
        // 具体实现...
    }
}
```
**必要性**：如果不添加`@Override`注解，当方法名拼写错误时，编译器不会报错，但实际是创建了一个新方法而非覆盖，这可能导致难以发现的bug。

---

## 四、接口继承

### 1. 接口继承概念

**定义**：使用`implements`关键字指定子类能力的方式称为接口继承。

**特点**：

1. 子类"继承"接口规范
    
2. 只规定子类能做什么，不规定如何实现
    
3. 子类必须重写所有接口方法
    
4. 编译时严格检查
    

**示例**：

```java

public interface List61B<Item> {
    void addFirst(Item x);
    void addLast(Item y);
    Item getFirst();
    Item getLast();
    Item removeLast();
    void insert(Item x, int position);
    int size();
}

public class SLList<Item> implements List61B<Item> {
    // 必须实现所有接口方法...
}
```
### 2. 接口继承的优势

**解决longest方法问题**：

```java

// 通用版本，可处理任何实现了List61B接口的列表
public static String longest(List61B<String> list) {
    int maxDex = 0;
    for (int i = 0; i < list.size(); i++) {
        String longestString = list.get(maxDex);
        String thisString = list.get(i);
        if (thisString.length() > longestString.length()) {
            maxDex = i;
        }
    }
    return list.get(maxDex);
}

// 使用示例
SLList<String> slList = new SLList<>();
Alist<String> aList = new Alist<>();

String longest1 = longest(slList);  // 可以工作
String longest2 = longest(aList);   // 也可以工作
```
**扩展性**：该方法可工作在任何实现了`List61B`接口的列表类型上，包括未来可能发明的"`AQList`"等新列表类型。

### 3. is-a关系

**内存机制**：内存盒只能持有适当类型的64位地址。

**示例**：

```java

List61B<String> inputList = new Alist<String>();
```
- `inputList`变量只能持有`List61B<String>`类型的引用
    
- 由于`Alist`是`List61B`的子类（is-a关系），`inputList`可以持有`Alist`的引用
    

**类型兼容性**：当方法参数声明为`List61B`类型时，可以接受任何实现该接口的类实例。

---

## 五、默认方法

### 1. 默认方法概念

**定义**：在接口中使用`default`关键字定义方法实现，子类可以选择是否覆盖。

**语法**：

```java

public interface List61B<Item> {
    // 普通抽象方法
    void addFirst(Item x);
    
    // 默认方法
    default public void print() {
        for (int i = 0; i < size(); i++) {
            System.out.print(get(i) + " ");
        }
        System.out.println();
    }
}
```
**特点**：

1. 子类不仅可以继承接口签名，还能继承具体实现代码
    
2. 子类可以选择是否覆盖默认方法
    
3. 即使不知道具体子类类型，也能通过接口保证的方法编写通用实现
    

### 2. 默认方法的效率问题

**示例分析**：

```java

default public void print() {
    for (int i = 0; i < size(); i++) {
        System.out.print(get(i) + " ");
    }
    System.out.println();
}
```
**效率分析**：

- **对于`Alist`**：`size()`和`get()`都是O(1)操作，整体效率高
    
- **对于`SLList`**：`get()`需要从头节点开始遍历，导致O(n)时间复杂度，多次调用导致整体O(n²)时间复杂度
    

**结论**：默认方法对`Alist`高效，但对`SLList`效率低下。

### 3. 覆盖默认方法

**动机**：当默认方法对特定子类效率不高时，可以重写优化。

**示例**：

```java

public class SLList<Item> implements List61B<Item> {
    @Override
    public void print() {
        // 使用链表特有的高效遍历方式
        for (Node p = sentinel.next; p != null; p = p.next) {
            System.out.print(p.item + " ");
        }
        System.out.println();
    }
}
```
**语法要点**：

1. 方法签名必须与接口完全一致
    
2. 使用`@Override`注解可防止拼写错误
    
3. 可以访问子类特有成员变量和方法
    

---

## 六、静态类型与动态类型

### 1. 静态类型（编译时类型）

**定义**：每个Java变量都有"编译时类型"，也称为"静态类型"，这是在变量声明时指定的类型。

**特点**：

1. 声明后永远不会改变
    
2. 变量不能被重新声明来改变其静态类型
    
3. 确定方法：查看变量声明时的类型指定
    

**示例**：

```java

Animal a1;           // a1的静态类型是Animal
LivingThing lt1;     // lt1的静态类型是LivingThing
List61B<String> lst; // lst的静态类型是List61B<String>
```
### 2. 动态类型（运行时类型）

**定义**：变量还有"运行时类型"，也称为"动态类型"，这是在实例化时指定的类型。

**特点**：

1. 等于变量当前指向的对象类型
    
2. 会随着赋值操作而改变
    
3. 与静态类型可能不同
    

**示例**：

```java

List61B<String> someList = new SLList<String>();
// 静态类型：List61B<String>
// 动态类型：SLList<String>

someList = new Alist<String>();
// 静态类型：List61B<String>（不变）
// 动态类型：Alist<String>（改变）
```
**与静态类型的区别**：

- 静态类型看声明，动态类型看当前指向
    
- 静态类型不变，动态类型可变
    

### 3. 动态方法选择

**核心规则**：当调用对象方法时，如果运行时类型重写了该方法，则使用重写方法而非编译时类型的方法。

**判断流程**：

1. 查看变量的静态类型（声明类型）
    
2. 查看变量当前指向对象的实际类型（动态类型）
    
3. 检查动态类型是否覆盖了该方法
    
4. 如果覆盖，则使用动态类型的实现
    

**示例**：

```java

public static void main(String[] args) {
    List61B<String> someList = new SLList<String>();
    someList.addLast("elk");
    someList.addLast("are");
    someList.addLast("watching");
    someList.print(); // 调用SLList的print()方法（如果覆盖了）
}
```
**术语**：这一过程称为"动态方法选择"（dynamic method selection）。

**"覆盖优先"原则**：动态类型的覆盖方法优先于静态类型的原始方法，这一机制实现了Java的多态特性。

---

## 七、接口继承与实现继承的区别

### 1. 接口继承（Interface Inheritance）

**特点**：

- 只继承行为规范（如列表必须能`addLast`）
    
- 不提供具体实现方式
    
- 子类必须实现所有声明方法
    

**作用**：允许以强大而简单的方式泛化代码。

### 2. 实现继承（Implementation Inheritance）

**特点**：

- 通过`default`关键字继承具体实现
    
- 子类可选择是否覆盖默认实现
    
- 允许代码复用
    

**作用**：

1. 允许代码复用：子类可以依赖超类或接口的默认实现
    
2. 给予子类设计者更多控制权：可以决定是否覆盖默认实现
    

### 3. 关键原则

**必须建立"is-a"关系而非"has-a"关系**：

- **正确示例**：`Dog`实现`Animal`，`SLList`实现`List61B`
    
- **错误示例**：`Cat`实现`Claw`，`Set`实现`SLList`
    

### 4. 实现继承的风险

**实现继承的特定危险**：

1. **追踪困难**：难以确定方法实际实现位置（虽然现代IDE有所改善）
    
2. **冲突解决**：规则可能晦涩难懂（如多个接口提供冲突的默认方法）
    
3. **代码复杂度**：容易导致过度复杂的代码结构（特别是对新手而言）
    
4. **设计错误**：常见混淆"has-a"和"is-a"关系
    
5. **封装破坏**：违反封装原则
    

---

## 八、知识总结

### 核心知识点速查表

|知识点|核心内容|考试重点/易混淆点|难度系数|
|---|---|---|---|
|**继承概念**|通过父子类关系实现代码复用，子类继承父类特性|区分接口继承与实现继承|★★★|
|**接口 (Interface)**|仅定义方法签名（无实现），强制子类实现所有声明方法|`@Override`注解的作用与必要性|★★★|
|**方法重写 (Override)**|子类覆盖父类同名同参数方法，运行时调用子类实现|动态方法选择规则：运行时类型决定调用版本|★★★★|
|**默认方法 (Default Method)**|接口中使用`default`提供默认实现，子类可选覆盖|默认方法性能陷阱（如链表遍历的O(n²)问题）|★★★★|
|**多态类型系统**|变量存在编译时类型（声明类型）和运行时类型（实际对象类型）|编译时类型决定可调用方法，运行时类型决定具体执行逻辑|★★★★★|

### 设计原则

1. **通过超类统一接口**：使用超类（如`List61B`）统一接口，避免针对具体子类（`Alist`/`SLList`）重复编码
    
2. **类型兼容性**：超类变量可引用任意子类对象
    
3. **语法特性**：`implements`声明接口实现，`super`访问父类成员
    
4. **接口与抽象类的选择**：根据具体需求选择合适的抽象机制
    

### 调试技巧

1. **使用`@Override`检测拼写错误**：避免意外新建方法而非重写
    
2. **注意未添加注解时的隐蔽错误风险**：编译器不会提示拼写错误
    
3. **测试覆盖所有实现类**：确保通用方法在所有子类上都能正常工作
    

### 实际应用建议

1. **优先使用接口而非具体类**：在方法参数和返回值类型中优先使用接口类型
    
2. **合理使用默认方法**：为通用操作提供默认实现，但要注意性能影响
    
3. **避免过度继承**：保持继承层次简单明了，避免过度复杂的继承关系
    
4. **正确使用`@Override`**：为所有覆盖方法添加注解，提高代码可读性和安全性
    

---

## 九、关键代码实现

### 1. 接口定义

```java

public interface List61B<Item> {
    public void addFirst(Item x);
    public void addLast(Item y);
    public Item getFirst();
    public Item getLast();
    public Item removeLast();
    public void insert(Item x, int position);
    public int size();
    
    // 默认方法
    default public void print() {
        for (int i = 0; i < size(); i++) {
            System.out.print(get(i) + " ");
        }
        System.out.println();
    }
}
```
### 2. 实现类示例

```java

public class SLList<Item> implements List61B<Item> {
    // 私有内部类
    private class Node {
        public Item item;
        public Node next;
        
        public Node(Item i, Node n) {
            item = i;
            next = n;
        }
    }
    
    private Node sentinel;
    private int size;
    
    public SLList() {
        sentinel = new Node(null, null);
        size = 0;
    }
    
    @Override
    public void addFirst(Item x) {
        sentinel.next = new Node(x, sentinel.next);
        size += 1;
    }
    
    @Override
    public void addLast(Item x) {
        Node p = sentinel;
        while (p.next != null) {
            p = p.next;
        }
        p.next = new Node(x, null);
        size += 1;
    }
    
    @Override
    public Item getFirst() {
        return sentinel.next.item;
    }
    
    @Override
    public Item getLast() {
        Node p = sentinel;
        while (p.next != null) {
            p = p.next;
        }
        return p.item;
    }
    
    @Override
    public Item removeLast() {
        // 实现...
    }
    
    @Override
    public void insert(Item x, int position) {
        // 实现...
    }
    
    @Override
    public int size() {
        return size;
    }
    
    @Override
    public Item get(int i) {
        Node p = sentinel.next;
        while (i > 0 && p != null) {
            p = p.next;
            i -= 1;
        }
        return p.item;
    }
    
    @Override
    public void print() {
        // 优化版本，避免O(n²)时间复杂度
        for (Node p = sentinel.next; p != null; p = p.next) {
            System.out.print(p.item + " ");
        }
        System.out.println();
    }
}
```
### 3. 通用方法

```java

public class WordUtils {
    public static String longest(List61B<String> list) {
        if (list.size() == 0) {
            return null;
        }
        
        int maxDex = 0;
        for (int i = 0; i < list.size(); i++) {
            String longestString = list.get(maxDex);
            String thisString = list.get(i);
            if (thisString.length() > longestString.length()) {
                maxDex = i;
            }
        }
        return list.get(maxDex);
    }
}
```
### 4. 使用示例

```java

public class TestInheritance {
    public static void main(String[] args) {
        // 创建不同类型的列表
        List61B<String> slList = new SLList<>();
        List61B<String> aList = new Alist<>();
        
        // 添加数据
        slList.addLast("cat");
        slList.addLast("elephant");
        slList.addLast("dog");
        
        aList.addLast("apple");
        aList.addLast("banana");
        aList.addLast("strawberry");
        
        // 使用通用方法
        String longest1 = WordUtils.longest(slList); // "elephant"
        String longest2 = WordUtils.longest(aList);  // "strawberry"
        
        System.out.println("Longest in SLList: " + longest1);
        System.out.println("Longest in Alist: " + longest2);
        
        // 调用print方法
        System.out.print("SLList contents: ");
        slList.print();  // 调用SLList的优化版本
        
        System.out.print("Alist contents: ");
        aList.print();   // 调用默认版本
    }
}
```
---

通过本讲的学习，你应该能够：

1. 理解继承的概念和必要性
    
2. 掌握接口的定义和实现方法
    
3. 区分方法覆盖与方法重载
    
4. 理解静态类型与动态类型的区别
    
5. 掌握动态方法选择机制
    
6. 理解接口继承与实现继承的区别
    
7. 能够设计合理的类层次结构
    
8. 编写通用代码处理多种类型
    

**关键要点**：

- 继承的核心是建立"is-a"关系
    
- 接口定义"能做什么"，实现类定义"如何做"
    
- 多态通过动态方法选择实现
    
- `@Override`注解是重要的安全机制
    
- 默认方法提供了实现继承的机制，但需要谨慎使用