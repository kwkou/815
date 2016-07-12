<properties
 pageTitle="管理 HPC Pack 群集计算节点 | Azure"
 description="了解 PowerShell 脚本工具如何添加、删除、启动和停止 Azure 的 HPC Pack 群集计算节点"
 services="virtual-machines-windows"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""
 tags="azure-service-management,hpc-pack"/>
<tags
	ms.service="virtual-machines-windows"
	ms.date="04/18/2016"
	wacn.date="06/29/2016"/>

# 管理 Azure 的 HPC Pack 群集中计算节点的数量和可用性

如果你在 Azure VM 中创建了一个 HPC Pack 群集，你可能希望有轻松添加、删除、启动（设置）或停止（取消设置）群集中大量计算节点 VM 的方法。若要执行这些任务，请运行头节点 VM 中安装的 Azure PowerShell 脚本。这些脚本可帮助你控制 HPC Pack 群集资源的数量和可用性，以便你可以控制成本。

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

## 先决条件

* **Azure VM 中的 HPC Pack 群集** - 在经典部署模型中，使用 HPC Pack 2012 R2 Update 1 以上版本创建一个 HPC Pack 群集。例如，可以通过使用 Azure 库中的 HPC Pack VM 映像和 Azure PowerShell 脚本，自动执行部署。有关信息和先决条件，请参阅[使用 HPC Pack IaaS 部署脚本创建 HPC 群集](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-powershell-script/)。

* **Azure 发布设置文件或管理证书** - 你需要在头节点上执行下列操作之一：

	部署之后，在头节点上的 %CCP\_HOME%bin 文件夹中找到节点管理脚本。你必须以管理员身份运行各个脚本。

    * **导入 Azure 发布设置文件**。为此，在头节点上运行以下 Azure PowerShell cmdlet：

    		Get-AzurePublishSettingsFile

    		Import-AzurePublishSettingsFile -PublishSettingsFile <publish settings file>

    * **在头节点上配置 Azure 管理证书**。如果你有 .cer 文件，将其导入至 CurrentUser\\My certificate store，然后为 Azure 环境（AzureCloud 或 AzureChinaCloud）运行以下 Azure PowerShell cmdlet：

    		Set-AzureSubscription -SubscriptionName <Sub Name> -SubscriptionId <Sub ID> -Certificate (Get-Item Cert:\CurrentUser\My<Cert Thrumbprint>) -Environment <AzureCloud | AzureChinaCloud>

## 添加计算节点 VM

使用 **Add-HpcIaaSNode.ps1** 脚本添加计算节点。

### 语法

	Add-HPCIaaSNode.ps1 [-ServiceName] <String> [-ImageName] <String> `
		[-Quantity] <Int32> [-InstanceSize] <String> [-DomainUserName] <String> `
		[[-DomainUserPassword] <String>] [[-NodeNameSeries] <String>] [<CommonParameters>]

### Parameters

* **ServiceName** - 云服务的名称，新计算节点 VM 将添加到该服务。

* **ImageName** - Azure VM 映像名称，可以通过 Azure 经典管理门户或 Azure PowerShell cmdlet **Get-AzureVMImage** 获得。映像必须满足以下要求：

    1. 必须安装了 Windows 操作系统。

    2. HPC Pack 必须安装在计算节点角色中。

    3. 映像的“用户”类别必须是私有映像，而不是公共 Azure VM 映像。

* **Quantity**- 要添加的计算节点 VM 的数量。

* **InstanceSize** - 计算节点 VM 的大小。

* **DomainUserName** - 域用户名称，用于将新 VM 加入到域。

* **DomainUserPassword** - 域用户的密码。

* **NodeNameSeries**（可选）- 计算节点的命名模式。格式必须为 &lt;*Root\_Name*&gt;&lt;*Start\_Number*&gt;%。例如，MyCN%10% 是指从 MyCN11 开始的一系列计算节点名称。如果未指定，脚本会使用 HPC 群集中配置的节点名称系列。

### 示例

