<properties
   pageTitle="跨云数据库的分布式事务"
   description="Azure SQL 数据库的弹性数据库事务概述"
   services="sql-database"
   documentationCenter=""
   authors="torsteng"
   manager="jeffreyg"
   editor="sidneyh"/>

<tags
   ms.service="sql-database"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="sql-database"
   ms.date="05/27/2016"
   wacn.date="12/26/2016"
   ms.author="torsteng"/>

# 跨云数据库的分布式事务

Azure SQL 数据库 (SQL DB) 的弹性数据库事务可让你在 SQL DB 中运行跨多个数据库的事务。SQL DB 的弹性数据库事务适用于使用 ADO .NET 的 .NET 应用程序，并且与你熟悉的使用 [System.Transaction](https://msdn.microsoft.com/zh-cn/library/system.transactions.aspx) 类的编程体验相集成。若要获取该库，请参阅 [.NET Framework 4.6.1（Web 安装程序）](https://www.microsoft.com/zh-cn/download/details.aspx?id=49981)。

在本地，这种方案通常需要运行 Microsoft 分布式事务处理协调器 (MSDTC)。由于 MSDTC 不适用于 Azure 中的平台即服务应用程序，协调分布式事务的功能现在已直接集成到 SQL DB 中。应用程序可以连接到任何 SQL 数据库来启动分布式事务，其中一个数据库会以透明的方式协调分布式事务，如下图所示。

  ![使用弹性数据库事务在 Azure SQL 数据库中执行分布式事务][1]

## 常见方案

SQL DB 的弹性数据库事务可让应用程序对多个不同 SQL 数据库中存储的数据进行原子性更改。预览版着重于 C# 和 .NET 的客户端开发体验。已计划在将来添加使用 T-SQL 的服务器端体验。  
弹性数据库事务面向以下方案：

* Azure 中的多数据库应用程序：在此方案中，数据垂直分区到 SQL DB 中的多个数据库，使得不同种类的数据位于不同的数据库。某些操作需要更改两个以上的数据库中保存的数据。应用程序使用弹性数据库事务来协调数据库之间的更改并确保原子性。

* Azure 中的分区数据库应用程序：在此方案中，数据层使用[弹性数据库客户端库](/documentation/articles/sql-database-elastic-database-client-library/)或自我分片，将数据水平分区到 SQL DB 中的多个数据库。常见的用例之一是在分片的多租户应用程序中，当更改涉及到多个租户时，需要执行原子更改。例如，从一个租户转移到另一个租户，而两者位于不同的数据库。第二种方案是以细致分片来适应大租户的容量需求，这又通常表示某些原子操作需要扩展到用于同一租户的多个数据库。第三种方案是以原子更新来引用数据库之间复制的数据。现在，可以使用预览版跨多个数据库协调这几个方面原子性事务操作。弹性数据库事务使用两阶段提交，确保跨数据库的事务原子性。如果事务涉及的数据库少于 100 个，则适合并入单个事务内。这些限制不是强制施加的，但是，如果超出这些限制，弹性数据库事务的性能和成功率很有可能会下降。


## 安装和迁移

我们更新了 .NET 库 System.Data.dll 和 System.Transactions.dll，以支持在 SQL DB 中执行弹性数据库事务。DLL 确保必要时使用两阶段事务提交，以确保原子性。若要使用弹性数据库事务来开始开发应用程序，请安装 [.NET 4.6.1](https://www.microsoft.com/zh-cn/download/details.aspx?id=49981) 或更高版本。在旧版 .NET Framework 上运行时，事务无法升级为分布式事务，并会引发异常。

安装后，可以使用 System.Transactions 中的分布式事务 API 和 SQL DB 连接。如果现有的 MSDTC 应用程序使用了这些 API，只需在安装 4.6.1 Framework 之后，以 .NET 4.6 为目标重建现有的应用程序。如果项目以 .NET 4.6 为目标，它们将自动使用新 Framework 版本中更新的 DLL，而结合 SQL DB 连接的分布式事务 API 调用现在会成功。

请记住，弹性数据库事务不需要安装 MSDTC。弹性数据库事务直接由 SQL DB 管理。这可以大幅简化云方案，因为 MSDTC 的部署不需要使用分布式事务和 SQL DB。第 4 部分更详细说明了如何将弹性数据库事务和所需的 .NET Framework 连同云应用程序一起部署到 Azure。

## 开发体验

###	多数据库应用程序

以下示例代码使用你熟悉的 .NET System.Transactions 编程体验。TransactionScope 类在 .NET 中创建环境事务。（“环境事务”是位于当前线程中的事务）。 在 TransactionScope 内打开的所有连接都参与该事务。如果有不同的数据库参与，事务将自动提升为分布式事务。通过设置完成范围来指示提交，即可控制事务的结果。

	using (var scope = new TransactionScope())
	{
		using (var conn1 = new SqlConnection(connStrDb1))
		{
			conn1.Open();
			SqlCommand cmd1 = conn1.CreateCommand();
			cmd1.CommandText = string.Format("insert into T1 values(1)");
			cmd1.ExecuteNonQuery();
		}

		using (var conn2 = new SqlConnection(connStrDb2))
		{
			conn2.Open();
			var cmd2 = conn2.CreateCommand();
			cmd2.CommandText = string.Format("insert into T2 values(2)");
			cmd2.ExecuteNonQuery();
		}

		scope.Complete();
	}

### 分片数据库应用程序
 
SQL DB 的弹性数据库事务还支持协调分布式事务，这需要使用弹性数据库客户端库的 OpenConnectionForKey 方法，打开扩大的数据层的连接。假设你需要保证事务一致性，以使更改跨多个不同的分片键值。与托管不同分片键值的分片的连接由 OpenConnectionForKey 来中转。一般情况下，可以连接到不同的分片，以确保事务保证需要分布式事务。以下代码示例演示了此方法。假设使用一个名为 shardmap 的变量代表来自弹性数据库客户端库的分片映射：

	using (var scope = new TransactionScope())
	{
		using (var conn1 = shardmap.OpenConnectionForKey(tenantId1, credentialsStr))
		{
			conn1.Open();
			SqlCommand cmd1 = conn1.CreateCommand();
			cmd1.CommandText = string.Format("insert into T1 values(1)");
			cmd1.ExecuteNonQuery();
		}
		
		using (var conn2 = shardmap.OpenConnectionForKey(tenantId2, credentialsStr))
		{
			conn2.Open();
			var cmd2 = conn2.CreateCommand();
			cmd2.CommandText = string.Format("insert into T1 values(2)");
			cmd2.ExecuteNonQuery();
		}

		scope.Complete();
	}


## 适用于 Azure 云服务的 .NET 安装

Azure 为托管 .NET 应用程序提供了多个产品。不同产品的比较可见于[Azure Web 应用、云服务和虚拟机比较](/documentation/articles/choose-web-site-cloud-service-vm/)。如果产品的来宾 OS 版本低于弹性事务所需的 .NET 4.6.1，则需要将来宾 OS 升级到 4.6.1。

对于 Azure App Service，当前不支持升级到来宾 OS。对于 Azure 虚拟机，只需要登录到 VM 并运行最新的 .NET framework 安装程序即可。对于 Azure 云服务，需要将更高版本的 .NET 安装包括到部署的启动任务中。[在云服务角色上安装 .NET](/documentation/articles/cloud-services-dotnet-install-dotnet/) 中说明了概念和步骤。

请注意，与 .NET 4.6 的安装程序相比，.NET 4.6.1 的安装程序在 Azure 云服务上执行引导过程时，可能需要更多的临时存储空间。为了确保安装成功，需要在 ServiceDefinition.csdef 文件中启动任务的 LocalResources 部分和环境设置中，增加 Azure 云服务的临时存储，如以下示例所示：

	<LocalResources>
	...
		<LocalStorage name="TEMP" sizeInMB="5000" cleanOnRoleRecycle="false" />
		<LocalStorage name="TMP" sizeInMB="5000" cleanOnRoleRecycle="false" />
	</LocalResources>
	<Startup>
		<Task commandLine="install.cmd" executionContext="elevated" taskType="simple">
			<Environment>
		...
				<Variable name="TEMP">
					<RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='TEMP']/@path" />
				</Variable>
				<Variable name="TMP">
					<RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='TMP']/@path" />
				</Variable>
			</Environment>
		</Task>
	</Startup>
	
