<!-- Ibiza portal: tested -->

<properties
	pageTitle="连接到 Windows Server VM | Azure"
	description="了解如何使用 Azure 门户预览和 Resource Manager 部署模型连接并登录到 Windows Server VM。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor="tysonn"
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="05/05/2016"
	wacn.date="06/29/2016"/>

# 如何连接并登录到运行 Windows 的 Azure 虚拟机 

你将在 Azure 门户预览中使用“连接”按钮来启动远程桌面（RDP）会话。首先，你将连接到虚拟机，然后将登录。

## 连接到虚拟机

1. 如果你尚未登录 [Azure 门户预览](https://portal.azure.cn/)，请先登录。

2.	在“中心”菜单中，单击“虚拟机”。

3.	从列表中选择虚拟机。

4. 在虚拟机边栏选项卡上，单击“连接”。

	![连接到虚拟机](./media/virtual-machines-windows-connect-logon/connect.png)

## 登录到虚拟机

[AZURE.INCLUDE [virtual-machines-log-on-win-server](../includes/virtual-machines-log-on-win-server.md)]

## 下一步

如果你尝试连接时遇到了麻烦，请参阅[对与基于 Windows 的 Azure 虚拟机的远程桌面连接进行故障排除](/documentation/articles/virtual-machines-windows-troubleshoot-rdp-connection)。此文将指导你完成诊断和解决常见问题。

<!---HONumber=Mooncake_0411_2016-->