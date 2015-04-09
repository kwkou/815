<properties linkid="virtual-machines-linux-mysql-use-opensuse" urlDisplayName="Install MongoDB" pageTitle="在 Azure 中运行 CentOS Linux 的虚拟机上安装 MongoDB" metaKeywords="Azure, MongoDB" description="了解如何在 Azure 中的虚拟机上安装 Mongo DB。" metaCanonical="" services="" documentationCenter="" title="Install MongoDB on a virtual machine running CentOS Linux in Azure" authors="" solutions="" manager="" editor="" />

# 在 Azure 中运行 OpenSUSE Linux 的虚拟机上安装 MySQL

[MySQL][MySQL] 是一个受欢迎的开源 SQL 数据库。本教程向您介绍：

- 如何从通过 Azure 提供的映像使用 [Azure 管理门户][AzurePortal]创建 OpenSUSE Linux 虚拟机。
- 如何使用 SSH 或 PuTTY 连接到该虚拟机。
- 如何在该虚拟机上安装 MySQL。

[WACOM.INCLUDE [antares-iaas-signup-iaas](../includes/antares-iaas-signup-iaas.md)]

## 创建运行 OpenSUSE Linux 的虚拟机

[WACOM.INCLUDE [create-and-configure-opensuse-vm-in-portal](../includes/create-and-configure-opensuse-vm-in-portal.md)]

##在虚拟机上安装和运行 MySQL

[WACOM.INCLUDE [install-and-run-mysql-on-opensuse-vm](../includes/install-and-run-mysql-on-opensuse-vm.md)]

##摘要
在本教程中，您已了解如何创建运行 OpenSUSE Linux 的虚拟机以及使用 SSH 或 PuTTY 远程连接到该虚拟机。您还了解了如何在 Linux 虚拟机上安装和配置 MySQL。有关 MySQL 的详细信息，请参阅 [MySQL 文档][MySQLDocs]。

[MySQLDocs]: http://dev.mysql.com/doc/
[MySQL]: http://www.mysql.com
[AzurePortal]: http://manage.windowsazure.cn
