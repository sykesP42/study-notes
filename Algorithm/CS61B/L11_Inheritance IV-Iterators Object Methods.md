# Java集合与迭代器：ArraySet实现详解

## 一、集合基础概念

### 1.1 集合的定义与特性

**集合**（Set）是计算机科学中的一种基本数据结构，具有以下核心特性：

1. **无序性**：元素没有固定顺序，不能通过索引访问特定位置元素
    
2. **唯一性**：集合中不允许存在重复元素
    
3. **确定性**：元素要么属于集合，要么不属于，没有模糊状态
    


```java

// Java标准库中的Set示例
import java.util.HashSet;
import java.util.Set;

public class SetDemo {
    public static void main(String[] args) {
        Set<String> cities = new HashSet<>();
        cities.add("Beijing");
        cities.add("Shanghai");
        cities.add("Guangzhou");
        cities.add("Beijing");  // 重复添加，集合中只有一个"Beijing"
        
        System.out.println(cities.size());  // 输出：3
        System.out.println(cities.contains("Shanghai"));  // 输出：true
    }
}
```
### 1.2 ArraySet设计目标

课程目标：构建一个名为`ArraySet`的自定义集合实现，基于数组存储元素，逐步添加工业级特性。

**初始功能需求**：

- `add(value)`：添加元素，确保唯一性
    
- `contains(value)`：检查元素是否存在
    
- `size()`：返回集合大小
    

**简化假设**：暂不考虑数组扩容（resizing），使用固定大小数组。

## 二、ArraySet基本实现

### 2.1 数据结构设计

```java

public class ArraySet<T> {
    // 底层存储：使用泛型数组
    private T[] items;
    // 当前元素数量
    private int size;
    
    // 构造函数
    public ArraySet() {
        // 初始化数组大小为100
        items = (T[]) new Object[100];  // 警告：unchecked cast
        size = 0;
    }
    
    // 基本方法
    public void add(T value) {
        if (value == null) {
            throw new IllegalArgumentException("Cannot add null to ArraySet");
        }
        
        // 检查元素是否已存在
        if (contains(value)) {
            return;  // 已存在，不添加
        }
        
        // 添加到数组末尾
        items[size] = value;
        size++;
    }
    
    public boolean contains(T value) {
        for (int i = 0; i < size; i++) {
            // 使用equals()比较对象内容，而非==比较引用
            if (items[i].equals(value)) {
                return true;
            }
        }
        return false;
    }
    
    public int size() {
        return size;
    }
}
```
### 2.2 关键实现细节

#### 泛型数组的创建问题

Java中不能直接创建泛型数组，需要使用类型转换：

java

items = (T[]) new Object[100];

这会产生"unchecked cast"编译器警告，但在本实现中可以接受。

#### 元素比较策略

- 使用`.equals()`方法而非`==`运算符进行元素比较
    
- `.equals()`比较对象内容，`==`比较对象引用地址
    
- 对于自定义对象，需要正确实现`equals()`方法
    

#### null值处理

- 可选择禁止null值（如示例中的抛出异常）
    
- 也可选择允许null值，但需要特殊处理
    

## 三、迭代器（Iterator）实现

### 3.1 增强for循环的工作原理

Java的增强for循环（foreach循环）是迭代集合的语法糖：

```java

// 简洁写法
for (String city : cities) {
    System.out.println(city);
}

// 实际等价于
Iterator<String> iterator = cities.iterator();
while (iterator.hasNext()) {
    String city = iterator.next();
    System.out.println(city);
}
```
### 3.2 迭代器接口要求

要使类支持增强for循环，必须实现两个关键接口：

1. **Iterable 《T》**：表示对象可以被迭代
    
    - 包含方法：`Iterator<T> iterator()`
        
2. **Iterator 《T》**：实际的迭代器对象
    
    - 包含方法：
        
        - `boolean hasNext()`：是否还有下一个元素
            
        - `T next()`：返回下一个元素
            
        - `void remove()`：可选，删除当前元素
            

### 3.3 ArraySet迭代器实现

```java

import java.util.Iterator;

public class ArraySet<T> implements Iterable<T> {
    private T[] items;
    private int size;
    
    // ... 其他代码 ...
    
    // 实现Iterable接口
    @Override
    public Iterator<T> iterator() {
        return new ArraySetIterator();
    }
    
    // 内部类实现Iterator接口
    private class ArraySetIterator implements Iterator<T> {
        private int wizPos;  // "巫师位置"，当前迭代位置
        
        public ArraySetIterator() {
            wizPos = 0;  // 从第一个元素开始
        }
        
        @Override
        public boolean hasNext() {
            return wizPos < size;  // 检查是否还有元素
        }
        
        @Override
        public T next() {
            // 返回当前元素，并移动到下一个位置
            T returnItem = items[wizPos];
            wizPos += 1;
            return returnItem;
        }
        
        // 可选：实现remove()方法
        @Override
        public void remove() {
            throw new UnsupportedOperationException("Remove operation not supported");
        }
    }
}
```
### 3.4 迭代器设计模式解析

