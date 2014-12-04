<properties linkid="manage-linux-common-task-mongodb-virtual-machine" urlDisplayName="Install MongoDB" pageTitle="Install MongoDB on a Linux virtual machine in Azure" metaKeywords="Azure vm CentOS, Azure vm Linux, Linux vm, Linux MongoDB" description="Learn how to create an Azure virtual machine with CentOS Linux, and then use SSH or PuTTY to install MongoDB." metaCanonical="" services="virtual-machines" documentationCenter="" title="Install MongoDB on a virtual machine running CentOS Linux in Azure" authors="kathydav" solutions="" manager="jeffreyg" editor="tysonn" />

# 在 Azure 中运行 CentOS Linux 的虚拟机上安装 MongoDB

[MongoDB][MongoDB] 是一个受欢迎的开源、高性能 NoSQL 数据库。使用 Azure 管理门户，你可从映像库创建运行 CentOS Linux 的虚拟机。然后，你可以在虚拟机上安装和配置 MongoDB 数据库。

你将了解到以下内容：

-   如何使用管理门户从库中选择并安装一台运行 CentOS Linux 的 Linux 虚拟机。

-   如何使用 SSH 或 PuTTY 连接到该虚拟机。
-   如何在虚拟机上安装 MongoDB。

## 创建运行 CentOS Linux 的虚拟机

[WACOM.INCLUDE [create-and-configure-centos-vm-in-portal](../includes/create-and-configure-centos-vm-in-portal.md)]

## 附加数据磁盘

[WACOM.INCLUDE [attach-data-disk-centos-vm-in-portal](../includes/attach-data-disk-centos-vm-in-portal.md)]

## 在该虚拟机上安装和运行 MongoDB

[WACOM.INCLUDE [install-and-run-mongo-on-centos-vm](../includes/install-and-run-mongo-on-centos-vm.md)]

## 摘要

在本教程中，你已了解如何创建 Linux 虚拟机以及使用 SSH 或 PuTTY 远程连接到该虚拟机。你还了解了如何在 Linux 虚拟机上安装和配置 MongoDB。有关 MongoDB 的详细信息，请参阅 [MongoDB 文档][MongoDB 文档]。

  [MongoDB]: http://www.mongodb.org/
  [create-and-configure-centos-vm-in-portal]: ../includes/create-and-configure-centos-vm-in-portal.md
  [attach-data-disk-centos-vm-in-portal]: ../includes/attach-data-disk-centos-vm-in-portal.md
  [install-and-run-mongo-on-centos-vm]: ../includes/install-and-run-mongo-on-centos-vm.md
  [MongoDB 文档]: http://www.mongodb.org/display/DOCS/Home
