<properties
	pageTitle="如何通过 Python 使用表存储 | Azure"
	description="使用 Azure 表存储（一种 NoSQL 数据存储）将结构化数据存储在云中。"
	services="storage"
	documentationCenter="python"
	authors="emgerner-msft"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="storage"
	ms.date="04/29/2016"
	wacn.date="06/13/2016"/>


# 如何通过 Python 使用表存储

[AZURE.INCLUDE [storage-selector-table-include](../includes/storage-selector-table-include.md)]

## 概述

本指南将演示如何使用 Azure 表存储服务执行常见方案。这些示例通过 Python 编写并使用 [Azure Storage SDK for Python]。涉及的方案包括创建和删除表、以及在表中插入和查询实体。

[AZURE.INCLUDE [storage-table-concepts-include](../includes/storage-table-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

## 创建表

可通过 **TableService** 对象使用表服务。以下代码创建 **TableService** 对象。在你希望在其中以编程方式访问 Azure 存储空间的任何 Python 文件中，将代码添加到文件的顶部附近：

	from azure.storage.table import TableService, Entity

以下代码使用存储帐户名称和帐户密钥创建 **TableService** 对象。使用你的帐户名称和密钥替换“myaccount”和“mykey”。

	table_service = TableService(account_name='myaccount', account_key='mykey', endpoint_suffix='core.chinacloudapi.cn')

	table_service.create_table('tasktable')

## 将实体添加到表

若要添加实体，请首先创建定义实体属性名称和值的字典或实体。请注意，对于每个实体，你必须指定 **PartitionKey** 和 **RowKey**。这些是您的实体的唯一标识符。你可以使用这些值进行查询，比查询其他属性快得多。系统使用 **PartitionKey** 自动将表的实体分发到多个存储节点上。具有相同的 **PartitionKey** 的实体存储在同一个节点上。**RowKey** 是实体在其所属分区内的唯一 ID。

若要将实体添加到表中，请将字典对象传递给 **insert_entity** 方法。

	task = {'PartitionKey': 'tasksSeattle', 'RowKey': '1', 'description' : 'Take out the trash', 'priority' : 200}
	table_service.insert_entity('tasktable', task)

你还可以将 **Entity** 类的实例传递给 **insert_entity** 方法。

	task = Entity()
	task.PartitionKey = 'tasksSeattle'
	task.RowKey = '2'
	task.description = 'Wash the car'
	task.priority = 100
	table_service.insert_entity('tasktable', task)

## 更新实体

此代码演示如何使用更新版本替换现有实体的旧版本。

	task = {'PartitionKey': 'tasksSeattle', 'RowKey': '1', 'description' : 'Take out the garbage', 'priority' : 250}
	table_service.update_entity('tasktable', task)

如果要更新的实体不存在，更新操作将失败。如果你要存储实体，而不管它以前是否已存在，请使用 **insert_or_replace_entity**。在下面的示例中，第一次调用将替换现有实体。第二次调用将插入新实体，因为表中不存在具有指定的 **PartitionKey** 和 **RowKey** 的实体。

	task = {'PartitionKey': 'tasksSeattle', 'RowKey': '1', 'description' : 'Take out the garbage again', 'priority' : 250}
	table_service.insert_or_replace_entity('tasktable', task)

	task = {'PartitionKey': 'tasksSeattle', 'RowKey': '3', 'description' : 'Buy detergent', 'priority' : 300}
	table_service.insert_or_replace_entity('tasktable', task)

## 更改实体组

有时，有必要成批地同时提交多项操作以确保通过服务器进行原子处理。若要完成此操作，请使用 **TableBatch** 类。在你确实要按批提交时，可调用 **commit\_batch**。请注意，所有实体必须在相同分区中才能更改为批处理。下面的示例将两个实体一起添加到批处理中。

	from azure.storage.table import TableBatch
	batch = TableBatch()
	task10 = {'PartitionKey': 'tasksSeattle', 'RowKey': '10', 'description' : 'Go grocery shopping', 'priority' : 400}
	task11 = {'PartitionKey': 'tasksSeattle', 'RowKey': '11', 'description' : 'Clean the bathroom', 'priority' : 100}
	batch.insert_entity(task10)
	batch.insert_entity(task11)
	table_service.commit_batch('tasktable', batch)

还可以通过上下文管理器语法来使用批处理：

	task12 = {'PartitionKey': 'tasksSeattle', 'RowKey': '12', 'description' : 'Go grocery shopping', 'priority' : 400}
	task13 = {'PartitionKey': 'tasksSeattle', 'RowKey': '13', 'description' : 'Clean the bathroom', 'priority' : 100}

	with table_service.batch('tasktable') as batch:
		batch.insert_entity(task12)
		batch.insert_entity(task13)


## 查询实体

若要查询表中的实体，请使用 **get_entity** 方法并传递 **PartitionKey** 和 **RowKey**。

	task = table_service.get_entity('tasktable', 'tasksSeattle', '1')
	print(task.description)
	print(task.priority)

## 查询实体集

此示例基于 **PartitionKey** 查找 Seattle 中的所有任务。

	tasks = table_service.query_entities('tasktable', filter="PartitionKey eq 'tasksSeattle'")
	for task in tasks:
		print(task.description)
		print(task.priority)

## 查询一部分实体属性

对表的查询可以只检索实体中的少数几个属性。此方法称为“投影”，可减少带宽并提高查询性能，尤其适用于大型实体。使用 **select** 参数并传递你希望显示给客户端的属性的名称。

以下代码中的查询只返回表中实体的说明。

[AZURE.NOTE]下面的代码段仅对云存储服务有效。不受存储模拟器支持。

	tasks = table_service.query_entities('tasktable', filter="PartitionKey eq 'tasksSeattle'", select='description')
	for task in tasks:
		print(task.description)

## 删除实体

可以使用实体的分区键和行键删除实体。

	table_service.delete_entity('tasktable', 'tasksSeattle', '1')

## 删除表

以下代码从存储帐户中删除一个表。

	table_service.delete_table('tasktable')

## 后续步骤

现在，你已了解表存储的基础知识，可单击下面的链接了解详细信息。

- [Python 开发人员中心](/develop/python/)
- [Azure 存储空间服务 REST API](http://msdn.microsoft.com/zh-cn/library/azure/dd179355)
- [Azure 存储团队博客]
- [Azure Storage SDK for Python]

[Azure 存储团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
[Azure Storage SDK for Python]: https://github.com/Azure/azure-storage-python

<!---HONumber=Mooncake_0606_2016-->