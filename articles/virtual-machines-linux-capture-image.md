<properties linkid="manage-linux-howto-capture-an-image" urlDisplayName="Capture an image" pageTitle="捕获运行 Linux 的虚拟机的映像" metaKeywords="Azure Linux vm, Linux vm" description="了解如何捕获运行 Linux 的 Azure 虚拟机 (VM) 的映像。 " metaCanonical="" services="virtual-machines" documentationCenter="" title="How to Capture an Image of a Virtual Machine Running Linux" authors="kathydav" solutions="" manager="jeffreyg" editor="tysonn" />



# 如何捕获一台 Linux 虚拟机以用作模板##

本文介绍如何捕获一台运行 Linux 的 Azure 虚拟机以便将其用作创建其他虚拟机的模板。此模板包括操作系统磁盘以及附加到该虚拟机的任何数据磁盘。它不包括网络配置，所以您需要在创建使用该模板的其他虚拟机时配置网络。

Azure 将此模板视为一个映像，并将其存储在"我的映像"****下。这也是您之前上载的所有映像的存储位置。有关映像的详细信息，请参阅[关于 Azure 中的虚拟机映像][]。

##开始之前##

这些步骤假定您已经创建一台 Azure 虚拟机并配置了操作系统，包括附加任何数据磁盘。如果您尚未完成这些操作，请参阅以下说明：

- [如何创建自定义虚拟机][]
- [如何将数据磁盘附加到虚拟机][]

##捕获虚拟机##

1. 通过单击命令栏上的"连接"****连接到虚拟机。有关详细信息，请参阅[如何登录到运行 Linux 的虚拟机][]。

2. 在 SSH 窗口中，键入下面的命令，然后输入在虚拟机上创建的帐户的密码。请注意 `waagent` 的输出可能稍有不同，具体取决于此实用程序的版本：

	`sudo waagent -deprovision`

	![Deprovision the virtual machine](./media/virtual-machines-linux-capture-image/LinuxDeprovision.png)


3. 键入 **y** 以继续。

	![Deprovision of virtual machine successful](./media/virtual-machines-linux-capture-image/LinuxDeprovision2.png)

4. 键入 **Exit** 以关闭 SSH 客户端。

5. 在[管理门户](http://manage.windowsazure.cn)中，选择虚拟机，然后单击"关机"****。

6. 当虚拟机停止时，在命令栏上单击"捕获"****以打开"捕获虚拟机"****对话框。

7.	在"映像名称"****框中，为新映像输入名称。

8.	必须通过运行带有 `-deprovision` 选项的 `waagent` 命令来 *deprovisioned* 所有 Linux 映像。单击"我已在虚拟机上运行 waagent-deprovision"****以指示操作系统准备就绪，可随时将其用作映像。

9.	单击复选标记以捕获映像。

	新映像现在显示在"映像"****下。捕获映像后，将删除虚拟机。

	![Image capture successful](./media/virtual-machines-linux-capture-image/VMCapturedImageAvailable.png)

##后续步骤##
该映像已准备就绪，可用作创建虚拟机的模板。为此，您要通过使用"从库中"****方法并选择您刚创建的映像创建一个自定义虚拟机。有关说明，请参阅[如何创建自定义虚拟机][]。
	
[如何登录到运行 Linux 的虚拟机]: ../virtual-machines-linux-how-to-log-on
[管理磁盘和映像]:http://msdn.microsoft.com/zh-cn/library/azure/jj672979.aspx
[如何创建自定义虚拟机]: ../virtual-machines-create-custom/
[如何将数据磁盘附加到虚拟机]: ../storage-windows-attach-disk/

