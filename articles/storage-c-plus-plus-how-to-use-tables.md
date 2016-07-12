<properties 
    pageTitle="如何使用表存储 (C++) | Azure" 
	description="使用 Azure 表存储（一种 NoSQL 数据存储）将结构化数据存储在云中。"
    services="storage" 
    documentationCenter=".net" 
    authors="tamram" 
    manager="adinah" 
    editor=""/>

<tags 
    ms.service="storage" 
	ms.date="04/29/2016"
    wacn.date="06/06/2016"/>

# 如何通过 C++ 使用表存储

[AZURE.INCLUDE [storage-selector-table-include](../includes/storage-selector-table-include.md)]

## 概述  
本指南将演示如何使用 Azure 表存储服务执行常见方案。示例用 C++ 编写，并使用[适用于 C++ 的 Azure 存储空间客户端库](https://github.com/Azure/azure-storage-cpp/blob/master/README.md)。涉及的方案包括**创建和删除表**，以及**使用表实体**。

>[AZURE.NOTE] 本指南主要面向适用于 C++ 的 Azure 存储空间客户端库 1.0.0 版及更高版本。建议的版本是存储空间客户端库 2.2.0，它可以通过 [NuGet](http://www.nuget.org/packages/wastorage) 或 [GitHub](https://github.com/Azure/azure-storage-cpp/) 获得。

[AZURE.INCLUDE [storage-table-concepts-include](../includes/storage-table-concepts-include.md)]
[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]


## 创建 C++ 应用程序  
在本指南中，你将使用存储功能，这些功能可以在 C++ 应用程序中运行。为此，你将需要安装适用于 C++ 的 Azure 存储客户端库，并在你的 Azure 订阅中创建 Azure 存储帐户。  

若要安装适用于 C++ 的 Azure 存储客户端库，你可以使用以下方法：

-	**Linux：**按照[适用于 C++ 的 Azure 存储空间客户端库自述文件](https://github.com/Azure/azure-storage-cpp/blob/master/README.md)页中提供的说明进行操作。  
-	**Windows：**在 Visual Studio 主菜单中，单击“工具”->“NuGet 程序包管理器”->“程序包管理器控制台”。在 [NuGet 包管理器控制台](http://docs.nuget.org/docs/start-here/using-the-package-manager-console)窗口中键入以下命令，然后按 Enter。  

		Install-Package wastorage

## 配置应用程序以访问表存储  
将以下 include 语句添加到要在其中使用 Azure 存储 API 访问表的 C++ 文件的顶部：

	#include "was/storage_account.h"
	#include "was/table.h"

## 设置 Azure 存储连接字符串  
Azure 存储客户端使用存储连接字符串来存储用于访问数据管理服务的终结点和凭据。运行客户端应用程序时，必须提供以下格式的存储连接字符串。使用[管理门户](https://manage.windowsazure.cn)中列出的存储帐户的存储帐户名称和存储访问密钥作为 *AccountName* 和 *AccountKey* 值。有关存储帐户和访问密钥的信息，请参阅[关于 Azure 存储帐户](/documentation/articles/storage-create-storage-account/)。此示例演示如何声明一个静态字段以保存连接字符串：

	// Define the connection string with your values.
	const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;AccountName=your_storage_account;AccountKey=your_storage_account_key;EndpointSuffix=core.chinacloudapi.cn"));

若要在本地基于 Windows 的计算机中测试你的应用程序，可以使用随 [Azure SDK](/downloads/) 一起安装的 Azure [存储模拟器](/documentation/articles/storage-use-emulator/)。存储模拟器是一种用于模拟本地开发计算机上提供的 Azure Blob、队列和表服务的实用程序。以下示例演示如何声明一个静态字段以将连接字符串保存到你的本地存储模拟器：

	// Define the connection string with Azure storage emulator.
	const utility::string_t storage_connection_string(U("UseDevelopmentStorage=true;"));  

若要启动 Azure 存储模拟器，请单击“开始”按钮或按 Windows 键。开始键入“Azure 存储模拟器”，然后从应用程序列表中选择“Azure 存储模拟器”。

下面的示例假定你使用了这两个方法之一来获取存储连接字符串。

## 检索你的连接字符串  
可以使用 **cloud_storage_account** 类来表示您的存储帐户信息。若要从存储连接字符串中检索你的存储帐户信息，你可以使用 parse 方法。

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

接下来，获取对 **cloud_table_client** 类的引用，因为使用它可以获取表存储服务中存储的表和实体的引用对象。以下代码使用我们在上面检索到的存储帐户对象创建 **cloud_table_client** 对象：  

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

## 创建表
使用 **cloud_table_client** 对象，可以获得表和实体的引用对象。以下代码将创建 **cloud_table_client** 对象并使用它创建新表。 

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);  

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Retrieve a reference to a table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Create the table if it doesn't exist.
	table.create_if_not_exists();  

## 将实体添加到表
若要将实体添加到表，请创建一个新的 **table_entity** 对象并将其传递到 **table_operation::insert_entity**。以下代码使用客户的名字作为行键，并使用姓氏作为分区键。实体的分区键和行键共同唯一地标识表中的实体。查询分区键相同的实体的速度快于查询分区键不同的实体的速度，但使用不同的分区键可实现更高的并行操作可伸缩性。有关详细信息，请参阅 [Azure 存储性能和可伸缩性清单](/documentation/articles/storage-performance-checklist/)。

以下代码创建了包含要存储的某些客户数据的 **table_entity** 类的新实例。接下来，该代码调用 **table_operation::insert_entity** 以创建一个 **table_operation** 对象，以便将实体插入表中，并将新的表实体与之关联。最后，该代码调用 **cloud_table** 对象的 execute 方法。并且新的 **table_operation** 向表服务发送请求，以将新的客户实体插入"people"表中。  

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Retrieve a reference to a table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Create the table if it doesn't exist.
	table.create_if_not_exists();

	// Create a new customer entity.
	azure::storage::table_entity customer1(U("Harp"), U("Walter"));

	azure::storage::table_entity::properties_type& properties = customer1.properties();
	properties.reserve(2);
	properties[U("Email")] = azure::storage::entity_property(U("Walter@contoso.com"));

	properties[U("Phone")] = azure::storage::entity_property(U("425-555-0101"));

	// Create the table operation that inserts the customer entity.
	azure::storage::table_operation insert_operation = azure::storage::table_operation::insert_entity(customer1);

	// Execute the insert operation.
	azure::storage::table_result insert_result = table.execute(insert_operation);

## 插入一批实体
你可以通过一次写入操作将一批实体插入到表服务。以下代码创建一个 **table_batch_operation** 对象，然后向其中添加三个插入操作。每个插入操作的添加方法如下：创建一个新的实体对象，设置它的值，然后对 **table_batch_operation** 对象调用 insert 方法以将实体与新的插入操作相关联。然后调用 **cloud_table.execute** 以执行此操作。  

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Create a cloud table object for the table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Define a batch operation.
	azure::storage::table_batch_operation batch_operation;

	// Create a customer entity and add it to the table.
	azure::storage::table_entity customer1(U("Smith"), U("Jeff"));

	azure::storage::table_entity::properties_type& properties1 = customer1.properties();
	properties1.reserve(2);
	properties1[U("Email")] = azure::storage::entity_property(U("Jeff@contoso.com"));
	properties1[U("Phone")] = azure::storage::entity_property(U("425-555-0104"));

	// Create another customer entity and add it to the table.
	azure::storage::table_entity customer2(U("Smith"), U("Ben"));

	azure::storage::table_entity::properties_type& properties2 = customer2.properties();
	properties2.reserve(2);
	properties2[U("Email")] = azure::storage::entity_property(U("Ben@contoso.com"));
	properties2[U("Phone")] = azure::storage::entity_property(U("425-555-0102"));

	// Create a third customer entity to add to the table.
	azure::storage::table_entity customer3(U("Smith"), U("Denise"));

	azure::storage::table_entity::properties_type& properties3 = customer3.properties();
	properties3.reserve(2);
	properties3[U("Email")] = azure::storage::entity_property(U("Denise@contoso.com"));
	properties3[U("Phone")] = azure::storage::entity_property(U("425-555-0103"));

	// Add customer entities to the batch insert operation.
	batch_operation.insert_or_replace_entity(customer1);
	batch_operation.insert_or_replace_entity(customer2);
	batch_operation.insert_or_replace_entity(customer3);

	// Execute the batch operation.
	std::vector<azure::storage::table_result> results = table.execute_batch(batch_operation);

批处理操作的注意事项如下：

-	您在单次批处理操作中最多可以执行 100 个插入、删除、合并、替换、插入或合并以及插入或替换操作（可以是这些操作的任意组合）。  
-	批处理操作也可以包含检索操作，但前提是检索操作是批处理中仅有的操作。  
-	单次批处理操作中的所有实体都必须具有相同的分区键。  
-	批处理操作的数据负载限制为 4MB。  

## 检索分区中的所有实体
若要查询表以获取分区中的所有实体，请使用 **table_query** 对象。以下代码示例指定了一个筛选器，以筛选分区键为  'Smith' 的实体。此示例会将查询结果中每个实体的字段输出到控制台。  

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Create a cloud table object for the table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Construct the query operation for all customer entities where PartitionKey="Smith".
	azure::storage::table_query query;

	query.set_filter_string(azure::storage::table_query::generate_filter_condition(U("PartitionKey"), azure::storage::query_comparison_operator::equal, U("Smith")));

	// Execute the query.
	azure::storage::table_query_iterator it = table.execute_query(query);

	// Print the fields for each customer.
	azure::storage::table_query_iterator end_of_results;
	for (; it != end_of_results; ++it)
	{
		const azure::storage::table_entity::properties_type& properties = it->properties();

		std::wcout << U("PartitionKey: ") << it->partition_key() << U(", RowKey: ") << it->row_key()
			<< U(", Property1: ") << properties.at(U("Email")).string_value()
			<< U(", Property2: ") << properties.at(U("Phone")).string_value() << std::endl;
	}  

此示例中的查询将检索出与筛选条件匹配的所有实体。如果你有大型表并需要经常下载表实体，我们建议你改为将数据存储在 Azure 存储 Blob 中。

## 检索分区中的一部分实体
如果不想查询分区中的所有实体，则可以通过结合使用分区键筛选器与行键筛选器来指定一个范围。以下代码示例使用两个筛选器来获取分区“Smith”中的、行键（名字）以字母“E”前面的字母开头的所有实体，然后输出查询结果。

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Create a cloud table object for the table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Create the table query.
	azure::storage::table_query query;

	query.set_filter_string(azure::storage::table_query::combine_filter_conditions(
		azure::storage::table_query::generate_filter_condition(U("PartitionKey"),
		azure::storage::query_comparison_operator::equal, U("Smith")),
		azure::storage::query_logical_operator::op_and,
		azure::storage::table_query::generate_filter_condition(U("RowKey"), azure::storage::query_comparison_operator::less_than, U("E"))));

	// Execute the query.
	azure::storage::table_query_iterator it = table.execute_query(query);

	// Loop through the results, displaying information about the entity.
	azure::storage::table_query_iterator end_of_results;
	for (; it != end_of_results; ++it)
	{
		const azure::storage::table_entity::properties_type& properties = it->properties();

		std::wcout << U("PartitionKey: ") << it->partition_key() << U(", RowKey: ") << it->row_key()
			<< U(", Property1: ") << properties.at(U("Email")).string_value()
			<< U(", Property2: ") << properties.at(U("Phone")).string_value() << std::endl;
	}  

## 检索单个实体
你可以编写查询以检索单个特定实体。以下代码使用 **table_operation::retrive_entity** 来指定客户  'Jeff Smith'。此方法只返回一个实体，而不是一个集合，并且返回的值在 **table_result** 中。在查询中指定分区键和行键是从表服务中检索单个实体的最快方法。  

	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Create a cloud table object for the table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Retrieve the entity with partition key of "Smith" and row key of "Jeff".
	azure::storage::table_operation retrieve_operation = azure::storage::table_operation::retrieve_entity(U("Smith"), U("Jeff"));
	azure::storage::table_result retrieve_result = table.execute(retrieve_operation);

	// Output the entity.
	azure::storage::table_entity entity = retrieve_result.entity();
	const azure::storage::table_entity::properties_type& properties = entity.properties();

	std::wcout << U("PartitionKey: ") << entity.partition_key() << U(", RowKey: ") << entity.row_key()
		<< U(", Property1: ") << properties.at(U("Email")).string_value()
		<< U(", Property2: ") << properties.at(U("Phone")).string_value() << std::endl;

## 替换实体
若要替换实体，请从表服务中检索它，修改实体对象，然后将更改保存回表服务。以下代码更改现有客户的电话号码和电子邮件地址。此代码不是调用 **table_operation::insert_entity**，而是使用 **table_operation::replace_entity**。这将导致在服务器上完全替换该实体，除非服务器上的该实体自检索到它以后发生更改，在此情况下，该操作将失败。操作失败将防止您的应用程序无意中覆盖应用程序的其他组件在检索与更新之间所做的更改。正确处理此失败的方法是再次检索实体，进行更改（如果仍有效），然后执行另一个 **table_operation::replace_entity** 操作。下一节将为你演示如何重写此行为。  

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Create a cloud table object for the table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Replace an entity.
	azure::storage::table_entity entity_to_replace(U("Smith"), U("Jeff"));
	azure::storage::table_entity::properties_type& properties_to_replace = entity_to_replace.properties();
	properties_to_replace.reserve(2);

	// Specify a new phone number.
	properties_to_replace[U("Phone")] = azure::storage::entity_property(U("425-555-0106"));

	// Specify a new email address.
	properties_to_replace[U("Email")] = azure::storage::entity_property(U("JeffS@contoso.com"));

	// Create an operation to replace the entity.
	azure::storage::table_operation replace_operation = azure::storage::table_operation::replace_entity(entity_to_replace);

	// Submit the operation to the Table service.
	azure::storage::table_result replace_result = table.execute(replace_operation);

## 插入或替换实体
如果该实体自从服务器中检索到它以后发生更改，则 **table_operation::replace_entity** 操作将失败。此外，你必须首先从服务器中检索该实体，**table_operation::replace_entity** 才能成功。但是，有时您不知道服务器上是否存在该实体以及存储在其中的当前值是否无关 - 更新操作会将其全部覆盖。为实现此目的，你将使用 **table_operation::insert_or_replace_entity** 操作。如果该实体不存在，此操作将插入它，如果存在，则替换它，而不管上次更新是何时进行的。在以下代码示例中，仍将检索 Jeff Smith 的客户实体，但稍后会使用 **table_operation::insert_or_replace_entity** 将其保存回服务器。将覆盖在检索与更新操作之间对实体进行的任何更新。  

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Create a cloud table object for the table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Insert-or-replace an entity.
	azure::storage::table_entity entity_to_insert_or_replace(U("Smith"), U("Jeff"));
	azure::storage::table_entity::properties_type& properties_to_insert_or_replace = entity_to_insert_or_replace.properties();

	properties_to_insert_or_replace.reserve(2);

	// Specify a phone number.
	properties_to_insert_or_replace[U("Phone")] = azure::storage::entity_property(U("425-555-0107"));

	// Specify an email address.
	properties_to_insert_or_replace[U("Email")] = azure::storage::entity_property(U("Jeffsm@contoso.com"));

	// Create an operation to insert-or-replace the entity.
	azure::storage::table_operation insert_or_replace_operation = azure::storage::table_operation::insert_or_replace_entity(entity_to_insert_or_replace);

	// Submit the operation to the Table service.
	azure::storage::table_result insert_or_replace_result = table.execute(insert_or_replace_operation);

## 查询一部分实体属性  
对表的查询可以只检索实体中的少数几个属性。以下代码中的查询使用 **table_query::set_select_columns** 方法，仅返回表中实体的电子邮件地址。  

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Create a cloud table object for the table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Define the query, and select only the Email property.
	azure::storage::table_query query;
	std::vector<utility::string_t> columns;

	columns.push_back(U("Email"));
	query.set_select_columns(columns);

	// Execute the query.
	azure::storage::table_query_iterator it = table.execute_query(query);

	// Display the results.
	azure::storage::table_query_iterator end_of_results;
	for (; it != end_of_results; ++it)
	{
		std::wcout << U("PartitionKey: ") << it->partition_key() << U(", RowKey: ") << it->row_key();

		const azure::storage::table_entity::properties_type& properties = it->properties();
		for (auto prop_it = properties.begin(); prop_it != properties.end(); ++prop_it)
		{
			std::wcout << ", " << prop_it->first << ": " << prop_it->second.str();
		}

		std::wcout << std::endl;
	}

>[AZURE.NOTE] 查询实体的几个属性是比检索所有属性更高效的操作。

## 删除实体
你可以在检索到实体后轻松将其删除。检索到实体后，对要删除的实体调用 **table_operation::delete_entity**。然后调用 **cloud_table.execute** 方法。以下代码检索并删除分区键为"Smith"、行键为"Jeff"的实体。  

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Create a cloud table object for the table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Create an operation to retrieve the entity with partition key of "Smith" and row key of "Jeff".
	azure::storage::table_operation retrieve_operation = azure::storage::table_operation::retrieve_entity(U("Smith"), U("Jeff"));
	azure::storage::table_result retrieve_result = table.execute(retrieve_operation);

	// Create an operation to delete the entity.
	azure::storage::table_operation delete_operation = azure::storage::table_operation::delete_entity(retrieve_result.entity());

	// Submit the delete operation to the Table service.
	azure::storage::table_result delete_result = table.execute(delete_operation);  

## 删除表
最后，以下代码示例将从存储帐户中删除表。在删除表之后的一段时间内无法重新创建它。

	// Retrieve the storage account from the connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the table client.
	azure::storage::cloud_table_client table_client = storage_account.create_cloud_table_client();

	// Create a cloud table object for the table.
	azure::storage::cloud_table table = table_client.get_table_reference(U("people"));

	// Create an operation to retrieve the entity with partition key of "Smith" and row key of "Jeff".
	azure::storage::table_operation retrieve_operation = azure::storage::table_operation::retrieve_entity(U("Smith"), U("Jeff"));
	azure::storage::table_result retrieve_result = table.execute(retrieve_operation);

	// Create an operation to delete the entity.
	azure::storage::table_operation delete_operation = azure::storage::table_operation::delete_entity(retrieve_result.entity());

	// Submit the delete operation to the Table service.
	azure::storage::table_result delete_result = table.execute(delete_operation);

## 后续步骤
现在，你已了解表存储的基础知识，请打开以下链接了解有关 Azure 存储空间的详细信息：

-	[如何通过 C++ 使用 Blob 存储](/documentation/articles/storage-c-plus-plus-how-to-use-blobs/)
-	[如何通过 C++ 使用队列存储](/documentation/articles/storage-c-plus-plus-how-to-use-queues/)
-	[使用 C++ 列出 Azure 存储资源](/documentation/articles/storage-c-plus-plus-enumeration/)
-	[适用于 C++ 的存储空间客户端库参考](http://azure.github.io/azure-storage-cpp)
-	[Azure 存储文档](/documentation/services/storage)
 

<!---HONumber=Mooncake_0530_2016-->