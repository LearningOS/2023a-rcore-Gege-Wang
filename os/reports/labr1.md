# 练习
跟随前面文档的指引，扩展 easy-fs-fuse，使得它可以生成同时包含 Rust 和 C 用户程序的镜像。修改内核，使得 hello 测例给出正常输出（即给出一行以 my name is 开头的输出，且 exit_code为0）。

## 解题思路
根据任务指导书，主要讲了 Makefile 的内容和参数传递的过程。也就是应用如何被放到文件系统，以及如何从操作系统拿出来。
根据指导书改进打包文件系统可以完成第一个任务。

对于执行一个程序来说，需要从文件系统加载程序到对应的位置才能进行下面的工作，操作系统内的加载器必须和应用程序的规范相配合才能顺利的找到数据的位置。二进制加载器将特定的内核级信息传递给用户进程，这些加载器是内核子系统的一部分，可以是内核内置的，也可以是内核模块。二进制加载器将二进制文件（程序）转换为系统上的进程。通常，每种二进制格式都有一个不同的加载器；

对于elf 格式的文件来说，进程的堆栈是这样的： https://articles.manugarg.com/aboutelfauxiliaryvectors.html
所以需要重新安排堆栈的布局。

## 问答问题
elf 文件和 bin 文件有什么区别？
```bash
$ file ch6_file0.elf
```
> ch6_file0.elf: ELF 64-bit LSB executable, UCB RISC-V, version 1 (SYSV), statically linked, stripped

```bash
$ file ch6_file0.bin
```
> ch6_file0.bin: data

```bash
$ riscv64-linux-musl-objdump -ld ch6_file0.bin > debug.S
```
> riscv64-linux-musl-objdump: ch6_file0.bin: file format not recognized
ch6_file0.bin 不是有效的 elf 格式文件。objdump 需要处理的是一个有效的二进制文件，否则无法正确解析。
elf 是一种通用的可执行文件格式，广泛用于多种操作系统，包括Linux、Unix等。它具有丰富的头部信息、节表、段表等结构，用于描述程序的各个部分。支持多种体系结构（如x86、ARM、RISC-V等），并且能够包含调试信息、符号表等。支持动态链接和共享库，允许在运行时解析和链接程序的不同部分。
bin 文件只包含程序的机器码和数据。缺乏像 ELF 文件那样详细的头部信息和段表。通常用于直接加载到内存并执行，而不涉及动态链接和运行时加载。