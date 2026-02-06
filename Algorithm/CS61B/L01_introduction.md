## 一、Java 编程入门：从 Python 到 Java

课程通过对比 **Python** 和 **Java** 来引入概念，这种对照方法能清晰地揭示 Java 的设计哲学，即使没有 Python 基础也能直观理解。

### 1. Hello World 程序对比

- **Python**: `print("hello world")`
    - **特点**：直接、简洁，是解释型脚本语言的典型风格。
- **Java**:

```java

public class HelloWorld {
	public static void main(String[] args) {
		System.out.println("hello world");
	}
}
```
- **核心差异**：
	
	1. **强制封装**：Java 中所有代码都必须存在于 **类（class）** 中。`HelloWorld` 类就是一个容器。
		
	2. **程序入口**：`public static void main(String[] args)` 是 JVM 约定的固定入口方法，程序由此开始执行。
		
	3. **输出方式**：通过 `System.out.println` 调用标准输出流对象的方法，体现了 Java 的面向对象特性。
		

### 2. Hello Numbers 程序与变量声明

**目标**：使用循环打印 0 到 9 的数字。

- **Python**:
``` python
x = 0  # 动态类型：x 被自动识别为整数
while x < 10:
	print(x)
	x = x + 1
```
- **Java**:

```java

int x; // 1. 声明：必须显式指定变量 x 为 int（整数）类型
x = 0; // 2. 初始化：然后才能赋值
// 常用写法：int x = 0; // 声明与初始化合并
while (x < 10) {
	System.out.println(x);
	x = x + 1;
}
```
- **关键概念**：**静态类型**
    
    - **声明要求**：Java 中，变量**必须先声明类型，后使用**。这允许编译器在编译阶段知晓 `x` 的内存大小和允许的操作。
        
    - **类型安全**：一旦声明为 `int`，`x` 就不能再被赋值为字符串（如 `x = "horse"`），否则会引发 **编译错误**，从而在运行前杜绝此类类型错误。
        
    - **工程意义**：这种严格性牺牲了部分编写灵活性，但换来了代码的**可靠性、可读性和运行时效率**。
        

---

## 二、函数/方法定义深度对比

### 1. Python 函数

```python

def larger(x, y):  # 无需声明参数和返回值的类型
    if x > y:
        return x
    else:
        return y
print(larger(-5, 10)) # 输出 10
```
**特点**：灵活，函数可以作为独立单元存在。

### 2. Java 方法

在 Java 中，所有“函数”都必须定义在类内部，因此它们被称为 **“方法”**。这强化了面向对象的“一切皆对象，行为封装在类中”的理念。

```java

public class LargerDemo {
    // 方法定义：必须明确指定参数类型和返回值类型
    public static int larger(int x, int y) {
        if (x > y) {
            return x;
        } else {
            return y;
        }
    }

    public static void main(String[] args) {
        // 方法调用：在同一类中可直接调用静态方法
        System.out.println(larger(-5, 10)); // 输出 10
    }
}
```
- **语法解剖**：
    
    - `public static`：访问修饰符。`static` 表示该方法属于类本身，而非类的某个实例，因此无需创建对象就能调用（在 `main` 中直接使用）。
        
    - `int`：**返回值类型**。方法必须声明其返回结果的数据类型，若无返回值则使用 `void`。
        
    - `(int x, int y)`：**参数列表**。每个参数都必须单独声明类型。
        
- **与Python的核心区别**：Java 的方法签名（方法名+参数类型列表）包含了丰富的类型信息，编译器借此进行严格的类型检查。**Java 方法只能返回一个值**，而 Python 函数可以返回多个值（实质上是返回一个元组）。
    

---

## 三、编译与解释：深入理解静态类型的优势与代价

### 1. Java 的两步执行过程

Java 被称为“编译型”语言，但其编译过程有其特殊性：

1. **编译 (Compilation)**：
    
    - 使用 `javac` 命令将 `.java` 源代码文件编译成 `.class` 字节码文件。
        
    - **核心活动**：编译器进行**语法检查**和**类型检查**。如果发现类型不匹配（如将 `String` 赋给 `int`），编译会**失败**，并给出明确的错误信息，不会生成 `.class` 文件。
        
2. **解释/运行 (Interpretation/Execution)**：
    
    - Java 虚拟机 (JVM) 加载并解释执行 `.class` 字节码文件。
        
    - 因为类型检查已在编译期完成，所以 JVM 的运行时负担更轻。
        

### 2. 静态类型语言的优缺点分析

