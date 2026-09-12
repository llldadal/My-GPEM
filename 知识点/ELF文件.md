# ELF文件
## 总述
ELF 是一种文件格式，全名叫 Executable and Linkable Format，可以翻成“可执行与可链接格式”。

常见类别包括：
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
├─ .symtab         完整符号表（文件可能不包含）
├─ .strtab         字符串表
└─ .rela.text      重定位信息（名称随架构和目标文件而异）
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
ELF Header 表明了该 ELF 文件的基本类型，可执行命令：
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
- DYN 共享目标文件，也可能是位置无关可执行文件（PIE）
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
重定位信息表储存了当前 ELF 文件的[重定位](链接.md#重定位)信息

可使用如下命令来显示重定位信息表的内容：
```bash
readelf -r main.o
```
输出类似如下：
```text
Relocation section '.rela.text'

Offset          Info           Type                 Symbol
000000000006    ...            R_X86_64_PC32        global
000000000012    ...            R_X86_64_PLT32       add
```
- Type：重定位类型名称。以下是 x86-64 平台上的常见示例，并非完整清单：
  - 普通 `.o` 文件中常见 `R_X86_64_PC32`、`R_X86_64_PLT32`、`R_X86_64_64`、`R_X86_64_32` 和 `R_X86_64_32S`
  - 使用 `-fPIC` 编译的 `.o` 文件中还常见 `R_X86_64_GOTPCREL`、`R_X86_64_GOTPCRELX` 和 `R_X86_64_REX_GOTPCRELX`
  - `.so` 文件的动态重定位中常见 `R_X86_64_RELATIVE`、`R_X86_64_GLOB_DAT`、`R_X86_64_JUMP_SLOT` 和 `R_X86_64_IRELATIVE`
- Symbol: 重定位符号名

以上比较需要关注的是 Type 字段，不同的 Type 对应不同的重定位计算方法。重定位类型可辅助判断编译方式，但实际结果还会受到代码写法、编译器版本、优化选项和链接方式影响，不能只凭单个类型下结论。

## 补充
最后，如果想要观察汇编代码，可以执行：
```bash
objdump -dr main.o
```
来进行反汇编，得到类似：
```text
0000000000000000 <main>:
   0:   ...
   6:   mov ...
        8: R_X86_64_PC32 global-0x4

  10:   call ...
        11: R_X86_64_PLT32 add-0x4
```
的含重定位信息的汇编代码（或许有用呢？）
