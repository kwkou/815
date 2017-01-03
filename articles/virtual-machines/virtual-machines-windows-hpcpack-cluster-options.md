<properties
 pageTitle="云中的 Windows HPC Pack 群集选项 | Azure"
 description="了解在 Azure 云中使用 Microsoft HPC Pack 创建和管理 Windows 高性能计算 (HPC) 群集时可用的选项"
 services="virtual-machines-windows,cloud-services,batch"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""
 tags="azure-resource-manager,azure-service-management,hpc-pack"/>  

<tags
ms.service="virtual-machines-windows"
 ms.devlang="na"
 ms.topic="article"
 ms.tgt_pltfrm="vm-windows"
 ms.workload="big-compute"
 ms.date="11/17/2016"
 wacn.date="12/30/2016"
 ms.author="danlep"/>  


# 在 Azure 中使用 HPC Pack 创建和管理 Windows HPC 群集时可用的选项

[AZURE.INCLUDE [virtual-machines-common-hpcpack-cluster-options](../../includes/virtual-machines-common-hpcpack-cluster-options.md)]

## 在 Azure VM 中运行 HPC Pack 群集

### Azure 模板

>[AZURE.NOTE] 必须修改从 GitHub 存储库“azure-quickstart-templates”下载的模板，以适应 Azure 中国云环境。例如，替换某些终结点（将“blob.core.chinacloudapi.cn”替换为“blob.core.chinacloudapi.cn”，将“chinacloudapp.cn”替换为“chinacloudapp.cn”）；更改某些不受支持的 VM 映像；更改某些不受支持的 VM 大小。

* （快速入门）[创建 HPC 群集](https://github.com/Azure/azure-quickstart-templates/tree/master/create-hpc-cluster)

* （快速入门）[使用自定义计算节点映像创建 HPC 群集](https://github.com/Azure/azure-quickstart-templates/tree/master/create-hpc-cluster-custom-image)

### PowerShell 部署脚本

* [使用 HPC Pack IaaS 部署脚本创建 HPC 群集](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-powershell-script/)

### 教程

* [教程：开始使用 Azure 中的 HPC Pack 群集运行 Excel 和 SOA 工作负荷](/documentation/articles/virtual-machines-windows-excel-cluster-hpcpack/)



### 使用 Azure 门户预览手动部署

* [在 Azure VM 中设置 HPC Pack 群集的头节点](/documentation/articles/virtual-machines-windows-hpcpack-cluster-headnode/)

### 群集管理

* [在 Azure 中管理 HPC Pack 群集的计算节点](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-node-manage/)

* [增加和减少 HPC Pack 群集中的 Azure 计算资源](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-node-autogrowshrink/)

* [将作业提交到 Azure 中的 HPC Pack 群集](/documentation/articles/virtual-machines-windows-hpcpack-cluster-submit-jobs/)

* [HPC Pack 中的作业管理](https://technet.microsoft.com/zh-cn/library/jj899585.aspx)


## 将辅助角色节点添加到 HPC Pack 群集


* [使用 HPC Pack 迸发到 Azure 辅助角色实例](https://technet.microsoft.com/zh-cn/library/gg481749.aspx)

* [教程：使用 Azure 中的 HPC Pack 设置混合群集](/documentation/articles/cloud-services-setup-hybrid-hpcpack-cluster/)

* [将 Azure“突发”节点添加到 Azure 中的 HPC Pack 头节点](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-node-burst/)


## 与 Azure 批处理集成 

* [使用 HPC Pack 迸发到 Azure Batch](https://technet.microsoft.com/zh-cn/library/mt612877.aspx)

<!---HONumber=Mooncake_1121_2016-->