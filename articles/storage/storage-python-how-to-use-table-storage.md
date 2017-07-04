<properties
    pageTitle="如何通过 Python 使用 Azure 表存储 | Azure"
    description="使用 Azure 表存储（一种 NoSQL 数据存储）将结构化数据存储在云中。"
    services="storage"
    documentationcenter="python"
    author="mmacy"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="7ddb9f3e-4e6d-4103-96e6-f0351d69a17b"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="python"
    ms.topic="article"
    ms.date="05/16/2017"
    wacn.date="05/31/2017"
    ms.author="marsma"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="9a649438b2d9a7141d88c4c85aa3ec10f099ed2f"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="how-to-use-table-storage-in-python"></a>如何在 Python 中使用表存储

[AZURE.INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]

## 概述
本指南将演示在 Python 中如何使用[用于 Python 的 Azure 存储 SDK](https://github.com/Azure/azure-storage-python) 执行常见的 Azure 表存储方案。 涉及的情景包括创建和删除表、插入和查询实体。

在解决本教程中的方案的同时，你可能想要参考[用于 Python API 的 Azure 存储 SDK 参考](https://azure-storage.readthedocs.io/en/latest/index.html)。

[AZURE.INCLUDE [storage-table-concepts-include](../../includes/storage-table-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

## <a name="install-the-azure-storage-sdk-for-python"></a>安装用于 Python 的 Azure 存储 SDK

创建存储帐户后，下一步就是安装[用于 Python 的 Microsoft Azure 存储 SDK](https://github.com/Azure/azure-storage-python)。 有关安装 SDK 的详细信息，请参阅 GitHub 上用于 Python 的存储 SDK 存储库中的 [README.rst](https://github.com/Azure/azure-storage-python/blob/master/README.rst) 文件。

## <a name="create-a-table"></a>创建表

若要使用 Python 中的 Azure 表服务，必须导入 [TableService][py_TableService] 模块。 因为要使用表实体，因此还需要[实体][ py_Entity]类。 将此代码添加到 Python 文件顶部附近，以便同时导入：

	from azure.storage.table import TableService, Entity

创建 [TableService][py_TableService] 对象，传入存储帐户名和帐户密钥。 将 `myaccount` 和 `mykey` 替换为帐户名和密钥，然后调用[create_table][py_create_table]，在 Azure 存储中创建表。

	table_service = TableService(account_name='myaccount', account_key='mykey', endpoint_suffix='core.chinacloudapi.cn')

	table_service.create_table('tasktable')

## <a name="add-an-entity-to-a-table"></a>将实体添加到表

若要添加实体，首先创建一个表示实体的对象，然后将该对象传递给 [TableService][py_TableService].[insert_entity][py_insert_entity] 方法。 实体对象可以是一个[实体][py_Entity]类型的字典和对象，且可定义实体的属性名和值。 除了为实体定义的任何其他属性外，每个实体还必须包含 [PartitionKey 和 RowKey](#partitionkey-and-rowkey) 属性。

此示例将创建一个表示实体的字典对象，然后将该对象传递给 [insert_entity][py_insert_entity] 方法以将其添加到表：

    task = {'PartitionKey': 'tasksSeattle', 'RowKey': '001', 'description' : 'Take out the trash', 'priority' : 200}
    table_service.insert_entity('tasktable', task)

此示例将创建一个[实体][py_Entity]对象，然后将该对象传递给 [insert_entity][py_insert_entity] 方法以将其添加到表：

    task = Entity()
    task.PartitionKey = 'tasksSeattle'
    task.RowKey = '002'
    task.description = 'Wash the car'
    task.priority = 100
    table_service.insert_entity('tasktable', task)

### <a name="partitionkey-and-rowkey"></a>PartitionKey 和 RowKey

必须为每个实体同时指定 PartitionKey 和 RowKey 属性。 这些是实体的唯一标识符，它们一起构成了实体的主键。 由于只对这些属性编制了索引，因此可使用这些值进行查询，速度比查询任何其他实体属性都要快。

表服务使用 PartitionKey 在存储节点中智能分发。 具有相同的 PartitionKey 的实体存储在同一个节点上。 RowKey 是实体在其所属分区内的唯一 ID。

## <a name="update-an-entity"></a>更新条目

若要更新实体的所有属性值，请调用 [update_entity][py_update_entity] 方法。 此示例演示如何使用更新版本替换现有实体：

    task = {'PartitionKey': 'tasksSeattle', 'RowKey': '001', 'description' : 'Take out the garbage', 'priority' : 250}
    table_service.update_entity('tasktable', task)

如果要更新的实体不存在，更新操作将失败。 如果要存储实体，无论它是否已存在，请使用 [insert_or_replace_entity][py_insert_or_replace_entity]。 在下面的示例中，第一次调用将替换现有实体。 第二个调用将插入新实体，因为表中不存在具有指定 PartitionKey 和 RowKey 的实体。

    # Replace the entity created earlier
    task = {'PartitionKey': 'tasksSeattle', 'RowKey': '001', 'description' : 'Take out the garbage again', 'priority' : 250}
    table_service.insert_or_replace_entity('tasktable', task)

    # Insert a new entity
    task = {'PartitionKey': 'tasksSeattle', 'RowKey': '003', 'description' : 'Buy detergent', 'priority' : 300}
    table_service.insert_or_replace_entity('tasktable', task)

> [AZURE.TIP]
> [Update_entity][py_update_entity] 方法将替换现有实体的所有属性和值，还可将其用于从现有实体中删除属性。 可以使用 [merge_entity][py_merge_entity] 方法更新含新的或修改后的属性值的现有实体，而无需彻底替换该实体。

## <a name="modify-multiple-entities"></a>修改多个实体

若要确保通过表服务进行原子处理，可以批量同时提交多个操作。 首先，使用 [TableBatch][py_TableBatch] 类将多个操作添加到单个批处理中。 接下来，调用 [TableService][py_TableService].[commit_batch][py_commit_batch] 以原子操作的形式提交操作。 批处理中要修改的实体必须位于同一分区。

该示例将两个实体一起添加到批处理中：

    from azure.storage.table import TableBatch
    batch = TableBatch()
    task004 = {'PartitionKey': 'tasksSeattle', 'RowKey': '004', 'description' : 'Go grocery shopping', 'priority' : 400}
    task005 = {'PartitionKey': 'tasksSeattle', 'RowKey': '005', 'description' : 'Clean the bathroom', 'priority' : 100}
    batch.insert_entity(task004)
    batch.insert_entity(task005)
    table_service.commit_batch('tasktable', batch)

还可以通过上下文管理器语法来使用批处理：

    task006 = {'PartitionKey': 'tasksSeattle', 'RowKey': '006', 'description' : 'Go grocery shopping', 'priority' : 400}
    task007 = {'PartitionKey': 'tasksSeattle', 'RowKey': '007', 'description' : 'Clean the bathroom', 'priority' : 100}

    with table_service.batch('tasktable') as batch:
        batch.insert_entity(task006)
        batch.insert_entity(task007)

## <a name="query-for-an-entity"></a>查询条目

若要查询表中的实体，请将 PartitionKey 和 RowKey 传递给 [TableService][py_TableService].[get_entity][py_get_entity] 方法。

    task = table_service.get_entity('tasktable', 'tasksSeattle', '001')
    print(task.description)
    print(task.priority)

## <a name="query-a-set-of-entities"></a>查询一组条目

在筛选器字符串中提供 filter 参数，可以查询一组实体。 此示例通过对 PartitionKey 应用筛选器来查找 Seattle 中的所有任务：

	tasks = table_service.query_entities('tasktable', filter="PartitionKey eq 'tasksSeattle'")
	for task in tasks:
		print(task.description)
		print(task.priority)

## <a name="query-a-subset-of-entity-properties"></a>查询条目属性的子集

还可以限制查询中每个实体所返回的属性。 此方法称为“投影”，可减少带宽并提高查询性能，尤其适用于大型实体或结果集。 使用 select 参数并传递希望返回给客户端的属性的名称。

以下代码中的查询只返回表中实体的说明。

> [AZURE.NOTE]
> 下面的代码段仅对 Azure 存储有效。 但不受存储模拟器支持。

	tasks = table_service.query_entities('tasktable', filter="PartitionKey eq 'tasksSeattle'", select='description')
	for task in tasks:
		print(task.description)

## <a name="delete-an-entity"></a>删除实体

可通过将实体的 PartitionKey 和 RowKey 传递给 [delete_entity][py_delete_entity] 方法将其删除。

    table_service.delete_entity('tasktable', 'tasksSeattle', '001')

## <a name="delete-a-table"></a>删除表

如果不再需要表或其内的任何实体，请调用 [delete_table][py_delete_table] 方法将该表从 Azure 存储中永久删除。

	table_service.delete_table('tasktable')

## <a name="next-steps"></a>后续步骤

* [用于 Python API 的 Azure 存储 SDK 参考](https://azure-storage.readthedocs.io/en/latest/index.html)
* [Azure Storage SDK for Python](https://github.com/Azure/azure-storage-python)
* [Python 开发人员中心](/develop/python/)
* [Azure 存储资源管理器](/documentation/articles/vs-azure-tools-storage-manage-with-storage-explorer/)：一款跨平台的免费应用程序，用于直观处理 Windows、MacOS 和 Linux 上的 Azure 存储数据。

[py_commit_batch]: https://azure-storage.readthedocs.io/en/latest/ref/azure.storage.table.tableservice.html#azure.storage.table.tableservice.TableService.commit_batch
[py_create_table]: https://azure-storage.readthedocs.io/en/latest/ref/azure.storage.table.tableservice.html#azure.storage.table.tableservice.TableService.create_table
[py_delete_entity]: https://azure-storage.readthedocs.io/en/latest/ref/azure.storage.table.tableservice.html#azure.storage.table.tableservice.TableService.delete_entity
[py_delete_table]: https://azure-storage.readthedocs.io/en/latest/ref/azure.storage.table.tableservice.html#azure.storage.table.tableservice.TableService.delete_table
[py_Entity]: https://azure-storage.readthedocs.io/en/latest/ref/azure.storage.table.models.html#azure.storage.table.models.Entity
[py_get_entity]: https://azure-storage.readthedocs.io/en/latest/ref/azure.storage.table.tableservice.html#azure.storage.table.tableservice.TableService.get_entity
[py_insert_entity]: https://azure-storage.readthedocs.io/en/latest/ref/azure.storage.table.tableservice.html#azure.storage.table.tableservice.TableService.insert_entity
[py_insert_or_replace_entity]: https://azure-storage.readthedocs.io/en/latest/ref/azure.storage.table.tableservice.html#azure.storage.table.tableservice.TableService.insert_or_replace_entity
[py_merge_entity]: https://azure-storage.readthedocs.io/en/latest/ref/azure.storage.table.tableservice.html#azure.storage.table.tableservice.TableService.merge_entity
[py_update_entity]: https://azure-storage.readthedocs.io/en/latest/ref/azure.storage.table.tableservice.html#azure.storage.table.tableservice.TableService.update_entity
[py_TableService]: https://azure-storage.readthedocs.io/en/latest/ref/azure.storage.table.tableservice.html
[py_TableBatch]: https://azure-storage.readthedocs.io/en/latest/ref/azure.storage.table.tablebatch.html#azure.storage.table.tablebatch.TableBatch
<!--Update_Description:whole content refine-->