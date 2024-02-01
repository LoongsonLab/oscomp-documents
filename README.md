# StarryOS LoongArch64 文档

这个文档 `README.md` 是StarryOS的LoongArch64分支的文档说明


 - [代码组织](/code_glossary.md) - 介绍相关LA相关代码结构
 - [Getting started](/getting_started.md) - 如何使用StarryOS，介绍新手如何快速入门
 - [启动流程](/bootstrap.md) - 简要的分析一个应用启动的流程，以及如何增加syscall
 - [平台支持](/platforms.md) - 目前此LA分支支持的平台，此外还包括常见的编译工具链说明
 - [应用迁移](/app_ports.md) - 如何进行应用的迁移（待补充）
 - [测试集](/testsuits.md) - 用于OSComp的一系列测试集，包括如何编译和使用
 - [调试相关](/debug_skills.md) - 一些常见的调试方法与技巧


### 说明

此文档会持续更新，其中涉及到的新的问题（比如调试），都会在此查阅。目的是为了更方便的解决移植
期间遇到的各种问题。




# 比赛参考资料

### LoongArch架构相关文档

 - [龙芯架构参考手册卷一](https://github.com/LoongsonLab/oscomp-documents/blob/main/pdf/%E9%BE%99%E8%8A%AF%E6%9E%B6%E6%9E%84%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C%E5%8D%B7%E4%B8%80.pdf)
 
 - [计算机体系结构基础(LoongArch)(第三版)](https://github.com/LoongsonLab/oscomp-documents/blob/main/pdf/%E8%AE%A1%E7%AE%97%E6%9C%BA%E4%BD%93%E7%B3%BB%E7%BB%93%E6%9E%84%E5%9F%BA%E7%A1%80(LoongArch)-3rd.pdf)

 - [LoongArch 系统调用(syscall)ABI](https://github.com/LoongsonLab/oscomp-documents/blob/main/pdf/LoongArch%20%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8(syscall)ABI.pdf)

 - [LoongArch-工具链约定](https://github.com/LoongsonLab/oscomp-documents/blob/main/pdf/LoongArch-%E5%B7%A5%E5%85%B7%E9%93%BE%E7%BA%A6%E5%AE%9A.pdf)

 - [LoongArch ELF ABI(中文版)](https://github.com/LoongsonLab/oscomp-documents/blob/main/pdf/LoongArch-ELF-ABI-CN.pdf)

 - 更多龙架构相关文档，可以参考[龙芯开源社区](http://loongnix.cn)，[龙芯中科公司官网](https://loongson.cn/)，[龙芯在github的官方账号](https://github.com/loongson)以及[龙芯实验室为大赛设置的文档仓库](https://github.com/LoongsonLab/oscomp-documents)


### 内核赛道选用的2K1000开发板参考资料

 - [开发板资料包 *提取码：1111* ](https://pan.baidu.com/s/1yWOJT8TD1tLlNtY6UW_QFQ?pwd=1111)。其中包括但不限于开发板和2k1000处理器用户手册，主板设计资料，uboot、内核和文件系统二进制以及源代码等信息。

 - [在线论坛](https://bbs.elecfans.com/group_1650)

 - [开发者社区](gitee.com/loongarch_community)

 - [2k1000LA 开发板套件](https://m.tb.cn/h.5sYX3Ja?tk=m0U1WjTPIUW)

 - [龙芯2K1000LA处理器用户手册_V1.0](https://github.com/LoongsonLab/oscomp-documents/blob/main/pdf/%E9%BE%99%E8%8A%AF2K1000LA%E5%A4%84%E7%90%86%E5%99%A8%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8C_V1.0.pdf)

 - [龙芯2K1000星云板用户手册V1.1](https://github.com/LoongsonLab/oscomp-documents/blob/main/pdf/%E5%B9%BF%E4%B8%9C%E9%BE%99%E8%8A%AF2K1000%E6%98%9F%E4%BA%91%E6%9D%BF%E7%94%A8%E6%88%B7%E6%89%8B%E5%86%8CV1.1.pdf)


### 参考OS

[龙芯实验室为大赛准备的参考开源OS](https://github.com/LoongsonLab/StarryOS-LoongArch.git)。StarryOS LoongArch版会持续更新。


### 常用的仓库
 
 - [Linux LoongArch](https://git.kernel.org/pub/scm/linux/kernel/git/chenhuacai/linux-loongson.git)

 - [Loongson Community](https://github.com/loongson-community)

 - [LoongArch 旧世界与新世界](https://github.com/loongson-community/areweloongyet/blob/main/docs/old-and-new-worlds.md)

 - [Loongson github](https://github.com/loongson)

 - [Qemu上运行LoongArch](https://github.com/foxsen/qemu-loongarch-runenv)

 - [Loongson 官方开源软件下载](http://www.loongnix.cn/zh/proj/)


### 开源的OS

 - [mit xv6-loongarch](https://github.com/skt-cpuos/xv6-loongarch-exp)。 xv6 是MIT开发的一个类Unix教学操作系统，与Linux或BSD不同，xv6非常简单，足以在一个学期内讲完，但仍包含Unix的重要概念和组织结构。xv6被全世界很多高校用于操作系统教学。 开发者： 深圳大学罗老师。 含OS代码、实验代码、实验指导书和PPT演示资料，可以直接用于操作系统教学。

 - [mit xv6-labs](https://github.com/Fan33oo/xv6-labs-loongarch). 本项目是xv6-labs-2021相关实验在LoongArch平台的参考实现。具体的实验设计参见xv6主页 的labs标签页。

 - [uCore](https://github.com/cyyself/ucore-loongarch32).  [实验指导书](https://cyyself.github.io/ucore_la32_docs)

 - [rCore](https://github.com/Godones/rCoreloongArch). 2022年全国大学生操作系统大赛-功能挑战赛二等奖。

 - [MaQueOS](https://gitee.com/dslab-lzu/maqueos). 本项目是用于兰州大学的教学操作系统，兰州大学相关团队为其编写了教材《MaQueOS：基于龙芯LoongArch架构的教学版操作系统》。

  - [Yocto](https://www.yoctoproject.org/). Yocto是用于定制嵌入式Linux系统的主流工具之一，它已经支持LoongArch.

  - [seL4](https://github.com/tyyteam/la-seL4). 2022年全国大学生操作系统大赛-功能挑战赛一等奖。

  - [NuttX](https://github.com/LA-NuttX). NuttX是完全兼容Posix和ANSI标准的嵌入式实时系统，有着轻量级、定制化的特点，已被广泛应用在成熟的商业系统或软件中，如小米Vela系统、三星Tizen RT系统、px4飞行控制软件。


 ### 资料持续更新




