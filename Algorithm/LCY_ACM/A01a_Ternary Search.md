# 三分查找（Ternary Search）笔记扩展

## 🔍 一、三分查找的核心概念

### 1. 基本定义

三分查找是一种**在凸函数（单峰函数）上寻找极值点**的算法，通过每次将搜索区间缩减为原来的2/3来逼近极值点。

### 2. 适用条件

- **函数必须是凸函数（单峰函数）**
    
    - 上凸函数（concave）：有最大值点
        
    - 下凸函数（convex）：有最小值点
        
- **不要求函数可导或连续**（但需要能在区间内计算函数值）
    
- **适用于不可导的分段函数**
    

---

## 📊 二、三分查找 vs 二分查找

|特性|二分查找|三分查找|
|---|---|---|
|**适用对象**|单调序列/函数|凸函数（单峰函数）|
|**查找目标**|特定值（target）|极值点（最大值/最小值）|
|**比较对象**|目标值与中点值|两个三等分点的函数值|
|**区间缩减**|每次减半（1/2）|每次减为2/3|
|**时间复杂度**|O(log₂n)|O(log₁.₅n) ≈ O(log₂n)|
|**应用场景**|有序数组查找、单调函数求根|凸函数优化、单峰函数极值|

---

## 🔧 三、三分查找的详细步骤

### 1. 基本算法流程


```text

1. 初始化搜索区间 [left, right]，设置精度阈值 eps
2. while (right - left > eps):
    计算两个三等分点：
        mid1 = left + (right - left) / 3
        mid2 = right - (right - left) / 3
    计算 f(mid1) 和 f(mid2)
    
    根据函数凸性更新区间：
    上凸函数（求最大值）：
        if f(mid1) < f(mid2): left = mid1  # 极值点在右区间
        else: right = mid2                  # 极值点在左区间
    下凸函数（求最小值）：
        if f(mid1) > f(mid2): left = mid1  # 极值点在右区间
        else: right = mid2                  # 极值点在左区间
3. 返回 (left + right) / 2 作为极值点近似值
```
### 2. 区间更新逻辑解析

#### 上凸函数（求最大值）：

```text

    f(mid1)       f(mid2)
       ↓            ↓
    ___/¯¯¯¯¯¯¯¯¯¯¯\___
      left   mid1  mid2   right
 ```     
情况1：f(mid1) < f(mid2)
    → 极值点肯定在 [mid1, right]
    → 舍弃左区间 [left, mid1]
    
情况2：f(mid1) ≥ f(mid2)
    → 极值点肯定在 [left, mid2]
    → 舍弃右区间 [mid2, right]

#### 下凸函数（求最小值）：

```text

    f(mid1)       f(mid2)
       ↓            ↓
    ¯¯¯\____________/¯¯¯
      left   mid1  mid2   right
```
情况1：f(mid1) > f(mid2)
    → 极值点肯定在 [mid1, right]
    → 舍弃左区间 [left, mid1]
    
情况2：f(mid1) ≤ f(mid2)
    → 极值点肯定在 [left, mid2]
    → 舍弃右区间 [mid2, right]

---

## 💻 四、代码实现示例

### 1. 递归实现（下凸函数求最小值）

```cpp

double ternary_search(double left, double right, double eps) {
    if (right - left < eps) {
        return (left + right) / 2;
    }
    
    double mid1 = left + (right - left) / 3;
    double mid2 = right - (right - left) / 3;
    
    double f1 = f(mid1);  // 计算函数值
    double f2 = f(mid2);
    
    if (f1 > f2) {
        // 对于下凸函数，f1 > f2 说明最小值在 [mid1, right]
        return ternary_search(mid1, right, eps);
    } else {
        // 否则最小值在 [left, mid2]
        return ternary_search(left, mid2, eps);
    }
}
```
### 2. 非递归实现（上凸函数求最大值）

