<properties
	pageTitle="如何通过 PHP 使用表存储 | Azure"
	description="了解如何通过 PHP 使用表服务来创建和删除表以及插入、删除和查询表。"
	services="storage"
	documentationCenter="php"
	authors="tfitzmac"
	manager="adinah"
	editor=""/>

<tags
	ms.service="storage"
	ms.date="04/08/2016"
	wacn.date="05/23/2016"/>


# 如何通过 PHP 使用表存储

[AZURE.INCLUDE [storage-selector-table-include](../includes/storage-selector-table-include.md)]

## 概述

本指南演示如何使用 Azure 表服务执行常见任务。示例是用 PHP 编写的并使用了 [Azure SDK for PHP][download]。所涉及的任务包括“创建和删除表以及在表中插入、删除和查询实体”。有关 Azure 表服务的详细信息，请参阅[后续步骤](#next-steps)部分。

[AZURE.INCLUDE [storage-table-concepts-include](../includes/storage-table-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

## 创建 PHP 应用程序

创建访问 Azure 表服务的 PHP 应用程序的唯一要求是从代码中引用 Azure SDK for PHP 中的类。你可以使用任何开发工具（包括“记事本”）创建应用程序。

在本指南中，你将使用表服务功能，这些功能可在 PHP 应用程序中本地调用，或通过在 Azure 的 Web 角色、辅助角色或 Web 应用中运行的代码调用。

## 获取 Azure 客户端库

[AZURE.INCLUDE [get-client-libraries](../includes/get-client-libraries.md)]

## 配置你的应用程序以访问表服务

若要使用 Azure 表服务 API，你需要：

1. 使用 [require_once][require_once] 语句引用 autoloader 文件，并
2. 引用可使用的所有类。

下面的示例演示了如何包括 autoloader 文件并引用 **ServicesBuilder** 类。

> [AZURE.NOTE] 本示例（以及本文中的其他示例）假定您已通过 Composer 安装用于 Azure 的 PHP 客户端库。如果你已手动安装这些库或将其作为 PEAR 包安装，则需要引用 <code>WindowsAzure.php</code> autoloader 文件。

	require_once 'vendor\autoload.php';
	use WindowsAzure\Common\ServicesBuilder;


在下面的示例中，`require_once` 语句将始终显示，但只会引用执行该示例所需的类。

## 设置 Azure 存储连接

若要实例化 Azure 表服务客户端，你必须首先具有有效的连接字符串。表服务连接字符串的格式为：

对于访问实时服务：

	DefaultEndpointsProtocol=[http|https];AccountName=[yourAccount];AccountKey=[yourKey];EndpointSuffix=core.chinacloudapi.cn

对于访问模拟器存储：

	UseDevelopmentStorage=true


若要创建任何 Azure 服务客户端，你将需要使用 **ServicesBuilder** 类。你可以：

* 将连接字符串直接传递给此类或
* 使用 **CloudConfigurationManager (CCM)** 检查多个外部源以获取连接字符串：
	* 默认情况下，它附带了对一个外部源的支持 - 环境变量
	* 你可通过扩展 **ConnectionStringSource** 类来添加新源

在此处列出的示例中，将直接传递连接字符串。

	require_once 'vendor\autoload.php';

	use WindowsAzure\Common\ServicesBuilder;

	$tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);


## 创建表

利用 **TableRestProxy** 对象，可以使用 **createTable** 方法创建表。创建表时，可以设置表服务超时。（有关表服务超时的详细信息，请参阅[为表服务操作设置超时][table-service-timeouts]。）

	require_once 'vendor\autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use WindowsAzure\Common\ServiceException;

	// Create table REST proxy.
	$tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

	try	{
		// Create table.
		$tableRestProxy->createTable("mytable");
	}
	catch(ServiceException $e){
		$code = $e->getCode();
		$error_message = $e->getMessage();
		// Handle exception based on error codes and messages.
		// Error codes and messages can be found here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
	}

有关表名称的限制的信息，请参阅[了解表服务数据模型][table-data-model]。

## 将实体添加到表

若要将实体添加到表，请创建一个新的 **Entity** 对象并将其传递到 **TableRestProxy->insertEntity**。请注意，在创建实体时，你必须指定 `PartitionKey` 和 `RowKey`。这些值是实体的唯一标识符，并且其查询速度比其他实体属性的查询速度快得多。系统使用 `PartitionKey` 自动将表的实体分发到多个存储节点上。具有相同 `PartitionKey` 的实体存储在同一个节点上。（对存储在同一节点上的多个实体执行操作要将比对存储在不同节点上的实体执行的操作的效果更佳。） `RowKey` 是实体在分区中的唯一 ID。

	require_once 'vendor\autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use WindowsAzure\Common\ServiceException;
	use WindowsAzure\Table\Models\Entity;
	use WindowsAzure\Table\Models\EdmType;

	// Create table REST proxy.
	$tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

	$entity = new Entity();
	$entity->setPartitionKey("tasksSeattle");
	$entity->setRowKey("1");
	$entity->addProperty("Description", null, "Take out the trash.");
	$entity->addProperty("DueDate",
						 EdmType::DATETIME,
						 new DateTime("2012-11-05T08:15:00-08:00"));
	$entity->addProperty("Location", EdmType::STRING, "Home");

	try{
		$tableRestProxy->insertEntity("mytable", $entity);
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
	}

有关表属性和类型的信息，请参阅[了解表服务数据模型][table-data-model]。

**TableRestProxy** 类提供了用于插入实体的两个替代方法：**insertOrMergeEntity** 和 **insertOrReplaceEntity**。若要使用这些方法，请创建一个新的 **Entity**，并将其作为参数传递到上述任一方法。如果实体不存在，则每种方法都将插入实体。在实体已存在的情况下，如果属性已存在，则 **insertOrMergeEntity** 将更新属性值；如果属性不存在，则该方法将添加新属性，而 **insertOrReplaceEntity** 将完全替换现有实体。下面的示例演示如何使用 **insertOrMergeEntity**。如果实体具有 `PartitionKey`“tasksSeattle”并且 `RowKey`“1”不存在，则将插入该实体。但是，如果之前已插入该实体（如上面的示例所示），则将更新 `DueDate` 属性并添加 `Status` 属性。系统还将更新 `Description` 和 `Location` 属性，但使用的值实际上会使其保持不变。如果后面两个属性不是如示例中所示添加的，但已存在于目标实体上，则其现有值将保持不变。

	require_once 'vendor\autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use WindowsAzure\Common\ServiceException;
	use WindowsAzure\Table\Models\Entity;
	use WindowsAzure\Table\Models\EdmType;

	// Create table REST proxy.
	$tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

	//Create new entity.
	$entity = new Entity();

	// PartitionKey and RowKey are required.
	$entity->setPartitionKey("tasksSeattle");
	$entity->setRowKey("1");

	// If entity exists, existing properties are updated with new values and
	// new properties are added. Missing properties are unchanged.
	$entity->addProperty("Description", null, "Take out the trash.");
	$entity->addProperty("DueDate", EdmType::DATETIME, new DateTime()); // Modified the DueDate field.
	$entity->addProperty("Location", EdmType::STRING, "Home");
	$entity->addProperty("Status", EdmType::STRING, "Complete"); // Added Status field.

	try	{
		// Calling insertOrReplaceEntity, instead of insertOrMergeEntity as shown,
		// would simply replace the entity with PartitionKey "tasksSeattle" and RowKey "1".
		$tableRestProxy->insertOrMergeEntity("mytable", $entity);
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}


## 检索单个实体

利用 **TableRestProxy->getEntity** 方法，可以通过查询实体的 `PartitionKey` 和 `RowKey` 来检索它。在以下示例中，分区键 `tasksSeattle` 和行键 `1` 传递给 **getEntity** 方法。

	require_once 'vendor\autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use WindowsAzure\Common\ServiceException;

	// Create table REST proxy.
	$tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

	try	{
		$result = $tableRestProxy->getEntity("mytable", "tasksSeattle", 1);
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

	$entity = $result->getEntity();

	echo $entity->getPartitionKey().":".$entity->getRowKey();

## 检索分区中的所有实体

使用筛选器来构造实体查询（有关详细信息，请参阅[查询表和实体][filters]）。若要检索分区中的所有实体，请使用筛选器“PartitionKey eq partition_name”。下面的示例演示如何通过将筛选器传递到 **queryEntities** 方法来检索 `tasksSeattle` 分区中的所有实体。

	require_once 'vendor\autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use WindowsAzure\Common\ServiceException;

	// Create table REST proxy.
	$tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

	$filter = "PartitionKey eq 'tasksSeattle'";

	try	{
		$result = $tableRestProxy->queryEntities("mytable", $filter);
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

	$entities = $result->getEntities();

	foreach($entities as $entity){
		echo $entity->getPartitionKey().":".$entity->getRowKey()."<br />";
	}

## 检索分区中的一部分实体

可以使用上一示例中使用的同一模式来检索分区中的部分实体。您检索的部分实体将由您使用的筛选器确定（有关详细信息，请参阅[查询表和实体][filters]）。下面的示例演示如何使用筛选器检索具有特定的 `Location` 和早于指定日期 `DueDate` 的所有实体。

	require_once 'vendor\autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use WindowsAzure\Common\ServiceException;

	// Create table REST proxy.
	$tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

	$filter = "Location eq 'Office' and DueDate lt '2012-11-5'";

	try	{
		$result = $tableRestProxy->queryEntities("mytable", $filter);
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

	$entities = $result->getEntities();

	foreach($entities as $entity){
		echo $entity->getPartitionKey().":".$entity->getRowKey()."<br />";
	}

## 检索一部分实体属性

查询可检索一部分实体属性。此方法称为“投影”，可减少带宽并提高查询性能，尤其适用于大型实体。若要指定要检索的属性，请将该属性的名称传递到 **Query->addSelectField** 方法。可以多次调用此方法来添加更多属性。执行 **TableRestProxy->queryEntities** 后，返回的实体将仅具有选定的属性。（若要返回一部分表实体，请使用上述查询中所示的筛选器。）

	require_once 'vendor\autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use WindowsAzure\Common\ServiceException;
	use WindowsAzure\Table\Models\QueryEntitiesOptions;

	// Create table REST proxy.
	$tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

	$options = new QueryEntitiesOptions();
	$options->addSelectField("Description");

	try	{
		$result = $tableRestProxy->queryEntities("mytable", $options);
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

	// All entities in the table are returned, regardless of whether
	// they have the Description field.
	// To limit the results returned, use a filter.
	$entities = $result->getEntities();

	foreach($entities as $entity){
		$description = $entity->getProperty("Description")->getValue();
		echo $description."<br />";
	}

## 更新实体

可通过对现有实体使用 **Entity->setProperty** 和 **Entity->addProperty** 方法并调用 **TableRestProxy->updateEntity** 来更新该实体。下面的示例将检索一个实体、修改一个属性、删除另一个属性并添加一个新属性。请注意，通过将属性的值设为 **null** 可删除该属性。

	require_once 'vendor\autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use WindowsAzure\Common\ServiceException;
	use WindowsAzure\Table\Models\Entity;
	use WindowsAzure\Table\Models\EdmType;

	// Create table REST proxy.
	$tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

	$result = $tableRestProxy->getEntity("mytable", "tasksSeattle", 1);

	$entity = $result->getEntity();

	$entity->setPropertyValue("DueDate", new DateTime()); //Modified DueDate.

	$entity->setPropertyValue("Location", null); //Removed Location.

	$entity->addProperty("Status", EdmType::STRING, "In progress"); //Added Status.

	try	{
		$tableRestProxy->updateEntity("mytable", $entity);
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

## 删除实体

若要删除实体，请将表名称以及实体的 `PartitionKey` 和 `RowKey` 传递到 **TableRestProxy->deleteEntity** 方法。

	require_once 'vendor\autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use WindowsAzure\Common\ServiceException;

	// Create table REST proxy.
	$tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

	try	{
		// Delete entity.
		$tableRestProxy->deleteEntity("mytable", "tasksSeattle", "2");
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

请注意，为了进行并发检查，可以使用 **DeleteEntityOptions->setEtag** 方法并将 **DeleteEntityOptions** 对象作为第四个参数传递到 **deleteEntity**，来为要删除的实体设置 Etag。

## 对表操作进行批处理

利用 **TableRestProxy->batch** 方法，你可以通过一个请求执行多个操作。此处的模式涉及将操作添加到 **BatchRequest** 对象，然后将 **BatchRequest** 对象传递到 **TableRestProxy->batch** 方法。若要将操作添加到 **BatchRequest** 对象，可以多次调用以下任一方法：

* **addInsertEntity**（添加 insertEntity 操作）
* **addUpdateEntity**（添加 updateEntity 操作）
* **addMergeEntity**（添加 mergeEntity 操作）
* **addInsertOrReplaceEntity**（添加 insertOrReplaceEntity 操作）
* **addInsertOrMergeEntity**（添加 insertOrMergeEntity 操作）
* **addDeleteEntity**（添加 deleteEntity 操作）

下面的示例演示了如何通过单个请求执行 **insertEntity** 和 **deleteEntity** 操作：

	require_once 'vendor\autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use WindowsAzure\Common\ServiceException;
	use WindowsAzure\Table\Models\Entity;
	use WindowsAzure\Table\Models\EdmType;
	use WindowsAzure\Table\Models\BatchOperations;

 	// Create table REST proxy.
	$tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

	// Create list of batch operation.
	$operations = new BatchOperations();

	$entity1 = new Entity();
	$entity1->setPartitionKey("tasksSeattle");
	$entity1->setRowKey("2");
	$entity1->addProperty("Description", null, "Clean roof gutters.");
	$entity1->addProperty("DueDate",
						  EdmType::DATETIME,
						  new DateTime("2012-11-05T08:15:00-08:00"));
	$entity1->addProperty("Location", EdmType::STRING, "Home");

	// Add operation to list of batch operations.
    $operations->addInsertEntity("mytable", $entity1);

	// Add operation to list of batch operations.
	$operations->addDeleteEntity("mytable", "tasksSeattle", "1");

	try	{
		$tableRestProxy->batch($operations);
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

有关对表操作进行批处理的详细信息，请参阅[执行实体组事务][entity-group-transactions]。

## 删除表

最后，若要删除表，请将表名传递到 **TableRestProxy->deleteTable** 方法。

	require_once 'vendor\autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use WindowsAzure\Common\ServiceException;

	// Create table REST proxy.
	$tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

	try	{
		// Delete table.
		$tableRestProxy->deleteTable("mytable");
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

## 后续步骤

现在，您已了解了 Azure 表服务的基础知识，单击下面的链接可了解有关更复杂的存储任务的详细信息。

- 访问 [Azure 存储空间团队博客](http://blogs.msdn.com/b/windowsazurestorage/)

有关详细信息，另请参阅 [PHP 开发人员中心](/develop/php/)。

[download]: /documentation/articles/php-download-sdk/
[require_once]: http://php.net/require_once
[table-service-timeouts]: http://msdn.microsoft.com/zh-cn/library/azure/dd894042.aspx

[table-data-model]: http://msdn.microsoft.com/zh-cn/library/azure/dd179338.aspx
[filters]: http://msdn.microsoft.com/zh-cn/library/azure/dd894031.aspx
[entity-group-transactions]: http://msdn.microsoft.com/zh-cn/library/azure/dd894038.aspx


<!---HONumber=Mooncake_0516_2016-->