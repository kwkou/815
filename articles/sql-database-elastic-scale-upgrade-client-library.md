<properties	
	pageTitle="升级到最新的弹性数据库客户端库 | Azure" 
	description="使用 Nuget 升级应用程序和库" 
	services="sql-database" 
	documentationCenter="" 
	manager="jhubbard" 
	authors="ddove"/>

<tags 
	ms.service="sql-database" 
	ms.date="04/04/2016" 
	wacn.date="06/14/2016" />

# 升级到最新的弹性数据库客户端库

可通过 [NuGet](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client) 和 Visual Studio 中的 NuGet 包管理器界面获取[弹性数据库客户端库](/documentation/articles/sql-database-elastic-database-client-library/)的新版本。升级包含客户端库的 bug 修复和新功能支持。

使用新库重新生成你的应用程序，以及更改 Azure SQL 数据库中存储的现有分片映射管理器元数据以支持新功能。

按顺序执行这些步骤可确保在更新元数据对象时旧版本的客户端库不再存在于你的环境中，这意味着在升级后不会创建旧版本的元数据对象。

## 升级步骤

**1.升级你的应用程序。** 在 Visual Studio 中，下载最新的客户端库版本并在使用该库的所有开发项目中引用该版本；然后重新生成并部署。

 * 在 Visual Studio 解决方案中，选择“工具”“NuGet 包管理器”“管理解决方案的 NuGet 包”。 
 * (Visual Studio 2013) 在左面板中，选择“更新”，然后选择窗口中显示的包“Azure SQL 数据库弹性扩展客户端库”上的“更新”按钮。
 * (Visual Studio 2015) 将“筛选器”框设置为“可用升级”。选择要更新的包，然后单击“更新”按钮。
	
 
 * 生成并部署。

**2.升级你的脚本。** 如果你使用 **PowerShell** 脚本来管理分片，请[下载新的库版本](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client)并将其复制到你从其执行脚本的目录中。

**3.升级拆分/合并服务。** 如果你使用弹性数据库拆分/合并工具来重新组织分片数据，请[下载并部署最新版本的工具](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Service.SplitMerge)。可在[此处](/documentation/articles/sql-database-elastic-scale-overview-split-and-merge/)找到该服务的详细升级步骤。

**4.升级分片映射管理器数据库**。升级 Azure SQL 数据库中支持分片映射的元数据。有两种方法可以完成此操作：使用 PowerShell 或 C#。这两个选项在下面说明。

***选项 1：使用 PowerShell 升级元数据***

1. 在[此处](http://nuget.org/nuget.exe)下载 NuGet 的最新命令行实用工具并将其保存到一个文件夹。 

2. 打开命令提示符，导航到同一文件夹，并发出命令：
`nuget install Microsoft.Azure.SqlDatabase.ElasticScale.Client`

3. 导航到包含你刚下载的新客户端 DLL 版本的子文件夹，例如：
`cd .\Microsoft.Azure.SqlDatabase.ElasticScale.Client.1.0.0\lib\net45`

4. 从[脚本中心](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-Database-Elastic-6442e6a9)下载弹性数据库客户端升级 scriptlet，并将其保存到包含 DLL 的同一文件夹中。

5. 在命令提示符下从该文件夹运行“PowerShell .\\upgrade.ps1”，并按照提示进行操作。
 
***选项 2：使用 C# 升级元数据***

或者，创建一个 Visual Studio 应用程序，它打开 ShardMapManager，循环访问所有分片并通过调用 [UpgradeLocalStore](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.upgradelocalstore.aspx) 和 [UpgradeGlobalStore](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.upgradeglobalstore.aspx) 方法执行元数据升级，如此示例所示：

	ShardMapManager smm =
	   ShardMapManagerFactory.GetSqlShardMapManager
	   (connStr, ShardMapManagerLoadPolicy.Lazy); 
	smm.UpgradeGlobalStore(); 
	
	foreach (ShardLocation loc in
	 smm.GetDistinctShardLocations()) 
	{   
	   smm.UpgradeLocalStore(loc); 
	} 

这些元数据升级技术可以应用多次而无危害。例如，如果在你已更新后，旧的客户端版本无意中创建了一个分片，你可以对所有分片再次运行升级，以确保最新的元数据版本在整个基础结构中存在。

**注意：**到目前为止发布的客户端库的新版本将继续使用 Azure SQL 数据库上早期版本的分片映射管理器元数据，反之亦然。但是，为了利用最新客户端中的一些新功能，需要对元数据进行升级。请注意，元数据升级不会影响任何用户数据或特定于应用程序的数据，只会影响分片映射管理器创建和使用的对象。并且应用程序将继续按照前面所述的升级顺序操作。

## 弹性数据库客户端版本历史记录 

**版本 1.0 -- 2015 年 4 月**

* 正式版
* 添加了日期时间类型分片键的支持

**版本 0.8 – 2015 年 3 月**

* 使用新的 ShardMap.OpenConnectionForKeyAsync 方法为依赖于数据的路由添加异步支持。 
* 为 ShardMap 添加了公共 KeyType 属性 
* 为分片添加了支持数据库还原和灾难恢复方案的改进 

**版本 0.7 – 2014 年 10 月**

初始预览版


[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]


<!--Image references-->
[1]: ./media/sql-database-elastic-scale-upgrade-client-library/nuget-upgrade.png
 
<!---HONumber=Mooncake_0606_2016-->
