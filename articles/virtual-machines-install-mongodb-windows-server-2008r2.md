<properties linkid="manage-windows-common-task-mongodb-vm" urlDisplayName="Install MongoDB" pageTitle="在 Windows Server 虚拟机上安装 MongoDB" metaKeywords="Azure vm, Azure MongoDB, Azure remote desktop" description="了解如何使用 Windows Server 2008 R2 创建 Azure 虚拟机，然后使用远程桌面安装 MongoDB。" metaCanonical="" services="virtual-machines" documentationCenter="" title="在 Azure 中，在运行 Windows Server 的虚拟机上安装 MongoDB" authors="kathydav" solutions="" manager="jeffreyg" editor="tysonn" />

# 在 Azure 中，在运行 Windows Server 的虚拟机上安装 MongoDB

[MongoDB][MongoDB] 是一个受欢迎的开源、高性能 NoSQL 数据库。通过使用 [Azure 管理门户][Azure 管理门户]，您可以从映像库中创建运行 Windows Server 的虚拟机。然后，您可以在虚拟机上安装和配置 MongoDB 数据库。

在本教程中，您将学习：

-   如何使用管理门户从库创建 Windows Server 虚拟机。
-   如何使用远程桌面连接到虚拟机。
-   如何在虚拟机上安装 MongoDB。

## 创建运行 Windows Server 2008 R2 的虚拟机

[WACOM.INCLUDE [create-and-configure-windows-server-2008-vm-in-portal](../includes/create-and-configure-windows-server-2008-vm-in-portal.md)]

## 附加数据磁盘

[WACOM.INCLUDE [attach-data-disk-windows-server-2008-vm-in-portal](../includes/attach-data-disk-windows-server-2008-vm-in-portal.md)]

## 在该虚拟机上安装和运行 MongoDB

[WACOM.INCLUDE [install-and-run-mongo-on-win2k8-vm](../includes/install-and-run-mongo-on-win2k8-vm.md)]

## 摘要

在本教程中，您已了解如何创建 Windows Server 虚拟机以及如何以远程方式连接到该虚拟机。您还了解了如何在 Windows 虚拟机上安装和配置 MongoDB。有关 MongoDB 的详细信息，请参阅 [MongoDB 文档][MongoDB 文档]。

  [MongoDB]: http://www.mongodb.org/
  [Azure 管理门户]: http://manage.windowsazure.cn
  [MongoDB 文档]: http://www.mongodb.org/display/DOCS/Home
