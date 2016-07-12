<properties
	pageTitle="在经典 Windows 虚拟机上设置终结点 | Azure"
	description="了解如何在 Azure 经典管理门户中设置终结点以允许与 Azure 中的 Windows 虚拟机通信。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="04/19/2016"
	wacn.date="06/29/2016"/>

# 如何在经典 Azure 虚拟机上设置终结点

在 Azure 中使用经典部署模型创建的所有虚拟机都可以通过专用网络通道与同一云服务或虚拟网络中的其他虚拟机自动通信。但是，Internet 上的计算机或其他虚拟网络需要终结点才能定向虚拟机的入站网络流量。这篇文章也适用于 [Linux 虚拟机](/documentation/articles/virtual-machines-linux-classic-setup-endpoints/)。

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用[资源管理器模型](/documentation/articles/virtual-machines-windows-nsg-quickstart-portal/)。在 **Resource Manager** 部署模型中，终结点是通过**网络安全组（NSG）**配置的。


在 Azure 经典管理门户中创建 Windows 虚拟机时，通常会为你自动创建常用终结点（如用于远程桌面和 Windows PowerShell 远程处理的终结点），具体取决于你选择的操作系统。你可以在创建虚拟机时或之后根据需要配置其他终结点。

[AZURE.INCLUDE [virtual-machines-common-classic-setup-endpoints](../includes/virtual-machines-common-classic-setup-endpoints.md)]

## 下一步


* 使用 Azure PowerShell cmdlet 设置 VM 终结点，请参阅 [Add-AzureEndpoint](https://msdn.microsoft.com/zh-cn/library/azure/dn495300.aspx)。

* 使用 Azure PowerShell cmdlet 管理终结点的 ACL，请参阅[使用 PowerShell 管理终结点的访问控制列表 (ACL)](/documentation/articles/virtual-networks-acl-powershell/)。

<!---HONumber=Mooncake_0215_2016-->