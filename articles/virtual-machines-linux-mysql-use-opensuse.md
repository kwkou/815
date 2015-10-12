<properties
	pageTitle="在 Azure 中运行 OpenSUSE Linux 的虚拟机上安装 MySQL"
	description="了解如何在 Azure 中的虚拟机上安装 MySQL。"
	services="virtual-machines"
	documentationCenter=""
	authors="KBDAzure"
	manager="timlt"
	editor=""
	tags="mysql"/>

<tags
	ms.service="virtual-machines"
	ms.date="05/22/2015"
	wacn.date="09/18/2015"/>

# 在 Azure 中运行 OpenSUSE Linux 的虚拟机上安装 MySQL

[MySQL][MySQL] 是一种受欢迎的 SQL 开源数据库。本教程演示：

- 如何使用 [Azure 管理门户][AzurePortal]从 Azure 所提供的映像中创建 OpenSUSE Linux 虚拟机。
- 如何使用 SSH 或 PuTTY 连接到该虚拟机。
- 如何在该虚拟机上安装 MySQL。

[AZURE.INCLUDE [antares-iaas-signup-iaas](../includes/antares-iaas-signup-iaas.md)]

## 创建运行 OpenSUSE Linux 的虚拟机

[AZURE.INCLUDE [create-and-configure-opensuse-vm-in-portal](../includes/create-and-configure-opensuse-vm-in-portal.md)]

##在虚拟机上安装和运行 MySQL

[AZURE.INCLUDE [install-and-run-mysql-on-opensuse-vm](../includes/install-and-run-mysql-on-opensuse-vm.md)]

##摘要
在本教程中，你已了解如何创建运行 OpenSUSE Linux 的虚拟机以及使用 SSH 或 PuTTY 远程连接到该虚拟机。你还了解了如何在 Linux 虚拟机上安装和配置 MySQL。有关 MySQL 的更多信息，请参见 [MySQL 文档][MySQLDocs]。

[MySQLDocs]: http://dev.mysql.com/doc/
[MySQL]: http://www.mysql.com
[AzurePortal]: http://manage.windowsazure.cn

<!---HONumber=70-->