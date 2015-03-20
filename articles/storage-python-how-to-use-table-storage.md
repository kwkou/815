<properties linkid="develop-python-table-service" urlDisplayName="Table Service" pageTitle="如何使用表存储 (Python) | Microsoft Azure" metaKeywords="Azure table Python, creating table Azure, deleting table Azure, inserting table Azure, querying table Azure" description="了解如何从 Python 使用表服务来创建和删除表，以及插入、删除和查询表。" metaCanonical="" services="storage" documentationCenter="Python" title="How to Use the Table Storage Service from Python" authors="" solutions="" manager="" editor="" />





# 如何从 Python 使用表存储服务
本指南将演示如何使用 Microsoft Azure 表存储服务
执行常见方案。相关示例是使用
Python API 编写的。涉及的方案包括创建和删除表、在表中插入和查询实体。有关表的详细信息，请参阅[后续步骤][]部分。

## 目录

[什么是表服务？][]   
 [概念][]   
 [创建 Azure 存储帐户][]   
 [如何：创建表][]   
 [如何：向表中添加实体][]   
 [如何：更新实体][]   
 [如何：更改实体组][]   
 [如何：查询实体][]   
 [如何：查询实体集][]   
 [如何：查询一部分实体属性][]   
 [如何：删除实体][]   
 [如何：删除表][]   
 [后续步骤][]

[WACOM.INCLUDE [howto-table-storage](../includes/howto-table-storage.md)]

## <a name="create-account"> </a>创建 Azure 存储帐户
[WACOM.INCLUDE [create-storage-account](../includes/create-storage-account.md)]

**注意：**如果你需要安装 Python 或客户端库，请参阅 [Python 安装指南](../python-how-to-install/)。


## <a name="create-table"> </a>如何创建表

可通过 TableService 对象使用表服务。以下代码创建 TableService 对象。在你希望在其中以编程方式访问 Azure 存储空间的任何 Python 文件中，将以下代码添加到文件的顶部附近：

	from azure.storage import TableService, Entity

以下代码使用存储帐户名称和帐户密钥创建 TableService 对象。使用实际帐户和密钥替换  'myaccount' 和  'mykey'。

	table_service = TableService(account_name='myaccount', account_key='mykey')

	table_service.create_table('tasktable')

## <a name="add-entity"> </a>如何向表中添加实体

若要添加实体，请首先创建定义实体属性名称和值的字典。请注意，对于每个实体，你必须指定 PartitionKey 和 RowKey。这些值是实体的唯一标识符，并且查询它们比查询其他属性快很多。系统使用 PartitionKey 自动将表的实体分发到多个存储节点上。
具有相同的 PartitionKey 的实体存储在同一个节点上。将显示
RowKey 是实体在其所属分区内的唯一 ID。

若要将实体添加到表中，请将字典对象传递给 **insert\_entity** 方法。

	task = {'PartitionKey': 'tasksSeattle', 'RowKey': '1', 'description' : 'Take out the trash', 'priority' : 200}
	table_service.insert_entity('tasktable', task)

你还可以将 **Entity** 类的实例传递给 **insert\_entity** 方法。

	task = Entity()
	task.PartitionKey = 'tasksSeattle'
	task.RowKey = '2'
	task.description = 'Wash the car'
	task.priority = 100
	table_service.insert_entity('tasktable', task)

## <a name="update-entity"> </a>如何更新实体

此代码演示如何使用更新版本替换现有实体的旧版本。

	task = {'description' : 'Take out the garbage', 'priority' : 250}
	table_service.update_entity('tasktable', 'tasksSeattle', '1', task)

如果要更新的实体不存在，更新操作将失败。如果你要存储实体，而不管它以前是否已存在，请使用 **insert\_or\_replace_entity**。 
在下面的示例中，第一次调用将替换现有实体。第二个调用将插入新实体，因为表中不存在具有指定 PartitionKey 和 RowKey 的实体。

	task = {'description' : 'Take out the garbage again', 'priority' : 250}
	table_service.insert_or_replace_entity('tasktable', 'tasksSeattle', '1', task)

	task = {'description' : 'Buy detergent', 'priority' : 300}
	table_service.insert_or_replace_entity('tasktable', 'tasksSeattle', '3', task)

## <a name="change-entities"> </a>如何更改实体组

有时，有必要成批地同时提交多项操作以确保通过服务器进行原子处理。若要实现该目的，请对 **TableService** 使用 **begin\_batch** 方法，然后像通常一样调用一系列操作。在你确实要按批提交时，可调用 **commit\_batch**。请注意，所有实体必须在相同分区中才能更改为批处理。下面的示例将两个实体一起添加到批处理中。

	task10 = {'PartitionKey': 'tasksSeattle', 'RowKey': '10', 'description' : 'Go grocery shopping', 'priority' : 400}
	task11 = {'PartitionKey': 'tasksSeattle', 'RowKey': '11', 'description' : 'Clean the bathroom', 'priority' : 100}
	table_service.begin_batch()
	table_service.insert_entity('tasktable', task10)
	table_service.insert_entity('tasktable', task11)
	table_service.commit_batch()

## <a name="query-for-entity"> </a>如何查询实体

若要查询表中的实体，请使用 **get\_entity** 方法并传递 PartitionKey 和 RowKey。

	task = table_service.get_entity('tasktable', 'tasksSeattle', '1')
	print(task.description)
	print(task.priority)

## <a name="query-set-entities"> </a>如何查询实体集

此示例基于 PartitionKey 查找 Seattle 中的所有任务。

	tasks = table_service.query_entities('tasktable', "PartitionKey eq 'tasksSeattle'")
	for task in tasks:
		print(task.description)
		print(task.priority)

## <a name="query-entity-properties"> </a>如何查询实体属性子集

对表的查询可以只检索实体中的少数几个属性。
此方法称为  *projection*，可减少带宽并提高查询性能，尤其适用于大型实体。使用 select 参数**并传递你希望显示给客户端的属性**的名称。

以下代码中的查询只返回表中实体的说明。

*请注意，下面的代码段只适用于云存储服务，存储
模拟器不支持它。*

	tasks = table_service.query_entities('tasktable', "PartitionKey eq 'tasksSeattle'", 'description')
	for task in tasks:
		print(task.description)

## <a name="delete-entity"> </a>如何删除实体

可以使用实体的分区键和行键删除实体。

	table_service.delete_entity('tasktable', 'tasksSeattle', '1')

## <a name="delete-table"> </a>如何删除表

以下代码从存储帐户中删除一个表。

	table_service.delete_table('tasktable')

## <a name="next-steps"> </a>后续步骤

现在，你已了解有关表存储的基础知识，可单击下面的链接来了解如何执行更复杂的存储任务。

-   查看 MSDN 参考：[在 Azure 中存储和访问数据][]
-   [访问 Azure 存储空间团队博客][]

  [后续步骤]: #next-steps
  [什么是表服务？]: #what-is
  [概念]: #concepts
  [创建 Azure 存储帐户]: #create-account
  [如何：创建表]: #create-table
  [如何：向表中添加实体]: #add-entity
  [如何：更新实体]: #update-entity
  [如何：更改实体组]: #change-entities
  [如何：查询实体]: #query-for-entity
  [如何：查询实体集]: #query-set-entities
  [如何：查询一部分实体属性]: #query-entity-properties
  [如何：删除实体]: #delete-entity
  [如何：删除表]: #delete-table
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [访问 Azure 存储空间团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
<!--HONumber=41-->
