<properties
    pageTitle="开始使用 Azure 表存储和 Visual Studio 连接服务 (ASP.NET) | Azure"
    description="在使用 Visual Studio 连接服务连接到存储帐户后，如何开始在 Visual Studio 的 ASP.NET 项目中使用 Azure 表存储"
    services="storage"
    documentationcenter=""
    author="TomArcher"
    manager="douge"
    editor="" />
<tags
    ms.assetid="af81a326-18f4-4449-bc0d-e96fba27c1f8"
    ms.service="storage"
    ms.workload="web"
    ms.tgt_pltfrm="vs-getting-started"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/02/2016"
    wacn.date="01/06/2017"
    ms.author="tarcher" />  


# 开始使用 Azure 表存储和 Visual Studio 连接服务 (ASP.NET)

## 概述

Azure 表存储使用户可以存储大量结构化数据。该服务是一个 NoSQL 数据存储，接受来自 Azure 云内部和外部的通过验证的呼叫。Azure 表最适合存储结构化非关系型数据。

本文介绍如何以编程方式管理 Azure 表存储实体，执行常见任务，例如创建和删除表，以及使用表实体。

> [AZURE.NOTE]
> 
> 本文的代码部分假定用户已使用连接服务连接到 Azure 存储帐户。连接服务的配置方法是：打开 Visual Studio 解决方案资源管理器，右键单击项目，然后从上下文菜单中选择“添加”->“连接服务”选项。在该处按照对话框的说明连接到所需的 Azure 存储帐户。

## 使用代码创建表

以下步骤演示了如何以编程方式创建表。在 ASP.NET MVC 应用中，该代码会置于控制器中。

1. 添加以下 *using* 指令：

         using Microsoft.Azure;
         using Microsoft.WindowsAzure.Storage;
         using Microsoft.WindowsAzure.Storage.Auth;
         using Microsoft.WindowsAzure.Storage.Table;

2. 获取表示存储帐户信息的 **CloudStorageAccount** 对象。使用下面的代码获取存储连接字符串和 Azure 服务配置中的存储帐户信息。（将 *<storage-account-name>* 更改为要访问的 Azure 存储帐户的名称。）

         CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
           CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));

3. 获取表示表服务客户端的 **CloudTableClient** 对象。

        CloudTableClient tableClient = storageAccount.CreateCloudTableClient();


4. 获取表示所需表名称引用的 **CloudTable** 对象。（将 *<table-name>* 更改为要创建的表的名称。）

		CloudTable table = tableClient.GetTableReference(<table-name>);

5. 如果表不存在，则调用 **CloudTable.CreateIfNotExists** 方法来创建表。
   
    	table.CreateIfNotExists();

## 向表中添加条目

以下步骤演示了如何以编程方式向表添加实体。在 ASP.NET MVC 应用中，该代码会置于控制器中。

1. 添加以下 *using* 指令：

         using Microsoft.Azure;
         using Microsoft.WindowsAzure.Storage;
         using Microsoft.WindowsAzure.Storage.Auth;
         using Microsoft.WindowsAzure.Storage.Table;

2. 获取表示存储帐户信息的 **CloudStorageAccount** 对象。使用下面的代码获取存储连接字符串和 Azure 服务配置中的存储帐户信息。（将 *<storage-account-name>* 更改为要访问的 Azure 存储帐户的名称。）

         CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
           CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));

3. 获取表示表服务客户端的 **CloudTableClient** 对象。

        CloudTableClient tableClient = storageAccount.CreateCloudTableClient();


4. 获取表示所需表名称引用的 **CloudTable** 对象。（将 *<table-name>* 更改为要向其添加实体的表的名称。）

		CloudTable table = tableClient.GetTableReference(<table-name>);

5. 若要向表添加实体，请定义一个派生自 **TableEntity** 的类。以下代码定义了将客户的名字和姓氏分别用作行键和分区键的 **CustomerEntity** 实体类。

	    public class CustomerEntity : TableEntity
	    {
	        public CustomerEntity(string lastName, string firstName)
	        {
	            this.PartitionKey = lastName;
	            this.RowKey = firstName;
	        }
	
	        public CustomerEntity() { }
	
	        public string Email { get; set; }
	    }

6. 实例化实体。

	    CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
	    customer1.Email = "Walter@contoso.com";

7. 创建插入客户实体的 **TableOperation** 对象。

	    TableOperation insertOperation = TableOperation.Insert(customer1);

8. 通过调用 **CloudTable.Execute** 方法执行插入操作。可以通过检查 **TableResult.HttpStatusCode** 属性验证操作的结果。状态代码为 2xx 指示客户端请求的操作已成功处理。例如，成功插入新的实体会生成 HTTP 状态代码 204，这意味着操作已成功处理，服务器没有返回任何内容。

    	TableResult result = table.Execute(insertOperation);

		// Inspect result.HttpStatusCode for success/failure.

## 向表添加一批实体

