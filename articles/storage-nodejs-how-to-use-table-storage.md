<properties linkid="dev-nodejs-how-to-table-services" urlDisplayName="Table Service" pageTitle="How to use table storage (Node.js) | Microsoft Azure" metaKeywords="Azure table storage service, Azure table service Node.js, table storage Node.js" description="Learn how to use the table storage service in Azure. Code samples are written using the Node.js API." metaCanonical="" services="storage" documentationCenter="Node.js" title="How to Use the Table Service from Node.js" authors="larryfr" solutions="" manager="" editor="" />

# 如何从 Node.js 使用表服务

本指南将演示如何使用 Windows Azure 表服务执行常见方案。
示例是用 Node.js API 编写的。
涉及的方案包括**创建和删除表、在表中插入和查询实体**。
有关表的详细信息，请参阅
[后续步骤][]部分。

## 目录

-   [什么是表服务？][]
-   [概念][]
-   [创建 Azure 存储帐户][]
-   [创建 Node.js 应用程序][]
-   [配置应用程序以访问存储][]
-   [设置 Azure 存储连接][]
-   [如何：创建表][]
-   [如何：向表中添加实体][]
-   [如何：更新实体][]
-   [如何：使用实体组][]
-   [如何：查询实体][]
-   [如何：查询实体集][]
-   [如何：查询一部分实体属性][]
-   [如何：删除实体][]
-   [如何：删除表][]
-   [后续步骤][]

[WACOM.INCLUDE [howto-table-storage][]]

## 创建帐户创建 Azure 存储帐户

[WACOM.INCLUDE [create-storage-account][]]

## 创建 Node.js 应用程序

创建一个空的 Node.js 应用程序。有关创建 Node.js 应用程序的说明，请参阅[创建 Node.js 应用程序并将其部署到 Azure 网站][]、[Node.js 云服务][]（使用 Windows PowerShell）或[使用 WebMatrix 构建网站][]。

## 配置应用程序以访问存储

若要使用 Azure 存储空间，你需要下载并使用 Node.js azure 包，
其中包括一组便于与存储 REST 服务
进行通信的库。

### 使用 Node 包管理器 (NPM) 可获取该程序包

1.  使用 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (Unix) 等命令行界面导航到你在其中创建了示例应用程序的文件夹。

2.  在命令窗口中键入 **npm install azure**，这应该产生
    以下输出：

        azure@0.7.5 node_modules\azure
        ├── dateformat@1.0.2-1.2.3
        ├── xmlbuilder@0.4.2
        ├── node-uuid@1.2.0
        ├── mime@1.2.9
        ├── underscore@1.4.4
        ├── validator@1.1.1
        ├── tunnel@0.0.2
        ├── wns@0.5.3
        ├── xml2js@0.2.7 (sax@0.5.2)
        └── request@2.21.0 (json-stringify-safe@4.0.0, forever-agent@0.5.0, aws-sign@0.3.0, tunnel-agent@0.3.0, oauth-sign@0.3.0, qs@0.6.5, cookie-jar@0.3.0, node-uuid@1.4.0, http-signature@0.9.11, form-data@0.0.8, hawk@0.13.1)

3.  可以手动运行 **ls** 命令来验证是否创建了
    **node\_modules** 文件夹。在该文件夹中，你将
    找到 **azure** 包，其中包含你访问存储
    所需的库。

### 导入包

使用记事本或其他文本编辑器将以下内容添加到你要在其中使用存储的
应用程序的 **server.js** 文件的顶部：

    var azure = require('azure');

## 设置 Azure 存储连接

azure 模块将读取环境变量 AZURE\_STORAGE\_ACCOUNT 和 AZURE\_STORAGE\_ACCESS\_KEY 以获取连接到你的 Azure 存储帐户所需的信息。如果未设置这些环境变量，则在调用 **TableService** 时必须指定帐户信息。

有关在 Azure 云服务的配置文件中设置环境变量的示例，请参阅[使用存储构建 Node.js 云服务][]。

有关在管理门户中为 Azure 网站设置环境变量的示例，请参阅[使用存储构建 Node.js Web 应用程序][]。

## 如何创建表

以下代码将创建一个 **TableService** 对象并使用它
创建新表。将以下内容添加到 **server.js** 顶部附近。

    var tableService = azure.createTableService();

