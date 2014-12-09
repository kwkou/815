<properties linkid="store-mongodb-virtual-machines-linux-centos" urlDisplayName="Install MongoDB" pageTitle="nstall MongoDB on a virtual machine running CentOS Linux in Azure" metaKeywords="Azure, MongoDB" description="Learn how to install Mongo DB on a virtual machine in Azure." metaCanonical="" services="" documentationCenter="" title="Install MongoDB on a virtual machine running CentOS Linux in Azure" authors="" solutions="" manager="" editor="" />

# 在 Azure 中运行 CentOS Linux 的虚拟机上安装 MongoDB

[MongoDB][MongoDB] 是一个受欢迎的开源、高性能 NoSQL 数据库。使用 [Azure 管理门户][Azure 管理门户]，你可从映像库创建运行 CentOS Linux 的虚拟机。然后，你可以在虚拟机上安装和配置 MongoDB 数据库。

你将了解到以下内容：

-   如何使用管理门户从库中创建运行 CentOS Linux 的虚拟机。
-   如何使用 SSH 或 PuTTY 连接到该虚拟机。
-   如何在虚拟机上安装 MongoDB。

## 注册虚拟机预览版功能

为了创建虚拟机，你将需要注册 Azure 虚拟机预览版功能。如果你没有 Azure 帐户，也可以注册一个免费试用版帐户。

[WACOM.INCLUDE [antares-iaas-signup-iaas](../includes/antares-iaas-signup-iaas.md)]

## 创建运行 CentOS Linux 的虚拟机

[WACOM.INCLUDE [create-and-configure-centos-vm-in-portal](../includes/create-and-configure-centos-vm-in-portal.md)]

## 在该虚拟机上安装和运行 MongoDB

[WACOM.INCLUDE [install-and-run-mongo-on-centos-vm](../includes/install-and-run-mongo-on-centos-vm.md)]

## 摘要

在本教程中，你已了解如何创建 Linux 虚拟机以及使用 SSH 或 PuTTY 远程连接到该虚拟机。你还了解了如何在 Linux 虚拟机上安装和配置 MongoDB。有关 MongoDB 的详细信息，请参阅 [MongoDB 文档][MongoDB 文档]。

  [MongoDB]: http://www.mongodb.org/
  [Azure 管理门户]: http://manage.windowsazure.cn
  [antares-iaas-signup-iaas]: ../includes/antares-iaas-signup-iaas.md
  [create-and-configure-centos-vm-in-portal]: ../includes/create-and-configure-centos-vm-in-portal.md
  [install-and-run-mongo-on-centos-vm]: ../includes/install-and-run-mongo-on-centos-vm.md
  [MongoDB 文档]: http://www.mongodb.org/display/DOCS/Home
