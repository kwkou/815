<properties
    pageTitle="在 Azure 中重新部署 Windows 虚拟机 | Azure"
    description="如何通过在 Azure 中重新部署 Windows 虚拟机来缓解 RDP 连接问题。"
    services="virtual-machines-windows"
    documentationcenter="virtual-machines"
    author="iainfoulds"
    manager="timlt"
    tags="azure-resource-manager,top-support-issue" />
<tags
    ms.assetid="0ee456ee-4595-4a14-8916-72c9110fc8bd"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="support-article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="infrastructure"
    ms.date="12/16/2016"
    wacn.date="01/20/2017"
    ms.author="iainfou" />  


# 将 Windows 虚拟机重新部署到新的 Azure 节点
如果在对远程桌面 (RDP) 连接或应用程序对基于 Windows 的 Azure 虚拟机 (VM) 的访问进行故障排除时遇到困难，重新部署 VM 可能会有帮助。重新部署 VM 时，将 VM 移到 Azure 基础结构中的新节点，然后重新打开它，同时保留所有配置选项和关联的资源。本文介绍如何使用 Azure PowerShell 或 Azure 门户重新部署 VM。

> [AZURE.NOTE]
重新部署 VM 后，临时磁盘将丢失，与虚拟网络接口关联的动态 IP 地址将更新。

## 使用 Azure PowerShell
请确保已在计算机上安装最新的 Azure PowerShell 1.x。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)。

以下示例在名为 `myResourceGroup` 的资源组中部署名为 `myVM` 的 VM：

    Set-AzureRmVM -Redeploy -ResourceGroupName "myResourceGroup" -Name "myVM"

[AZURE.INCLUDE [virtual-machines-common-redeploy-to-new-node](../../includes/virtual-machines-common-redeploy-to-new-node.md)]

## 后续步骤
如果在连接 VM 时遇到问题，可以在 [RDP 连接故障排除](/documentation/articles/virtual-machines-windows-troubleshoot-rdp-connection/)或[详细的 RDP 故障排除步骤](/documentation/articles/virtual-machines-windows-detailed-troubleshoot-rdp/)中找到特定的帮助。如果无法访问在 VM 上运行的应用程序，还可以阅读[应用程序故障排除问题](/documentation/articles/virtual-machines-windows-troubleshoot-app-connection/)。

<!---HONumber=Mooncake_0116_2017-->
<!--Update_Description: update meta properties & wording update-->