---
title: "从DPU的产生与发展说起"
date: 2021-12-29
draft: false
tags: ["DPU"]
# categories: ["ca1"]
---

Nvidia（英伟达）于2020年收购Mellanox公司（专注于提供IB网络、Ethernet网络产品，如ConnectX系列网卡），并在同年推出BlueField DPU，自此，DPU成为芯片领域一个新的聚焦点，Marvell、Pensando、Broadcom（博通）、Intel等国外传统半导体/芯片公司纷纷入局，国内的中科驭数、星云智联等芯片公司和一大批初创公司也提出了自己的DPU方案。

Nvidia CEO 黄仁勋在 GTC（GPU技术大会）上提出：“用于通用计算的 CPU，用于加速计算的 GPU，用于网络数据处理的 DPU，将成为未来计算的三大支柱”[1]。究竟什么是 DPU？为什么它将有可能与传统的 CPU、GPU平起平坐呢？

## 一、DPU是什么

数据处理单元（Data Processing Unit），通常称为DPU，是一种新型的可重新编程的结合高性能网络接口的高性能处理器。这些网络接口经过优化，可以执行和加速由数据中心服务器执行的网络和存储功能[2]。DPU就像GPU一样插入服务器的PCIe插槽，它们允许服务器将网络和存储功能从CPU卸载到DPU，让CPU只专注于运行操作系统和系统应用程序。DPU通常使用可重新编程的FPGA结合网络接口卡来加速网络流量，就像使用GPU通过将数学运算从CPU卸载到GPU来加速人工智能(AI)应用程序一样。



## 二、DPU的产生与发展

网络的发展推动着DPU的出现，DPU的发展史也是网卡的发展史。

![图片](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211647284.png)

### **第⼀阶段：基础功能网卡**

基础功能网卡即传统网卡只提供了最基础的网络接口，通过PCIE等总线，作为主机和外部网络连接的桥梁。基础网卡一般提供2x10G或2x25G带宽吞吐能力，网络的TCP/IP协议栈的处理由主机的操作系统实现，其硬件卸载能力较弱，主要是Checksum，LRO/LSO等，支持SR-IOV（Single Root I/O Virtualization），以及有限的多队列能力。



### 第⼆阶段：智能网卡

借助软硬件融合的思想，将操作系统的部分网络协议栈（TCP/IP、VLAN、GENEVE）由硬件来实现，实现对于各种网络基础功能的支持和加速，从而释放CPU的通用算力，由此诞生了各种offload NIC（支持卸载的网卡）。随着网络协议的复杂化和多样化，固定协议的offload无法与网络协议的快速更新发展相匹配，经历1-2年研发周期的固化网卡面临着被迅速淘汰的危机，在这样的背景下就要求网卡具有一定的可编程能力，从而满足协议更新的需求，延长网卡的市场周期。我们把拥有可编程能力的硬件卸载网卡叫做SmartNIC（智能网卡），它具有更加丰富的硬件卸载能力和一定的可编程性支持，如：OVS Fastpath硬件卸载，基于RoCE和RoCEv2的RDMA网络硬件卸载，融合网络中无损网络能力（PFC，ECN，ETS等）的硬件卸载，存储领域NVMe-oF（NVMe-over-Fabrics）的硬件卸载，以及安全传输的数据面卸载等。此时期的智能网卡以数据平面的卸载为主。



### 第三阶段：DPU智能⽹卡

DPU可以看作是第二代智能网卡，它在第一代智能网卡的基础上加入了片上CPU Core（ARM/MIPS），可以卸载控制平面的任务和一些灵活复杂的数据平面任务，有更强的可编程性。DPU主要分为网络单元和计算单元，网络单元负责与片上CPU、外设、主机的连接，计算单元赋予了DPU通用计算能力，与各种硬件加速器协同实现各种计算任务的加速，如DPI（深度包解析）、RegEx（正则匹配）、IPSec/AES（加密）等。DPU的出现使得网卡的卸载不再局限于网络功能，在网络、计算、存储、安全各个方面都有了更加丰富的支持。目前DPU智能网卡支持PCIe Root Complex模式和Endpoint模式，在配置为PCIe Root Complex模式时，可以实现NVMe存储控制器，与NVMe SSD磁盘一起构建存储服务器。