下面的示例基于 VM 映像 *hpccnimage1*，在云服务 *hpcservice1* 中添加了 20 个大型计算节点 VM。

	Add-HPCIaaSNode.ps1 -ServiceName hpcservice1 -ImageName hpccniamge1 `
		-Quantity 20 -InstanceSize Large -DomainUserName <username> `
		-DomainUserPassword <password> `

## 删除计算节点 VM

使用 **Remove-HpcIaaSNode.ps1** 脚本删除计算节点。

### 语法

	Remove-HPCIaaSNode.ps1 -Name <String[]> [-DeleteVHD] [-Force] [-WhatIf] [-Confirm] [<CommonParameters>]

	Remove-HPCIaaSNode.ps1 -Node <Object> [-DeleteVHD] [-Force] [-Confirm] [<CommonParameters>]

### Parameters

* **Name** - 要删除的群集节点的名称。支持通配符。参数集名称是“名称”。你无法指定**名称**和**节点**参数。

* **Node** - 要删除的节点的 HpcNode 对象，可通过 HPC PowerShell cmdlet [Get-HpcNode](https://technet.microsoft.com/zh-cn/library/dn887927.aspx) 获得。参数集名称是“节点”。你无法指定**名称**和**节点**参数。

* **DeleteVHD**（可选）- 删除已删除 VM 的关联磁盘的设置。

* **Force**（可选）- 在删除 HPC 节点之前强制其脱机的设置。

* **Confirm**（可选）- 执行命令前的确认提示。

* **WhatIf** - 一项设置，用于说明如果你执行了命令，但命令没有被实际执行时出现情况。

### 示例

下面的示例强制名称以 *HPCNode-CN-* 开头的脱机节点，然后删除这些节点及其关联磁盘。

	Remove-HPCIaaSNode.ps1 -Name HPCNodeCN-* -DeleteVHD -Force

## 启动计算节点 VM

使用 **Start-HpcIaaSNode.ps1** 脚本启动计算节点。

### 语法

	Start-HPCIaaSNode.ps1 -Name <String[]> [<CommonParameters>]

	Start-HPCIaaSNode.ps1 -Node <Object> [<CommonParameters>]

### Parameters

* **Name** - 要启动的群集节点的名称。支持通配符。参数集名称是“名称”。你无法指定**名称**和**节点**参数。

* **Node** - 要启动的节点的 HpcNode 对象，可以通过 HPC PowerShell cmdlet [Get-HpcNode](https://technet.microsoft.com/zh-cn/library/dn887927.aspx) 获得。参数集名称是“节点”。你无法指定**名称**和**节点**参数。

### 示例

下面的示例启动名称以 *HPCNode-CN-* 开头的节点。

	Start-HPCIaaSNode.ps1 -Name HPCNodeCN-*

## 停止计算节点 VM

使用 **Stop-HpcIaaSNode.ps1** 脚本停止计算节点。

### 语法

	Stop-HPCIaaSNode.ps1 -Name <String[]> [-Force] [<CommonParameters>]

	Stop-HPCIaaSNode.ps1 -Node <Object> [-Force] [<CommonParameters>]

### Parameters


* **Name** - 要停止的群集节点的名称。支持通配符。参数集名称是“名称”。你无法指定**名称**和**节点**参数。

* **Node** - 要停止的节点的 HpcNode 对象，可通过 HPC PowerShell cmdlet [Get-HpcNode](https://technet.microsoft.com/zh-cn/library/dn887927.aspx) 获得。参数集名称是“节点”。你无法指定**名称**和**节点**参数。

* **Force**（可选）- 在停止 HPC 节点之前强制其脱机的设置。

### 示例

下面的示例强制名称以 *HPCNode-CN-* 开头的脱机节点，然后停止这些节点。

	Stop-HPCIaaSNode.ps1 -Name HPCNodeCN-* -Force

## 后续步骤

* 如果你想要有一种方法能够根据群集上作业及任务的当前工作负荷自动增加或减少群集节点，请参阅[根据集群工作流自动增加和减少 Azure 中的 HPC Pack 群集资源](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-node-autogrowshrink/)。

<!---HONumber=Mooncake_0215_2016-->