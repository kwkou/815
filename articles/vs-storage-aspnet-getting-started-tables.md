<properties 
	pageTitle="开始使用 Azure 表存储和 Visual Studio 连接服务" 
	description="如何开始在 Visual Studio 的 ASP.NET 项目中使用 Azure 表存储" 
	services="storage" 
	documentationCenter="" 
	authors="patshea123" 
	manager="douge" 
	editor="tglee"/>

<tags ms.service="storage"

	ms.date="08/04/2015" 
	wacn.date="09/16/2015"/>

# 开始使用 Azure 表存储和 Visual Studio 连接服务

> [AZURE.SELECTOR]
> - [Getting Started](/documentation/articles/vs-storage-aspnet-getting-started-tables)
> - [What Happened](/documentation/articles/vs-storage-aspnet-what-happened)

> [AZURE.SELECTOR]
> - [Blobs](/documentation/articles/vs-storage-aspnet-getting-started-blobs)
> - [Queues](/documentation/articles/vs-storage-aspnet-getting-started-queues)
> - [Tables](/documentation/articles/vs-storage-aspnet-getting-started-tables)

## 概述
本文介绍通过使用 Visual Studio 中的“添加连接服务”对话框在 ASP.NET 项目中创建或引用 Azure 存储帐户之后，如何开始在 Visual Studio 中使用 Azure 表存储。本文向你展示如何使用 Azure 表执行常见任务，包括创建和删除表以及使用表实体。示例是用 C# 代码编写的，并使用了 [Azure .NET 存储客户端库](https://msdn.microsoft.com/zh-cn/library/azure/dn261237.aspx)。有关使用 Azure 表存储的更多常规信息，请参阅[如何通过 .NET 使用表存储](/documentation/articles/storage-dotnet-how-to-use-tables)。


Azure 表存储服务使用户可以存储大量结构化数据。该服务是一个 NoSQL 数据存储，接受来自 Azure 云内部和外部的通过验证的呼叫。Azure 表最适合存储结构化非关系型数据。




##使用代码访问表 

1. 请确保 C# 文件顶部的命名空间声明包括这些 `using` 语句。

	using Microsoft.Azure; using Microsoft.WindowsAzure.Storage; using Microsoft.WindowsAzure.Storage.Auth; using Microsoft.WindowsAzure.Storage.Table;

2. 获取表示存储帐户信息的 `CloudStorageAccount` 对象。使用下面的代码获取存储连接字符串和 Azure 服务配置中的存储帐户信息。

		 CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
		   CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));

    **注意：**在下列示例中，在代码的前面使用上述全部代码。

3. 获取 `CloudTableClient` 对象，以引用存储帐户中的表对象。

	    // Create the table client.
    	CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

4. 获取 `CloudTable` 引用对象，以引用特定的表和实体。
	
    	// Get a reference to a table named "peopleTable"
	    CloudTable table = tableClient.GetTableReference("peopleTable");

##使用代码创建表

若要创建 Azure 表，只需在前面的代码中添加对 `CreateIfNotExistsAsync()` 的调用。

	// Create the CloudTable if it does not exist
	await table.CreateIfNotExistsAsync();


##插入一批实体

您可以通过单个写入操作将多个实体插入表中。以下代码示例将创建两个实体对象（“Jeff Smith”和“Ben Smith”），使用 Insert 方法将它们添加到 `TableBatchOperation` 对象，然后通过调用 `CloudTable.ExecuteBatchAsync` 启动操作。

	// Get a reference to a CloudTable object named 'peopleTable' as described in "Access a table in code"
	
	// Create the batch operation.
	TableBatchOperation batchOperation = new TableBatchOperation();
	
	// Create a customer entity and add it to the table.
	CustomerEntity customer1 = new CustomerEntity("Smith", "Jeff");
	customer1.Email = "Jeff@contoso.com";
	customer1.PhoneNumber = "425-555-0104";
	
	// Create another customer entity and add it to the table.
	CustomerEntity customer2 = new CustomerEntity("Smith", "Ben");
	customer2.Email = "Ben@contoso.com";
	customer2.PhoneNumber = "425-555-0102";
	
	// Add both customer entities to the batch insert operation.
	batchOperation.Insert(customer1);
	batchOperation.Insert(customer2);
	
	// Execute the batch operation.
	await peopleTable.ExecuteBatchAsync(batchOperation);

##获取分区中的所有实体
若要查询表以获取分区中的所有实体，请使用 `TableQuery` 对象。以下代码示例指定了一个筛选器，以筛选分区键为“Smith”的实体。此示例会将查询结果中每个实体的字段输出到控制台。

	// Get a reference to a CloudTable object named 'peopleTable' as described in "Access a table in code"

	// Construct the query operation for all customer entities where PartitionKey="Smith".
    TableQuery<CustomerEntity> query = new TableQuery<CustomerEntity>().Where(TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"));

    // Print the fields for each customer.
    TableContinuationToken token = null;
    do
    {
    	TableQuerySegment<CustomerEntity>
        resultSegment = await peopleTable.ExecuteQuerySegmentedAsync(query, token);
        token = resultSegment.ContinuationToken;

        foreach (CustomerEntity entity in resultSegment.Results)
        {
        Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
        entity.Email, entity.PhoneNumber);
        }
        } while (token != null);

        return View();


##获取单个实体
您可以编写查询以获取单个特定实体。以下代码使用 `TableOperation` 对象来指定名为“Ben Smith”的客户。此方法仅返回一个实体，而不是一个集合，并且 `TableResult.Result` 中的返回值是一个 `CustomerEntity` 对象。在查询中同时指定分区键和行键是从表服务中检索单个实体的最快方法。

        // Get a reference to a CloudTable object named 'peopleTable' as described in "Access a table in code"

        // Create a retrieve operation that takes a customer entity.
        TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");
	
	// Execute the retrieve operation.
	TableResult retrievedResult = await peopleTable.ExecuteAsync(retrieveOperation);
	
	// Print the phone number of the result.
	if (retrievedResult.Result != null)
	   Console.WriteLine(((CustomerEntity)retrievedResult.Result).PhoneNumber);
	else
	   Console.WriteLine("The phone number could not be retrieved.");

##删除实体
您可以在找到实体后将其删除。以下代码将查找名为“Ben Smith”的客户实体，如果找到，会将其删除。

	// Get a reference to a CloudTable object named 'peopleTable' as described in "Access a table in code"
	
	// Create a retrieve operation that expects a customer entity.
	TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");
	
	// Execute the operation.
	TableResult retrievedResult = peopleTable.Execute(retrieveOperation);
	
	// Assign the result to a CustomerEntity object.
	CustomerEntity deleteEntity = (CustomerEntity)retrievedResult.Result;
	
	// Create the Delete TableOperation and then execute it.
	if (deleteEntity != null)
	{
	   TableOperation deleteOperation = TableOperation.Delete(deleteEntity);
	
	   // Execute the operation.
	   await peopleTable.ExecuteAsync(deleteOperation);
	
	   Console.WriteLine("Entity deleted.");
	}
	
	else
	   Console.WriteLine("Couldn't delete the entity.");

## 后续步骤

[AZURE.INCLUDE [vs-storage-dotnet-tables-next-steps](../includes/vs-storage-dotnet-tables-next-steps.md)]



  

<!---HONumber=69-->