## 三、DPU的优势

DPU的功能主要包括了网络、计算、存储、安全多个方面。

![图片](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211647742.png)

基于网卡发展而来的DPU，其最核心的功能集中在网络方面，可以用于卸载网络虚拟化（OVS等）、SR-IOV、防火墙或任何其他需要高速数据包处理的应用程序。目前，主流的DPU支持高达200Gb/s的以太网和InfiniBand网络处理，释放CPU算力，从而解决主机上计算密集型应用的性能瓶颈。

![图片](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211647020.png)



对于存储，DPU可以作为标准的NVMe设备呈现给主机系统，同时它可以采用NVMe-oF解决方案，使用来自数据中心其他服务器的远程NVMe存储器。DPU也可以直接通过PCIe连接NVMe SSD，然后通过网络暴露给数据中心的其他DPU，所有这些活动都不需要传统的主机服务器的参与。DPU可以卸载NVMe-oF存储直连、加密、弹性存储、数据完整性、压缩和去重等，这使得远程存储的延迟与性能和直连存储相接近，提供了构建数据中心的高性能池化存储的新方式。

对于计算，DPU可用于运行与服务器上的主管理程序不同的管理程序，从而使x86 CPU或GPU甚至FPGA成为另一种跨越整个数据中心多台服务器的集中资源。或者只是将这些资源直接连接到DPU，由DPU将它们暴露到网络上。这样以来，数据中心内的任何主机都可以访问各种处理引擎的资源，任何主机上的任何应用程序都可以自由地利用这些加速器，无论它们实际物理位置在哪里。

![图片](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211647730.png)

在数据中心 “以数据为中心” 的趋势下，DPU为云服务提供商提供了资源池化的新方式，使其能够在基础架构内的任何位置组合存储、网络和计算资源，实现按需分配，进一步提高资源的利用率，为数据中心降低成本，达到更高的经济效益。此外，通过将安全任务卸载到DPU上，云服务提供商能够在为云租户提供裸机即服务（bare-metal-as-a-service）的同时保证服务环境的正确性与安全性。AWS/阿里云纷纷自研，英特尔/英伟达竞相布局，DPU已经在各大数据中心展现出巨大的价值。





## 四、DPU的核心要素

DPU SoC的核心并不是一个高性能的嵌入式CPU，由于能耗限制，期待DPU的嵌入式CPU达到比主机CPU更强的数据处理能力是不现实的。对于传统的x86CPU来说，100Gb/s的数据包处理速度已经会带来巨大的处理负担，极端情况下甚至会导致数据包的堆积和丢失。在400Gb/s的高速网络下，期待DPU的低功耗嵌入式CPU去处理每个数据包并不是一个合理的解决方案。Nvidia的Bluefield DPU和Pensando的Elba DPU解决方案都表明，在数据包处理负担过重的情况下，由嵌入式CPU负责控制路径的初始化和异常情况的处理可能是更好的DPU实现方式。

![图片](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211647795.png)

DPU在架构上主要包含两个部分。首先是继承于智能网卡的网络处理单元，例如Nvidia Bluefield DPU集成了ConnectX-6网卡单元，Pensando的Elba DPU集成了P4的数据包处理流水线。网络处理单元具有高性能的网络接口，用来连接外部高速网络，目前的主流DPU产品支持100Gb/s~200Gb/s的网络接口，未来两到三年会逐步提升至400Gb/s甚至800Gb/s的水平。第二个部分是SoC，主要包含低功耗的嵌入式CPU和各种HAC（Hardware Accelerator）。嵌入式CPU通常会使用ARM核或者其他的低功耗处理器（Fungible使用了MIPS 64处理器）以控制DPU的整体功耗。有了嵌入式CPU的支撑，DPU都会运行完整一个完整的操作系统（通常是完整的Linux），带来了很强的可编程性，并配合各种灵活可编程的加速引擎用来提供更强的卸载和加速能力。

