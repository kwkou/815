<properties
 pageTitle="Windows VM 的计算基准测试分数 | Azure"
 description="比较运行 Windows Server 的 Azure VM 的 SPECint 计算基准测试分数"
 services="virtual-machines-windows"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""
 tags="azure-resource-manager,azure-service-management"/>
<tags
ms.service="virtual-machines-windows"
 ms.devlang="na"
 ms.topic="article"
 ms.tgt_pltfrm="vm-windows"
 ms.workload="infrastructure-services"
 ms.date="03/22/2017"
 wacn.date="05/31/2017"
 ms.author="cynthn"/>

# Windows VM 的计算基准测试分数

以下 SPECInt 基准测试分数显示运行 Windows Server 的 Azure 高性能 VM 产品阵容的计算性能。此外，还提供了 [Linux VM](/documentation/articles/virtual-machines-linux-compute-benchmark-scores/) 的计算基准测试分数。

## Dv2 系列


大小 | vCPU | NUMA 节点 | CPU | 运行次数 | 平均基本速率 | 标准偏差
------- | ------ | ---- | -------| ---- | ---- | -----
Standard_D1_v2 | 1 | 1 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 83 | 36.6 | 2.6
Standard_D2_v2 | 2 | 1 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 27 | 70.0 | 3.7
Standard_D3_v2 | 4 | 1 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 19 | 130.5 | 4.4
Standard_D4_v2 | 8 | 1 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 19 | 238.1 | 5.2
Standard_D5_v2 | 16 | 2 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 14 | 460.9 | 15.4
Standard_D11_v2 | 2 | 1 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 19 | 70.1 | 3.7
Standard_D12_v2 | 4 | 1 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 2 | 132.0 | 1.4
Standard_D13_v2 | 8 | 1 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 17 | 235.8 | 3.8
Standard_D14_v2 | 16 | 2 | Intel Xeon E5-2673 v3 @ 2.4 GHz | 15 | 460.8 | 6.5

## 关于 SPECint

Windows 分数是通过在 Windows Server 上运行 [SPECint 2006](https://www.spec.org/cpu2006/results/rint2006.html) 计算得出的。SPECint 是使用基本速率选项 (SPECint\_rate2006) 运行的，每个核心一个副本。SPECint 包括 12 项单独的测试，每项测试运行三次，取每次测试的中间值并为值加权，形成综合分数。然后，在多个 VM 上运行这些测试，提供表中所示的平均分数。


## 后续步骤

* 有关存储容量、磁盘详细信息以及选择 VM 大小的注意事项，请参阅 [Sizes for virtual machines](/documentation/articles/virtual-machines-windows-sizes/)（虚拟机的大小）。

<!---HONumber=Mooncake_0829_2016-->