**迭代器设计模式**将集合的遍历逻辑与集合本身分离，具有以下优势：

1. **封装性**：隐藏集合内部实现细节
    
2. **统一接口**：为不同集合提供一致的遍历方式
    
3. **支持多种遍历**：可为同一集合提供不同的遍历策略
    
4. **简化客户端代码**：客户端无需了解集合内部结构
    

```java

// 使用示例
public class IteratorDemo {
    public static void main(String[] args) {
        ArraySet<Integer> numbers = new ArraySet<>();
        numbers.add(5);
        numbers.add(23);
        numbers.add(42);
        
        // 方式1：增强for循环（推荐）
        for (Integer num : numbers) {
            System.out.println(num);
        }
        
        // 方式2：显式使用迭代器
        Iterator<Integer> iterator = numbers.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```
### 3.5 Java集合框架中的迭代器层次

```text

Iterable<T> (接口)
    ↑
Collection<T> (接口)
    ↑
Set<T> (接口)    List<T> (接口)    Queue<T> (接口)
    ↑              ↑                  ↑
HashSet<T>       ArrayList<T>      LinkedList<T>
TreeSet<T>       Vector<T>         PriorityQueue<T>
LinkedHashSet<T> LinkedList<T>
```
**关键点**：

- 所有标准Java集合都实现了`Iterable`接口
    
- 自定义集合遵循相同模式即可获得标准迭代支持
    
- 迭代顺序由具体实现决定（如`ArraySet`按添加顺序）
    

## 四、toString()方法重写

### 4.1 Object.toString()默认行为

所有Java类都是`Object`类的子类，继承了`toString()`方法：

```java

// Object类中的默认实现
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```
默认输出格式：`类名@哈希码`（如`ArraySet@5010be6`）

### 4.2 重写toString()方法

为`ArraySet`提供更友好的字符串表示：

```java

public class ArraySet<T> implements Iterable<T> {
    // ... 其他代码 ...
    
    @Override
    public String toString() {
        // 使用StringBuilder提高性能
        StringBuilder sb = new StringBuilder();
        sb.append("{");
        
        // 使用迭代器遍历元素
        Iterator<T> iterator = iterator();
        while (iterator.hasNext()) {
            sb.append(iterator.next());
            if (iterator.hasNext()) {
                sb.append(", ");
            }
        }
        
        sb.append("}");
        return sb.toString();
    }
    
    // 更简洁的实现（使用增强for循环）
    @Override
    public String toString() {
        List<String> elements = new ArrayList<>();
        for (T item : this) {  // 使用增强for循环
            elements.add(item.toString());
        }
        return "{" + String.join(", ", elements) + "}";
    }
}
```
### 4.3 StringBuilder vs String拼接

**性能对比**：

```java

// 方式1：直接拼接（性能差）
String result = "{";
for (T item : this) {
    result += item + ", ";  // 每次循环创建新String对象
}
result = result.substring(0, result.length() - 2) + "}";

// 方式2：StringBuilder（性能好）
StringBuilder sb = new StringBuilder("{");
for (T item : this) {
    sb.append(item).append(", ");
}
if (sb.length() > 1) {
    sb.setLength(sb.length() - 2);  // 删除最后的", "
}
sb.append("}");
return sb.toString();
```
**StringBuilder优势**：

- 可变字符序列，避免重复创建String对象
    
- 适合大量字符串操作场景
    
- 线程不安全但性能更好（单线程环境下）
    

## 五、equals()方法重写

### 5.1 == 与 equals() 的区别

|比较维度|`==` 运算符|`.equals()` 方法|
|---|---|---|
|**作用**|比较引用地址或基本类型值|比较对象内容|
|**对象比较**|检查是否同一对象|检查内容是否相等|
|**默认实现**|语言内置|Object类中实现为`==`|
|**可重写性**|不可重写|可重写以实现自定义比较|

```java

// 示例：String对象的比较
String s1 = new String("hello");
String s2 = new String("hello");
String s3 = s1;

System.out.println(s1 == s2);      // false：不同对象
System.out.println(s1 == s3);      // true：同一对象
System.out.println(s1.equals(s2)); // true：内容相同
System.out.println(s1.equals(s3)); // true：内容相同
```
### 5.2 重写equals()方法的需求

