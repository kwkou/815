<properties linkid="manage-windows-howto-capture-an-image" urlDisplayName="Capture an image" pageTitle="捕获运行 Windows Server 的虚拟机的映像" metaKeywords="Azure capture image vm, capturing vm" description="了解如何捕获运行 Windows Server 2008 R2 的 Azure 虚拟机 (VM) 的映像。 " metaCanonical="" services="virtual-machines" documentationCenter="" title="How to Capture an Image of a Virtual Machine Running Windows Server" authors="kathydav" solutions="" manager="jeffreyg" editor="tysonn" />

#如何捕获一台要用作模板的 Windows 虚拟机#

本文介绍如何捕获一台运行 Windows 的 Azure 虚拟机，以便将其用作创建其他虚拟机的模板。此模板包括操作系统磁盘以及附加到该虚拟机的任何数据磁盘。它不包括网络配置，所以您需要在创建使用该模板的其他虚拟机时配置网络。

Azure 将此模板视为一个映像，并将其存储在"我的映像"****下。这也是您之前上载的所有映像的存储位置。有关映像的详细信息，请参阅[关于 Azure 中的虚拟机映像][]。

##开始之前##

这些步骤假定您已经创建一台 Azure 虚拟机并配置了操作系统，包括附加任何数据磁盘。如果您尚未完成这些操作，请参阅以下说明：

- [如何创建自定义虚拟机][]
- [如何将数据磁盘附加到虚拟机][]

##捕获虚拟机##

1. 通过单击命令栏上的"连接"****连接到虚拟机。有关详细信息，请参阅[如何登录到运行 Windows Server 的虚拟机][]。

2.	以管理员身份打开"命令提示符"窗口。


3.	将目录更改为"%windir%\system32\sysprep"，然后再运行 sysprep.exe。


4. 	将显示"系统准备工具"****对话框。请执行以下操作：


	- 在"系统清理操作"****中，选择"进入系统全新体验(OOBE)"****，并确保选中"通用化"****。有关使用 Sysprep 的详细信息，请参阅[如何使用 Sysprep：简介][]。

	- 在"关机选项"****中，选择"关机"****。

	- 单击"确定"****。

	![Run Sysprep](./media/virtual-machines-capture-image-windows-server/SysprepGeneral.png)

7.	Sysprep 将关闭虚拟机，这会在[管理门户](http://manage.windowsazure.cn)中将虚拟机的状态更改为"已停止"****。


8.	单击"虚拟机"****，然后选择您要捕获的虚拟机。

9.	在命令栏上，单击"捕获"****。

	![Capture virtual machine](./media/virtual-machines-capture-image-windows-server/CaptureVM.png)

	将显示"捕获虚拟机"****对话框。

10.	在"映像名称"****中，为新映像键入名称。

11.	在将 Windows Server 映像添加到自定义映像组之前，该映像必须先按前面步骤中的说明通过运行 Sysprep 来通用化。单击"我已在虚拟机上运行了 Sysprep"****以指明您已经完成了此操作。

12.	单击复选标记以捕获映像。在捕获已通用化的虚拟机的映像时，会删除该虚拟机。

	新映像现在显示在"映像"****下。

	![Image capture successful](./media/virtual-machines-capture-image-windows-server/VMCapturedImageAvailable.png)

##后续步骤##
该映像已准备就绪，可用作创建虚拟机的模板。为此，您要通过使用"从库中"****方法并选择您刚创建的映像创建一个自定义虚拟机。有关说明，请参阅[如何创建自定义虚拟机][]。

	
[管理磁盘和映像]:http://msdn.microsoft.com/zh-cn/library/azure/jj672979.aspx
[如何创建自定义虚拟机]: ../virtual-machines-create-custom/
[如何将数据磁盘附加到虚拟机]: ../storage-windows-attach-disk/
[如何登录到运行 Windows Server 的虚拟机]:/zh-cn/documentation/articles/virtual-machines-log-on-windows-server/
[如何使用 Sysprep：简介]:http://technet.microsoft.com/zh-cn/library/bb457073.aspx
[运行 Sysprep.exe]: ./media/virtual-machines-capture-image-windows-server/SysprepCommand.png
[进入 Sysprep.exe 选项]: ./media/virtual-machines-capture-image-windows-server/SysprepGeneral.png
[虚拟机已停止]: ./media/virtual-machines-capture-image-windows-server/SysprepStopped.png
[捕获虚拟机的映像]: ./media/virtual-machines-capture-image-windows-server/CaptureVM.png
[输入映像名称]: ./media/virtual-machines-capture-image-windows-server/Capture.png
[成功捕获映像]: ./media/virtual-machines-capture-image-windows-server/CaptureSuccess.png
[使用捕获的映像]: ./media/virtual-machines-capture-image-windows-server/MyImagesWindows.png
