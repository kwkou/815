<properties
	pageTitle="在 Azure 中登录到运行 Linux 的虚拟机"
	description="了解如何使用安全外壳 (SSH) 客户端登录到运行 Linux 的 Azure 虚拟机。"
	services="virtual-machines"
	documentationCenter=""
	authors="squillace"
	manager="timlt"
	editor=""/>

<tags
	ms.service="virtual-machines"
	ms.date="03/16/2015"
	wacn.date="09/18/2015"/>




#如何登录到运行 Linux 的虚拟机 #

对于运行 Linux 操作系统的虚拟机，你可以使用安全外壳 (SSH) 客户端来登录。

需要在用于登录到虚拟机的计算机上安装 SSH 客户端。您可以选择很多 SSH 客户端程序。以下是可能的选择：

- 在运行 Windows 操作系统的计算机上，你可能希望使用 PuTTY 之类的 SSH 客户端。有关详细信息，请参阅 [PuTTY 下载页](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)。
- 在运行 Linux 操作系统的计算机上，你可能希望使用 OpenSSH 之类的 SSH 客户端。有关详细信息，请参阅 [OpenSSH](http://www.openssh.org/)。

>[AZURE.NOTE]如需了解更多要求和疑难解答提示，请参阅[使用 RDP 或 SSH 连接到 Azure 虚拟机](http://go.microsoft.com/fwlink/p/?LinkId=398294)。

此过程将向你演示如何使用 PuTTY 程序访问虚拟机。

1. 请从“管理门户”查找“主机名”和“端口信息”。[](http://manage.windowsazure.cn)您可以从虚拟机的仪表板中找到所需信息。单击虚拟机名称并查看仪表板“速览”部分的“SSH 详细信息”。

	![获取 SSH 详细信息](./media/virtual-machines-linux-how-to-log-on/sshdetails.png)

2. 打开 PuTTY 程序。

3. 输入你从仪表板收集到的“主机名”和“端口信息”，然后单击“打开”。

	![打开 PuTTY](./media/virtual-machines-linux-how-to-log-on/putty.png)

4. 使用你在创建虚拟机时指定的帐户登录到虚拟机。有关如何使用用户名和密码创建虚拟机的详细信息，请参阅[创建运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-tutorial)。

	![登录到虚拟机](./media/virtual-machines-linux-how-to-log-on/sshlogin.png)

>[AZURE.NOTE]VMAccess 扩展可用于重置 SSH 密钥或密码（如果你忘记了它们）。如果你忘记了用户名，你可以通过该扩展使用 sudo 权限创建一个新的。有关说明，请参阅[如何为 Linux 虚拟机重置密码或 SSH]。

您现在可以像使用任何其他服务器一样使用该虚拟机。

<!-- LINKS -->
[如何为 Linux 虚拟机重置密码或 SSH]: /documentation/articles/virtual-machines-linux-use-vmaccess-reset-password-or-ssh
<!---HONumber=70-->