如果指定的表已存在，则调用 **createTableIfNotExists** 将
返回指定的表，如果不存在，则使用指定的名称创建
一个新表。下面的示例将创建一个名为“mytable”的新表（如果该表尚不存在）：

    tableService.createTableIfNotExists('mytable', function(error){
    if(!error){
    // 表存在或已创建
        }
    });

### 筛选器

可以向使用 **TableService** 执行的操作应用可选的筛选操作。筛选操作可以包括日志记录、自动重试等。筛选器是实现了具有签名的方法的对象：

        function handle (requestOptions, next)

在对请求选项执行预处理后，该方法需要调用“next”并且传递具有以下签名的回调：

        function (returnObject, finalCallback, next)

在此回调中并且在处理 returnObject（来自对服务器请求的响应）后，回调需要调用 next（如果它存在以便继续处理其他筛选器）或只调用 finalCallback 以便结束服务调用。

Azure SDK for Node.js 中附带了两个实现了重试逻辑的筛选器，分别是 **ExponentialRetryPolicyFilter** 和 **LinearRetryPolicyFilter**。下面的代码将创建一个 **TableService** 对象，该对象使用 **ExponentialRetryPolicyFilter**：

    var retryOperations = new azure.ExponentialRetryPolicyFilter();
    var tableService = azure.createTableService().withFilter(retryOperations);

## 如何向表中添加实体

若要添加实体，应先创建一个定义你的实体属性及其数据类型的
对象。请注意，对于每个实体，你必须指定
**PartitionKey** 和 **RowKey**。这些值是你的实体的
唯一标识符，并且查询它们比查询其他属性快很多。
系统使用 **PartitionKey** 自动将表的实体
分发到许多存储节点上。具有相同 **PartitionKey** 的
实体存储在相同的节点上。
**RowKey** 是实体在其所属分区内的唯一 ID。
若要将实体添加到表中，应将实体对象传递
给 **insertEntity** 方法。

    var task = {
    PartitionKey :'hometasks'
    , RowKey : '1'
    , Description :'Take out the trash'
    , DueDate:new Date(2012, 6, 20)
    };
    tableService.insertEntity('mytable', task, function(error){
    if(!error){
    // 实体已插入
        }
    });

## 如何更新实体

可使用多种方法来更新现有实体：

-   **updateEntity** - 通过替换现有实体来更新现有实体。

-   **mergeEntity** - 通过将新属性值合并到现有实体来更新现有实体。

-   **insertOrReplaceEntity** - 通过替换现有实体来更新现有实体。如果不存在实体，将插入一个新实体。

-   **insertOrMergeEntity** - 通过将新属性值合并到现有实体来更新现有实体。如果不存在实体，将插入一个新实体。

以下示例演示了使用 **updateEntity** 更新实体：

    var task = {
    PartitionKey :'hometasks'
    , RowKey : '1'
    , Description :'Wash Dishes'
    }
    tableService.updateEntity('mytable', task, function(error){
    if(!error){
    // 实体已更新
        }
    });

对于 **updateEntity** 和 **mergeEntity**，如果待更新的实体不存在，则更新操作将失败。因此，如果你希望存储某个实体而不考虑它是否已存在，则应改用 **insertOrReplaceEntity** 或 **insertOrMergeEntity**。

## 如何操作实体组

有时，有必要成批地同时提交多项操作以确保通过服务器进行原子处理。
若要完成此操作，可以对 **TableService** 使用 **beginBatch** 方法，
然后像通常一样调用操作系列。
不同之处在于，这些运算符的回调
函数将指明该操作已经过批处理，
不提交到服务器。在你确实要按批提交时，则应
调用 **commitBatch**。提供给该方法的回调将指示整个批次
是否成功提交。下面的示例演示了在一个批次中提交两个实体：

    var tasks=[
        {
    PartitionKey :'hometasks'
    , RowKey : '1'
    , Description :'Take out the trash.'
    , DueDate:new Date(2012, 6, 20)
        }
        , {
    PartitionKey :'hometasks'
    , RowKey : '2'
    , Description :'Wash the dishes.'
    , DueDate:new Date(2012, 6, 20)
        }
    ]
    tableService.beginBatch();
    var async=require('async');

    async.forEach(tasks
    , function taskIterator(task, callback){
    tableService.insertEntity('mytable', task, function(error){
    if(!error){
    // 实体已插入
    callback(null);
    } else {
    callback(error);
                }
            });
        }
    , function(error){
    if(!error){
    // 所有插入已完成
    tableService.commitBatch(function(error){
    if(!error){
    // 批已成功提交
                    }
                });
            }
        });

