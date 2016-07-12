<properties
	pageTitle="Oracle WebLogic Server 和 Database VM | Azure"
	description="使用 Resource Manager 部署模型创建在 Windows Server 2012 上运行的 Oracle WebLogic Server 12c 和 Oracle Database 12c Azure 映像。"
	services="virtual-machines-windows"
	authors="bbenz"
	documentationCenter=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="06/22/2015"
	wacn.date="03/21/2016"/>

#在 Azure 中创建 Oracle WebLogic Server 12c 和 Oracle Database 12c 虚拟机

本文演示如何在 Azure 中，基于 Windows Server 2012 上运行的由 Microsoft 提供的 Oracle WebLogic Server 12c 和 Oracle Database 12c 映像创建虚拟机。

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代经典部署模型。

##在 Azure 中创建 Oracle WebLogic Server 12c 和 Oracle Database 12c 虚拟机

1. 登录到 [Azure 经典管理门户](https://manage.windowsazure.cn/)。

2. 单击“新建”>“计算”>“从库中”。

3.	选择“Windows Server 2012 R2 Datacenter (zh-cn)”或“Windows Server 2012 R2 Datacenter (zh-cn)”映像。检查有关此映像的信息（如最小建议大小），然后单击**“下一步”**。

4.	指定 VM 的“虚拟机名称”。

7.	选择“层”和“大小”。如果你需要 DS 系列 VM，请通过 Azure PowerShell 创建一个。有关详细信息，请参阅[使用 Powershell 和经典部署模型创建 Windows 虚拟机](/documentation/articles/virtual-machines-windows-classic-create-powershell/)

5.	指定 VM 的“新用户名”。请注意，此用户用于远程登录 VM；这不是 Oracle 数据库用户名。

6.	指定并确认 VM 的密码，然后单击“下一步”。

8.	根据需要设置可选配置，但请注意以下事项：
	1. 创建新的云服务或选择现有的云服务。
	2. 选择一个**位置**
	1. 将“存储帐户”保持不变，以使用 VM 名称创建新的存储帐户。
	2. 将“可用性集”保留为“无”。
	3. 此时请不要添加任何**终结点**。单击“下一步”。
	

10. 选择是否安装“配置扩展”和“安全扩展”，然后单击“完成”。


##创建在此虚拟机中托管的数据库

按照以**在 Azure 中使用 Oracle Database 12c 虚拟机创建数据库**部分开头的[在 Azure 中创建 Oracle Database 12c 虚拟机](/documentation/articles/virtual-machines-windows-classic-create-oracle-database/)中的说明进行操作。

##配置在此虚拟机中托管的 Oracle WebLogic Server 12c
按照以**在 Azure 中配置 Oracle WebLogic Server 12c 虚拟机**部分开头的[在 Azure 中创建 Oracle WebLogic Server 12c 虚拟机](/documentation/articles/virtual-machines-windows-create-oracle-weblogic-server-12c/)中的说明进行操作。

##其他资源

[从 Java 应用程序连接到 Oracle 数据库](http://docs.oracle.com/cd/E11882_01/appdev.112/e12137/getconn.htm#TDPJD136)

[在 Azure 上使用 Linux 的 Oracle WebLogic Server 12c](http://www.oracle.com/technetwork/middleware/weblogic/learnmore/oracle-weblogic-on-azure-wp-2020930.pdf)

[Oracle Database 2 Day DBA 12c 发行版 1](http://docs.oracle.com/cd/E16655_01/server.121/e17643/toc.htm)

<!---HONumber=Mooncake_0314_2016-->