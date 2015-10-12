<properties
	pageTitle="登录到运行 Windows Server 的虚拟机"
	description="了解如何使用 Azure 门户登录到运行 Windows Server 的虚拟机。"
	services="virtual-machines"
	documentationCenter=""
	authors="KBDAzure"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="06/12/2015"
	wacn.date="09/18/2015"/>


# 如何登录到运行 Windows Server 的虚拟机#

你将在 Azure 门户中使用“连接”按钮来启动远程桌面会话。（有关 Linux VM 的信息，请参阅[如何登录到运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-how-to-log-on)。）

## 如何登录

[AZURE.INCLUDE [virtual-machines-log-on-win-server](../includes/virtual-machines-log-on-win-server.md)]

## 故障排除提示

下面是可以快速尝试的一些事项：

如果远程桌面连接出现问题，可尝试通过门户重置配置。在虚拟机仪表板的“速览”下，单击“重置远程配置”。

如果密码出现问题，可尝试在门户中重置密码。在虚拟机仪表板的“速览”下，单击“重置密码”。

如果这些方法无效，则需进行更广泛的故障排除操作。有关说明，请参阅[对与基于 Windows 的 Azure 虚拟机的远程桌面连接进行故障排除](/documentation/articles/virtual-machines-troubleshoot-remote-desktop-connections)。

<!---HONumber=70-->