对于`ArraySet`，我们希望两个包含相同元素的集合被视为相等，即使它们不是同一个对象：

```java

ArraySet<Integer> set1 = new ArraySet<>();
set1.add(1);
set1.add(2);
set1.add(3);

ArraySet<Integer> set2 = new ArraySet<>();
set2.add(3);
set2.add(2);
set2.add(1);

// 期望：set1.equals(set2) 返回 true
```
### 5.3 equals()方法实现规范

`equals()`方法必须满足以下数学性质：

1. **自反性**：`x.equals(x)` 必须返回 true
    
2. **对称性**：如果 `x.equals(y)` 返回 true，则 `y.equals(x)` 也必须返回 true
    
3. **传递性**：如果 `x.equals(y)` 且 `y.equals(z)`，则 `x.equals(z)` 必须返回 true
    
4. **一致性**：多次调用 `x.equals(y)` 应始终返回相同结果
    
5. **非空性**：`x.equals(null)` 必须返回 false
    

### 5.4 ArraySet.equals()实现

```java

public class ArraySet<T> implements Iterable<T> {
    // ... 其他代码 ...
    
    @Override
    public boolean equals(Object other) {
        // 1. 检查是否为同一对象
        if (this == other) {
            return true;
        }
        
        // 2. 检查是否为null
        if (other == null) {
            return false;
        }
        
        // 3. 检查类型是否匹配
        if (this.getClass() != other.getClass()) {
            return false;
        }
        
        // 4. 类型转换
        ArraySet<T> otherSet = (ArraySet<T>) other;
        
        // 5. 检查大小是否相同
        if (this.size() != otherSet.size()) {
            return false;
        }
        
        // 6. 检查所有元素是否相同
        for (T item : this) {
            if (!otherSet.contains(item)) {
                return false;
            }
        }
        
        return true;
    }
    
    // 注意：重写equals()必须同时重写hashCode()
    @Override
    public int hashCode() {
        int result = 0;
        for (T item : this) {
            // 使用元素的hashCode，考虑null情况
            result += (item == null ? 0 : item.hashCode());
        }
        return result;
    }
}
```
### 5.5 instanceof 关键字的深入理解

`instanceof`操作符用于检查对象的运行时类型：

```java

if (other instanceof ArraySet) {
    // 安全地进行类型转换
    ArraySet<T> otherSet = (ArraySet<T>) other;
    // ...
}
```
**instanceof的特性**：

1. 检查对象的动态类型（运行时类型）
    
2. 如果对象为null，返回false
    
3. 检查类型及其所有子类型
    
4. 结合类型转换使用，确保类型安全
    

```java

// instanceof的高级用法
Object obj = new ArraySet<Integer>();

// 基本用法
if (obj instanceof ArraySet) {
    System.out.println("obj is an ArraySet");
}

// Java 14+的模式匹配（简化代码）
if (obj instanceof ArraySet<?> otherSet) {
    // otherSet已自动转换并可用
    System.out.println(otherSet.size());
}
```
## 六、完整ArraySet实现

### 6.1 最终版本代码

