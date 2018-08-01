# 计算机系统综合实验报告

计53 张耀楠 戴臻旸 王润基

2018.08.01

## 实验概况

### 实验目标

以RISCV32为核心，分别实现编译器、OS移植、CPU，并以CPU为主要任务。

* 编译：将Decaf编译器后端移植到RISCV
* OS：将RustOS移植到RISCV
* CPU：实现支持RV32I标准的单核CPU

### 时间安排

共5周：

第一周：学习研究RISCV和工具链。完成Decaf后端的移植。开始RustOS移植。

第二周：完成RustOS移植。开始进行CPU的设计和实现。

后三周：CPU实现和调试。

### 实验结果

* 编译：
  * 完成从 MIPS 后端到 RV32I 后端的移植
  * 使用 C 实现简单的一个运行库，以 Decaf 生成代码和运行库链接生成可执行文件
  * 通过编译 PA4 测试
  * 可在QEMU上以riscv-pk、uCore、RustOS为后端运行，以及在板子上 bare metal 环境中运行
* OS：
  * 之前x86上已实现的功能都移植到RISCV32，并可正常运行。
  * 之前没有实现的部分平台无关功能（文件系统）由于时间原因也没有实现。
* CPU：
  * 实现了经典五段流水，50MHz主频的CPU
  * 可在板子上正常运行SystemOnCat组提供的监控程序
  * 正在持续改进以通过官方[riscv-tests](https://github.com/riscv/riscv-tests)的测例

## Decaf
Decaf 的工作分为两部分，
一个是生成 RiscV 的汇编代码，这个是编译的要求．
另一个是移植 Decaf 的内置函数如 `Print`, `new`　等．

### Decaf 后端代码生成
Decaf 框架中，后端代码生成的流程是
1. 对中间代码 TAC 做数据流分析．
2. 利用第一步的结果，完成寄存器分配，将 TAC 中有无限个的虚拟寄存器变成物理寄存器．
3. 使用目标架构的指令完成 TAC 需要的操作，如 TAC 的 `ADD` 对应 RiscV 指令 `addi` 和 `add`.

这方法最大的问题是优化能力不足（实际上 Decaf 可以说唯一的优化就是死代码删除）．
数据流分析止步于单个基本块．
每次离开基本块都需要保存变量到内存，进入基本块又需要加载变量到寄存器．
最大的优点在于简单，最后每条 TAC 对应的是确定的汇编指令串．

### 生成汇编代码
为了使得 Decaf 框架生成 Riscv 汇编代码，我们做了如下的改动

1. 将代码中 Mips 全都改成 Riscv

2. 修改寄存器: 主要是改名

3. 修改一些指令的实现．如 Riscv 没有 `slt r1, r2, r3` 指令．
最简单的方法就是把它变为 `sub r1 r2 r3; snez r1`.

可以看出，改动相对较小．
我们认为这是因为 Riscv 天生就和 Mips 非常相似的缘故，
毕竟它们都是 Patterson 那帮人设计的．

### 运行时
生成的汇编代码还无法运行，因为他们还缺乏 `Print` `new` 等函数的实现，
如何方便地实现 Decaf 语言自带的 `Print`, `new` 等函数呢？

一种方法是采用汇编编写，然后把汇编代码硬编码入 Decaf 代码生成框架．
但是这样不仅会遇到汇编代码难以编写的问题，更重要的是函数实现的变化变得非常繁琐．

我们采用的方法是，使用 C 语言实现一套 `Print`, `new` 等，相当于一个函数库．
Decaf 生成的汇编代码和这个函数库最后链接即可．实际使用中这套方法非常有效，
我们针对三种平台（riscv-pk，rucore，baremetal）各实现了一套库函数，
这样我们的 Decaf 程序只需要和不同的库链接，就能运行在不同的平台上．

另外，因为 Decaf 框架和 Riscv 标准的 ABI 有不同，
如 Decaf 用 x86 式的栈传参，而 Riscv 使用寄存器传参，
因此我们还是要在后端框架中包装一下库函数．
不过这个包装只是把参数从栈上读进寄存器，然后就调用库函数执行具体的功能了．

### 测试运行
Decaf 相关代码在 [github](https://github.com/riscv-and-rust-and-decaf/decaf-riscv-compiler) 上．
后端生成的测试在 `decaf-riscv-compiler/TestCases/{rv,wrjlibc,baremetal}` 中．

* **riscv-pk 环境下测试**: 在 `TestCases/rv` 下运行 `./testall` 即可. 需要预先安装 riscv-tools.
可以使用我们的编译好的版本 [riscv-tools](https://github.com/riscv-and-rust-and-decaf/riscv-prebuilt-toolchains/releases).
下载后需要将 `bin` 加入 `$PATH` 中.

* **rucore 环境下测试**: 执行如下命令
```
TestCases/wrjlibc $ HAS_LIBC=0 BAREMETAL=0 make
TestCases/wrjlibc $ cd ../rv
TestCases/rv $ HAS_LIBC=0 BAREMETAL=0 make exe
```
然后选择你希望运行的可执行文件, 利用 `mksfs` 放入 riscv-ucore 的磁盘中, 启动 riscv-ucore / rucore 就应该可以看到该程序了, 运行即可.

* **baremetal 环境下测试**: 执行如下命令
```
TestCases/wrjlibc $ HAS_LIBC=0 BAREMETAL=1 make
TestCases/wrjlibc $ cd ../baremetal
TestCases/baremetal $ make
```
然后将希望运行的 `.bin` 文件烧入 baseram 从 0 开始的位置, 再烧入 CPU 的 `.bit` 文件,
重置后应当在串口看到输出.

如果 `make` 在链接阶段出错, 检查 `wrjlibc` 是否使用了正确的 `HAS_LIBC` 和 `BAREMETAL`.

## RustOS

TODO

## CPU

TODO
