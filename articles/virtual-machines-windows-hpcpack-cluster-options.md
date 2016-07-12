<properties
 pageTitle="云中的 Windows HPC Pack 群集选项 | Azure"
 description="了解使用 Microsoft HPC Pack 在 Azure 云中创建和管理 Windows 高性能计算 (HPC) 群集时可用的选项。"
 services="virtual-machines-windows,cloud-services"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""
 tags="azure-resource-manager,azure-service-management,hpc-pack"/>
<tags
	ms.service="virtual-machines-windows"
 	ms.date="04/29/2016"
	wacn.date="06/29/2016"/>

# 使用 Microsoft HPC Pack 在 Azure 中创建和管理 Windows 高性能计算 (HPC) 群集时可用的选项

[AZURE.INCLUDE [virtual-machines-common-hpcpack-cluster-options](../includes/virtual-machines-common-hpcpack-cluster-options.md)]

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]

## 在 Azure VM 中运行 HPC Pack 群集

### PowerShell 部署节点

* [使用 HPC Pack IaaS 部署脚本创建 HPC 群集](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-powershell-script/)

### 教程

* [教程：开始使用 Azure 中的 HPC Pack 群集运行 Excel 和 SOA 工作负荷](/documentation/articles/virtual-machines-windows-excel-cluster-hpcpack/)

### 使用 Azure 经典管理门户手动部署

* [在 Azure VM 中设置 HPC Pack 群集的头节点](/documentation/articles/virtual-machines-windows-hpcpack-cluster-headnode/)

### 群集管理

* [在 Azure 中管理 HPC Pack 群集的计算节点](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-node-manage/)

* [增加和减少 HPC Pack 群集中的 Azure 计算资源](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-node-autogrowshrink/)

* [将作业提交到 Azure 中的 HPC Pack 群集](/documentation/articles/virtual-machines-windows-hpcpack-cluster-submit-jobs/)



## 将辅助角色节点添加到 HPC Pack 群集


* [使用 HPC Pack 迸发到 Azure](https://technet.microsoft.com/zh-cn/library/gg481749.aspx)

* [教程：使用 Azure 中的 HPC Pack 设置混合群集](/documentation/articles/cloud-services-setup-hybrid-hpcpack-cluster/)

* [将 Azure“突发”节点添加到 Azure 中的 HPC Pack 头节点](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-node-burst/)

* [增加和减少 HPC Pack 群集中的 Azure 计算资源](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-node-autogrowshrink/)


## 为 MPI 工作负荷创建 RDMA 群集

* [使用 HPC Pack 设置一个运行 MPI 应用程序的 Windows RDMA 群集](/documentation/articles/virtual-machines-windows-classic-hpcpack-rdma-cluster/)


<!---HONumber=Mooncake_1207_2015-->