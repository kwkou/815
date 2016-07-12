<properties
	pageTitle="捕获 Azure Windows VM 的映像| Azure"
	description="捕获使用经典部署模型创建的 Azure Windows 虚拟机的映像。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="03/11/2016"
    	wacn.date="05/24/2016"/>


#捕获使用经典部署模型创建的 Azure Windows 虚拟机的映像。

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用[资源管理器模型](/documentation/articles/virtual-machines-windows-capture-image/)。

本文将演示如何捕获运行 Windows 的 Azure 虚拟机，你可以将它用作映像来创建其他虚拟机。此映像包含操作系统磁盘和任何附加到虚拟机的数据磁盘。由于它不包括网络配置，因此你在使用此映像创建其他虚拟机时，需要进行相关配置。

Azure 将映像存储在**“我的映像”**下。你上载的任何映像都会存储在同一位置。有关映像的详细信息，请参阅[关于虚拟机的映像](/documentation/articles/virtual-machines-windows-classic-about-images/)。

##开始之前##

这些步骤假定你已创建了 Azure 虚拟机并配置了操作系统，包括附加任何数据磁盘。如果你尚未执行此操作，请参阅以下说明：

- [从映像创建虚拟机](/documentation/articles/virtual-machines-windows-classic-createportal/)
- [如何将数据磁盘附加到虚拟机](/documentation/articles/virtual-machines-windows-classic-attach-disk/)

> [AZURE.WARNING]此过程会在捕获原始虚拟机后将其删除。

这并不适合作为备份虚拟机的方式。执行此操作的一个可行方法是 Azure 备份，它在特定区域中作为预览版提供。认证合作伙伴提供了其他解决方案。若要了解当前提供的内容，请搜索 Azure 应用商店。


##捕获虚拟机

1. 在 [Azure 经典管理门户](http://manage.windowsazure.cn)中，**连接**到虚拟机。有关说明，请参阅[如何登录到运行 Windows Server 的虚拟机][]。

2.	以管理员身份打开“命令提示符”窗口。

3.	将目录更改为 `%windir%\system32\sysprep`，然后运行 sysprep.exe。

4. 	此时会显示**“系统准备工具”**对话框。请执行以下操作：

	- 在**“系统清理操作”**中，选择**“进入系统全新体验(OOBE)”**，并确保选中**“一般化”**。有关使用 Sysprep 的详细信息，请参阅[如何使用 Sysprep：简介][]。

	- 在**“关机选项”**中选择**“关机”**。

	- 单击**“确定”**。

	![运行 Sysprep](./media/virtual-machines-windows-classic-capture-image/SysprepGeneral.png)

7.	Sysprep 将关闭虚拟机，这会在 Azure 经典管理门户中将虚拟机的状态更改为**“已停止”**。

8.	在 Azure 经典管理门户中，单击**“虚拟机”**，然后选择要捕获的虚拟机。

9.	在命令栏中，单击**“捕获”**。

	![捕获虚拟机](./media/virtual-machines-windows-classic-capture-image/CaptureVM.png)

	此时将显示**“捕获虚拟机”**对话框。

10.	在**“映像名称”**中，键入新映像的名称。

11.	在将 Windows Server 映像添加到自定义映像组之前，必须先按前面步骤中的说明通过运行 Sysprep 将该映像通用化。单击**“我已经在虚拟机上运行了 Sysprep”**以指明你已完成此操作。

12.	单击复选标记以捕获映像。新映像现在将显示在**“映像”**下。

 	![成功捕获映像](./media/virtual-machines-windows-classic-capture-image/VMCapturedImageAvailable.png)

##后续步骤

该映像已就绪，可用于创建虚拟机了。为此，你将通过使用**“从库中”**菜单项并选择你刚创建的映像来创建一个虚拟机。有关说明，请参阅[从映像创建虚拟机](/documentation/articles/virtual-machines-windows-classic-createportal/)。



[如何登录到运行 Windows Server 的虚拟机]: /documentation/articles/virtual-machines-windows-classic-connect-logon/
[如何使用 Sysprep：简介]: http://technet.microsoft.com/zh-cn/library/bb457073.aspx
[Run Sysprep.exe]: ./media/virtual-machines-windows-classic-capture-image/SysprepCommand.png
[Enter Sysprep.exe options]: ./media/virtual-machines-windows-classic-capture-image/SysprepGeneral.png
[The virtual machine is stopped]: ./media/virtual-machines-windows-classic-capture-image/SysprepStopped.png
[Capture an image of the virtual machine]: ./media/virtual-machines-windows-classic-capture-image/CaptureVM.png
[Enter the image name]: ./media/virtual-machines-windows-classic-capture-image/Capture.png
[Image capture successful]: ./media/virtual-machines-windows-classic-capture-image/CaptureSuccess.png
[Use the captured image]: ./media/virtual-machines-windows-classic-capture-image/MyImagesWindows.png

<!---HONumber=Mooncake_1207_2015-->