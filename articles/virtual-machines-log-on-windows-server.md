<properties
	pageTitle="登录到 VM | Azure"
	description="使用 Azure 门户登录到使用经典部署模型创建的 Windows 虚拟机。"
	services="virtual-machines"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="10/05/2015"
	wacn.date="12/31/2015"/>


# 使用 Azure 门户登录到 Windows 虚拟机



在 Azure 门户中，使用“连接”按钮启动远程桌面会话，然后登录到 Windows VM。

是否要连接到 Linux VM？ 请参阅[如何登录到运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-how-to-log-on)。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-classic-include.md)]

## 连接到虚拟机

1. 登录到 Azure 门户。

2. 单击“虚拟机”，然后选择虚拟机。

3. 在页面底部的命令栏中，单击“连接”。

	![登录到虚拟机](./media/virtual-machines-log-on-windows-server/connectwindows.png)
	
> [AZURE.TIP]如果“连接”按钮不可用，请参阅本文末尾的故障排除提示。

## 登录到虚拟机

[AZURE.INCLUDE [virtual-machines-log-on-win-server](../includes/virtual-machines-log-on-win-server.md)]

## 故障排除提示

下面是可以尝试的一些事项：

-	如果“连接”按钮处于非活动状态或者你在使用远程桌面连接时遇到其他问题，可尝试重置配置。在虚拟机仪表板的“速览”下，单击“重置远程配置”。
-	如果密码出现问题，可尝试重置密码。在虚拟机仪表板的“速览”下，单击“重置密码”。

如果这些提示不起作用或不是你所需的内容，请参阅[对与基于 Windows 的 Azure 虚拟机的远程桌面连接进行故障排除](/documentation/articles/virtual-machines-troubleshoot-remote-desktop-connections)。此文将指导你完成诊断和解决常见问题。

<!---HONumber=Mooncake_1221_2015-->