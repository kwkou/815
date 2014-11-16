<properties linkid="manage-windows-howto-logon" urlDisplayName="Log on to a VM" pageTitle="Log on to a virtual machine running Windows Server" metaKeywords="Azure logging on vm, vm portal" description="Learn how to log on to a virtual machine running Windows Server 2008 R2 by using the Azure Management Portal." metaCanonical="" services="virtual-machines" documentationCenter="" title="How to Log on to a Virtual Machine Running Windows Server" authors="kathydav" solutions="" manager="jeffreyg" editor="tysonn" />




# 如何登录到运行 Windows Server 的虚拟机

对于运行 Windows Server 操作系统的虚拟机，你可以使用管理门户中的“连接”按钮建立远程桌面连接。

1.  登录到 [Azure 管理门户](http://manage.windowsazure.cn) - 如果你尚未这么做。

2.  单击**“虚拟机”**，然后选择相应的虚拟机。

3.  在命令栏中，单击“连接”。

	![登录到虚拟机](./media/virtual-machines-log-on-windows-server/connectwindows.png)

4.  单击“打开”以使用为虚拟机自动创建的远程桌面协议文件。

5.  单击“连接”继续连接过程。

	![继续连接](./media/virtual-machines-log-on-windows-server/connectpublisher.png)

6.  输入虚拟机上 Administrator 帐户的用户名和密码，然后单击“确定”。


7.  单击“是”以验证虚拟机的标识。

	![验证计算机的标识](./media/virtual-machines-log-on-windows-server/connectverify.png)
	你现在可以像使用任何其他服务器一样使用该虚拟机。

[WACOM.NOTE] 有关要求和疑难解答提示，请参阅[使用 RDP 或 SSH 连接到 Windows Azure 虚拟机](http://msdn.microsoft.com/zh-cn/library/azure/dn535788.aspx)。