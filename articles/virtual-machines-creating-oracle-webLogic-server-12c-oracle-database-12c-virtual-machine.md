<properties title="Creating an Oracle WebLogic Server 12c and Oracle Database 12c virtual machine in Azure" pageTitle="在 Azure 中创建 Oracle WebLogic Server 12c 和 Oracle Database 12c 虚拟机" description="逐步演示在 Windows Azure 中创建运行在 Windows Server 2012 上的 Oracle WebLogic Server 12c 和 Oracle Database 12c 映像的示例。" services="virtual-machines" authors="bbenz" documentationCenter=""/>
<tags ms.service="virtual-machines" ms.date="06/22/2015" wacn.date="08/29/2015"/>

#在 Azure 中创建 Oracle WebLogic Server 12c 和 Oracle Database 12c 虚拟机

本文演示如何在 Azure 中，基于 Windows Server 2012 上运行的由 Microsoft 提供的 Oracle WebLogic Server 12c 和 Oracle Database 12c 映像创建虚拟机。

##在 Azure 中创建 Oracle WebLogic Server 12c 和 Oracle Database 12c 虚拟机

1. 登录到 [Azure 门户](https://manage.windowsazure.cn)。

2.	单击**“应用商店”**，单击**“计算”**，然后在搜索框中键入 **Oracle**。

3.	选择 **Windows Server 2012 上的 Oracle Database 12c 和 WebLogic Server 12c Standard Edition** 或 **Windows Server 2012 上的 Oracle Database 12c 和 WebLogic Server 12c Enterprise Edition** 映像。检查有关此映像的信息（如最小建议大小），然后单击**“下一步”**。

4.	指定虚拟机的**“主机名”**。

5.	指定虚拟机的**“用户名”**。请注意，此用户名用于远程登录到虚拟机；这不是 Oracle 数据库用户名。

6.	指定并确认虚拟机的密码，或提供安全外壳 (SSH) 公钥。

7.	选择“定价层”。请注意，默认情况下会显示建议的定价层。若要查看所有配置选项，请单击右上角的**“全部查看”**。

8. 根据需要设置可选配置（请参阅[关于 Azure 虚拟机配置设置](https://msdn.microsoft.com/zh-cn/library/azure/dn763935.aspx)）。请注意以下事项：

	a.将**“存储帐户”**保持不变，以使用虚拟机名称创建新的存储帐户。

	b.将**“可用性集”**保留为**“未配置”**。

	c.此时请不要添加任何终结点。

9.	选择或创建资源组。有关详细信息，请参阅<!--[-->使用 Azure 预览门户管理 Azure 资源<!--](/documentation/articles/resource-group-portal)-->。

10. 选择一个**订阅**。

11. 选择**位置**。


##创建在此虚拟机中托管的数据库

按照以**在 Azure 中使用 Oracle Database 12c 虚拟机创建数据库**部分开头的[在 Azure 中创建 Oracle Database 12c 虚拟机](/documentation/articles/virtual-machines-creating-oracle-database-virtual-machine)中的说明进行操作。

##配置在此虚拟机中托管的 Oracle WebLogic Server 12c
按照以**在 Azure 中配置 Oracle WebLogic Server 12c 虚拟机**部分开头的[在 Azure 中创建 Oracle WebLogic Server 12c 虚拟机](/documentation/articles/virtual-machines-creating-oracle-webLogic-server-12c-virtual-machine)中的说明进行操作。如果要设置 WebLogic Server 群集，另请参阅[在 Azure 中创建 Oracle WebLogic Server 12c 群集](/documentation/articles/virtual-machines-creating-oracle-webLogic-server-12c-cluster)。

##其他资源
<!--[-->有关 Oracle 虚拟机映像的其他注意事项<!--](/documentation/articles/miscellaneous-considerations-for-oracle-virtual-machine-images-new-article)-->

[Oracle 虚拟机映像列表](/documentation/articles/virtual-machines-oracle-list-oracle-virtual-machine-images)

[从 Java 应用程序连接到 Oracle 数据库](http://docs.oracle.com/cd/E11882_01/appdev.112/e12137/getconn.htm#TDPJD136)

[在 Windows Azure 上使用 Linux 的 Oracle WebLogic Server 12c](http://www.oracle.com/technetwork/middleware/weblogic/learnmore/oracle-weblogic-on-azure-wp-2020930.pdf)

[Oracle Database 2 Day DBA 12c 发行版 1](http://docs.oracle.com/cd/E16655_01/server.121/e17643/toc.htm)

<!---HONumber=67-->