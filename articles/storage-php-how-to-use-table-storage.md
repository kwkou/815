<properties linkid="develop-php-table-service" urlDisplayName="Table Service" pageTitle="How to use table storage (PHP) | Microsoft Azure" metaKeywords="Azure Table service PHP, Azure creating table, Azure deleting table, Azure insert table, Azure query table" description="Learn how to use the Table service from PHP to create and delete a table, and insert, delete, and query the table." metaCanonical="" services="storage" documentationCenter="PHP" title="How to use the Table service from PHP" authors="" solutions="" manager="" editor="" />

# 如何通过 PHP 使用表服务

本指南将演示如何使用 Azure 表服务执行常见方案。示例是用 PHP 编写的并使用了 [Azure SDK for PHP][]。所涉及的任务包括**创建和删除表以及在表中插入、删除和查询实体**。有关 Azure 表服务的详细信息，请参阅[后续步骤][]部分。

## 目录

-   [什么是表服务？][]
-   [概念][]
-   [创建 Azure 存储帐户][]
-   [创建 PHP 应用程序][]
-   [配置你的应用程序以访问表服务][]
-   [设置 Azure 存储连接][]
-   [如何：创建表][]
-   [如何：将实体添加到表][]
-   [如何：检索单个实体][]
-   [如何：检索分区中的所有实体][]
-   [如何：检索分区中的一部分实体][]
-   [如何：检索一部分实体属性][]
-   [如何：更新实体][]
-   [如何：对表操作进行批处理][]
-   [如何：删除表][]
-   [后续步骤][]

[WACOM.INCLUDE [howto-table-storage][]]

## 创建 Azure 存储帐户

[WACOM.INCLUDE [create-storage-account][]]

## 创建 PHP 应用程序

创建访问 Azure 表服务的 PHP 应用程序的唯一要求是从代码中引用 Azure SDK for PHP 中的类。你可以使用任何开发工具（包括“记事本”）创建应用程序。

在本指南中，你将使用表服务功能，这些功能可在 PHP 应用程序中本地调用，或通过在 Azure 的 Web 角色、辅助角色或网站中运行的代码调用。

## 获取 Azure 客户端库

[WACOM.INCLUDE [get-client-libraries][]]

## 配置你的应用程序以访问表服务

若要使用 Azure 表服务 API，你需要：

1.  使用 [require\_once][] 语句引用 autoloader 文件，并
2.  引用可使用的所有类。

下面的示例演示了如何包括 autoloader 文件并引用 **ServicesBuilder** 类。

> [WACOM.NOTE]
> 本示例（以及本文中的其他示例）假定你已通过 Composer 安装了用于 Azure 的 PHP 客户端库。如果你已手动安装这些库或将其作为 PEAR 包安装，则需要引用 `WindowsAzure.php` autoloader 文件。

    require_once 'vendor\autoload.php';
    use WindowsAzure\Common\ServicesBuilder;

在下面的示例中，`require_once` 语句将始终显示，但只会引用执行该示例所需的类。

## 设置 Azure 存储连接

若要实例化 Azure 表服务客户端，你必须首先具有有效的连接字符串。表服务连接字符串的格式为：

对于访问实时服务：

    DefaultEndpointsProtocol=[http|https];AccountName=[yourAccount];AccountKey=[yourKey]

对于访问模拟器存储：

    UseDevelopmentStorage=true

若要创建任何 Azure 服务客户端，你将需要使用 **ServicesBuilder** 类。你可以：

-   将连接字符串直接传递给此类或
-   使用 **CloudConfigurationManager (CCM)** 检查多个外部源以获取连接字符串：

    -   默认情况下，它附带了对一个外部源的支持 - 环境变量
    -   你可通过扩展 **ConnectionStringSource** 类来添加新源

在此处列出的示例中，将直接传递连接字符串。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;

    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

## 如何：创建表

利用 **TableRestProxy** 对象，可以使用 **createTable** 方法创建表。创建表时，可以设置表服务超时。（有关表服务超时的详细信息，请参阅[为表服务操作设置超时][]。）

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // 创建表 REST 代理。
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    try {
    // 创建表。
    $tableRestProxy->createTable("mytable");
    }
    catch(ServiceException $e){
    $code = $e->getCode();
    $error_message = $e->getMessage();
    // 基于错误代码和消息处理异常。
    // 可以在以下位置找到错误代码和消息： 
    // http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
    }

有关表名称的限制的信息，请参阅[了解表服务数据模型][]。

## 如何：将实体添加到表

