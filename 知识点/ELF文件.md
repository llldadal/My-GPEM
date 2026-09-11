# ELF文件
## 总述
ELF 是一种文件格式，全名叫 Executable and Linkable Format，可以翻成“可执行与可链接格式”。

其包含几个类别
- 可重定位目标文件
- 共享库文件
- 可执行文件

## 具体组织

### 结构
```text
ELF 文件
│
├─ ELF Header      我是什么类型的文件
├─ .text           机器指令
├─ .data           已初始化全局数据
├─ .bss            零初始化/未初始化全局数据
├─ .symtab         符号表
├─ .strtab         字符串表
└─ .rela.text      重定位信息
```
下面以如下示例程序具体介绍重要的部分
```c
// main.c

extern int global;

int add(int, int);

int main()
{
    return add(global, 2);
}
```
关于显示一个ELF文件的结构，可执行：
```bash
readelf -S main.o
```

---

### ELF Header
ELF Header表名了该ELF文件具体属于哪一种，可执行命令：
```bash
readelf -h main.o
```
输出：

```text
Class: ELF64
Data: 2's complement, little endian
Type: REL
Machine: Advanced Micro Devices X86-64
```
其中主要看Type字段：
- REL 可重定位目标文件
- EXEC 可执行文件
- DYN 共享库文件
- NONE 未知或未指定文件

根据这个，我们可以判断当前ELF文件的类型

---

### .symtab 符号表
符号表存储了当前ELF文件使用符号的定义和引用状态

可使用命令：
```bash
readelf -s main.o
```
来查看文件的符号表

```text
Num: Value Size Type   Bind   Vis      Ndx Name

...
4:   0     0    NOTYPE GLOBAL DEFAULT UND global
5:   0     0    NOTYPE GLOBAL DEFAULT UND add
6:   0    20    FUNC   GLOBAL DEFAULT   1 main
```
- Name 符号名
- Ndx 定义状态: 
    - 如果为数字i，代表定义在本文件的第i个节
    - 如果为UND，则代表只是引用，不在本文件定义
- Bind 符号绑定属性: 
    - LOCAL: 静态符号，只能在本文件被引用
    - GLOBAL：可被其他文件引用的符号，如果是已定义的符号，通常为强符号
    - WEAK: 可被其他文件引用的符号，如果是已定义的符号，通常为弱符号

如果只关注定义状态，可使用命令：
```bash
nm main.o
```
查看简化的符号表
```text
                 U add
                 U global
0000000000000000 T main
```
- U: 引用，未定义
- T: 定义在本文件.text节

---

### .rela.text 重定位信息表
