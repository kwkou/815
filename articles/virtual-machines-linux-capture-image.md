<properties linkid="manage-linux-howto-capture-an-image" urlDisplayName="Capture an image" pageTitle="Capture an image of a virtual machine running Linux" metaKeywords="Azure Linux vm, Linux vm" description="Learn how to capture an image of an Azure virtual machine (VM) running Linux. " metaCanonical="" services="virtual-machines" documentationCenter="" title="How to Capture an Image of a Virtual Machine Running Linux" authors="kathydav" solutions="" manager="jeffreyg" editor="tysonn" />

# 如何捕获一台 Linux 虚拟机以用作模板

本文介绍如何捕获一台 Linux 虚拟机以便将其用作创建其他虚拟机的模板。此虚拟机模板包括操作系统磁盘以及附加到该虚拟机的任何数据磁盘。它不包括网络配置，所以你需要在使用该模板创建其他虚拟机时配置网络。

捕获到虚拟机后，你在创建虚拟机时可以在“我的映像”下使用它。该映像文件在你的存储帐户中存储为 VHD。有关映像的详细信息，请参阅[管理磁盘和映像][管理磁盘和映像]。

## 开始之前

这些步骤假定你已经创建了 Azure 虚拟机并配置了操作系统，包括附加任何数据磁盘。如果尚未完成这些操作，请参阅以下说明：

-   [如何创建自定义虚拟机][如何创建自定义虚拟机]
-   [如何将数据磁盘附加到虚拟机][如何将数据磁盘附加到虚拟机]

## 捕获虚拟机

1.  通过单击命令栏上的“连接”连接到虚拟机。有关详细信息，请参阅[如何登录到运行 Linux 的虚拟机][如何登录到运行 Linux 的虚拟机]。

2.  在 SSH 窗口中，键入下面的命令，然后输入在虚拟机上创建的帐户的密码：请注意，来自`waagent` 的输出结果可能会因此实用程序的版本而略有差异：

    `sudo waagent -deprovision`

    ![取消设置虚拟机][取消设置虚拟机]

3.  键入 **y** 以继续。

    ![成功地取消设置虚拟机][成功地取消设置虚拟机]

4.  键入 **Exit** 以关闭 SSH 客户端。

5.  在[管理门户][管理门户]中，选择虚拟机，然后单击“关机”。

6.  当虚拟机停止时，在命令栏上单击“捕获”以打开“捕获虚拟机”对话框。

7.  在“映像名称”框中，为新映像输入名称。

8.  所有的 Linux 映像都必须通过运行`waagent` 命令（使用`-deprovision` 选项）*取消设置*。单击“我已在虚拟机上运行 waagent-deprovision”以指示操作系统准备就绪，可随时将其用作映像。

9.  单击复选标记以捕获映像。

    新映像现在显示在“映像”下。捕获映像后，将删除虚拟机。

    ![成功捕获映像][成功捕获映像]

## 后续步骤

该映像已就绪，可用作创建虚拟机的模板了。为此，你要通过使用“从库中”方法并选择你刚创建的映像创建一个自定义虚拟机。有关说明，请参阅[如何创建自定义虚拟机][如何创建自定义虚拟机]。

  [管理磁盘和映像]: http://msdn.microsoft.com/zh-cn/library/azure/jj672979.aspx
  [如何创建自定义虚拟机]: ../virtual-machines-create-custom/
  [如何将数据磁盘附加到虚拟机]: ../storage-windows-attach-disk/
  [如何登录到运行 Linux 的虚拟机]: ../virtual-machines-linux-how-to-log-on
  [取消设置虚拟机]: ./media/virtual-machines-linux-capture-image/LinuxDeprovision.png
  [成功地取消设置虚拟机]: ./media/virtual-machines-linux-capture-image/LinuxDeprovision2.png
  [管理门户]: http://manage.windowsazure.cn
  [成功捕获映像]: ./media/virtual-machines-linux-capture-image/VMCapturedImageAvailable.png
