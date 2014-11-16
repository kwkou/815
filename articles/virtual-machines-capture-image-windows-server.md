<properties linkid="manage-windows-howto-capture-an-image" urlDisplayName="Capture an image" pageTitle="Capture an image of a virtual machine running Windows Server" metaKeywords="Azure capture image vm, capturing vm" description="Learn how to capture an image of an Azure virtual machine (VM) running Windows Server 2008 R2. " metaCanonical="" services="virtual-machines" documentationCenter="" title="How to Capture an Image of a Virtual Machine Running Windows Server" authors="kathydav" solutions="" manager="jeffreyg" editor="tysonn" />

# 如何捕获一台用作模板的 Windows 虚拟机

本文介绍如何捕获一台 Windows 虚拟机以便将其用作创建其他虚拟机的模板。此虚拟机模板包括操作系统磁盘以及附加到该虚拟机的任何数据磁盘。它不包括网络配置，所以你需要在使用该模板创建其他虚拟机时配置网络。

捕获到虚拟机后，你在创建虚拟机时可以在“我的映像”下使用它。该映像文件在你的存储帐户中存储为 VHD。有关映像的详细信息，请参阅[管理磁盘和映像][管理磁盘和映像]。

## 开始之前

这些步骤假定你已经创建了 Azure 虚拟机并配置了操作系统，包括附加任何数据磁盘。如果尚未完成这些操作，请参阅以下说明：

-   [如何创建自定义虚拟机][如何创建自定义虚拟机]
-   [如何将数据磁盘附加到虚拟机][如何将数据磁盘附加到虚拟机]

## 捕获虚拟机

1.  通过单击命令栏上的“连接”连接到虚拟机。有关详细信息，请参阅[如何登录到运行 Windows Server 的虚拟机][如何登录到运行 Windows Server 的虚拟机]。

2.  以管理员身份打开“命令提示符”窗口。

3.  将目录更改为`%windir%\system32\sysprep`，然后运行 sysprep.exe。

4.  会显示**“系统准备工具”**对话框。请执行以下操作：

    -   在**“系统清理操作”**中，选择**“进入系统全新体验(OOBE)”**，并且一定要选中**“通用”**。有关使用 Sysprep 的更多信息，请参阅[如何使用 Sysprep：简介][如何使用 Sysprep：简介]。

    -   在**“关机选项”**中选择**“关机”**。

    -   单击“确定”。

    ![运行 Sysprep][运行 Sysprep]

5.  sysprep 命令将关闭虚拟机，这会在[管理门户][管理门户]中将虚拟机的状态更改为“已停止”。

6.  单击“虚拟机”，然后选择你要捕获的虚拟机。

7.  在命令栏中，单击“捕获”。

    ![捕获虚拟机][捕获虚拟机]

    将出现“捕获虚拟机”对话框。

8.  在“映像名称”框中，为新映像输入名称。

9.  在将 Windows Server 映像添加到自定义映像组之前，该映像必须先按前面步骤中的说明通过运行 Sysprep 来通用化。单击“我已在虚拟机上运行了 Sysprep ”以指明你已经完成了此操作。

10. 单击复选标记以捕获映像。在捕获已通用化的虚拟机的映像时，该虚拟机会被删除。

    新映像现在显示在“映像”下。

    ![成功捕获映像][成功捕获映像]

## 后续步骤

该映像已就绪，可用作创建虚拟机的模板了。为此，你要通过使用“从库中”方法并选择你刚创建的映像创建一个自定义虚拟机。有关说明，请参阅[如何创建自定义虚拟机][如何创建自定义虚拟机]。

  [管理磁盘和映像]: http://msdn.microsoft.com/zh-cn/library/azure/jj672979.aspx
  [如何创建自定义虚拟机]: ../virtual-machines-create-custom/
  [如何将数据磁盘附加到虚拟机]: ../storage-windows-attach-disk/
  [如何登录到运行 Windows Server 的虚拟机]: http://windowsazure.cn/zh-cn/documentation/articles/virtual-machines-log-on-windows-server/
  [如何使用 Sysprep：简介]: http://technet.microsoft.com/zh-cn/library/bb457073.aspx
  [运行 Sysprep]: ./media/virtual-machines-capture-image-windows-server/SysprepGeneral.png
  [管理门户]: http://manage.windowsazure.cn
  [捕获虚拟机]: ./media/virtual-machines-capture-image-windows-server/CaptureVM.png
  [成功捕获映像]: ./media/virtual-machines-capture-image-windows-server/VMCapturedImageAvailable.png
