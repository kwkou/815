<properties
	pageTitle="Oracle WebLogic Server 和 Database VM | Azure"
	description="使用 Resource Manager 部署模型创建在 Windows Server 2012 上运行的 Oracle WebLogic Server 12c 和 Oracle Database 12c Azure 映像。"
	services="virtual-machines-windows"
	authors="rickstercdn"
	manager="timlt"
	documentationCenter=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="05/17/2016"
	wacn.date="07/11/2016"/>

#在 Azure 中创建 Oracle WebLogic Server 12c 和 Oracle Database 12c 虚拟机

[AZURE.INCLUDE [virtual-machines-common-oracle-support](../includes/virtual-machines-common-oracle-support.md)]

本文介绍如何在 Azure 中，在以前使用 Windows Server 2012 上安装的 Oracle 软件创建的映像上创建 Oracle WebLogic Server 12c 和 Oracle Database 12c 数据库。

##创建在此虚拟机中托管的数据库

按照以**在 Azure 中使用 Oracle Database 12c 虚拟机创建数据库**部分开头的[在 Azure 中创建 Oracle Database 12c 虚拟机](/documentation/articles/virtual-machines-windows-classic-create-oracle-database/)中的说明进行操作。

##配置在此虚拟机中托管的 Oracle WebLogic Server 12c
按照以**在 Azure 中配置 Oracle WebLogic Server 12c 虚拟机**部分开头的[在 Azure 中创建 Oracle WebLogic Server 12c 虚拟机](/documentation/articles/virtual-machines-windows-create-oracle-weblogic-server-12c/)中的说明进行操作。

##其他资源
[有关 Oracle 虚拟机映像的其他注意事项](/documentation/articles/virtual-machines-windows-classic-oracle-considerations/)

[从 Java 应用程序连接到 Oracle 数据库](http://docs.oracle.com/cd/E11882_01/appdev.112/e12137/getconn.htm#TDPJD136)

[Azure 上使用 Linux 的 Oracle WebLogic Server 12c](http://www.oracle.com/technetwork/middleware/weblogic/learnmore/oracle-weblogic-on-azure-wp-2020930.pdf)

[Oracle Database 2 Day DBA 12c 发行版 1](http://docs.oracle.com/cd/E16655_01/server.121/e17643/toc.htm)

<!---HONumber=Mooncake_0704_2016-->