![图片](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211701374.png)

为了灵活使用DPU上的各种加速器，厂商通常会提供相应的SDK（通常会与开源生态相兼容），Nvidia DPU就提供了DOCA（Data-Center-Infrastructure-On-A-Chip Architecture）SDK来实现更加灵活便捷的硬件控制方式和编程手段，并集成P4、DPDK等，以利用开源生态。简单来说，DOCA于DPU就像CUDA于GPU，这也是Nvidia将DPU和CPU、GPU列为未来的三大计算支撑的一个着力点。



## 五、DPU解决方案

我们选择了几种主流的DPU产品对其配置和架构做简单的介绍：

### **1、Nvidia Bluefield-2**

Nvidia于2020年收购Mellanox，同年推出基于ConnectX网卡的BlueField DPU产品，并于2021年推出了Bluefield-2 DPU产品。BlueField-2 DPU在ConnectX-6的基础上增加了SoC部分，由嵌入式CPU处理控制平面，CX6的eSwitch处理数据平面，从而构建完整的DPU处理单元。

![图片](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211647019.png)

**BlueField-2的主要规格如下：**

- CPU：8x ARM A72核
- 内存控制器：8Gb 或 16Gb DDR4-3200内存
- 高速网络连接：2x100Gbps或1x 200Gbps以太网或InfiniBand，基于Mellanox ConnectX-6 Dx
- 高速数据包处理加速：类似于其他ConnectX-6 Dx解决方案的多个卸载引擎和eSwitch流逻辑
- 加速器：用于正则表达式、重复数据删除和压缩算法以及加密卸载
- PCIe Gen4通道：16通道PCIe Gen3/4 PCIe switch
- 安全和管理功能：Hardware RoT（Root of Trust），具有用于带外（out-of-band）管理的1GbE接口
- 运行的操作系统：许多Linux发行版，如Ubuntu、CentOS、Yocoto以及VMware ESXi



### 2、Fungible F1

Fungible是首批为其提供的这种新型处理器命名为DPU的公司之一。

![图片](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211647984.png)

**Fungible F1主要规格如下：**

- CPU：8个4x SMT MIPS-64核心的数据集群
- 内存控制器：2x DDR4控制器加上对8GB HBM2（High Bandwidth Memory）的支持
- 高速网络连接：2x 400Gbps网络接口，能够聚合高达800Gbps或8x 100GbE
- 高速数据包处理加速：用于解析、封装、解封装、查找和传输/接收加速的类P4语言
- 加速器：多个加速器，包括用于数据移动的加速器
- PCIe Gen4通道：四个x16主机单元，可以作为根或端点运行
- 安全和管理功能：4核x2路SMT控制集群，具有安全区域、安全启动和Hardware RoT（Root of Trust），还有加密引擎和随机数生成等功能
- 运行的操作系统：Linux



### 3、Pensando Elba

Pensando是一家云创业公司，由一群著名的前思科工程师创立。Elba DPU由嵌入式CPU处理控制平面，P4流水线处理数据平面。

![图片](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211647079.png)

**pensando Elba主要规格如下：**

- CPU：16个ARM A72核
- 内存控制器：双通道DDR4/DDR5内存支持8–64GB。Pensando在之前的型号中使用HBM，但后来转而使用更便宜、更灵活的DDR
- 高速网络连接：2x 200Gbps网络接口
- 高速包处理加速：P4可编程路径
- 加速器：用于加密、压缩和数据移动等
- PCIe Gen4通道：32x PCIe Gen4通道和8个端口
- 安全和管理功能：Hardware RoT（Root of Trust），具有用于带外管理的1GbE接口
- 运行的操作系统：支持DPDK的Linux，以及VMware ESXi

### 下一代DPU解决方案

Nvidia推出BlueField-2 DPU时，提出了未来三年DPU的路标。总体上来看，其核心是以400Gb/s的链路速度为代表的高速网络处理能力，以及更强的片上CPU处理能力。

