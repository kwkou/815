<properties
 pageTitle="云中的 Linux HPC Pack 群集选项 | Azure"
 description="了解使用 Microsoft HPC Pack 在 Azure 云中创建和管理 Linux 高性能计算 (HPC) 群集时可用的选项。"
 services="virtual-machines,cloud-services"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""
 tags="azure-resource-manager,azure-service-management,hpc-pack"/>
<tags
	ms.service="virtual-machines-linux"
 	ms.date="02/04/2016"
	wacn.date="03/28/2016"/>

# 使用 Microsoft HPC Pack 在 Azure 中创建和管理 Linux 高性能计算 (HPC) 群集时可用的选项

[AZURE.INCLUDE [virtual-machines-common-hpcpack-cluster-options](../includes/virtual-machines-common-hpcpack-cluster-options.md)]

如果你想使用 HPC Pack 运行 Windows HPC 工作流，请参阅 [使用 Microsoft HPC Pack 在 Azure 中创建和管理 Windows 高性能计算 (HPC) 群集时可用的选项](/documentation/articles/virtual-machines-windows-hpcpack-cluster-options)。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]

## 在 Azure VM 中运行 HPC Pack 群集

### PowerShell 部署节点

* [使用 HPC Pack IaaS 部署脚本创建 HPC 群集](/documentation/articles/virtual-machines-linux-classic-hpcpack-cluster-powershell-script)

### 教程

* [教程：Azure 的 HPC Pack 群集中的 Linux 计算节点入门](/documentation/articles/virtual-machines-linux-classic-hpcpack-cluster)

* [教程：在 Azure 中的 Linux 计算节点上使用 Microsoft HPC Pack 运行 NAMD](/documentation/articles/virtual-machines-linux-classic-hpcpack-cluster-namd)

* [教程：开始使用 Azure 中的 HPC Pack 群集运行 Excel 和 SOA 工作负荷](/documentation/articles/virtual-machines-linux-classic-hpcpack-cluster-openfoam)


### 群集管理

* [将作业提交到 Azure 中的 HPC Pack 群集](/documentation/articles/virtual-machines-linux-hpcpack-cluster-submit-jobs)


## 为 MPI 工作负荷创建 RDMA 群集

* [教程：在 Azure 的 Linux RDMA 集群使用 Microsoft HPC Pack 运行 OpenFOAM](/documentation/articles/virtual-machines-linux-classic-hpcpack-cluster-openfoam)

* [设置 Linux RDMA 集群来运行 MPI 应用程序](/documentation/articles/virtual-machines-linux-classic-rdma-cluster)

<!---HONumber=Mooncake_1207_2015-->