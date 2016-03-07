<properties
 pageTitle="云中的 HPC Pack 群集选项 | Azure"
 description="了解使用 Microsoft HPC Pack 在 Azure 云中创建和管理高性能计算 (HPC) 群集时可用的选项。"
 services="virtual-machines,cloud-services"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""
 tags="azure-resource-manager,azure-service-management,hpc-pack"/>
<tags
	ms.service="virtual-machines"
 	ms.date="10/08/2015"
	wacn.date="02/17/2016"/>

# 使用 Microsoft HPC Pack 在 Azure 中创建和管理高性能计算 (HPC) 群集时可用的选项

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]


利用 Microsoft HPC Pack 和 Azure 的计算与基础结构服务可以创建和管理基于云的高性能计算 (HPC) 群集。[HPC Pack](https://technet.microsoft.com/zh-cn/library/jj899572.aspx) 是在 Azure 和 Windows Server 技术基础之上构建的 Microsoft 免费 HPC 解决方案，支持 Windows 和 Linux HPC 工作负荷。基于云的 HPC Pack 群集为群集管理员或独立软件供应商 (ISV) 提供弹性的可缩放平台，能够在运行计算密集型应用程序的同时，降低本地计算群集基础结构的投资。


## 在 Azure VM 中运行 HPC Pack 群集

### PowerShell 部署节点

* [使用 HPC Pack IaaS 部署脚本创建 HPC 群集](/documentation/articles/virtual-machines-hpcpack-cluster-powershell-script)

### 教程

* [教程：Azure 的 HPC Pack 群集中的 Linux 计算节点入门](/documentation/articles/virtual-machines-linux-cluster-hpcpack)

* [教程：在 Azure 中的 Linux 计算节点上使用 Microsoft HPC Pack 运行 NAMD](/documentation/articles/virtual-machines-linux-cluster-hpcpack-namd)

* [教程：开始使用 Azure 中的 HPC Pack 群集运行 Excel 和 SOA 工作负荷](/documentation/articles/virtual-machines-excel-cluster-hpcpack)



### 使用 Azure 门户手动部署



* [在 Azure VM 中设置 HPC Pack 群集的头节点](/documentation/articles/virtual-machines-hpcpack-cluster-headnode)

### 群集管理

* [在 Azure 中管理 HPC Pack 群集的计算节点](/documentation/articles/virtual-machines-hpcpack-cluster-node-manage)

* [增加和减少 HPC Pack 群集中的 Azure 计算资源](/documentation/articles/virtual-machines-hpcpack-cluster-node-autogrowshrink)

* [将作业提交到 Azure 中的 HPC Pack 群集](/documentation/articles/virtual-machines-hpcpack-cluster-submit-jobs)



## 将辅助角色节点添加到 HPC Pack 群集


* [使用 HPC Pack 迸发到 Azure](https://technet.microsoft.com/zh-cn/library/gg481749.aspx)

* [教程：使用 Azure 中的 HPC Pack 设置混合群集](/documentation/articles/cloud-services-setup-hybrid-hpcpack-cluster)

* [将 Azure“突发”节点添加到 Azure 中的 HPC Pack 头节点](/documentation/articles/virtual-machines-hpcpack-cluster-node-burst)

* [增加和减少 HPC Pack 群集中的 Azure 计算资源](/documentation/articles/virtual-machines-hpcpack-cluster-node-autogrowshrink)


## 为 MPI 工作负荷创建 RDMA 群集

* [使用 HPC Pack 设置一个运行 MPI 应用程序的 Windows RDMA 群集](/documentation/articles/virtual-machines-windows-hpcpack-cluster-rdma)

* [Set up a Linux RDMA cluster to run MPI applications](/documentation/articles/virtual-machines-linux-cluster-rdma)

<!---HONumber=Mooncake_1207_2015-->