![图片](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211648303.png)

目前，Nvidia已经发布了新一代DPU BlueField-3。它支持400Gbps网络，采用32通道PCIe Gen 5.0，搭载16核Armv8.2+ A78 CPU，具有16GB板载DDR5[7]，较上一代DPU性能实现了极大的提升。



## 六、工业界应用

### 亚马逊AWS Nitro系统

AWS的Nitro是DPU在云基础设施中应用的首批示例之一。Amazo Web Services分解了传统服务器，添加Nitro IO加速卡(ASIC)以通过整体Nitro卡控制器处理VPC（虚拟私有云）、EBS、实例存储、安全性等。Nitro将虚拟机管理程序、网络虚拟化和存储虚拟化任务分流到专用硬件，以释放主CPU。

AWS Elastic Compute Cloud实例基于PCIe连接的Nitro卡以及X86或Arm处理器和DRAM。有各种EC2实例类型—通用型或针对计算、内存、存储、机器学习和横向扩展用例进行了优化[8]。

![图片](https://hugo-github-io.oss-cn-beijing.aliyuncs.com/img/202306211648297.png)

## 七、总结

随着5G、大数据和云计算等新技术的蓬勃发展，数据中心的规模增大，基础架构需要的网络带宽不断提升。同时服务器物理核密度不断提高，支撑CPU的网络从25Gb/s增加到200Gb/s，服务器本身对网络基础功能的处理要求不断提高，在CPU内核上产生了过多的计算开销，这是DPU智能网卡产生的最关键原因之一。

DPU旨在卸载和基础网络处理相关的计算任务，利用各种硬件加速器，以比主机CPU更低的成本实现对各种网络功能和虚拟化的支持，进一步支持计算、存储、安全等方面的加速卸载，从而释放主机CPU的通用算力。

进一步，云服务提供商借助DPU实现各种计算、存储、网络资源的池化和按需分配，能够进一步提高资源的利用率，为数据中心降低成本，实现更高的经济效益。

## Reference

[1] Cadalyst Staff.What Is a DPU? The Third Pillar of Computing, Says NVIDIA [EB/OL]. (2020-5-20). https://www.cadalyst.com/design-related-technologies/graphics-cards-and-cpus/what-dpu-third-pillar-computing-says-nvidia-7585

[2] Premio Inc. What Is A DPU (Data Processing Unit)? [EB/OL]. (2021-5-12). https://premioinc.com/blogs/blog/what-is-a-dpu-data-processing-unit

[3] 中国科学院计算技术研究所，鄢贵海等.专用数据处理器（DPU）技术白皮书[EB/OL]. (2021-10). http://www.yusur.tech/zkls/pdf/DPU-whitepaper-v1.0-final-21.pdf

[4] KEVIN DEIERLING.What Is a DPU? [EB/OL].(2020-5-20). https://blogs.nvidia.com/blog/2020/05/20/whats-a-dpu-data-processing-unit/

[5] Patrick Kennedy What is a DPU A Data Processing Unit Quick Primer. [EB/OL]. (2020-9-29). https://www.servethehome.com/what-is-a-dpu-a-data-processing-unit-quick-primer/

[6] Patrick Kennedy. NVIDIA Shows DPU Roadmap Combining Arm Cores GPU and Networking [EB/OL]. (2020-10-5). https://www.servethehome.com/nvidia-shows-dpu-roadmap-combining-arm-cores-gpu-and-networking/

[7] NVIDIA Corporation.NVIDIA BLUEFIELD-3 DPU datasheet [EB/OL]. https://www.nvidia.com/content/dam/en-zz/Solutions/Data-Center/documents/datasheet-nvidia-bluefield-3-dpu.pdf

[8] AWS Events. Deep dive into the Nitro system[EB/OL]. (2019-12-10). https://www.youtube.com/watch?v=rUY-00yFlE4

[9] DPU家族大探秘：https://mp.weixin.qq.com/s/jmQG5uLjOlyIe-5QFL0PiQ
