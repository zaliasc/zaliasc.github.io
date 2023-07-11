---
title: "Dramsim3接入记录"
date: 2023-07-06
draft: false
tags: ["Linux","simulation"]
# categories: ["ca1"]
---

近期对SIMT架构在网络处理中的应用场景进行模拟，使用 Dramsim3 作为处理器的 memory system。

## Dramsim3 介绍

A Cycle-Accurate, Thermal-Capable DRAM Simulator. [^1]





## 框架接入方式

TODO.





## 接入参考

很多通用的模拟器设施都有使用 Dramsim3 进行 memory 模拟的实现，这里列出一些进行自己接入时的参考：

- Dramsim3.[^2]
- Zsim-dramsim3.[^3]



## ref:

[^1]: Dramsim3 pape :  https://par.nsf.gov/servlets/purl/10216399

[^2]: Gem5 dramsim3 wrapper : https://github.com/gem5/gem5/blob/stable/src/mem/dramsim3.cc
[^3]: zsim-dramsim3 : https://github.com/zoumo4913/Zsim-DRAMsim3

