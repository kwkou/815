<properties
    pageTitle="如何通过 Ruby 使用 Azure 表存储 | Azure"
    description="使用 Azure 表存储（一种 NoSQL 数据存储）将结构化数据存储在云中。"
    services="storage"
    documentationcenter="ruby"
    author="mmacy"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="047cd9ff-17d3-4c15-9284-1b5cc61a3224"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="ruby"
    ms.topic="article"
    ms.date="12/08/2016"
    wacn.date="01/06/2017"
    ms.author="marsma" />

# 如何通过 Ruby 使用 Azure 表存储

[AZURE.INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]

## 概述
本指南演示如何使用 Azure 表服务执行常见任务。相关示例是使用 Ruby API 编写的。涉及的情景包括创建和删除表、在表中插入和查询条目。

[AZURE.INCLUDE [storage-table-concepts-include](../../includes/storage-table-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

## 创建 Ruby 应用程序

有关如何创建 Ruby 应用程序的说明，请参阅 [Azure VM 上的 Ruby on Rails Web 应用程序](/documentation/articles/virtual-machines-linux-classic-ruby-rails-web-app/)。

## 配置应用程序以访问存储
要使用 Azure 存储，需下载和使用 Ruby Azure 包，其中包括与存储 REST 服务通信的一组方便的库。

### 使用 RubyGems 获取该程序包
1. 使用命令行接口，例如 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (Unix)。
2. 在命令窗口中键入 **gem install azure** 以安装 gem 和依赖项。

### 导入包
使用常用的文本编辑器将以下内容添加到要在其中使用存储的 Ruby 文件的顶部：

	require "azure"

## 设置 Azure 存储连接

Azure 模块将读取环境变量 **AZURE\_STORAGE\_ACCOUNT** 和 **AZURE\_STORAGE\_ACCESS\_KEY**，以便获取连接到 Azure 存储帐户所需的信息。如果未设置这些环境变量，则在使用 **Azure::TableService** 之前必须通过以下代码指定帐户信息：

	Azure.config.storage_account_name = "<your azure storage account>"
	Azure.config.storage_access_key = "<your azure storage access key>"

在 Azure 门户中，从经典或 Resource Manager 存储帐户获取这些值：

1. 登录到 [Azure 门户](https://portal.azure.cn)。
2. 导航到要使用的存储帐户。
3. 在右侧的“设置”边栏选项卡中，单击“访问密钥”。
4. 在显示的“访问密钥”边栏选项卡中，可看到访问密钥 1 和访问密钥 2。可以使用其中任意一个密钥。
5. 单击复制图标以将密钥复制到剪贴板。

从 Azure 经典管理门户中的存储帐户中获取这些值：

1. 登录到[Azure 经典管理门户](https://manage.windowsazure.cn)。
2. 导航到要使用的存储帐户。
3. 单击导航窗格底部的“管理访问密钥”。
4. 在弹出对话框中，可看到存储帐户名称、主访问密钥和辅助访问密钥。对于访问密钥，可以使用主访问密钥，也可以使用辅助访问密钥。
5. 单击复制图标以将密钥复制到剪贴板。

## 创建表

使用 **Azure::TableService** 对象可用于处理表和条目。若要创建表，请使用 **create\_table()** 方法。以下示例将创建表或输出存在的错误。

	azure_table_service = Azure::TableService.new
	begin
	  azure_table_service.create_table("testtable")
	rescue
	  puts $!
	end

## 向表中添加条目
若要添加条目，应首先创建定义条目属性的哈希对象。请注意，必须为每个条目指定 **PartitionKey** 和 **RowKey**。这些值是条目的唯一标识符，查询它们比查询其他属性快很多。Azure 存储使用 **PartitionKey** 将表中条目自动分发到多个存储节点。具有相同 **PartitionKey** 的条目存储在同一个节点上。**RowKey** 是条目在其所属分区内的唯一 ID。

	entity = { "content" => "test entity",
	  :PartitionKey => "test-partition-key", :RowKey => "1" }
	azure_table_service.insert_entity("testtable", entity)

## 更新条目

可使用多种方法来更新现有条目：

* **update\_entity()：** 通过替换更新现有条目。
* **merge\_entity()：** 通过并入新属性值来更新现有条目。
* **insert\_or\_merge\_entity()：** 通过替换更新现有条目。如果不存在条目，则插入新条目：
* **insert\_or\_replace\_entity()：** 通过并入新属性值来更新现有条目。如果不存在条目，则插入新条目。

以下示例演示如何使用 **update\_entity()** 更新条目：

	entity = { "content" => "test entity with updated content",
	  :PartitionKey => "test-partition-key", :RowKey => "1" }
	azure_table_service.update_entity("testtable", entity)

对于 **update\_entity()** 和 **merge\_entity()**，如果要更新的条目不存在，更新操作将失败。因此，如果希望存储某个条目而不考虑它是否已存在，应改用 **insert\_or\_replace\_entity()** 或 **insert\_or\_merge\_entity()**。

## 处理条目组

有时，有必要成批地同时提交多项操作以确保通过服务器进行原子处理。若要完成此操作，首先要创建一个 **Batch** 对象，然后对 **TableService** 使用 **execute\_batch()** 方法。以下示例演示在一个批次中提交 RowKey 为 2 和 3 的两个条目。请注意，此操作仅适用于具有相同 PartitionKey 的条目。

	azure_table_service = Azure::TableService.new
	batch = Azure::Storage::Table::Batch.new("testtable",
	  "test-partition-key") do
	  insert "2", { "content" => "new content 2" }
	  insert "3", { "content" => "new content 3" }
	end
	results = azure_table_service.execute_batch(batch)

## 查询条目

若要查询表中条目，请使用 **get\_entity()** 方法并传递表名称、**PartitionKey** 和 **RowKey**。

	result = azure_table_service.get_entity("testtable", "test-partition-key",
	  "1")

## 查询一组条目

若要查询表中的一组条目，请创建查询哈希对象并使用 **query\_entities()** 方法。以下示例演示如何获取具有相同 **PartitionKey** 的所有条目：

	query = { :filter => "PartitionKey eq 'test-partition-key'" }
	result, token = azure_table_service.query_entities("testtable", query)

> [AZURE.NOTE] 如果结果集太大，一个查询无法全部返回，将会返回一个继续标记，你可以使用该标记检索后续页面。

## 查询条目属性的子集
对表的查询可以只检索条目的几个属性。这种技术称为“投影”，可减少带宽并提高查询性能，尤其适用于大型条目。请使用 select 子句并传递你希望显示给客户端的属性的名称。

	query = { :filter => "PartitionKey eq 'test-partition-key'",
	  :select => ["content"] }
	result, token = azure_table_service.query_entities("testtable", query)

## <a id="how-to-delete-an-entity"></a>删除条目

若要删除条目，请使用 **delete\_entity()** 方法。需要传入包含该条目的表的名称、条目的 PartitionKey 和 RowKey。

		azure_table_service.delete_entity("testtable", "test-partition-key", "1")

## <a id="how-to-delete-a-table"></a>删除表

若要删除表，请使用 **delete\_table()** 方法并传入要删除的表的名称。

		azure_table_service.delete_table("testtable")

## <a id="next-steps"></a>后续步骤

若要了解有关更复杂存储任务的信息，请访问下面的链接：

- [Azure 存储团队博客](http://blogs.msdn.com/b/windowsazurestorage/)
- GitHub 上的 [Azure SDK for Ruby](http://github.com/WindowsAzure/azure-sdk-for-ruby) 存储库

<!---HONumber=Mooncake_0103_2017-->