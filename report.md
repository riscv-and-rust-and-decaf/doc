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
  * TODO
  * 可在QEMU上以riscv-pk、uCore、RustOS为后端运行，以及在板子上bare metal环境中运行
* OS：
  * 之前x86上已实现的功能都移植到RISCV32，并可正常运行。
  * 之前没有实现的部分平台无关功能（文件系统）由于时间原因也没有实现。
* CPU：
  * 实现了经典五段流水，50MHz主频的CPU
  * 可在板子上正常运行SystemOnCat组提供的监控程序
  * 正在持续改进以通过官方[riscv-tests](https://github.com/riscv/riscv-tests)的测例

## Decaf

TODO

## RustOS

TODO

## CPU

TODO