|维度|优点|缺点（或代价）|
|---|---|---|
|**可靠性**|**提前捕获错误**：类型错误在编译时即被暴露，方便在开发阶段修复，避免了程序交付后因类型问题崩溃。|**编写繁琐**：代码更冗长，需要大量类型声明，降低了编写简单脚本的速度。|
|**可维护性**|**代码即文档**：类型声明明确了方法对输入和输出的要求，使代码更易于他人理解和维护。|**灵活性降低**：代码通用性变差。例如，需要一个能比较`整数`、`浮点数`甚至`字符串`的 `larger` 函数，在 Java 中可能需要重载多个方法或使用泛型（后续学习）。|
|**性能**|**运行效率高**：编译器知晓确切类型，可生成更优的机器码，且运行时无需进行动态类型检查和推断。|**设计复杂度**：在涉及高度抽象和灵活性的设计时，类型系统有时会带来额外的设计挑战。|
|**开发体验**|**强大的 IDE 支持**：IDE 可以利用类型信息提供精准的代码补全、重构和错误提示。|**学习曲线**：需要理解并熟练运用类型系统，对初学者有一定门槛。|

---

## 四、面向对象编程 (OOP) 核心概念与实践

### 1. 核心思想

- **封装**：将**数据（属性/字段）** 和对数据的**操作（方法）** 捆绑在一起，形成一个独立的单元——**对象**。对外隐藏内部实现细节，仅通过定义好的接口进行交互。
    
- **类**：创建对象的**蓝图**。它定义了一类对象共有的属性和方法。
    
- **对象**：根据类创建出的**具体实例**。每个对象都拥有类中定义的属性（但值可能不同）和方法。
    

### 2. 实践：构建一个“汽车”类

下面通过一个逐步演进的 `Car` 类来展示 Java OOP 的关键语法。

```java

// Car.java
public class Car {
    // 1. 实例变量 (Instance Variables) / 字段 (Fields)
    // 用于描述对象的状态，必须在类内部、方法外部声明。
    String model; // 汽车型号
    int wheels;   // 轮子数量

    // 2. 构造函数 (Constructor)
    // 用于初始化新创建的对象。名称必须与类名完全相同。
    public Car(String model, int wheels) {
        // ‘this’ 关键字用于区分同名的实例变量和参数
        this.model = model; // 将参数model的值，赋给当前对象的model字段
        this.wheels = wheels;
    }

    // 3. 方法 (Methods) - 定义对象的行为
    // a. 修改状态的方法
    public void driveIntoDitch() {
        if (this.wheels > 0) {
            this.wheels--; // 减少一个轮子
            System.out.println(this.model + " 掉进沟里，轮子还剩 " + this.wheels + " 个。");
        }
    }

    // b. 返回信息的方法
    public int getNumWheels() {
        return this.wheels; // 返回当前轮子数量
    }

    public void printStatus() {
        System.out.println("这是一辆" + this.model + "，它有" + this.wheels + "个轮子。");
    }
}
```
- **`this` 关键字详解**：用于引用**当前对象**的实例变量或方法。当构造函数的参数名与实例变量名相同时，`this.model` 明确指向当前对象的 `model` 字段。如果不存在命名冲突，可以省略 `this.`。


### 3. 创建并使用对象

```java

// Main.java (包含 main 方法的类)
public class Main {
    public static void main(String[] args) {
        // 使用 ‘new’ 关键字和构造函数创建对象
        Car myCar = new Car("Toyota Camry", 4);
        Car yourCar = new Car("Ford Focus", 4);

        // 调用对象的方法
        myCar.printStatus(); // 输出：这是一辆Toyota Camry，它有4个轮子。
        myCar.driveIntoDitch(); // 输出：Toyota Camry 掉进沟里，轮子还剩 3 个。

        // 每个对象的状态是独立的
        yourCar.printStatus(); // 输出：这是一辆Ford Focus，它有4个轮子。(状态未受影响)
        System.out.println(myCar.getNumWheels()); // 输出：3
    }
}
```
---

## 五、总结

### 核心概念对比表

| 主题        | Java 关键特性                               | 与 Python 的关键差异                  | 学习重点                                             |
| --------- | --------------------------------------- | ------------------------------- | ------------------------------------------------ |
| **程序结构**  | 一切皆在类中，程序入口是 `main` 方法。                 | Python 脚本可独立执行。                 | 理解 `public static void main(String[] args)` 的含义。 |
| **变量**    | **静态类型**：先声明（类型），后使用。                   | **动态类型**：类型随赋值而定。               | 掌握基本数据类型（`int`, `double`, `boolean`等）的声明与初始化。    |
| **函数/方法** | 称为“方法”，必须定义在类内，需声明返回类型和参数类型。            | 称为“函数”，可独立定义，无需类型声明。            | 理解方法签名、`static` 关键字，以及如何调用方法。                    |
| **面向对象**  | 严格基于类的 OOP，使用 `new` 创建对象，`this` 引用当前对象。 | 同样支持 OOP，但语法更灵活（如 `self` 显式传递）。 | 理解类与对象的关系，掌握构造函数、实例变量和实例方法的定义与使用。                |
| **错误处理**  | **编译时类型检查**，大量错误在运行前被捕获。                | **运行时类型检查**，错误在执行到该行时才暴露。       | 学会阅读和理解 Java 编译器的错误信息。                           |
|           |                                         |                                 |                                                  |
