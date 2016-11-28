<properties
 pageTitle="将突发节点添加到 HPC Pack 群集 | Azure"
 description="了解如何添加云服务中运行的辅助角色实例在 Azure 中按需扩展 HPC Pack 群集"
 services="virtual-machines-windows"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""
 tags="azure-service-management,hpc-pack"/>  

<tags
ms.service="virtual-machines-windows"
 ms.devlang="na"
 ms.topic="article"
 ms.tgt_pltfrm="vm-multiple"
 ms.workload="big-compute"
 ms.date="10/14/2016"
 wacn.date="11/28/2016"
 ms.author="danlep"/>  


# 在 Azure 中将按需“突发”节点添加到 HPC Pack 群集



如果在 Azure 中设置了 [Microsoft HPC Pack](https://technet.microsoft.com/zh-cn/library/cc514029)，可能希望能够快速扩展/收缩群集容量，而无需保留一组预配的计算节点 VM。本文介绍了如何按需将“突发”节点（云服务中运行的辅助角色实例）作为计算资源添加到 Azure 中的头节点。

[AZURE.INCLUDE [了解部署模型](../../includes/learn-about-deployment-models-classic-include.md)]

![突发节点][burst]  


本文中的步骤有助于将 Azure 节点快速添加到基于云的 HPC Pack 头节点 VM，进行测试或概念证明部署。这些高级步骤与“迸发到 Azure”的步骤相同，用于为本地 HPC Pack 群集添加云计算能力。有关教程，请参阅[使用 Microsoft HPC Pack 设置混合计算群集](/documentation/articles/cloud-services-setup-hybrid-hpcpack-cluster/)。有关生产部署的详细指南和注意事项，请参阅[使用 Microsoft HPC Pack 迸发到 Azure ](https://technet.microsoft.com/zh-cn/library/gg481749.aspx)。


## 先决条件

* **Azure VM 中部署的 HPC Pack 头节点** - 可使用独立头节点或较大群集中的头节点。若要创建独立头节点，请参阅[在 Azure VM 中部署 HPC Pack 头节点](/documentation/articles/virtual-machines-windows-hpcpack-cluster-headnode/)。有关 HPC Pack 群集的自动部署选项，请参阅[在 Azure 中使用 Microsoft HPC Pack 创建和管理 Windows HPC 群集时可用的选项](/documentation/articles/virtual-machines-windows-hpcpack-cluster-options/)。

    >[AZURE.TIP] 如果使用 [HPC Pack IaaS 部署脚本](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-powershell-script/)在 Azure 中创建群集，则可以在自动化部署中包含 Azure 突发节点。请参阅该文中的示例。

* **Azure 订阅** - 若要添加 Azure 节点，可选择部署头节点 VM 时所用的相同订阅，还可选用一个或多个不同订阅。

* **内核配额** — 你可能需要增加核心配额，尤其是在你选择部署具有多核大小的多个 Azure 节点时。

## 步骤 1：为 Azure 节点创建云服务和存储帐户

使用 Azure 经典管理门户或等效工具配置部署 Azure 节点时所需的以下资源：

* 新的 Azure 云服务
* 新的 Azure 存储帐户

>[AZURE.NOTE] 请勿重复使用订阅中的现有云服务。

**注意事项**

* 为你计划创建的各个 Azure 节点模板配置一个单独的云服务。但是，多个节点模板可以使用相同的存储帐户。

* 建议在同一 Azure 区域中查找用于部署的云服务和存储帐户。




## 步骤 2：配置 Azure 管理证书

若要将 Azure 节点添加为计算资源，头节点上必须有管理证书，并必须将相应证书上传到用于部署的 Azure 订阅。

对于此案例，可以选择 HPC Pack 在头节点上自动安装和配置的**默认 HPC Azure 管理证书**。此证书对进行测试和概念证明部署很有用。若要使用此证书，请将文件 C:\\Program Files\\Microsoft HPC Pack 2012\\Bin\\hpccert.cer 从头节点 VM中上传到订阅。若要在 [Azure 经典管理门户](https://manage.windowsazure.cn)中上传证书，请单击“设置”>“管理证书”。

有关配置管理证书的其他选项，请参阅[为 Azure 突发部署配置 Azure 管理证书的方案](http://technet.microsoft.com/zh-cn/library/gg481759.aspx)。

## 步骤 3：向群集部署 Azure 节点



本方案中添加和启动 Azure 节点的步骤通常与本地头节点中使用的步骤相同。有关详细信息，请参阅[使用 Microsoft HPC Pack 部署 Azure 节点的步骤](https://technet.microsoft.com/zh-cn/library/gg481758.aspx)中的以下部分：

* 创建 Azure 节点模板

* 将 Azure 节点添加到 Windows HPC 群集

* 启动（设置）Azure 节点

添加和启动节点后，它们就已准备好用于运行群集作业。

如果部署 Azure 节点时遇到问题，请参阅[排除使用 Microsoft HPC Pack 部署 Azure 节点时发生的故障](http://technet.microsoft.com/zh-cn/library/jj159097.aspx)。

## 后续步骤

* 如果想根据群集工作负荷自动增加或减少 Azure 计算资源，请参阅[自动增加和减少 HPC Pack 群集中的 Azure 计算资源](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-node-autogrowshrink/)。

<!--Image references-->

[burst]: ./media/virtual-machines-windows-classic-hpcpack-cluster-node-burst/burst.png

<!---HONumber=Mooncake_1121_2016-->