```cpp

double ternary_search(double left, double right, double eps) {
    while (right - left > eps) {
        double mid1 = left + (right - left) / 3;
        double mid2 = right - (right - left) / 3;
        
        double f1 = f(mid1);
        double f2 = f(mid2);
        
        // 对于上凸函数，求最大值
        if (f1 < f2) {
            left = mid1;  // 极值点在右区间
        } else {
            right = mid2; // 极值点在左区间
        }
    }
    return (left + right) / 2;
}
```
---

## 📐 五、精度控制与迭代次数

### 1. 精度计算公式

初始区间长度为 ==`L = right - left`==  
经过 `k` 次迭代后，区间长度变为：

$$
L_k = L \times \left(\frac{2}{3}\right)^k
$$
### 2. 所需迭代次数

要达到精度 `eps` 所需的迭代次数：


$$
k = \log_{1.5}\left(\frac{L}{\text{eps}}\right) = \frac{\ln(L/\text{eps})}{\ln(1.5)}
$$
### 3. 实用迭代方式


```cpp

// 常用精度控制方式
const double eps = 1e-7;  // 1e-6到1e-9通常足够

// 方法1：直接比较区间长度
while (right - left > eps) {
    // 三分查找逻辑
}

// 方法2：固定迭代次数（更稳定，避免无限循环）
for (int iter = 0; iter < 100; iter++) {
    // 三分查找逻辑
    // 100次迭代通常能达到很高精度
}
```


- 推导
$$
\begin{aligned}
L_k &= L \times \left(\frac{2}{3}\right)^k \\
\text{要求} & L_k \leq eps \\
L \times \left(\frac{2}{3}\right)^k &\leq eps \\
\left(\frac{2}{3}\right)^k &\leq \frac{eps}{L} \\
k \times \ln\left(\frac{2}{3}\right) &\leq \ln\left(\frac{eps}{L}\right) \\
k &\geq \frac{\ln(eps/L)}{\ln(2/3)} \\
k &\geq \frac{\ln(L/eps)}{\ln(3/2)} \\
k &\geq \log_{1.5}\left(\frac{L}{eps}\right)
\end{aligned}
$$

- 近似公式：
$$
k \approx 1.71 \times \ln\left(\frac{L}{eps}\right)
\quad \text{(因为 } \ln(1.5) \approx 0.405 \text{)}
$$
---

## 🎯 六、实战技巧与注意事项

### 1. **凸性验证**

在应用三分查找前，务必验证函数在搜索区间内的凸性：

- 绘制函数图像观察
    
- 计算二阶导数（如果可导）
    
- 采样多个点验证单峰特性
    

### 2. **边界条件处理**

```cpp

// 检查边界是否为极值点
double left_val = f(left);
double right_val = f(right);

// 如果边界值更优，可能需要调整搜索逻辑
if (left_val < min_val) {
    min_val = left_val;
    min_point = left;
}
```
### 3. **数值稳定性**

- 避免在函数值非常接近时过早停止
    
- 考虑使用相对误差而非绝对误差
    
- 对于非常平坦的函数区域，可能需要特殊处理
    

### 4. **整数三分查找**

```cpp

// 当函数定义域为整数时
int ternary_search_int(int left, int right) {
    while (right - left > 3) {  // 区间足够大时使用三分
        int mid1 = left + (right - left) / 3;
        int mid2 = right - (right - left) / 3;
        
        if (f(mid1) > f(mid2)) {
            left = mid1;
        } else {
            right = mid2;
        }
    }
    
    // 最后的小区间直接枚举
    int best = left;
    for (int i = left + 1; i <= right; i++) {
        if (f(i) > f(best)) {
            best = i;
        }
    }
    return best;
}
```
---

## 🌟 七、经典例题与应用

### 1. **HDU 2899 - Strange Function**

```cpp

// 求函数 f(x) = 6x^7 + 8x^6 + 7x^3 + 5x^2 - y*x 在 [0,100] 的最小值
// 这是三分查找的经典例题
```
### 2. **常见应用场景**

- **物理问题**：寻找最优抛射角度、最小能量路径
    
- **经济学**：利润最大化、成本最小化
    