```java

import java.util.Iterator;
import java.util.NoSuchElementException;

public class ArraySet<T> implements Iterable<T> {
    private static final int DEFAULT_CAPACITY = 100;
    
    private T[] items;
    private int size;
    
    // 构造函数
    @SuppressWarnings("unchecked")
    public ArraySet() {
        items = (T[]) new Object[DEFAULT_CAPACITY];
        size = 0;
    }
    
    @SuppressWarnings("unchecked")
    public ArraySet(int initialCapacity) {
        if (initialCapacity <= 0) {
            throw new IllegalArgumentException("Initial capacity must be positive");
        }
        items = (T[]) new Object[initialCapacity];
        size = 0;
    }
    
    // 基本操作
    public void add(T value) {
        if (value == null) {
            throw new IllegalArgumentException("Cannot add null to ArraySet");
        }
        
        if (contains(value)) {
            return;  // 元素已存在
        }
        
        // 简单扩容机制（简化版）
        if (size == items.length) {
            resize(items.length * 2);
        }
        
        items[size] = value;
        size++;
    }
    
    public boolean contains(T value) {
        if (value == null) {
            return false;  // 根据设计，不包含null
        }
        
        for (int i = 0; i < size; i++) {
            if (items[i].equals(value)) {
                return true;
            }
        }
        return false;
    }
    
    public int size() {
        return size;
    }
    
    public boolean isEmpty() {
        return size == 0;
    }
    
    // 扩容机制
    @SuppressWarnings("unchecked")
    private void resize(int newCapacity) {
        T[] newItems = (T[]) new Object[newCapacity];
        System.arraycopy(items, 0, newItems, 0, size);
        items = newItems;
    }
    
    // 迭代器实现
    @Override
    public Iterator<T> iterator() {
        return new ArraySetIterator();
    }
    
    private class ArraySetIterator implements Iterator<T> {
        private int currentIndex;
        
        public ArraySetIterator() {
            currentIndex = 0;
        }
        
        @Override
        public boolean hasNext() {
            return currentIndex < size;
        }
        
        @Override
        public T next() {
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            T item = items[currentIndex];
            currentIndex++;
            return item;
        }
    }
    
    // toString实现
    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder("{");
        for (int i = 0; i < size; i++) {
            sb.append(items[i]);
            if (i < size - 1) {
                sb.append(", ");
            }
        }
        sb.append("}");
        return sb.toString();
    }
    
    // equals实现
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        
        ArraySet<?> other = (ArraySet<?>) obj;
        if (this.size != other.size) return false;
        
        // 检查所有元素
        for (int i = 0; i < size; i++) {
            if (!other.contains(items[i])) {
                return false;
            }
        }
        return true;
    }
    
    // hashCode实现
    @Override
    public int hashCode() {
        int result = 1;
        for (int i = 0; i < size; i++) {
            T element = items[i];
            result = 31 * result + (element == null ? 0 : element.hashCode());
        }
        return result;
    }
}
```
### 6.2 测试代码

```java

public class ArraySetTest {
    public static void main(String[] args) {
        // 测试基本功能
        ArraySet<String> set1 = new ArraySet<>();
        set1.add("Apple");
        set1.add("Banana");
        set1.add("Apple");  // 重复添加
        set1.add("Orange");
        
        System.out.println("Set1: " + set1);  // {Apple, Banana, Orange}
        System.out.println("Size: " + set1.size());  // 3
        System.out.println("Contains Banana: " + set1.contains("Banana"));  // true
        
        // 测试迭代功能
        System.out.println("\nIteration:");
        for (String fruit : set1) {
            System.out.println(fruit);
        }
        
        // 测试equals方法
        ArraySet<String> set2 = new ArraySet<>();
        set2.add("Orange");
        set2.add("Banana");
        set2.add("Apple");
        
        System.out.println("\nSet1 equals Set2: " + set1.equals(set2));  // true
        System.out.println("Set1 hashCode: " + set1.hashCode());
        System.out.println("Set2 hashCode: " + set2.hashCode());
        
        // 测试null处理
        try {
            set1.add(null);
        } catch (IllegalArgumentException e) {
            System.out.println("\nCorrectly rejected null: " + e.getMessage());
        }
    }
}
```
## 七、高级主题与扩展

### 7.1 集合的数学运算扩展

可以为`ArraySet`添加集合运算功能：

```java

public class ArraySet<T> implements Iterable<T> {
    // ... 现有代码 ...
    
    // 并集
    public ArraySet<T> union(ArraySet<T> other) {
        ArraySet<T> result = new ArraySet<>(this.size() + other.size());
        
        // 添加当前集合所有元素
        for (T item : this) {
            result.add(item);
        }
        
        // 添加另一个集合所有元素（add会自动去重）
        for (T item : other) {
            result.add(item);
        }
        
        return result;
    }
    
    // 交集
    public ArraySet<T> intersection(ArraySet<T> other) {
        ArraySet<T> result = new ArraySet<>(Math.min(this.size(), other.size()));
        
        for (T item : this) {
            if (other.contains(item)) {
                result.add(item);
            }
        }
        
        return result;
    }
    
    // 差集
    public ArraySet<T> difference(ArraySet<T> other) {
        ArraySet<T> result = new ArraySet<>(this.size());
        
        for (T item : this) {
            if (!other.contains(item)) {
                result.add(item);
            }
        }
        
        return result;
    }
    
    // 子集判断
    public boolean isSubsetOf(ArraySet<T> other) {
        for (T item : this) {
            if (!other.contains(item)) {
                return false;
            }
        }
        return true;
    }
}
```
### 7.2 性能优化考虑

1. **查找优化**：使用哈希表优化`contains()`方法
    
2. **内存优化**：动态调整数组大小，避免浪费
    
