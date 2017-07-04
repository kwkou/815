<properties
    pageTitle="在 Azure VM 中创建 HPC Pack 头节点 | Azure"
    description="了解如何使用 Azure 经典管理门户和经典部署模型在 Azure VM 中创建 Microsoft HPC Pack 头节点。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="dlepow"
    manager="timlt"
    editor=""
    tags="azure-service-management,hpc-pack"/>
<tags
    ms.assetid="e6a13eaf-9124-47b4-8d75-2bc4672b8f21"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="big-compute"
    ms.date="12/29/2016"
    wacn.date="02/24/2017"
    ms.author="danlep" />

# 在 Azure VM 中使用应用商店映像创建 HPC Pack 群集的头节点

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代经典部署模型。

头节点需要加入到 Azure 虚拟网络的 Active Directory 域中。可以使用此头节点在 Azure 中进行 HPC Pack 概念验证部署，并将计算资源添加到该群集以运行 HPC 工作负荷。


![HPC Pack 头节点][headnode]

>[AZURE.NOTE]目前，HPC Pack VM 映像为基于预安装了 HPC Pack 2012 R2 Update 2 的 Windows Server 2012 R2 Datacenter。还预安装了 Microsoft SQL Server 2014 Express。


对于 Azure 中 HPC Pack 群集的生产部署，我们建议采用自动部署方法，如 [HPC Pack IaaS 部署脚本](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-powershell-script/)。

## 规划注意事项

* **Active Directory 域** - 在启动 HPC 服务之前，HPC Pack 头节点必须加入到 Azure 中的 Active Directory 域。一种方法是部署一个单独的域控制器并部署在 Azure 中部署林，将 VM 加入到林。对于概念验证部署，在启动 HPC 服务之前，可以将为头节点创建的 VM 作为域控制器。

* **Azure 虚拟网络** - 如果计划将群集计算节点 VM 添加到 HPC 群集中，或者为群集创建一个单独的域控制器，则你将需要在 Azure 虚拟网络 (VNet) 中部署头节点。在没有 VNet 的情况下，你仍可以向群集中添加 Azure“突发”节点。

## 创建头节点的步骤

以下是为 HPC Pack 头节点创建 Azure VM 的大致步骤。可以使用各种 Azure 工具在 Azure 经典（服务管理）部署模型中执行这些步骤。


1. 如果你打算为头节点 VM 创建 VNet，请参阅[使用 Azure 经典管理门户创建虚拟网络（经典）](/documentation/articles/virtual-networks-create-vnet-classic-portal/)。

    **注意事项**
    
    * 你可以接受虚拟网络地址空间和子网的默认配置。

2. 如果你需要在单独的 VM 上创建新的 Active Directory 林，请参阅[在 Azure 虚拟网络中安装新的 Active Directory 林](/documentation/articles/active-directory-new-forest-virtual-machine/)。

    **注意事项**

    * 对于许多测试部署，可以在 Azure 中创建单个域控制器。为了确保 Active Directory 域的高可用性，可以部署一个额外的备份域控制器。
    
    * 对于简单的概念验证部署，可以忽略此步骤，稍后将头节点 VM 提升为域控制器。

3. 在 Azure 经典管理门户中，通过从 Azure 库中选择 HPC Pack 2012 R2 映像，创建一台经典 VM。（请参阅[此处](/documentation/articles/virtual-machines-windows-classic-tutorial/)的经典管理门户步骤。）

    **注意事项**

    * 选择一个 VM 大小，至少为 A4。

    * 如果你想在一个 VNet 中部署头节点，请确保在 VM 配置中指定该 VNet。

    * 我们建议你为 VM 创建一个新的云服务。
       
4. 在创建 VM且 VM 运行后，将 VM 加入现有的域林中，或在 VM 上创建一个新域林。

    **注意事项**

    * 如果使用现有的域林在 Azure 虚拟网络中创建了 VM，请使用标准的 Server Manager 或 Windows PowerShell 工具将 VM 加入到该林。然后重新启动。

    * 如果在新的虚拟网络中创建了 VM（未使用现有域林），则将该 VM 提升为域控制器。使用标准步骤安装和配置头节点上的 Active Directory 域服务角色。有关详细步骤，请参阅[安装新的 Windows Server 2012 Active Directory 林](https://technet.microsoft.com/zh-cn/library/jj574166.aspx)。

5. 在 VM 运行并加入到 Active Directory 林后启动 HPC Pack 服务，如下所示：

    a.使用一个属于本地管理员组的域帐户连接到头节点 VM。例如，使用创建头节点 VM 时设置的管理员帐户。

    b.对于默认头节点配置，以管理员身份启动 Windows PowerShell 并键入以下命令：

	    & $env:CCP_HOME\bin\HPCHNPrepare.ps1 –DBServerInstance ".\ComputeCluster"

    HPC Pack 服务启动可能需要几分钟时间。

    对于其他头节点配置选项，请键入 `get-help HPCHNPrepare.ps1`。


## 后续步骤

* 现在即可使用 HPC Pack 群集的头节点。例如，启动 HPC 群集管理器，并完成[部署待办事项列表](https://technet.microsoft.com/zh-cn/library/jj884141.aspx)。
* 若要按需提高群集计算容量，可在云服务中添加 [Azure 迸发节点](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-node-burst/)。

* 尝试在群集上运行测试工作负荷。例如，请参阅 HPC Pack [入门指南](https://technet.microsoft.com/zh-cn/library/jj884144)。

<!--Image references-->
[headnode]: ./media/virtual-machines-windows-hpcpack-cluster-headnode/headnode.png

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: meta update-->