**说明**

上面的示例使用“异步”模块以确保实体全都在调用\*\*commitBatch\*\*之前成功提交。

## 如何查询实体

若要查询表中的实体，请使用 **queryEntity** 方法并
传递 **PartitionKey** 和 **RowKey**。

    tableService.queryEntity('mytable'
    , 'hometasks'
        , '1'
    , function(error, entity){
    if(!error){
    // entity 包含返回的实体
            }
        });

## 如何查询实体集

若要查询表，请使用 **TableQuery** 对象通过一些子句构建查询表达式，
这些子句有 **select**、**from**、**where**（包括
**wherePartitionKey**、**whereRowKey**、
**whereNextPartitionKey** 和 **whereNextRowKey** 等便利子句）、
**and**、**or** 以及 **top**。然后将查询表达式
传递给 **queryEntities** 方法。你可以将结果用在回调内
的 **for** 循环中。

此示例基于 **PartitionKey** 查找 Seattle 中的所有任务。

    var query = azure.TableQuery
    .select()
    .from('mytable')
    .where('PartitionKey eq ?', 'hometasks');
    tableService.queryEntities(query, function(error, entities){
    if(!error){
    //entities 包含实体的数组
        }
    });

## 如何查询实体属性子集

对表的查询可以只检索实体中的少数几个属性。此方法称为**“投影”，
可减少带宽并提高查询性能，尤其适用于
大型实体。请使用 **select** 子句并
传递你希望显示给客户端的属性的名称。

以下代码中的查询只返回表中实体的**“说明”**。请注意，在程序输出中，
**DueDate** 将显示为 **undefined**，
因为服务器未发送它。

**说明**

请注意，下面的代码段只对云存储服务有效，存储模拟器不支持 **select** 关键字。

    var query = azure.TableQuery
    .select('Description')
    .from('mytable')
    .where('PartitionKey eq ?', 'hometasks');
    tableService.queryEntities(query, function(error, entities){
    if(!error){
    //entities 包含实体的数组
        }
    });

## 如何删除实体

可以使用实体的分区键和行键删除实体。在本例中，
**task1** 对象包含要删除的实体的 **RowKey** 和
**PartitionKey** 值。然后，该对象被
传递给 **deleteEntity** 方法。

    tableService.deleteEntity('mytable'
        , {
    PartitionKey :'hometasks'
    , RowKey : '1'
        }
    , function(error){
    if(!error){
    // 实体已删除
            }
        });

## 如何删除表

以下代码从存储帐户中删除一个表。

    tableService.deleteTable('mytable', function(error){
    if(!error){
    // 表已删除
        }
    });

## 后续步骤

现在，你已了解有关表存储的基础知识，可单击下面的链接来了解如何
执行更复杂的存储任务。

-   查看 MSDN 参考：[在 Azure 中存储和访问数据][]。
-   [访问 Azure 存储空间团队博客][]。
-   访问 GitHub 上的 [Azure SDK for Node][] 存储库。

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
  [howto-table-storage]: ../includes/howto-table-storage.md
  [create-storage-account]: ../includes/create-storage-account.md
  [创建 Node.js 应用程序并将其部署到 Azure 网站]: /en-us/documentation/articles/web-sites-nodejs-develop-deploy-mac/
  [Node.js 云服务]: /en-us/documentation/articles/cloud-services-nodejs-develop-deploy-app/
  [使用 WebMatrix 构建网站]: /en-us/documentation/articles/web-sites-nodejs-use-webmatrix/
  [使用存储构建 Node.js 云服务]: /en-us/documentation/articles/storage-nodejs-use-table-storage-cloud-service-app/
  [使用存储构建 Node.js Web 应用程序]: /en-us/documentation/articles/storage-nodejs-use-table-storage-web-site/
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [访问 Azure 存储空间团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
  [Azure SDK for Node]: https://github.com/WindowsAzure/azure-sdk-for-node
