<properties
 pageTitle="将突发节点添加到 HPC Pack 群集 | Azure"
 description="了解如何按需将云服务中运行的辅助角色实例作为计算资源添加到 Azure 中的 HPC Pack 头节点。"
 services="virtual-machines"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""
 tags="azure-service-management,hpc-pack"/>
<tags
	ms.service="virtual-machines-windows"
	ms.date="04/13/2016"
	wacn.date="05/24/2016"/>

# 将按需“突发”节点（辅助角色实例）作为计算资源添加到 Azure 中的 HPC Pack 群集

本文介绍如何按需将 Azure“突发”节点（在云服务中运行的辅助角色实例）作为计算资源添加到 Azure 的现有 HPC Pack 头节点。这可让你按需增加 Azure 中 HPC 群集的计算能力，而无需维护一组预配置的计算节点 VM。

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

![突发节点][burst]

>[AZURE.TIP] 如果使用 [HPC Pack IaaS 部署脚本](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-powershell-script/)在 Azure 中创建群集，则可以在自动化部署中包含 Azure 突发节点。请参阅该文中的示例。

本文中的步骤有助于你将 Azure 节点快速添加到基于云的 HPC Pack 头节点 VM，进行测试或概念证明部署。此过程实质上与“迸发到 Azure”的过程相同，用于为本地 HPC Pack 群集添加云计算能力。有关教程，请参阅[使用 Microsoft HPC Pack 设置混合计算群集](/documentation/articles/cloud-services-setup-hybrid-hpcpack-cluster/)。有关生产部署的详细指南和注意事项，请参阅[使用 Microsoft HPC Pack 迸发到 Azure ](https://technet.microsoft.com/zh-cn/library/gg481749.aspx)。

## 先决条件

* **Azure VM 中部署的 HPC Pack 头节点** — 请参阅[在 Azure VM 中部署 HPC Pack 头节点](/documentation/articles/virtual-machines-windows-hpcpack-cluster-headnode/)，了解在经典（服务管理）部署模型中创建群集头节点的步骤。

* **Azure 订阅** — 若要添加 Azure 节点，可以选择与用来部署头节点 VM 的订阅相同的订阅，也可以选择一个（或多个）不同的订阅。

* **内核配额** — 你可能需要增加核心配额，尤其是在你选择部署具有多核大小的多个 Azure 节点时。

## 步骤 1：创建云服务和存储帐户，以添加 Azure 节点

使用 Azure 经典管理门户或等效工具配置部署 Azure 节点所需的以下项：

* 新的 Azure 云服务
* 新的 Azure 存储帐户

>[AZURE.NOTE] 请勿重复使用订阅中的现有云服务。

**注意事项**

* 为你计划创建的各个 Azure 节点模板配置一个单独的云服务。但是，多个节点模板可以使用相同的存储帐户。

* 你通常应在相同 Azure 区域内查找部署的云服务和存储帐户。




## 步骤 2：配置 Azure 管理证书

若要将 Azure 节点添加为计算资源，在头节点上需要有一个管理证书，并将相应证书上载到用于部署的 Azure 订阅。

对于此案例，可以选择 HPC Pack 在头节点上自动安装和配置的**默认 HPC Azure 管理证书**。此证书对进行测试和概念证明部署很有用。若要使用此证书，只需从头节点 VM 将文件 C:\\Program Files\\Microsoft HPC Pack 2012\\Bin\\hpccert.cer 上载到订阅即可。

有关配置管理证书的其他选项，请参阅[为 Azure 突发部署配置 Azure 管理证书的方案](http://technet.microsoft.com/zh-cn/library/gg481759.aspx)。

## 步骤 3：向群集部署 Azure 节点



本方案中添加和启动 Azure 节点的步骤通常与本地头节点使用的步骤相同。有关详细信息，请参阅[使用 Microsoft HPC Pack 部署 Azure 节点的步骤](https://technet.microsoft.com/zh-cn/library/gg481758.aspx)中的以下部分：

* 创建 Azure 节点模板

* 将 Azure 节点添加到 Windows HPC 群集

* 启动（设置）Azure 节点

添加和启动节点后，它们就已准备好用于运行群集作业。

如果部署 Azure 节点时遇到问题，请参阅[排除使用 Microsoft HPC Pack 部署 Azure 节点时发生的故障](http://technet.microsoft.com/zh-cn/library/jj159097.aspx)。

## 后续步骤

* 如果你想要有一种方法能够根据群集上作业及任务的当前工作负荷自动增加或减少 Azure 计算资源，请参阅[自动增加和减少 HPC Pack 群集中的 Azure 计算资源](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-node-autogrowshrink/)。

<!--Image references-->
[burst]: ./media/virtual-machines-windows-classic-hpcpack-cluster-node-burst/burst.png

<!---HONumber=Mooncake_0215_2016-->