- **工程设计**：最优参数选择、材料最省设计
    
- **机器学习**：凸函数优化、参数调优
    

### 3. **变种问题**

- **多峰函数**：需要先分割为多个单峰区间
    
- **离散函数**：定义域为整数的凸函数
    
- **高维凸函数**：需要结合其他优化方法
    

---

## ⚠️ 八、常见错误与陷阱

### 1. **误用于非凸函数**

- 三分查找只能用于单峰函数
    
- 多峰函数会找到局部极值而非全局极值
    

### 2. **精度设置不当**

- `eps` 过大会导致结果不精确
    
- `eps` 过小会导致浮点数误差累积或无限循环
    

### 3. **函数凸性判断错误**

- 必须确保在整个搜索区间内函数都是凸的
    
- 边界附近可能出现凸性变化
    

### 4. **浮点数比较问题**

```cpp

// 错误：直接比较浮点数
if (f1 == f2) { ... }

// 正确：使用容差比较
if (fabs(f1 - f2) < 1e-10) {
    // 函数值非常接近，需要特殊处理
    // 可以选择任意方向更新，或进一步细分
}
```
---

## 🔄 九、三分查找的优化变体

### 1. **黄金分割搜索（Golden Section Search）**

- 使用黄金比例（约0.618）而非三等分
    
- 只需计算一个新的函数值每次迭代
    
- 理论最优的区间缩减比例
    

### 2. **二次插值法**

- 用二次函数拟合当前三点
    
- 直接计算拟合函数的极值点
    
- 收敛速度更快（超线性收敛）
    

### 3. **自适应三分查找**

- 根据函数变化率动态调整分割点
    
- 在平坦区域使用更激进的缩减策略
    
- 在陡峭区域使用更保守的策略
    

---

## 📈 十、复杂度分析与优化

### 1. **时间复杂度**

- 每次迭代：O(1) 次函数计算
    
- 总迭代次数：O(log₁.₅(L/eps)) ≈ O(log₂(L/eps))
    
- 实际运行：通常30-50次迭代足够
    

### 2. **空间复杂度**

- 非递归实现：O(1)
    
- 递归实现：O(log n) 栈空间
    

### 3. **优化技巧**

- **预计算**：如果函数计算代价高，可缓存计算结果
    
- **并行计算**：同时计算 f(mid1) 和 f(mid2)
    
- **提前终止**：当函数值变化很小时提前停止
    

---

## 📚 十一、学习建议与练习

### 1. **建议练习顺序**

1. 理解凸函数概念与判断方法
    
2. 实现基础的二分查找（对比理解）
    
3. 实现三分查找的基本框架
    
4. 解决HDU 2899等经典问题
    
5. 尝试变种问题和优化方法
    

### 2. **调试技巧**

```cpp

// 添加调试输出
void debug_ternary(double left, double right, int iter) {
    cout << "迭代 " << iter << ": [" << left << ", " << right << "]" << endl;
    cout << "  区间长度: " << right - left << endl;
}

// 在循环中添加
while (right - left > eps) {
    debug_ternary(left, right, iter_count++);
    // ... 三分查找逻辑
}
```
### 3. **扩展阅读**

- 数值分析中的最优化方法
    
- 凸优化理论
    
- 梯度下降法（对比学习）
    
- 牛顿法（更高阶的优化方法）
    

---

## 🎓 总结要点

|关键点|说明|
|---|---|
|**核心思想**|通过比较三等分点的函数值，每次排除1/3的搜索区间|
|**适用条件**|单峰函数（凸函数），不要求可导|
|**时间复杂度**|O(log n)，与二分查找同数量级|
|**精度控制**|通过设置eps或固定迭代次数控制|
|**常见错误**|误用于多峰函数、浮点数精度问题|
|**优化方向**|黄金分割、二次插值、自适应策略|

三分查找是解决**凸函数极值问题**的强大工具，在算法竞赛和工程优化中都有广泛应用。掌握其原理、实现细节和适用场景，能够有效解决一大类优化问题。