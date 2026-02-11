## 1. Shell 脚本基础

Shell 脚本允许你将一系列命令组合起来，并加入控制流（如条件判断、循环），从而完成复杂任务。与通用脚本语言不同，Shell 脚本专门为执行 Shell 相关任务（如创建命令管道、文件重定向）进行了优化[](https://juejin.cn/post/7109097359652421663)。

### 变量赋值与引用

在 Bash 中为变量赋值和使用变量的方法如下：


```bash

# 正确的赋值方式：等号两侧不能有空格
foo=bar

# 引用变量值
echo $foo  # 输出: bar

# 错误的示例：Shell 会将 `foo` 解释为命令名
foo = bar  # 命令不存在，会报错
```
### 字符串引号的区别

使用单引号和双引号定义字符串有重要区别：

|引号类型|变量是否扩展|示例与结果|
|---|---|---|
|**单引号 `'`**|**否** - 字符串内容完全按字面处理|`echo '$foo'` 输出 `$foo`|
|**双引号 `"`**|**是** - 字符串中的变量会被替换为值|`echo "$foo"` 输出 `bar`|

### 特殊变量

Bash 提供了多个特殊变量，用于访问脚本参数、进程状态等信息[](https://juejin.cn/post/7109097359652421663)[](https://awesome.ecosyste.ms/projects/github.com%2Fmirror4s%2Fmissing-semester)：

- `$0`：当前运行的脚本名称。
- `$1` 到 `$9`：脚本的第1到第9个参数。
- `$@`：所有参数的列表。
- `$#`：传递给脚本的参数个数。
- `$?`：**上一个命令的退出状态码**。0 通常表示成功，非0 表示失败。
- `$$`：当前 Shell 脚本的进程 ID (PID)。
- `!!`：完整的上一条命令（包括参数）。常用场景：权限不足时，用 `sudo !!` 快速重试。
- `$_`：上一条命令的最后一个参数。
    

### 返回码与条件执行

命令的退出状态码可用于控制执行流程：

```bash

# true 命令总是返回 0（成功），false 总是返回 1（失败）
true
echo $?  # 输出 0
false
echo $?  # 输出 1

# 使用 &&（与）和 ||（或）进行条件执行，它们都是短路运算符
false && echo "这行不会执行"
true || echo "这行也不会执行"
```
### 命令替换与进程替换

- **命令替换 `$(CMD)`**：先执行 `CMD` 命令，然后用其**标准输出**替换整个 `$(CMD)` 部分。
    

```bash

# 将当前目录列表赋值给变量
files=$(ls)
```
- **进程替换 `<(CMD)`**：先执行 `CMD` 命令，将其输出存入一个**临时文件**，然后用该临时文件的路径替换 `<(CMD)`。常用于期望接收文件路径而非标准输入的命令[](https://juejin.cn/post/7109097359652421663)[](https://awesome.ecosyste.ms/projects/github.com%2Fmirror4s%2Fmissing-semester)。
    

```bash

# 比较两个目录下的文件列表差异
diff <(ls dir1) <(ls dir2)
```
### 花括号扩展与通配符

花括号 `{}` 可以自动展开命令中的公共部分，非常适合批量操作[](https://juejin.cn/post/7109097359652421663)[](https://awesome.ecosyste.ms/projects/github.com%2Fmirror4s%2Fmissing-semester)：

```bash

# 同时创建多个目录
mkdir proj_{frontend,backend,data}

# 批量复制或转换文件
cp /path/to/file.{txt,md} /backup/  # 相当于复制 file.txt 和 file.md
convert image.{png,jpg}             # 相当于 convert image.png image.jpg
```
常用的通配符：

- `?`：匹配任意单个字符。
- `*`：匹配任意长度的任何字符（包括零个）。
    

---

## 2. 强大的 Shell 工具

### 查找文件：`find`

`find` 命令功能强大，可以递归地根据名称、类型、修改时间、大小等条件搜索文件，并能对找到的文件执行操作[](https://juejin.cn/post/7109097359652421663)[](https://awesome.ecosyste.ms/projects/github.com%2Fmirror4s%2Fmissing-semester)。

```bash

# 基础查找：按名称和类型
find . -name "*.tmp" -type f        # 查找当前目录下所有 .tmp 文件
find /home -name "src" -type d      # 查找 /home 下所有名为 src 的目录

# 按时间查找：找出最近一天修改过的文件
find . -mtime -1

# 按大小查找：找出大小在500k到10M之间的压缩包
find . -size +500k -size -10M -name "*.tar.gz"

# 对找到的文件执行操作：删除所有 .tmp 文件
find . -name "*.tmp" -exec rm {} \;

# 结合转换：将所有 PNG 转换为 JPG
find . -name "*.png" -exec convert {} {.}.jpg \;
```
### 查找文件内容：`grep`

`grep` 用于在文本中搜索匹配模式的行[](https://awesome.ecosyste.ms/projects/github.com%2Fmirror4s%2Fmissing-semester)。

```bash

# 基本搜索
grep "pattern" file.txt

# 递归搜索目录下所有文件
grep -r "TODO" ./src/

# 显示匹配行的上下文（前后各5行）
grep -C 5 "error" log.txt

# 反向选择，输出不匹配的行
grep -v "success" results.txt
```
### 如何学习命令用法

当你遇到不熟悉的命令或参数时，可以按以下顺序寻求帮助[](https://juejin.cn/post/7109097359652421663)：

1. **内置帮助**：大多数命令支持 `-h` 或 `--help` 参数，提供简明用法。
    
2. **手册页**：使用 `man command`（如 `man find`）查看完整、权威的说明文档。
    
3. **TLDR 页**：如果觉得手册页过于详细，可以安装 `tldr` 工具，它提供命令的**常用示例**，更直观实用。
    

### 搜索历史命令

高效复用历史命令能极大提升效率：

- **`Ctrl + R`**：进入反向搜索模式，输入关键词即可搜索历史命令。
    
- **`history | grep "keyword"`**：通过管道直接筛选包含特定关键词的历史命令。
    
- **`fzf`**：一个模糊查找工具，可以与历史命令结合 (`history | fzf`)，提供交互式、更友好的搜索体验[](https://blog.csdn.net/weixin_42089190/article/details/109279085)。
    

---

## 3. 进阶脚本与技巧

### Shell 函数 vs Shell 脚本

了解函数和脚本的区别很重要[](https://juejin.cn/post/7109097359652421663)：

|特性|Shell 函数|Shell 脚本|
|---|---|---|
|**语言**|必须与当前 Shell 语言相同（如 Bash）|可以是任何语言（需正确的 shebang）|
|**加载**|在 Shell 会话中定义，一次加载，多次使用|每次执行时都会加载一次文件|
|**执行环境**|**在当前 Shell 进程中运行**，可修改环境变量（如 `cd`）|在独立的子进程中运行，无法直接修改父 Shell 环境|
|**用途**|适合模块化、复用性高的代码块|适合独立、完整的任务|

**示例函数**：一个简单的导航函数。

```bash

# 定义函数：将当前目录保存到环境变量
marco() {
    export MARCO=$(pwd)
    echo "当前目录已保存: $MARCO"
}

# 定义函数：跳回保存的目录
polo() {
    if [ -n "$MARCO" ]; then
        cd "$MARCO"
        echo "已跳转至: $MARCO"
    else
        echo "错误：未设置 MARCO 目录"
    fi
}
```
_注：在脚本中定义的函数，需要使用 `source script.sh` 命令加载到当前 Shell 会话中才能直接调用[](https://blog.csdn.net/weixin_42089190/article/details/109279085)。_

### 提升脚本可移植性：使用 Shebang

脚本开头的 `#!`（称为 shebang）告诉系统使用哪个解释器来执行此脚本。为了提高在不同系统间的可移植性，推荐使用 `env` 来寻找解释器的路径[](https://juejin.cn/post/7109097359652421663)[](https://awesome.ecosyste.ms/projects/github.com%2Fmirror4s%2Fmissing-semester)。

```bash

#!/usr/bin/env bash
# 这是一个更可移植的 Bash 脚本开头

#!/usr/bin/env python3
# 这是一个 Python 脚本的开头
```
### 脚本调试工具：`shellcheck`

Shell 脚本的语法有些“微妙”，容易出错。`shellcheck` 是一个静态分析工具，可以检查脚本中的语法错误、常见陷阱，并给出改进建议[](https://juejin.cn/post/7109097359652421663)[](https://blog.csdn.net/weixin_42089190/article/details/109279085)。

```bash

# 安装 shellcheck (以 Debian/Ubuntu 为例)
sudo apt install shellcheck

# 检查你的脚本
shellcheck your_script.sh
```
---

## 4. 关键要点回顾

- **变量与字符串**：赋值无空格，引用加 `$`；单引号硬编码，双引号可扩展。
    
- **流程控制**：善用 `$?`、`&&`、`||` 基于命令成功与否决定执行路径。
    
- **高效操作**：掌握 `$(...)` 命令替换、`{...}` 花括号扩展和通配符，减少重复输入。
    
- **文件处理**：`find` 和 `grep` 是**文件搜索和内容过滤的利器**，结合 `-exec` 参数能实现批量自动化操作。
    
- **学习与复用**：遇到新命令，先 `--help`，再 `man`，或用 `tldr` 看例子；多用 `Ctrl+R` 或 `fzf` 搜索历史命令。
    
- **脚本与函数**：需要修改当前环境（如切换目录）时用函数；独立的、复杂的任务写成脚本。
    
- **追求稳健**：编写脚本时使用 `#!/usr/bin/env` 提升可移植性，并用 `shellcheck` 进行代码检查。