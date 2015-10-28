<properties 
	pageTitle="Azure 存储入门" 
	description="如何开始在 Visual Studio 中的 Azure WebJobs 5 项目中使用 Azure 表存储" 
	services="storage" 
	documentationCenter="" 
	authors="patshea123" 
	manager="douge" 
	editor="tglee"/>

<tags 
	ms.service="storage"
	ms.date="07/13/2015" 
	wacn.date="08/29/2015"/>

# Azure 存储入门（Azure WebJob 项目）

> [AZURE.SELECTOR]
> - [Getting Started](/documentation/articles/vs-storage-webjobs-getting-started-tables)
> - [What Happened](/documentation/articles/vs-storage-webjobs-what-happened)
> - [Blobs](/documentation/articles/vs-storage-webjobs-getting-started-blobs)
> - [Queues](/documentation/articles/vs-storage-webjobs-getting-started-queues)
> - [Tables](/documentation/articles/vs-storage-webjobs-getting-started-tables)



## 概述

Azure 表存储服务使用户可以存储大量结构化数据。该服务是一个 NoSQL 数据存储，接受来自 Azure 云内部和外部的通过验证的呼叫。Azure 表最适合存储结构化非关系型数据。有关详细信息，请参阅[如何通过.NET 使用表存储](/documentation/articles/storage-dotnet-how-to-use-tables#create-a-table "如何通过 .NET 使用表存储")。

本文章提供了 C# 代码示例，用于演示如何在 Azure 表存储服务中使用 Azure WebJobs SDK 版本 1.x。这些代码示例使用 [WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk) 版本 1.x。

		
一些代码段显示了[手动调用](/documentation/articles/vs-storage-webjobs-getting-started-blobs#manual)（即：不是使用触发器属性之一调用）的函数中使用的 `Table` 属性。

##如何向表中添加实体

若要将实体添加到表中，请使用具有 `ICollector<T>` 或 `IAsyncCollector<T>` 参数的 `Table` 属性，其中 `T` 指定想要添加的实体的架构。属性构造函数使用指定表名称的字符串参数。

下面的代码示例将 `Person` 实体添加到名 *Ingress* 的表中。

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

通常与 `ICollector` 一起使用的类型派生自 `TableEntity` 或实现 `ITableEntity`，但它并不是必需的。以下 `Person` 类之一适用于前面 `Ingress` 方法中所示的代码。

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

如果你想要直接使用 Azure 存储 API，则可以向方法签名添加 `CloudStorageAccount` 参数。

##实时监视

因为数据入口函数通常处理大量数据，WebJobs SDK 仪表板提供了实时监视的数据。“调用日志”部分告诉你函数是否仍在运行。

![Ingress 函数正在运行](./media/vs-storage-webjobs-getting-started-tables/ingressrunning.png)

“调用详细信息”页在运行时报告函数的进度（写入的实体数），并且为你提供中止的机会。

![Ingress 函数正在运行](./media/vs-storage-webjobs-getting-started-tables/ingressprogress.png)

函数完成时，“调用详细信息”页回报告写入的行数。

![Ingress 函数已完成](./media/vs-storage-webjobs-getting-started-tables/ingresssuccess.png)

##如何从表中读取多个实体

若要读取表，请使用具有 `IQueryable<T>` 参数的 `Table` 属性，其中类型 `T` 派生自 `TableEntity` 或实现`ITableEntity`。

下面的代码示例读取并记录 `Ingress` 表中的所有行：
 
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

###如何从表中读取单个实体

有一个 `Table` 属性构造函数具有两个附加参数，当你想要绑定到单个表实体时，该参数可以用于指定分区键和行键的。

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


在此示例中的 `Person` 类不一定实现 `ITableEntity`。

##如何直接使用 .NET 存储 API 处理表

你还可以使用包含 `CloudTable` 对象的 `Table` 属性更灵活地处理表。

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

有关如何使用 `CloudTable` 对象的详细信息，请参阅[如何通过 .NET 使用表存储](/documentation/articles/storage-dotnet-how-to-use-tables)。

##队列操作指南文章涵盖的相关主题

有关如何处理队列消息触发的表处理，或者不特定于表处理的 WebJobs SDK 方案的信息，请参阅[如何通过 WebJobs SDK 使用 Azure 队列存储](/documentation/articles/vs-storage-webjobs-getting-started-queues)。



##后续步骤

本文章提供了代码示例，演示如何处理用于操作 Azure 表的常见方案。有关如何使用 Azure WebJobs 和 WebJobs SDK 的详细信息，请参阅 [Azure WebJobs 推荐资源](http://www.windowsazure.cn/documentation/articles/websites-webjobs-resources/)。
 

<!---HONumber=67-->