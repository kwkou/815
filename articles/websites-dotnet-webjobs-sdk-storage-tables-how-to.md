<properties 
	pageTitle="如何通过 WebJobs SDK 使用 Azure 表存储" 
	description="如何通过 WebJobs SDK 使用 Azure 表存储创建表，将实体添加到表，并读取现有表。" 
	services="app-service\web, storage" 
	documentationCenter=".net" 
	authors="tdykstra" 
	manager="wpickett" 
	editor="jimbe"/>

<tags
	ms.service="app-service-web"
	ms.date="02/29/2016"
	wacn.date="04/26/2016"/>

# 如何通过 WebJobs SDK 使用 Azure 表存储

## 概述

本指南提供了 C# 代码示例，用于演示如何使用 [WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk/) 版本 1.x 读取和写入 Azure 存储表。

本指南假设你了解[如何使用指向存储帐户的连接字符串在 Visual Studio 中创建 WebJob 项目](/documentation/articles/websites-dotnet-webjobs-sdk-get-started/)或创建[多个存储帐户](https://github.com/Azure/azure-webjobs-sdk/blob/master/test/Microsoft.Azure.WebJobs.Host.EndToEndTests/MultipleStorageAccountsEndToEndTests.cs)。
		
一些代码段显示了[手动调用](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/#manual)（即：不是使用触发器属性之一调用）的函数中使用的 `Table` 属性。

## <a id="ingress"></a>如何向表中添加实体

若要将实体添加到表中，请使用包含 `ICollector<T>` 或 `IAsyncCollector<T>` 参数的 `Table` 属性，其中 `T` 指定您想要添加的实体的架构。属性构造函数使用指定表名称的字符串参数。

下面的代码示例将 `Person` 实体添加到名为 *Ingress* 的表。

		[NoAutomaticTrigger]
		public static void IngressDemo(
		    [Table("Ingress")] ICollector<Person> tableBinding)
		{
		    for (int i = 0; i < 100000; i++)
		    {
		        tableBinding.Add(
		            new Person() { 
		                PartitionKey = "Test", 
		                RowKey = i.ToString(), 
		                Name = "Name" }
		            );
		    }
		}

通常您用于 `ICollector` 的类型派生自 `TableEntity` 或实现 `ITableEntity`，但它并不一定要执行这些操作。以下 `Person` 类之一适用于前面 `Ingress` 方法中所示的代码。

		public class Person : TableEntity
		{
		    public string Name { get; set; }
		}

		public class Person
		{
		    public string PartitionKey { get; set; }
		    public string RowKey { get; set; }
		    public string Name { get; set; }
		}

如果您想要直接使用 Azure 存储 API，可以将 `CloudStorageAccount` 参数添加到方法签名。

## <a id="monitor"></a>实时监视

因为数据入口函数通常处理大量数据，WebJobs SDK 仪表板提供了实时监视的数据。“调用日志”部分告诉你函数是否仍在运行。

![Ingress 函数正在运行](./media/websites-dotnet-webjobs-sdk-storage-tables-how-to/ingressrunning.png)

“调用详细信息”页在运行时报告函数的进度（写入的实体数），并且为你提供中止的机会。

![Ingress 函数正在运行](./media/websites-dotnet-webjobs-sdk-storage-tables-how-to/ingressprogress.png)

该函数完成时，**调用详细信息**页会报告写入的行数。

![Ingress 函数已完成](./media/websites-dotnet-webjobs-sdk-storage-tables-how-to/ingresssuccess.png)

## <a id="multiple"></a>如何从表中读取多个实体

若要读取表，请将 `Table` 属性和 `IQueryable<T>` 参数一起使用，其中类型 `T` 派生自 `TableEntity` 或 实现 `ITableEntity`。

下面的代码示例读取并记录 `Ingress` 表中所有行：
 
		public static void ReadTable(
		    [Table("Ingress")] IQueryable<Person> tableBinding,
		    TextWriter logger)
		{
		    var query = from p in tableBinding select p;
		    foreach (Person person in query)
		    {
		        logger.WriteLine("PK:{0}, RK:{1}, Name:{2}", 
		            person.PartitionKey, person.RowKey, person.Name);
		    }
		}

### <a id="readone"></a>如何从表中读取单个实体

通过包含两个其他参数的 `Table` 属性构造函数，您可以在想要绑定到单个表实体时，指定分区键和行键。

下面的代码示例基于队列消息中接收的分区键和行键值读取 `Person` 实体的表行：

		public static void ReadTableEntity(
		    [QueueTrigger("inputqueue")] Person personInQueue,
		    [Table("persontable","{PartitionKey}", "{RowKey}")] Person personInTable,
		    TextWriter logger)
		{
		    if (personInTable == null)
		    {
		        logger.WriteLine("Person not found: PK:{0}, RK:{1}",
		                personInQueue.PartitionKey, personInQueue.RowKey);
		    }
		    else
		    {
		        logger.WriteLine("Person found: PK:{0}, RK:{1}, Name:{2}",
		                personInTable.PartitionKey, personInTable.RowKey, personInTable.Name);
		    }
		}


本示例中的 `Person` 类不必实现 `ITableEntity`。

## <a id="storageapi"></a>如何直接使用.NET 存储 API 处理表

您还可以将 `Table` 属性和 `CloudTable` 对象一起使用，以便能够更灵活地处理表。

下面的代码示例使用 `CloudTable` 对象将单个实体添加到 *Ingress* 表中。
 
		public static void UseStorageAPI(
		    [Table("Ingress")] CloudTable tableBinding,
		    TextWriter logger)
		{
		    var person = new Person()
		        {
		            PartitionKey = "Test",
		            RowKey = "100",
		            Name = "Name"
		        };
		    TableOperation insertOperation = TableOperation.Insert(person);
		    tableBinding.Execute(insertOperation);
		}

有关如何使用 `CloudTable` 对象的详细信息，请参阅[如何通过 .NET 使用表存储](/documentation/articles/storage-dotnet-how-to-use-tables/)。

## <a id="queues"></a>队列操作说明文章中介绍了相关主题

有关如何处理队列消息触发的表处理，或者不特定于表处理的 WebJobs SDK 方案的信息，请参阅[如何通过 WebJobs SDK 使用 Azure 队列存储](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/)。

该文章涵盖的主题包括：

* 异步函数
* 多个实例
* 正常关闭
* 在函数正文中使用 WebJobs SDK 属性
* 在代码中设置 SDK 连接字符串。
* 在代码中设置 WebJobs SDK 构造函数参数的值
* 手动触发函数
* 写入日志

## <a id="nextsteps"></a>后续步骤

本指南提供的代码示例演示了如何处理常见方案以操作 Azure 表。有关如何使用 Azure WebJobs 和 WebJobs SDK 的详细信息，请参阅 [Azure WebJobs 推荐资源](/documentation/articles/websites-webjobs-resources/)。
 

<!---HONumber=Mooncake_0215_2016-->