除了可以向表逐个添加实体，还可以成批添加实体。这可以减少代码与 Azure 表服务之间的重复操作次数。以下步骤演示了如何以编程方式使用单个操作向表添加多个实体。在 ASP.NET MVC 应用中，该代码会置于控制器中。

1. 添加以下 *using* 指令：

         using Microsoft.Azure;
         using Microsoft.WindowsAzure.Storage;
         using Microsoft.WindowsAzure.Storage.Auth;
         using Microsoft.WindowsAzure.Storage.Table;

2. 获取表示存储帐户信息的 **CloudStorageAccount** 对象。使用下面的代码获取存储连接字符串和 Azure 服务配置中的存储帐户信息。（将 *<storage-account-name>* 更改为要访问的 Azure 存储帐户的名称。）

         CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
           CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));

3. 获取表示表服务客户端的 **CloudTableClient** 对象。

        CloudTableClient tableClient = storageAccount.CreateCloudTableClient();


4. 获取表示所需表名称引用的 **CloudTable** 对象。（将 *<table-name>* 更改为要向其添加实体的表的名称。）

		CloudTable table = tableClient.GetTableReference(<table-name>);

5. 若要向表添加实体，请定义一个派生自 **TableEntity** 的类。以下代码定义了将客户的名字和姓氏分别用作行键和分区键的 **CustomerEntity** 实体类。

	    public class CustomerEntity : TableEntity
	    {
	        public CustomerEntity(string lastName, string firstName)
	        {
	            this.PartitionKey = lastName;
	            this.RowKey = firstName;
	        }
	
	        public CustomerEntity() { }
	
	        public string Email { get; set; }
	    }

6. 实例化实体。

	    CustomerEntity customer1 = new CustomerEntity("Smith", "Jeff");
	    customer1.Email = "Jeff@contoso.com";
	
	    CustomerEntity customer2 = new CustomerEntity("Smith", "Ben");
	    customer2.Email = "Ben@contoso.com";

7. 获取 **TableBatchOperation** 对象。

	    TableBatchOperation batchOperation = new TableBatchOperation();

8. 向批量插入操作添加实体。

	    batchOperation.Insert(customer1);
	    batchOperation.Insert(customer2);

9. 通过调用 **CloudTable.ExecuteBatch** 方法执行批量插入操作。**CloudTable.ExecuteBatch** 方法返回 **TableResult** 对象的列表。可以通过检查列表中每个 **TableResult** 对象的 **TableResult.HttpStatusCode** 属性验证批量插入操作的结果。状态代码为 2xx 指示客户端请求的操作已成功处理。例如，成功插入新的实体会生成 HTTP 状态代码 204，这意味着操作已成功处理，服务器没有返回任何内容。
    
		IList<TableResult> results = table.ExecuteBatch(batchOperation);

		// Inspect the HttpStatusCode property of each TableResult object
		// in the results list for success/failure.

## 获取单个实体

以下步骤演示了如何以编程方式获取表中的实体。在 ASP.NET MVC 应用中，该代码会置于控制器中。

> [AZURE.NOTE]
> 
> 本部分的代码引用[向表添加一批实体](#add-a-batch-of-entities-to-a-table)部分提供的 **CustomerEntity** 类和数据。

1. 添加以下 *using* 指令：

         using Microsoft.Azure;
         using Microsoft.WindowsAzure.Storage;
         using Microsoft.WindowsAzure.Storage.Auth;
         using Microsoft.WindowsAzure.Storage.Table;

2. 获取表示存储帐户信息的 **CloudStorageAccount** 对象。使用下面的代码获取存储连接字符串和 Azure 服务配置中的存储帐户信息。（将 *<storage-account-name>* 更改为要访问的 Azure 存储帐户的名称。）

         CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
           CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));

3. 获取表示表服务客户端的 **CloudTableClient** 对象。

        CloudTableClient tableClient = storageAccount.CreateCloudTableClient();


4. 获取表示所需表名称引用的 **CloudTable** 对象。（将 *<table-name>* 更改为要向其添加实体的表的名称。）

		CloudTable table = tableClient.GetTableReference(<table-name>);

