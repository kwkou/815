<properties linkid="virtual-machines-linux-mysql-use-opensuse" urlDisplayName="Install MongoDB" pageTitle="nstall MongoDB on a virtual machine running CentOS Linux in Azure" metaKeywords="Azure, MongoDB" description="Learn how to install Mongo DB on a virtual machine in Azure." metaCanonical="" services="" documentationCenter="" title="Install MongoDB on a virtual machine running CentOS Linux in Azure" authors="" solutions="" manager="" editor="" />

# 在 Azure 中运行 OpenSUSE Linux 的虚拟机上安装 MySQL

[MySQL][MySQL] 是一种受欢迎的 SQL 开源数据库。使用 [Azure 管理门户][Azure 管理门户]，你可从映像库创建运行 OpenSUSE Linux 的虚拟机。然后，你可以在虚拟机上安装和配置 MySQL 数据库。

在本教程中，你将学习：

-   如何使用管理门户从库中创建 OpenSUSE Linux 虚拟机。
-   如何使用 SSH 或 PuTTY 连接到该虚拟机。
-   如何在该虚拟机上安装 MySQL。

## 注册虚拟机预览版功能

为了创建虚拟机，你将需要注册 Azure 虚拟机预览版功能。如果你没有 Azure 帐户，也可以注册一个免费试用版帐户。

[WACOM.INCLUDE [antares-iaas-signup-iaas][antares-iaas-signup-iaas]]

## 创建运行 OpenSUSE Linux 的虚拟机

[WACOM.INCLUDE [create-and-configure-opensuse-vm-in-portal][create-and-configure-opensuse-vm-in-portal]]

## 在虚拟机上安装和运行 MySQL

[WACOM.INCLUDE [install-and-run-mysql-on-opensuse-vm][install-and-run-mysql-on-opensuse-vm]]

## 摘要

在本教程中，你已了解如何创建运行 OpenSUSE Linux 的虚拟机以及使用 SSH 或 PuTTY 远程连接到该虚拟机。你还了解了如何在 Linux 虚拟机上安装和配置 MySQL。有关 MySQL 的详细信息，请参阅 [MySQL 文档][MySQL 文档]。

  [MySQL]: http://www.mysql.com
  [Azure 管理门户]: http://manage.windowsazure.cn
  [antares-iaas-signup-iaas]: ../includes/antares-iaas-signup-iaas.md
  [create-and-configure-opensuse-vm-in-portal]: ../includes/create-and-configure-opensuse-vm-in-portal.md
  [install-and-run-mysql-on-opensuse-vm]: ../includes/install-and-run-mysql-on-opensuse-vm.md
  [MySQL 文档]: http://dev.mysql.com/doc/