## 跨多个服务器的事务

Azure SQL 数据库中支持跨不同逻辑服务器的弹性数据库事务。事务跨越逻辑服务器边界时，参与的服务器将首先需要进入相互通信关系。一旦建立了通信关系，任意两个服务器中的任何数据库都可以与另一服务器中的数据库参与弹性事务。事务跨越两个以上的逻辑服务器时，需要准备好任意逻辑服务器对之间的通信关系。

使用以下 PowerShell cmdlet 来管理弹性数据库事务的跨服务器通信关系：

* **New-AzureRmSqlServerCommunicationLink**：使用此 cmdlet 在 Azure SQL DB 中的两个逻辑服务器之间创建新的通信关系。这种通信关系是对称的，这意味着两台服务器都可以使用另一台服务器启动事务。
* **Get AzureRmSqlServerCommunicationLink**：使用此 cmdlet 来检索现有通信关系及其属性。
* **Get AzureRmSqlServerCommunicationLink**：使用此 cmdlet 来删除现有通信关系。


## 监视事务状态

使用 SQL DB 中的动态管理视图 (DMV) 来监视正在进行的弹性数据库事务的状态和进度。与事务相关的所有 DMV 都与 SQL DB 中的分布式事务相关。可以在此处找到相应的 DMV 列表：[事务相关的动态管理视图和函数 (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms178621.aspx)。
 
这些 DMV 特别有用：

* **sys.dm\_tran\_active\_transactions**：列出当前正在使用的事务及其状态。UOW（工作单位）列可以标识属于同一分布式事务的不同子事务。同一分布式事务中的所有事务具有相同的 UOW 值。有关详细信息，请参阅 [DMV 文档](https://msdn.microsoft.com/zh-cn/library/ms174302.aspx)。
* **sys.dm\_tran\_database\_transactions**：提供有关事务的其他信息，例如事务在日志中的位置。有关详细信息，请参阅 [DMV 文档](https://msdn.microsoft.com/zh-cn/library/ms186957.aspx)。
* **sys.dm\_tran\_locks**：提供当前正在进行的事务持有的锁的相关信息。有关详细信息，请参阅 [DMV 文档](https://msdn.microsoft.com/zh-cn/library/ms190345.aspx)。

## 限制 

SQL DB 中的弹性数据库事务当前存在以下限制：

* 仅支持 SQL DB 中跨数据库的事务。其他 [X/Open XA](https://zh.wikipedia.org/wiki/X/Open_XA) 资源提供程序和除 SQL DB 以外的数据库无法参与弹性数据库事务。这意味着，弹性数据库事务无法扩展到本地 SQL Server 和 Azure SQL 数据库。对于本地的分布式事务，请继续使用 MSDTC。
* 仅支持来自 .NET 应用程序的客户端协调事务。目前已规划 T-SQL 的服务器端支持，例如 BEGIN DISTRIBUTED TRANSACTION，但尚未推出。
* 仅支持 Azure SQL DB V12 上的数据库。
* 不支持跨 WCF 服务的事务。例如，你有一个执行事务的 WCF 服务方法。在事务范围内封装调用将失败，并显示异常 [System.ServiceModel.ProtocolException](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.protocolexception)。


<!--Image references-->
[1]: ./media/sql-database-elastic-transactions-overview/distributed-transactions.png

<!---HONumber=Mooncake_Quality_Review_1215_2016-->