5. 创建检索操作对象，该对象采用派生自 **TableEntity** 的实体对象。第一个参数是 *partitionKey* ，第二个参数是 *rowKey* 。以下代码片段使用[向表添加一批实体](#add-a-batch-of-entities-to-a-table)部分提供的 **CustomerEntity** 类和数据，通过  *partitionKey* 值“Smith”和 *rowKey* 值“Ben”查询表中的 **CustomerEntity** 实体。

        TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

6. 执行检索操作。

	    TableResult retrievedResult = table.Execute(retrieveOperation);

7. 通过检查 **TableOperation.HttpStatusCode** 属性验证操作的结果，在该属性中，状态代码为 200 指示客户端请求的操作已成功处理。也可检查 **TableResult.Result** 属性，其中会包含返回的实体（如果操作成功）。

        CustomerEntity customer = null;

        if (retrievedResult.HttpStatusCode == 200 && retrievedResult.Result != null)
        {
            // Process the customer entity.
			customer = retrievedResult.Result as CustomerEntity;
        }

## 获取分区中的所有实体

以下步骤演示了如何以编程方式获取分区中的所有实体。在 ASP.NET MVC 应用中，该代码会置于控制器中。

> [AZURE.NOTE]
> 
> 本部分的代码引用[向表添加一批实体](#add-a-batch-of-entities-to-a-table)部分提供的 **CustomerEntity** 类和数据。

1. 添加以下 *using* 指令：

         using Microsoft.Azure;
         using Microsoft.WindowsAzure.Storage;
         using Microsoft.WindowsAzure.Storage.Auth;
         using Microsoft.WindowsAzure.Storage.Table;

2. 获取表示存储帐户信息的 **CloudStorageAccount** 对象。使用下面的代码获取存储连接字符串和 Azure 服务配置中的存储帐户信息。（将 *<storage-account-name>* 更改为要访问的 Azure 存储帐户的名称。）

         CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
           CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));

3. 获取表示表服务客户端的 **CloudTableClient** 对象。

        CloudTableClient tableClient = storageAccount.CreateCloudTableClient();


4. 获取表示所需表名称引用的 **CloudTable** 对象。（将 *<table-name>* 更改为要向其添加实体的表的名称。）

		CloudTable table = tableClient.GetTableReference(<table-name>);

5. 实例化 **TableQuery** 对象，指定 **Where** 子句中的查询。以下代码片段使用[向表添加一批实体](#add-a-batch-of-entities-to-a-table)部分提供的 **CustomerEntity** 类和数据，通过 **PartitionKey** 值“Smith”查询表中的所有实体。

	    TableQuery<CustomerEntity> query = 
			new TableQuery<CustomerEntity>()
			.Where(TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"));

6. 在循环中调用 **CloudTable.ExecuteQuerySegmented** 方法，传递在上一步实例化的查询对象。**CloudTable.ExecuteQuerySegmented** 方法返回 **TableContinuationToken** 对象，该对象为 **null** 时指示没有更多可检索的实体。在循环中，使用另一个循环来循环访问返回的实体。

        TableContinuationToken token = null;
        do
        {
            TableQuerySegment<CustomerEntity>resultSegment = table.ExecuteQuerySegmented(query, token);
            token = resultSegment.ContinuationToken;

            foreach (CustomerEntity customer in resultSegment.Results)
            {
                // Process customer entity.
            }
        } while (token != null);

## 删除条目

以下步骤演示了如何搜索和删除实体。

1. 添加以下 *using* 指令：

         using Microsoft.Azure;
         using Microsoft.WindowsAzure.Storage;
         using Microsoft.WindowsAzure.Storage.Auth;
         using Microsoft.WindowsAzure.Storage.Table;

2. 获取表示存储帐户信息的 **CloudStorageAccount** 对象。使用下面的代码获取存储连接字符串和 Azure 服务配置中的存储帐户信息。（将 *<storage-account-name>* 更改为要访问的 Azure 存储帐户的名称。）

         CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
           CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));

3. 获取表示表服务客户端的 **CloudTableClient** 对象。

        CloudTableClient tableClient = storageAccount.CreateCloudTableClient();


4. 获取表示所需表名称引用的 **CloudTable** 对象。（将 *<table-name>* 更改为要向其添加实体的表的名称。）

		CloudTable table = tableClient.GetTableReference(<table-name>);

5. 创建检索操作对象，该对象采用派生自 **TableEntity** 的实体对象。第一个参数是 *partitionKey*，第二个参数是 *rowKey*。以下代码片段使用[向表添加一批实体](#add-a-batch-of-entities-to-a-table)部分提供的 **CustomerEntity** 类和数据，通过 *partitionKey* 值“Smith”和 *rowKey* 值“Ben”查询表中的 **CustomerEntity** 实体。

        TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");

6. 执行检索操作。

	    TableResult retrievedResult = table.Execute(retrieveOperation);

7. 通过检查 **TableOperation.HttpStatusCode** 属性验证操作的结果，在该属性中，状态代码为 200 指示客户端请求的操作已成功处理。也可检查 **TableResult.Result** 属性，其中会包含返回的实体（如果操作成功）。在条件语句中，若要验证操作是否成功，请创建一个删除操作（传递从查询返回的实体），然后执行该删除操作。

        if (retrievedResult.HttpStatusCode == 200 && retrievedResult.Result != null)
        {
            CustomerEntity customer = retrievedResult.Result as CustomerEntity;

            // Create the delete operation.
            TableOperation deleteOperation = TableOperation.Delete(customer);

            // Execute the delete operation.
            table.Execute(deleteOperation);
        }

## 后续步骤

[AZURE.INCLUDE [vs-storage-dotnet-tables-next-steps](../../includes/vs-storage-dotnet-tables-next-steps.md)]

<!---HONumber=Mooncake_0103_2017-->