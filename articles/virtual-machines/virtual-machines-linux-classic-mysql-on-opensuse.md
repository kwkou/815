<properties
	pageTitle="在 Azure 中运行 OpenSUSE Linux 的 VM 上安装 MySQL"
	description="了解如何在 Azure 中的虚拟机上安装 MySQL。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="07/19/2016"
	wacn.date="09/30/2016"/>

# 在 Azure 中运行 OpenSUSE Linux 的虚拟机上安装 MySQL

[MySQL][MySQL] 是一种受欢迎的 SQL 开源数据库。本教程介绍如何创建运行 OpenSUSE Linux 的虚拟机，然后安装 MySQL。

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)]

<br>

## 创建运行 OpenSUSE Linux 的虚拟机

[AZURE.INCLUDE [create-and-configure-opensuse-vm-in-portal](../../includes/create-and-configure-opensuse-vm-in-portal.md)]

## 在虚拟机上安装和运行 MySQL

[AZURE.INCLUDE [install-and-run-mysql-on-opensuse-vm](../../includes/install-and-run-mysql-on-opensuse-vm.md)]

## 后续步骤
有关 MySQL 的详细信息，请参阅 [MySQL 文档][MySQLDocs]。

[MySQLDocs]: http://dev.mysql.com/doc/index-topic.html
[MySQL]: http://www.mysql.com

<!---HONumber=Mooncake_1207_2015-->