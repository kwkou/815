<properties
	pageTitle="登录到 Azure 中的 Linux VM | Azure"
	description="了解如何使用安全外壳 (SSH) 客户端登录到运行 Linux 的 Azure 虚拟机。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="squillace"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="04/18/2016"
	wacn.date="06/13/2016"/>


#如何登录到运行 Linux 的虚拟机 #

> [AZURE.IMPORTANT] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用 [Resource Manager 部署模型](/documentation/articles/virtual-machines-linux-quick-create-portal/)。

需要在用于登录到虚拟机的计算机上安装 SSH 客户端。您可以选择很多 SSH 客户端程序。以下是可能的选择：

- 对于运行 Linux 操作系统的虚拟机，你使用 Secure Shell (SSH) 客户端来登录；很难想象你可以使用默认情况下不安装该客户端的分发。请参阅[如何使用 SSH](/documentation/articles/virtual-machines-linux-ssh-from-linux/)，了解有关 Linux 的更多信息。
- 在运行 Windows 操作系统的计算机上，你可能希望使用 PuTTY 之类的 SSH 客户端。有关详细信息，请参阅 [PuTTY 下载页](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)。


>[AZURE.NOTE]如需了解更多要求和疑难解答提示，请参阅[使用 RDP 或 SSH 连接到 Azure 虚拟机](/documentation/articles/virtual-machines-linux-about/)。

此过程将演示如何使用 OS X 上的 SSH 客户端访问虚拟机。

1. 请从“[经典管理门户](http://manage.windowsazure.cn)”查找“主机名”和“端口信息”。您可以从虚拟机的仪表板中找到所需信息。单击虚拟机名称并查看仪表板“速览”部分的“SSH 详细信息”。

	![获取 SSH 详细信息](./media/virtual-machines-linux-classic-log-on/portalsshdetails.png)

2. 使用你在创建虚拟机时指定的帐户以及相应的主机名和端口登录到虚拟机。有关如何使用用户名和密码创建虚拟机的详细信息，请参阅[创建运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-classic-createportal/)。

	![登录到虚拟机](./media/virtual-machines-linux-classic-log-on/sshport.png)

>[AZURE.NOTE] VMAccess 扩展可用于重置 SSH 密钥或密码（如果你忘记了它们）。如果你忘记了用户名，你可以通过该扩展使用 sudo 权限创建一个新的。有关说明，请参阅[如何为 Linux 虚拟机重置密码或 SSH]。

您现在可以像使用任何其他服务器一样使用该虚拟机。

<!-- LINKS -->
[如何为 Linux 虚拟机重置密码或 SSH]: /documentation/articles/virtual-machines-linux-classic-reset-access/

<!---HONumber=Mooncake_0606_2016-->