若要将实体添加到表，请创建一个新的 **Entity** 对象并将其传递到 **TableRestProxy-\>insertEntity**。请注意，在创建实体时，你必须指定 `PartitionKey` 和 `RowKey`。这些值是实体的唯一标识符，并且其查询速度比其他实体属性的查询速度快得多。系统使用 `PartitionKey` 自动将表的实体分发到多个存储节点上。具有相同的 `PartitionKey` 的实体存储在同一个节点上。（对存储在同一节点上的多个实体执行操作要将比对存储在不同节点上的实体执行的操作的效果更佳。）`RowKey` 是实体在分区中的唯一 ID。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\Table\Models\Entity;
    use WindowsAzure\Table\Models\EdmType;

    // 创建表 REST 代理。
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    $entity = new Entity();
    $entity->setPartitionKey("tasksSeattle");
    $entity->setRowKey("1");
    $entity->addProperty("Description", null, "Take out the trash.");
    $entity->addProperty("DueDate", 
    EdmType::DATETIME, 
    new DateTime("2012-11-05T08:15:00-08:00"));
    $entity->addProperty("Location", EdmType::STRING, "Home");

    try {
    $tableRestProxy->insertEntity("mytable", $entity);
    }
    catch(ServiceException $e){
    // 基于错误代码和消息处理异常。
    // 错误代码和消息位于以下位置： 
    // http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    }

有关表属性和类型的信息，请参阅[了解表服务数据模型][]。

**TableRestProxy** 类提供了用于插入实体的两个替代方法：**insertOrMergeEntity** 和 **insertOrReplaceEntity**。若要使用这些方法，请创建一个新的 **Entity**，并将其作为参数传递到上述任一方法。如果实体不存在，则每种方法都将插入实体。在实体已存在的情况下，如果属性已存在，则 **insertOrMergeEntity** 将更新属性值；如果属性不存在，则该方法将添加新属性，而 **insertOrReplaceEntity** 将完全替换现有实体。下面的示例演示如何使用 **insertOrMergeEntity**。如果 `PartitionKey` 为“tasksSeattle”且 `RowKey` 为“1”的实体不存在，则将会插入该实体。但是，如果之前已插入该实体（如上面的示例所示），则将更新 `DueDate` 属性并添加 `Status` 属性。系统还将更新 `Description` 和 `Location` 属性，但使用的值实际上会使其保持不变。如果后面两个属性不是如示例中所示添加的，但已存在于目标实体上，则其现有值将保持不变。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\Table\Models\Entity;
    use WindowsAzure\Table\Models\EdmType;

    // 创建表 REST 代理。
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    //创建新实体。
    $entity = new Entity();

    // PartitionKey 和 RowKey 是必需的。
    $entity->setPartitionKey("tasksSeattle");
    $entity->setRowKey("1");

    // 如果实体存在，则将使用新值更新现有属性并
    // 添加新属性。缺少的属性不会更改。
    $entity->addProperty("Description", null, "Take out the trash.");
    $entity->addProperty("DueDate", EdmType::DATETIME, new DateTime()); // 修改了 DueDate 字段。
    $entity->addProperty("Location", EdmType::STRING, "Home");
    $entity->addProperty("Status", EdmType::STRING, "Complete"); // 添加了 Status 字段。

    try {
    // 如果调用 insertOrReplaceEntity（而不是如下所示调用 insertOrMergeEntity），
    // 则只会简单地替换 PartitionKey 为“tasksSeattle”且 RowKey 为“1”的实体。
    $tableRestProxy->insertOrMergeEntity("mytable", $entity);
    }
    catch(ServiceException $e){
    // 基于错误代码和消息处理异常。
    // 错误代码和消息位于以下位置： 
    // http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.":".$error_message."<br />";
    }
       

## 如何：检索单个实体

利用 **TableRestProxy-\>getEntity** 方法，可以通过查询实体的 `PartitionKey` 和 `RowKey` 来检索单个实体。在下面的示例中，分区键 `tasksSeattle` 和行键 `1` 将传递到 **getEntity** 方法。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // 创建表 REST 代理。
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    try {
    $result = $tableRestProxy->getEntity("mytable", "tasksSeattle", 1);
    }
    catch(ServiceException $e){
    // 基于错误代码和消息处理异常。
    // 错误代码和消息位于以下位置： 
    // http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.":".$error_message."<br />";
    }

    $entity = $result->getEntity();

    echo $entity->getPartitionKey().":".$entity->getRowKey();

## 如何：检索分区中的所有实体

实体查询是使用筛选器构造的（有关详细信息，请参阅[查询表和实体][]）。若要检索分区中的所有实体，请使用筛选器“PartitionKey eq *partition\_name*”。下面的示例演示了如何通过将筛选器传递到 **queryEntities** 方法来检索 `tasksSeattle` 分区中的所有实体。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // 创建表 REST 代理。
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    $filter = "PartitionKey eq 'tasksSeattle'";

    try {
    $result = $tableRestProxy->queryEntities("mytable", $filter);
    }
    catch(ServiceException $e){
    // 基于错误代码和消息处理异常。
    // 错误代码和消息位于以下位置： 
    // http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.":".$error_message."<br />";
    }

    $entities = $result->getEntities();

    foreach($entities as $entity){
    echo $entity->getPartitionKey().":".$entity->getRowKey()."<br />";
    }

## 如何：检索分区中的一部分实体

可以使用上一示例中使用的同一模式来检索分区中的部分实体。你检索的部分实体将由你使用的筛选器确定（有关详细信息，请参阅[查询表和实体][]）。下面的示例演示了如何使用筛选器检索具有特定的 `Location` 且 `DueDate` 早于指定日期的所有实体。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // 创建表 REST 代理。
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    $filter = "Location eq 'Office' and DueDate lt '2012-11-5'";

    try {
    $result = $tableRestProxy->queryEntities("mytable", $filter);
    }
    catch(ServiceException $e){
    // 基于错误代码和消息处理异常。
    // 错误代码和消息位于以下位置： 
    // http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.":".$error_message."<br />";
    }

    $entities = $result->getEntities();

    foreach($entities as $entity){
    echo $entity->getPartitionKey().":".$entity->getRowKey()."<br />";
    }

## 如何：检索一部分实体属性

查询可检索一部分实体属性。此方法称为*投影*，可减少带宽并提高查询性能，尤其适用于大型实体。若要指定要检索的属性，请将该属性的名称传递到 **Query-\>addSelectField** 方法。可以多次调用此方法来添加更多属性。执行 **TableRestProxy-\>queryEntities** 后，返回的实体将仅具有选定的属性。（若要返回一部分表实体，请使用上述查询中所示的筛选器。）

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\Table\Models\QueryEntitiesOptions;

    // 创建表 REST 代理。
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    $options = new QueryEntitiesOptions();
    $options->addSelectField("Description");

    try {
    $result = $tableRestProxy->queryEntities("mytable", $options);
    }
    catch(ServiceException $e){
    // 基于错误代码和消息处理异常。
    // 错误代码和消息位于以下位置： 
    // http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.":".$error_message."<br />";
    }

    // 将返回表中的所有实体，无论它们 
    // 是否具有 Description 字段。
    // 若要限制返回的结果，请使用筛选器。
    $entities = $result->getEntities();

    foreach($entities as $entity){
    $description = $entity->getProperty("Description")->getValue();
    echo $description."<br />";
    }

## 如何：更新实体

可通过对现有实体使用 **Entity-\>setProperty** 和 **Entity-\>addProperty** 方法并调用 **TableRestProxy-\>updateEntity** 来更新该实体。下面的示例将检索一个实体、修改一个属性、删除另一个属性并添加一个新属性。请注意，通过将属性的值设为 **null** 可删除该属性。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\Table\Models\Entity;
    use WindowsAzure\Table\Models\EdmType;

    // 创建表 REST 代理。
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    $result = $tableRestProxy->getEntity("mytable", "tasksSeattle", 1);

    $entity = $result->getEntity();

    $entity->setPropertyValue("DueDate", new DateTime()); //修改了 DueDate。

    $entity->setPropertyValue("Location", null); //删除了 Location。

    $entity->addProperty("Status", EdmType::STRING, "In progress"); //添加了 Status。

    try {
    $tableRestProxy->updateEntity("mytable", $entity);
    }
    catch(ServiceException $e){
    // 基于错误代码和消息处理异常。
    // 错误代码和消息位于以下位置： 
    // http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.":".$error_message."<br />";
    }

## 如何：删除实体

若要删除实体，请将表名以及实体的 `PartitionKey` 和 `RowKey` 传递到 **TableRestProxy-\>deleteEntity** 方法。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // 创建表 REST 代理。
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    try {
    // 删除实体。
    $tableRestProxy->deleteEntity("mytable", "tasksSeattle", "2");
    }
    catch(ServiceException $e){
    // 基于错误代码和消息处理异常。
    // 错误代码和消息位于以下位置： 
    // http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.":".$error_message."<br />";
    }

请注意，为了进行并发检查，可以使用 **DeleteEntityOptions-\>setEtag** 方法并将 **DeleteEntityOptions** 对象作为第四个参数传递到 **deleteEntity**，来为要删除的实体设置 Etag。

## 如何：对表操作进行批处理

利用 **TableRestProxy-\>batch** 方法，你可以通过一个请求执行多个操作。此处的模式涉及将操作添加到 **BatchRequest** 对象，然后将 **BatchRequest** 对象传递到 **TableRestProxy-\>batch** 方法。若要将操作添加到 **BatchRequest** 对象，可以多次调用以下任一方法：

-   **addInsertEntity**（添加 insertEntity 操作）
-   **addUpdateEntity**（添加 updateEntity 操作）
-   **addMergeEntity**（添加 mergeEntity 操作）
-   **addInsertOrReplaceEntity**（添加 insertOrReplaceEntity 操作）
-   **addInsertOrMergeEntity**（添加 insertOrMergeEntity 操作）
-   **addDeleteEntity**（添加 deleteEntity 操作）

下面的示例演示了如何通过单个请求执行 **insertEntity** 和 **deleteEntity** 操作：

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;
    use WindowsAzure\Table\Models\Entity;
    use WindowsAzure\Table\Models\EdmType;
    use WindowsAzure\Table\Models\BatchOperations;

    // 创建表 REST 代理。
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    // 创建批处理操作列表。
    $operations = new BatchOperations();

    $entity1 = new Entity();
    $entity1->setPartitionKey("tasksSeattle");
    $entity1->setRowKey("2");
    $entity1->addProperty("Description", null, "Clean roof gutters.");
    $entity1->addProperty("DueDate", 
    EdmType::DATETIME, 
    new DateTime("2012-11-05T08:15:00-08:00"));
    $entity1->addProperty("Location", EdmType::STRING, "Home");

    // 将操作添加到批处理操作列表中。
    $operations->addInsertEntity("mytable", $entity1);

    // 将操作添加到批处理操作列表中。
    $operations->addDeleteEntity("mytable", "tasksSeattle", "1");

    try {
    $tableRestProxy->batch($operations);
    }
    catch(ServiceException $e){
    // 基于错误代码和消息处理异常。
    // 错误代码和消息位于以下位置： 
    // http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.":".$error_message."<br />";
    }

有关对表操作进行批处理的详细信息，请参阅[执行实体组事务][]。

## 如何：删除表

最后，若要删除表，请将表名传递到 **TableRestProxy-\>deleteTable** 方法。

    require_once 'vendor\autoload.php';

    use WindowsAzure\Common\ServicesBuilder;
    use WindowsAzure\Common\ServiceException;

    // 创建表 REST 代理。
    $tableRestProxy = ServicesBuilder::getInstance()->createTableService($connectionString);

    try {
    // 删除表。
    $tableRestProxy->deleteTable("mytable");
    }
    catch(ServiceException $e){
    // 基于错误代码和消息处理异常。
    // 错误代码和消息位于以下位置： 
    // http://msdn.microsoft.com/zh-cn/library/azure/dd179438.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.":".$error_message."<br />";
    }

## 后续步骤

现在，你已了解了 Azure 表服务的基础知识，单击下面的链接可了解如何执行更复杂的存储任务。

-   查看 MSDN 参考：[在 Azure 中存储和访问数据][]
-   访问 Azure 存储空间团队博客：<http://blogs.msdn.com/b/windowsazurestorage/>

  [Azure SDK for PHP]: http://go.microsoft.com/fwlink/?LinkID=252473
  [后续步骤]: #NextSteps
  [什么是表服务？]: #what-is
  [概念]: #concepts
  [创建 Azure 存储帐户]: #CreateAccount
  [创建 PHP 应用程序]: #CreateApplication
  [配置你的应用程序以访问表服务]: #ConfigureStorage
  [设置 Azure 存储连接]: #ConnectionString
  [如何：创建表]: #CreateTable
  [如何：将实体添加到表]: #AddEntity
  [如何：检索单个实体]: #RetrieveEntity
  [如何：检索分区中的所有实体]: #RetEntitiesInPartition
  [如何：检索分区中的一部分实体]: #RetrieveSubset
  [如何：检索一部分实体属性]: #RetPropertiesSubset
  [如何：更新实体]: #UpdateEntity
  [如何：对表操作进行批处理]: #BatchOperations
  [如何：删除表]: #DeleteTable
  [howto-table-storage]: ../includes/howto-table-storage.md
  [create-storage-account]: ../includes/create-storage-account.md
  [get-client-libraries]: ../includes/get-client-libraries.md
  [require\_once]: http://php.net/require_once
  [为表服务操作设置超时]: http://msdn.microsoft.com/zh-cn/library/azure/dd894042.aspx
  [了解表服务数据模型]: http://msdn.microsoft.com/zh-cn/library/azure/dd179338.aspx
  [查询表和实体]: http://msdn.microsoft.com/zh-cn/library/azure/dd894031.aspx
  [执行实体组事务]: http://msdn.microsoft.com/zh-cn/library/azure/dd894038.aspx
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
