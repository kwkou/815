#####创建表
利用 **CloudTableClient** 对象，您可以获得表和实体的引用对象。以下代码将创建 **CloudTableClient** 对象并使用它创建新表。该代码将尝试引用名为"people"的表。如果找不到该名称的表，它将创建一个。

**注意：**指南中的所有代码假定将构建的应用程序是 Azure 云服务项目，并且使用存储在 Azure 应用程序的服务配置中的存储连接字符串。

	// Create the table client.
	CloudTableClient tableClient = storageAccount.CreateCloudTableClient();
	
	// Create the table if it doesn't exist.
	CloudTable table = tableClient.GetTableReference("people");
	table.CreateIfNotExists();

#####将实体添加到表
若要将实体添加到表，请创建用于定义实体的属性的类。以下代码定义了将客户的名字和姓氏分别用作行键和分区键的 **CustomerEntity** 实体类。

	public class CustomerEntity : TableEntity
	{
	    public CustomerEntity(string lastName, string firstName)
	    {
	        this.PartitionKey = lastName;
	        this.RowKey = firstName;
	    }
	
	    public CustomerEntity() { }
	
	    public string Email { get; set; }
	
	    public string PhoneNumber { get; set; }
	}

将使用您之前在"创建表"中创建的 **CloudTable** 对象完成涉及实体的表操作。**TableOperation** 对象表示将完成的操作。以下代码示例演示如何创建 **CloudTable** 对象以及 **CustomerEntity** 对象。为准备此操作，会创建一个 **TableOperation** 以将客户实体插入该表中。最后，将通过调用 CloudTable.Execute 执行此操作。

	// Create the CloudTable object that represents the "people" table.
	CloudTable table = tableClient.GetTableReference("people");
	
	// Create a new customer entity.
	CustomerEntity customer1 = new CustomerEntity("Harp", "Walter");
	customer1.Email = "Walter@contoso.com";
	customer1.PhoneNumber = "425-555-0101";
	
	// Create the TableOperation that inserts the customer entity.
	TableOperation insertOperation = TableOperation.Insert(customer1);
	
	// Execute the insert operation.
	table.Execute(insertOperation);

#####插入一批实体
您可以通过单个写入操作将多个实体插入表中。以下代码示例将创建两个实体对象（"Jeff Smith"和"Ben Smith"），使用 Insert 方法将它们添加到 **TableBatchOperation** 对象，然后通过调用 CloudTable.Execute 启动操作。

	// Create the CloudTable object that represents the "people" table.
	CloudTable table = tableClient.GetTableReference("people");
	
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
	table.ExecuteBatch(batchOperation);

#####获取分区中的所有实体
若要查询表以获取分区中的所有实体，请使用 **TableQuery** 对象。以下代码示例指定了一个筛选器，以筛选分区键为  'Smith' 的实体。此示例会将查询结果中每个实体的字段输出到控制台。

	// Create the CloudTable object that represents the "people" table.
	CloudTable table = tableClient.GetTableReference("people");
	
	// Construct the query operation for all customer entities where PartitionKey="Smith".
	TableQuery<CustomerEntity> query = new TableQuery<CustomerEntity>().Where(TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, "Smith"));
	
	// Print the fields for each customer.
	foreach (CustomerEntity entity in table.ExecuteQuery(query))
	{
	    Console.WriteLine("{0}, {1}\t{2}\t{3}", entity.PartitionKey, entity.RowKey,
	        entity.Email, entity.PhoneNumber);
	}

#####获取单个实体
您可以编写查询以获取单个特定实体。以下代码使用 **TableOperation** 对象来指定名为  'Ben Smith' 的客户。此方法仅返回一个实体，而不是一个集合，并且 TableResult.Result 中的返回值是一个 **CustomerEntity** 对象。在查询中同时指定分区键和行键是从**表**服务中检索单个实体的最快方法。

	// Create the CloudTable object that represents the "people" table.
	CloudTable table = tableClient.GetTableReference("people");
	
	// Create a retrieve operation that takes a customer entity.
	TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");
	
	// Execute the retrieve operation.
	TableResult retrievedResult = table.Execute(retrieveOperation);
	
	// Print the phone number of the result.
	if (retrievedResult.Result != null)
	   Console.WriteLine(((CustomerEntity)retrievedResult.Result).PhoneNumber);
	else
	   Console.WriteLine("The phone number could not be retrieved.");

#####删除实体
您可以在找到实体后将其删除。以下代码将查找名为"Ben Smith"的客户实体，如果找到，会将其删除。

	// Create the CloudTable that represents the "people" table.
	CloudTable table = tableClient.GetTableReference("people");
	
	// Create a retrieve operation that expects a customer entity.
	TableOperation retrieveOperation = TableOperation.Retrieve<CustomerEntity>("Smith", "Ben");
	
	// Execute the operation.
	TableResult retrievedResult = table.Execute(retrieveOperation);
	
	// Assign the result to a CustomerEntity object.
	CustomerEntity deleteEntity = (CustomerEntity)retrievedResult.Result;
	
	// Create the Delete TableOperation and then execute it.
	if (deleteEntity != null)
	{
	   TableOperation deleteOperation = TableOperation.Delete(deleteEntity);
	
	   // Execute the operation.
	   table.Execute(deleteOperation);
	
	   Console.WriteLine("Entity deleted.");
	}
	
	else
	   Console.WriteLine("Couldn't delete the entity.");

[了解有关 Azure 存储的详细信息](/zh-cn/documentation/services/storage)
另请参阅[在服务器资源管理器中浏览存储资源](http://msdn.microsoft.com/zh-cn/library/azure/ff683677.aspx).<!--HONumber=41-->
