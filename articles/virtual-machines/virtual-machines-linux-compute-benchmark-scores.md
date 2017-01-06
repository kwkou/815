<properties
 pageTitle="Linux VM 的计算基准测试分数 | Azure"
 description="比较运行 Linux 的 Azure VM 的 CoreMark 计算基准测试分数"
 services="virtual-machines-linux"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""
 tags="azure-resource-manager,azure-service-management"/>  

<tags
ms.service="virtual-machines-linux"
 ms.devlang="na"
 ms.topic="article"
 ms.tgt_pltfrm="vm-linux"
 ms.workload="infrastructure-services"
 ms.date="09/22/2016"
 wacn.date="11/21/2016"
 ms.author="cynthn"/>

# Linux VM 的计算基准测试分数

以下 CoreMark 基准测试分数显示运行 Ubuntu 的 Azure 高性能 VM 产品阵容的计算性能。此外，还提供了 [Windows VM](/documentation/articles/virtual-machines-windows-compute-benchmark-scores/) 的计算基准测试分数。

## Dv2 系列


大小 | vCPU | NUMA 节点 | CPU | 运行次数 | 迭代次数/秒 | 标准偏差
------- | ------ | ---- | -------| ---- | ---- | -----
Standard_D1_v2 | 1 | 1 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 140 | 14,852 | 780
Standard_D2_v2 | 2 | 1 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 133 | 29,467 | 1,863
Standard_D3_v2 | 4 | 1 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 139 | 56,205 | 1,167
Standard_D4_v2 | 8 | 1 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 126 | 108,543 | 3,446
Standard_D5_v2 | 16 | 2 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 126 | 205,332 | 9,998
Standard_D11_v2 | 2 | 1 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 125 | 28,598 | 1,510
Standard_D12_v2 | 4 | 1 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 131 | 55,673 | 1,418
Standard_D13_v2 | 8 | 1 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 140 | 107,986 | 3,089
Standard_D14_v2 | 16 | 2 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 140 | 208,186 | 8,839
Standard_D15_v2 | 20 | 2 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 28 | 268,560 | 4,667

## F 系列

大小 | vCPU | NUMA 节点 | CPU | 运行次数 | 迭代次数/秒 | 标准偏差
------- | ------ | ---- | -------| ---- | ---- | -----
Standard_F1 | 1 | 1 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 154 | 15,602 | 787
Standard_F2 | 2 | 1 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 126 | 29,519 | 1,233
Standard_F4 | 4 | 1 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 147 | 58,709 | 1,227
Standard_F8 | 8 | 1 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 224 | 112,772 | 3,006
Standard_F16 | 16 | 2 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 42 | 218,571 | 5,113

## 关于 CoreMark

Linux 分数是通过在 Ubuntu 上运行 [CoreMark](http://www.eembc.org/coremark/faq.php) 计算得出的。CoreMark 中配置的线程数设置为虚拟 CPU 的数目，并发性设置为 PThreads。目标迭代次数已根据预期性能进行调整，提供至少 20 秒（通常更长）的运行时，最终分数表示已完成迭代次数除以运行测试所花费的秒数。每项测试在每个 VM 上至少运行了七次。测试运行时间为 2015 年 10 月，于测试运行当天在支持 VM 的每个 Azure 公共区域中的多个 VM 上运行。
## 后续步骤



* 有关存储容量、磁盘详细信息以及选择 VM 大小的注意事项，请参阅 [Sizes for virtual machines](/documentation/articles/virtual-machines-linux-sizes/)（虚拟机的大小）。

* 若要在 Linux VM 上运行 CoreMark 脚本，请下载 [CoreMark 脚本包](http://download.microsoft.com/download/3/0/5/305A3707-4D3A-4599-9670-AAEB423B4663/AzureCoreMarkScriptPack.zip)。

<!---HONumber=Mooncake_0829_2016-->