<properties 
	pageTitle="如何将用户添加到弹性数据库池" 
	description="必须将具有权限的用户添加到池中的每个数据库" 
	metaKeywords="azure sql database elastic databases credentials" 
	services="sql-database" documentationCenter=""  
	manager="jeffreyg" 
	authors="ddove"/>

<tags 
	ms.service="sql-database"
	ms.date="11/03/2015"
	wacn.date="12/22/2015" />

# 如何将用户添加到弹性数据库池

**弹性数据库作业**功能（预览版）可让你对 Azure SQL 数据库中包含自定义数据库集合、**弹性数据库池**或**弹性数据库池分片集**的一组数据库运行 Transact-SQL 脚本。若要运行脚本，必须将具有相应权限的用户添加到要执行作业的每个数据库中。有关详细信息，请参阅[在 Azure SQL 数据库中管理数据库和登录名](https://msdn.microsoft.com/zh-cn/library/azure/ee336235.aspx?f=255&MSPPError=-2147217396)或[将用户添加到 SQL Azure 数据库](http://azure.microsoft.com/blog/2010/06/21/adding-users-to-your-sql-azure-database/)

## 先决条件
* 安装[弹性作业组件](/documentation/articles/sql-database-elastic-jobs-service-installation/)。 

## 如何将用户添加到数据库

1.	首次，请连接到弹性数据库池中数据库所在的 Azure SQL 数据库服务器的**主机**，然后使用安装**弹性数据库作业**时提供的相同凭据创建新的登录名。

		CREATE LOGIN login1 WITH password='<ProvidePassword>';

2. 登录到池中的每个数据库，并使用相同的名称和密码创建一个用户。用户必须有足够的权限来执行该作业。必须在每个数据库上运行此代码。

		CREATE USER admin1 FROM LOGIN login1;
		
3. 该用户还必须有足够的权限执行针对作业指定的脚本。使用 [ALTER ROLE](https://msdn.microsoft.com/zh-cn/library/ms189775.aspx) 为用户提供所需的最低权限，以便能够成功执行脚本。

## 后续步骤

若要使用 PowerShell 创建作业，请参阅[使用 PowerShell 创建和管理 SQL 数据库弹性数据库作业（预览版）](/documentation/articles/sql-database-elastic-jobs-powershell/)。

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-jobs-overview/elastic-jobs.png
<!--anchors-->

<!---HONumber=Mooncake_1207_2015-->