<properties linkid="dev-nodejs-how-to-table-services" urlDisplayName="Table Service" pageTitle="如何使用表存储 (Node.js) | Microsoft Azure" metaKeywords="Azure table storage service, Azure table service Node.js, table storage Node.js" description="了解如何在 Azure 中使用表存储服务。代码示例使用 Node.js API 编写。" metaCanonical="" services="storage" documentationCenter="Node.js" title="How to Use the Table Service from Node.js" authors="larryfr" solutions="" manager="" editor="" />






# 如何从 Node.js 使用表服务

本指南将演示如何使用 Microsoft Azure 队列服务 Azure 表服务。相关示例是使用 Node.js API 编写的。涉及的方案包括**创建和删除表、插入和查询表中的实体**。有关表的详细信息，请参阅[后续步骤][]部分。

## 目录

* [什么是表服务？][]   
* [概念][]   
* [创建 Azure 存储帐户](#create-account)
* [创建 Node.js 应用程序](#create-app)
* [配置应用程序以访问存储](#configure-access)
* [设置 Azure 存储连接](#setup-connection-string)  
* [如何：创建表](#create-table)
* [如何：向表中添加实体](#add-entity)
* [如何：更新实体](#update-entity)
* [如何：使用实体组](#change-entities)
* [如何：检索实体](#query-for-entity)
* [如何：查询实体集](#query-set-entities)
* [如何：删除实体](#delete-entity)
* [如何：删除表](#delete-table)   
* [如何：使用共享访问签名](#sas)
* [后续步骤][]

[WACOM.INCLUDE [howto-table-storage](../includes/howto-table-storage.md)]

<h2><a name="create-account"></a>创建 Azure 存储帐户</h2>

[WACOM.INCLUDE [create-storage-account](../includes/create-storage-account.md)]

## <a name="create-app"> </a>创建 Node.js 应用程序

创建一个空的 Node.js 应用程序。有关创建 Node.js 应用程序的说明，请参阅 [创建 Node.js 应用程序并将其部署到 Azure 网站]、[Node.js 云服务][Node.js 云服务]（使用 Windows PowerShell）或 [使用 WebMatrix 构建网站]。

## <a name="configure-access"> </a>配置应用程序以访问存储

若要使用 Azure 存储空间，你需要 Azure Storage SDK for Node.js，其中包括一组
便于与存储 REST 服务进行通信的库。

### 使用 Node 包管理器 (NPM) 可获取该程序包

1.  使用 PowerShell (Windows)、Terminal (Mac) 或 Bash (Unix) 等命令行界面导航到你在其中创建了示例应用程序的文件夹。

2.  在命令窗口中键入 npm install azure-storage，这应该产生以下输出：

        azure-storage@0.1.0 node_modules\azure-storage
		├── extend@1.2.1
		├── xmlbuilder@0.4.3
		├── mime@1.2.11
		├── underscore@1.4.4
		├── validator@3.1.0
		├── node-uuid@1.4.1
		├── xml2js@0.2.7 (sax@0.5.2)
		└── request@2.27.0 (json-stringify-safe@5.0.0, tunnel-agent@0.3.0, aws-sign@0.3.0, forever-agent@0.5.2, qs@0.6.6, oauth-sign@0.3.0, cookie-jar@0.3.0, hawk@1.0.0, form-data@0.1.3, http-signature@0.10.0)

3.  可以手动运行 **ls** 命令来验证是否创建了
    **node\_modules** 文件夹。在该文件夹中，你会
    找到 **azure-storage** 包，其中包含你访问
    存储所需的库。

### 导入包

使用记事本或其他文本编辑器将以下内容添加到应用程序的
**server.js** 文件的顶部，以便在其中使用存储：

    var azure = require('azure-storage');

## <a name="setup-connection-string"> </a>设置 Azure 存储连接

Azure 模块将读取环境变量 AZURE\_STORAGE\_ACCOUNT 和 AZURE\_STORAGE\_ACCESS\_KEY 或 AZURE\_STORAGE\_CONNECTION\_STRING，以便获取连接到 Azure 存储帐户所需的信息。如果未设置这些环境变量，则在调用 TableService 时必须指定帐户信息。

有关在管理门户中为 Azure 网站设置环境变量的示例，请参阅[使用存储构建 Node.js Web 应用程序]

## <a name="create-table"> </a>如何创建表

以下代码将创建一个 TableService 对象并使用它
创建新表。将以下内容添加到 server.js 顶部附近。

    var tableSvc = azure.createTableService();

调用 **createTableIfNotExists** 将创建具有指定名称的新表，前提是该名称
不存在。下面的示例将创建一个名为  'mytable' 的新表（如果该表尚不存在）：

    tableSvc.createTableIfNotExists('mytable', function(error, result, response){
		if(!error){
			// Table exists or created
		}
	});

 `result` 将为  `true`（如果创建了新表），或者为  `false`（如果表已存在）。 `response` 将包含有关该请求的信息。

###筛选器

可以向使用 TableService 执行的操作应用可选的筛选操作。筛选操作可以包括日志记录、自动重试等。筛选器是实现了具有签名的方法的对象：

		function handle (requestOptions, next)

在对请求选项执行预处理后，该方法需要调用"next"并且传递具有以下签名的回调：

		function (returnObject, finalCallback, next)

在此回调中并且在处理 returnObject（来自对服务器请求的响应）后，回调需要调用 next（如果它存在以便继续处理其他筛选器）或只调用 finalCallback 以便结束服务调用。

Azure SDK for Node.js 中附带了两个实现了重试逻辑的筛选器，分别是 ExponentialRetryPolicyFilter 和 LinearRetryPolicyFilter。下面的代码将创建一个 TableService 对象，该对象使用 ExponentialRetryPolicyFilter：

	var retryOperations = new azure.ExponentialRetryPolicyFilter();
	var tableSvc = azure.createTableService().withFilter(retryOperations);

## <a name="add-entity"> </a>如何向表中添加实体

若要添加实体，应首先创建一个定义了你的实体
属性的哈希对象。所有实体都必须包含 **PartitionKey** 和 **RowKey**，即实体的唯一标识符。

* **PartitionKey** -确定实体存储在其中的分区。

* **RowKey** -唯一标识分区内的实体。

**PartitionKey** 和 **RowKey** 都必须是字符串值。有关详细信息，请参阅 [了解表服务数据模型](http://msdn.microsoft.com/library/azure/dd179338.aspx).

下面是如何定义实体的示例。请注意，**dueDate** 被定义为一种类型的 **Edm.DateTime**。可以选择性地指定类型。如果未指定类型，系统会进行推断。

	var task = { 
	  PartitionKey: {'_':'hometasks'},
	  RowKey: {'_': '1'},
	  description: {'_':'take out the trash'},
	  dueDate: {'_':new Date(2015, 6, 20), '$':'Edm.DateTime'}
	};

> [WACOM.NOTE] 每个记录还有一个**"时间戳"**字段，在插入或更新实体时，Azure 会设置该字段。

你还可以使用 **entityGenerator** 来创建实体。下面的示例使用 **entityGenerator** 来创建相同的任务实体。

	var entGen = azure.TableUtilities.entityGenerator;
    var task = {
	  PartitionKey: entGen.String('hometasks'),
      RowKey: entGen.String('1'),
      description: entGen.String('take out the trash'),
      dueDate: entGen.DateTime(new Date(Date.UTC(2015, 6, 20))),
    };

若要向表中添加实体，应将实体对象传递给
**insertEntity** 方法。

	tableSvc.insertEntity('mytable',task, function (error, result, response) {
		if(!error){
			// Entity inserted
		}
	});

如果操作成功， `result` 将包含 [ETag](http://en.wikipedia.org/wiki/HTTP_ETag) of the inserted record and `response` will contain information about the operation.

> [WACOM.NOTE] 默认情况下，**insertEntity** 不会在  `response` 信息中返回插入的实体。如果你计划对此实体执行其他操作，或者希望对信息进行缓存，则可在  `result` 中返回该实体。你可以通过启用 **echoContent** 来执行此操作，如下所示：
>
> `tableSvc.insertEntity('mytable', task, {echoContent: true}, function (error, result, response) {...}`

## <a name="update-entity"> </a>如何更新实体

可使用多种方法来更新现有实体：

* updateEntity - 通过替换现有实体来更新现有实体。

* mergeEntity - 通过将新属性值合并到现有实体来更新现有实体。

* insertOrReplaceEntity - 通过替换现有实体来更新现有实体。如果不存在实体，将插入一个新实体。

* insertOrMergeEntity - 通过将新属性值合并到现有实体来更新现有实体。如果不存在实体，将插入一个新实体。

以下示例演示了使用 updateEntity 更新实体：

	tableSvc.updateEntity('mytable', updatedTask, function(error, result, response){
      if(!error) {
        // Entity updated
      }
    });

> [WACOM.NOTE] 默认情况下，更新某个实体时，不会查看要更新的数据是否曾被其他进程更新过。若要支持并发更新，请执行以下步骤：
> 
> 1.获取要更新的对象的 ETag。对于任何实体相关操作，该 ETag 将在  `response` 中返回，并且可通过  `response['.metadata'].etag` 检索。
> 
> 2.对某个实体执行更新操作时，请将以前检索的 ETag 信息添加到新的实体。例如：
> 
>     `entity2['.metadata'].etag = currentEtag;`
>    
> 3.执行更新操作。如果实体在你检索 ETag 值后已被修改，例如被应用程序的其他实例修改，则会返回一条  `error`，指出未满足请求中指定的更新条件。
    
对于 updateEntity 和 mergeEntity，如果待更新的实体不存在，则更新操作将失败。因此，如果你希望存储某个实体而不考虑它是否已存在，则应改用 insertOrReplaceEntity 或 insertOrMergeEntity。

如果更新操作成功，则  `result` 会包含所更新实体的 **Etag**。

## <a name="change-entities"> </a>如何操作实体组

有时候，有必要将多个操作成批
提交，确保服务器对其进行原子处理。为此，你可以
使用 **TableBatch** 类来创建一个批处理，然后使用 **TableService** 的 **executeBatch** 方法来执行批处理操作。

 下面的示例演示了在一个批次中提交两个实体：

    var task1 = { 
	  PartitionKey: {'_':'hometasks'},
	  RowKey: {'_': '1'},
	  description: {'_':'Take out the trash'},
	  dueDate: {'_':new Date(2015, 6, 20)}
	};
	var task2 = { 
	  PartitionKey: {'_':'hometasks'},
	  RowKey: {'_': '2'},
	  description: {'_':'Wash the dishes'},
	  dueDate: {'_':new Date(2015, 6, 20)}
	};

	var batch = new azure.TableBatch();
	
	batch.insertEntity(task1, {echoContent: true});
	batch.insertEntity(task2, {echoContent: true});

	tableSvc.executeBatch('mytable', batch, function (error, result, response) {
	  if(!error) {
	    // Batch completed
	  }
	});

对于成功的批处理操作， `result` 将包含批处理中每个操作的信息。

###使用批处理操作

可以通过查看  `operations` 属性来检查添加到批处理中的操作。你可以使用以下方法来处理操作。

* **clear** - 清除批处理中的所有操作。

* **getOperations** - 获取批处理的操作。

* **hasOperations** - 如果批处理包含操作，则返回 true。

* **removeOperations** - 删除操作。

* **size** - 返回批处理中操作的数目。

## <a name="query-for-entity"> </a>如何检索实体

如果你想要返回基于 **PartitionKey** 和 **RowKey** 的特定实体，请使用 **retrieveEntity** 方法。

    tableSvc.retrieveEntity('mytable', 'hometasks', '1', function(error, result, response){
	  if(!error){
	    // result contains the entity
	  }
    });

此操作完成后， `result` 将包含该实体。

## <a name="query-set-entities"> </a>如何查询实体集

若要查询表，请使用 **TableQuery** 对象生成一个查询
表达式，只需使用以下子句即可：

* **select** - 将要从查询返回的字段。

* **where** - where 子句。

	* **and** - 一个  `and` where 条件。

	* **or** - 一个  `or` where 条件。

* **top** - 要提取的项的数目。


下面的示例生成的查询将返回 PartitionKey 为  'hometasks' 的前 5 项。

	var query = new azure.TableQuery()
	  .top(5)
	  .where('PartitionKey eq ?', 'hometasks');

由于未使用 **select**，因此将返回所有字段。若要对表执行查询，请使用 **queryEntities**。下面的示例使用此查询来返回  'mytable' 中的实体。

	tableSvc.queryEntities('mytable',query, null, function(error, result, response) {
	  if(!error) {
	    // query was successful
	  }
	});

如果成功， `result.entries` 将包含与查询匹配的一组实体。如果查询无法返回所有实体， `result.continuationToken` 就不会是 *null*，因此可用作 **queryEntities** 的第三个参数来检索更多结果。对于初始查询，第三个参数应为  *null*。

###如何查询实体属性子集

对表的查询可以只检索实体中的少数几个字段。
这可以减少带宽并提高查询性能，尤其适用于大型实体。使用 **select** 子句并传递要返回的字段的名称。例如，下面的查询将只返回**"说明"**和 **dueDate** 字段。

	var query = new azure.TableQuery()
	  .select(['description', 'dueDate'])
	  .top(5)
	  .where('PartitionKey eq ?', 'hometasks');

## <a name="delete-entity"> </a>如何删除实体

可以使用实体的分区键和行键删除实体。在此
示例中，**task1** 对象包含要删除的实体的 **RowKey** 和
**PartitionKey** 值。然后，该对象将
传递给 **deleteEntity** 方法。

	var task = { 
	  PartitionKey: {'_':'hometasks'},
	  RowKey: {'_': '1'}
	};

    tableSvc.deleteEntity('mytable', task, function(error, response){
	  if(!error) {
		// Entity deleted
	  }
	});

> [WACOM.NOTE] 你应该考虑在删除项时使用 ETag，以确保项尚未被其他进程修改。请参阅 [如何：更新实体][]，了解如何使用 ETag。

## <a name="delete-table"> </a>如何删除表

以下代码从存储帐户中删除一个表。

    tableSvc.deleteTable('mytable', function(error, response){
		if(!error){
			// Table deleted
		}
	});

如果你不确定表是否存在，则使用 **deleteTableIfExists**。

## <a name="sas"></a>如何：使用共享访问签名

共享访问签名 (SAS) 是一种安全的方法，用于对表进行细致访问而无需提供你的存储帐户名或密钥。通常使用 SAS 来提供对你的数据的有限访问权限，例如允许移动应用程序查询记录。

受信任的应用程序（例如基于云的服务）可使用 **TableService** 的 **generateSharedAccessSignature** 生成 SAS，然后将其提供给不受信任的或不完全受信任的应用程序。例如，移动应用程序。SAS 可使用策略生成，该策略描述了 SAS 的生效日期和失效日期，以及授予 SAS 持有者的访问级别。

下面的示例生成了一个新的共享访问策略，该策略将允许 SAS 持有者查询 ('r') 表，在创建后 100 分钟过期。

	var startDate = new Date();
	var expiryDate = new Date(startDate);
	expiryDate.setMinutes(startDate.getMinutes() + 100);
	startDate.setMinutes(startDate.getMinutes() - 100);
		
	var sharedAccessPolicy = {
	  AccessPolicy: {
	    Permissions: azure.TableUtilities.SharedAccessPermissions.QUERY,
	    Start: startDate,
	    Expiry: expiryDate
	  },
	};

	var tableSAS = tableSvc.generateSharedAccessSignature('mytable', sharedAccessPolicy);
	var host = tableSvc.host;

请注意，还必须提供主机信息，因为 SAS 持有者尝试访问表时，必须提供该信息。

然后，客户端应用程序将 SAS 用于 **TableServiceWithSAS**，以便针对表执行操作。下面的示例连接到该表，并执行一个查询。

	var sharedTableService = azure.createTableServiceWithSas(host, tableSAS);
	var query = azure.TableQuery()
	  .where('PartitionKey eq ?', 'hometasks');
		
	sharedTableService.queryEntities(query, null, function(error, result, response) {
	  if(!error) {
		// result contains the entities
	  }
	});

由于 SAS 在生成时只具有查询访问权限，因此如果尝试插入、更新或删除实体，则会返回错误。

###访问控制列表

你还可以使用访问控制列表 (ACL) 为 SAS 设置访问策略。如果你希望允许多个客户端访问某个表，但为每个客户端提供了不同的访问策略，则访问控制列表会很有用。

ACL 是使用一组访问策略实施的，每个策略都有一个关联的 ID。下面的示例定义了两个策略，一个用于"user1"，一个用于"user2"：

	var sharedAccessPolicy = [
	  {
	    AccessPolicy: {
	      Permissions: azure.TableUtilities.SharedAccessPermissions.QUERY,
	      Start: startDate,
	      Expiry: expiryDate
	    },
	    Id: 'user1'
	  },
	  {
	    AccessPolicy: {
	      Permissions: azure.TableUtilities.SharedAccessPermissions.ADD,
	      Start: startDate,
	      Expiry: expiryDate
	    },
	    Id: 'user2'
	  }
	];

下面的示例获取 **hometasks** 的当前 ACL，，然后使用 **setTableAcl** 添加新策略。此方法具有以下用途：

	tableSvc.getTableAcl('hometasks', function(error, result, response) {
      if(!error){
		//push the new policy into signedIdentifiers
		result.signedIdentifiers.push(sharedAccessPolicy);
		tableSvc.setTableAcl('hometasks', result, function(error, result, response){
	  	  if(!error){
	    	// ACL set
	  	  }
		});
	  }
	});

设置 ACL 后，你可以根据某个策略的 ID 创建 SAS。以下示例为  'user2': 创建新的 SAS

	tableSAS = tableSvc.generateSharedAccessSignature('hometasks', { Id: 'user2' });

## <a name="next-steps"> </a>后续步骤

现在，你已了解表存储的基础知识，单击下面的链接
可了解如何执行更复杂的存储任务。

-   查看 MSDN 参考：[在 Azure 中存储和访问数据][]。
-   [访问 Azure 存储空间团队博客][]。
-   访问 GitHub 上的 [Azure Storage SDK for Node][] 存储库。

  [Azure Storage SDK for Node]: https://github.com/Azure/azure-storage-node
  [后续步骤]: #next-steps
  [什么是表服务？]: #what-is
  [概念]: #concepts
  [创建 Azure 存储帐户]: #create-account
  [创建 Node.js 应用程序]: #create-app
  [配置应用程序以访问存储]: #configure-access
  [设置 Azure 存储连接]: #setup-connection-string
  [如何：创建表]: #create-table
  [如何：向表中添加实体]: #add-entity
  [如何：更新实体]: #update-entity
  [如何：使用实体组]: #change-entities
  [如何：查询实体]: #query-for-entity
  [如何：查询实体集]: #query-set-entities
  [如何：查询一部分实体属性]: #query-entity-properties
  [如何：删除实体]: #delete-entity
  [如何：删除表]: #delete-table

  [OData.org]: http://www.odata.org/
  [使用 REST API]: http://msdn.microsoft.com/zh-cn/library/azure/hh264518.aspx
  [Azure 管理门户]: http://manage.windowsazure.cn

  [Node.js 云服务]: /zh-cn/documentation/articles/cloud-services-nodejs-develop-deploy-app/
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [访问 Azure 存储空间团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
  [使用 WebMatrix 构建网站]: /zh-cn/documentation/articles/web-sites-nodejs-use-webmatrix/
  [使用存储构建 Node.js 云服务]: /zh-cn/documentation/articles/storage-nodejs-use-table-storage-cloud-service-app/
  [使用存储构建 Node.js Web 应用程序]: /zh-cn/documentation/articles/storage-nodejs-use-table-storage-web-site/
  [创建 Node.js 应用程序并将其部署到 Azure 网站]: /zh-cn/documentation/articles/web-sites-nodejs-develop-deploy-mac/
<!--HONumber=41-->
