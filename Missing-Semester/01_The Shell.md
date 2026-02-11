
### PATH

- show all of the path on machine
```shell
#linux/mac
echo $PATH

#powershell
echo $env:path
```

- separated way

| OS      | paths separated by |                  |
| ------- | ------------------ | ---------------- |
| linux   | `/`                | 所有内容属于根命名空间      |
| mac     | `/`                | 所有内容属于根命名空间      |
| windows | `\`                | 每个驱动器都有一个单独的路径结构 |
- **绝对路径**：能完全确定文件位置的路径
- **相对路径**：相对于你当前所在位置的路径

##### 找到当前位置  ==`pwd`==
```shell
pwd
```

##### 更改目录  ==`cd`==
```shell
cd path
```

-  **当前目录**： **==`.`==**
-  **父目录**： **==`..`==**
-  **主目录**： **==`~`==**
-  **两个不同目录间切换**： **==`-`==**
-  **显示帮助（大多都有）**： **==`--help`==**


##### 快速浏览目录内容  ==`ls`==
```shell
ls 
ls ..
ls ./path
ls -l #更详细的显示目录下内容
```

##### 移动目录/重命名  ==`mv`==

##### 复制文件  ==`cp`==

##### 删除文件  ==`rm`==

##### 删除目录  ==`rmdir`==

##### 新建目录  ==`mkdir`==

##### 打印文件内容  ==`cat`==

##### 使用`man`获取帮助信息  ==`man program`==

```shell
man ls
```

##### 打开文件（Linux）  ==`xdg-open`==

##### 搜索  ==`grep`==

### 多文件协作

#### 重定向

- ==`<`== :将这个程序的输入重定向为这个文件的内容
- ==`>`== :将前面程序的输出重定向到这个文件中
- ==`>>`== :表示追加

```shell
echo hello > hello.text
cat hello.txt
#hello

cat < hello.txt
#hello

cat < hello.txt >hello2.txt
cat hello2.txt
#hello

cat < hello.txt >> hello2.txt
cat hello2.txt
#hello
#hello
```

- ==`|`== 管道：将左边程序的输出作为右边程序的输入

```shell
ls -l / | tail -n1
# .....


ls -l / | tail -n1 > ls.txt #输出到ls.txt
cat ls.txt
# .....(同上)
```


#### 权限

- root权限运行shell
	- `#` 表示root超级管理员运行；`$` 非root用户运行
```shell

/sys
cd backlight/
cd intel_backlight/
cat brightness
# 1060(显示当前屏幕亮度)

echo 500 > brightness
#出错！没有权限。

sudo echo 500 > brightness
#依旧出错！因为运行的shell不是root身份（输入和输出的重定向是程序不知道的，通道和重定向是由shell设置的，打开内核文件brightness的不是sudo而是shell，所以不被允许）


#方法一
sudo su #获取root身份的shell
echo 500 > brightness#（更改成功！）


#方法二
echo 1060 | sudo tee brightness#（更改成功！）
#1060
#打开brightness文件的程序tee是作为root用户运行的 
```

- ==`tee`==:将输入的内容写入一个文件，同时也将它输出到标准输出。

```shell
# 例：手动打开滚轮锁定指示灯。

/sys
cd leds/
cd input4\:\:scrolllock/
cat brightness
#0
echo 1 | sudo tee brightness
#1
```