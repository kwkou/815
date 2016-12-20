<properties
    pageTitle="调整经典 Windows VM 的大小 | Azure"
    description="使用 Azure Powershell 调整在经典部署模型中创建的 Windows 虚拟机的大小。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="Drewm3"
    manager="timlt"
    editor=""
    tags="azure-service-management" />  

<tags
    ms.assetid="e3038215-001c-406e-904d-e0f4e326a4c7"
    ms.service="virtual-machines-windows"
    ms.workload="na"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/19/2016"
    wacn.date="12/20/2016"
    ms.author="drewm" />  


# 调整在经典部署模型中创建的 Windows VM 的大小
本文说明如何使用 Azure Powershell 调整在经典部署模型中创建的 Windows VM 的大小。

在考虑调整 VM 大小的能力时，有两个概念控制可用于调整虚拟机大小的大小范围。第一个概念是在其中部署 VM 的区域。在“Azure 区域”网页的“服务”选项卡下提供区域中可用的 VM 大小的列表。第二个概念是当前正在托管 VM 的物理硬件。托管 VM 的物理服务器在常见物理硬件的群集中分组在一起。更改 VM 大小的方法因当前托管 VM 的硬件群集是否支持所需的新 VM 大小而异。

[AZURE.INCLUDE [了解部署模型](../../includes/learn-about-deployment-models-classic-include.md)]

还可以[调整在 Resource Manager 部署模型中创建的 VM 的大小](/documentation/articles/virtual-machines-windows-resize-vm/)。

## 添加帐户
必须将 Azure PowerShell 配置为使用经典 Azure 资源。请按照以下步骤配置 Azure PowerShell 以管理经典资源。

1. 在 PowerShell 命令提示处，键入 `Add-AzureAccount -Environment AzureChinaCloud` 并单击“Enter”。
2. 键入与你的 Azure 订阅相关联的电子邮件地址并单击“继续”。
3. 键入你的帐户的密码。
4. 单击“登录”。

## 在同一硬件群集中调整大小
若要将 VM 的大小调整为托管 VM 的硬件群集中可用的大小，请执行以下步骤。

1. 运行以下 PowerShell 命令，列出托管包含 VM 的云服务的硬件群集中可用的 VM 大小。

        Get-AzureService | where {$_.ServiceName -eq "<cloudServiceName>"}

2. 运行以下命令以调整 VM 大小。

        Get-AzureVM -ServiceName <cloudServiceName> -Name <vmName> | Set-AzureVMSize -InstanceSize <newVMSize> | Update-AzureVM

## 在新的硬件群集上调整大小
若要将 VM 的大小调整为托管 VM 的硬件群集中不可用的大小，必须重新创建云服务和云服务中的所有 VM。单个硬件群集托管每个云服务，因此，云服务中的所有 VM 的大小必须都受单个硬件群集支持。以下步骤介绍如何通过删除并重新创建云服务来调整 VM 大小。

1. 运行以下 PowerShell 命令，列出区域中可用的 VM 大小。

        Get-AzureLocation | where {$_.Name -eq "<locationName>"}

2. 记下包含要调整大小的 VM 的云服务中每个 VM 的所有配置设置。
3. 删除云服务中的所有 VM 时选择保留每个 VM 的磁盘的选项。
4. 使用所需的 VM 大小重新创建要调整大小的 VM。
5. 使用现在托管云服务的硬件群集中可用的 VM 大小，重新创建云服务中原有的所有其他 VM。

删除云服务并使用新的 VM 大小重新创建云服务的示例脚本可在[此处](https://github.com/Azure/azure-vm-scripts)找到。

## 后续步骤
* [调整在 Resource Manager 部署模型中创建的 VM 的大小](/documentation/articles/virtual-machines-windows-resize-vm/)。

<!---HONumber=Mooncake_1212_2016-->