3. **迭代器并发修改检测**：
    
    ```java
    
    private class ArraySetIterator implements Iterator<T> {
        private int currentIndex;
        private final int expectedSize;  // 记录创建迭代器时的大小
        
        public ArraySetIterator() {
            currentIndex = 0;
            expectedSize = size;  // 捕获当前大小
        }
        
        @Override
        public T next() {
            if (expectedSize != size) {
                throw new ConcurrentModificationException();
            }
            // ... 其他代码 ...
        }
    }
    
```
### 7.3 实现Java标准Set接口

要使`ArraySet`完全兼容Java集合框架，可以实现`Set`接口：

```java

import java.util.Collection;
import java.util.Set;
import java.util.Spliterator;
import java.util.stream.Stream;
import java.util.stream.StreamSupport;

public class ArraySet<T> implements Set<T> {
    // 必须实现Set接口的所有方法
    // 包括：addAll, remove, removeAll, retainAll, clear, toArray等
    
    @Override
    public boolean addAll(Collection<? extends T> c) {
        boolean modified = false;
        for (T item : c) {
            if (add(item)) {
                modified = true;
            }
        }
        return modified;
    }
    
    @Override
    public void clear() {
        for (int i = 0; i < size; i++) {
            items[i] = null;  // 帮助垃圾回收
        }
        size = 0;
    }
    
    @Override
    public Object[] toArray() {
        return Arrays.copyOf(items, size);
    }
    
    @Override
    @SuppressWarnings("unchecked")
    public <E> E[] toArray(E[] a) {
        if (a.length < size) {
            return (E[]) Arrays.copyOf(items, size, a.getClass());
        }
        System.arraycopy(items, 0, a, 0, size);
        if (a.length > size) {
            a[size] = null;
        }
        return a;
    }
    
    // 其他Set接口方法...
}
```
## 八、知识总结与最佳实践

### 8.1 核心知识点总结

|主题|关键点|注意事项|
|---|---|---|
|**集合特性**|无序性、唯一性、确定性|与列表(List)的区别|
|**迭代器模式**|Iterable和Iterator接口分离关注点|内部类实现，支持增强for循环|
|**toString()**|提供友好的字符串表示|使用StringBuilder优化性能|
|**equals()/hashCode()**|重写以实现内容比较|必须同时重写，保持契约|
|**泛型数组**|类型擦除导致转型警告|使用@SuppressWarnings注解|
|**异常处理**|非法参数和并发修改|提供清晰的错误信息|

### 8.2 设计原则应用

1. **单一职责原则**：`ArraySet`专注于集合存储，迭代逻辑由独立类处理
    
2. **开闭原则**：通过迭代器模式，遍历算法可扩展而不修改集合
    
3. **里氏替换原则**：正确实现`equals()`确保子类可替换父类
    
4. **接口隔离原则**：`Iterable`和`Iterator`接口职责分离
    
5. **依赖倒置原则**：客户端依赖抽象(`Iterable`)，而非具体实现
    

### 8.3 常见陷阱与解决方案

```java

// 陷阱1：迭代器中的并发修改
ArraySet<Integer> set = new ArraySet<>();
set.add(1);
set.add(2);

// 错误：在迭代过程中修改集合
for (Integer num : set) {
    if (num == 1) {
        set.add(3);  // 可能抛出ConcurrentModificationException
    }
}

// 解决方案：先收集要修改的内容，再统一处理
List<Integer> toAdd = new ArrayList<>();
for (Integer num : set) {
    if (num == 1) {
        toAdd.add(3);
    }
}
set.addAll(toAdd);

// 陷阱2：equals()实现不完整
@Override
public boolean equals(Object obj) {
    // 缺少null检查
    if (this.getClass() != obj.getClass()) {  // 如果obj为null，抛出NullPointerException
        return false;
    }
    // ...
}

// 正确做法：先检查null
@Override
public boolean equals(Object obj) {
    if (obj == null) return false;
    if (this.getClass() != obj.getClass()) return false;
    // ...
}
```
### 8.4 进一步学习建议

1. **阅读源码**：研究Java标准库中`HashSet`、`TreeSet`的实现
    
2. **性能分析**：比较不同集合实现的性能特性
    
3. **设计模式**：深入了解迭代器模式及其变体
    
4. **并发集合**：学习`ConcurrentHashMap`、`CopyOnWriteArraySet`等线程安全集合
    
5. **函数式编程**：探索Java 8+的Stream API与集合的结合使用
    

## 总结

通过实现`ArraySet`，我们深入理解了：

1. 集合数据结构的基本特性与实现
    
2. 迭代器设计模式的工作原理
    
3. Java对象方法的正确重写方式
    
4. 泛型编程的实际应用
    
5. 工业级代码的质量要求