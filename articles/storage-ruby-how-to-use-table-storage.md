<properties 
	pageTitle="如何通过 Ruby 使用 Azure 表存储 | Azure" 
	description="使用 Azure 表存储（一种 NoSQL 数据存储）将结构化数据存储在云中。"
	services="storage"
	documentationCenter="ruby"
	authors="tfitzmac"
	manager="wpickett"
	editor=""/>

<tags 
	ms.service="storage" 
	ms.date="04/29/2016"
	wacn.date="06/13/2016"/>


# 如何通过 Ruby 使用 Azure 表存储

[AZURE.INCLUDE [storage-selector-table-include](../includes/storage-selector-table-include.md)]

## 概述

本指南演示如何使用 Azure 表服务执行常见任务。相关示例是使用 Ruby API 编写的。涉及的方案包括“创建和删除表、在表中插入和查询实体”。

[AZURE.INCLUDE [storage-table-concepts-include](../includes/storage-table-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

## 创建 Ruby 应用程序

有关如何创建 Ruby 应用程序的说明，请参阅 [Azure VM 上的 Ruby on Rails Web 应用程序](/documentation/articles/virtual-machines-linux-classic-ruby-rails-web-app/)。


## 配置应用程序以访问存储

要使用 Azure 存储空间，你需要下载和使用 Ruby azure 包，其中包括一组便于与存储 REST 服务进行通信的库。

### 使用 RubyGems 获取该程序包

1. 使用命令行接口，例如 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (Unix)。

2. 在命令窗口中键入 **gem install azure** 以安装 gem 和依赖项。

### 导入包

使用常用的文本编辑器将以下内容添加到你要在其中使用存储的 Ruby 文件的顶部：

	require "azure"

## 设置 Azure 存储连接

Azure 模块将读取环境变量 **AZURE_STORAGE_ACCOUNT** 和 **AZURE_STORAGE_ACCESS_KEY**，以便获取连接到 Azure 存储帐户所需的信息。如果未设置这些环境变量，则在使用 **Azure::TableService** 之前必须通过以下代码指定帐户信息：

	Azure.config.storage_account_name = "<your azure storage account>"
	Azure.config.storage_access_key = "<your azure storage access key>"

获取这些值：


从管理门户中的存储帐户中获取这些值：

1. 登录到[管理门户](https://manage.windowsazure.cn)。
2. 导航到要使用的存储帐户。
3. 单击导航窗格底部的“管理访问密钥”。
4. 在弹出对话框中，将会看到存储帐户名称、主访问密钥和辅助访问密钥。对于访问密钥，您可以使用主访问密钥，也可以使用辅助访问密钥。 
5. 单击复制图标以将键复制到剪贴板。

## 创建表

使用 **Azure::TableService** 对象可以对表和实体进行操作。若要创建表，请使用 **create_table()** 方法。以下示例将创建一个表或输出存在的错误。

	azure_table_service = Azure::TableService.new
	begin
	  azure_table_service.create_table("testtable")
	rescue
	  puts $!
	end

## 将实体添加到表

若要添加实体，应首先创建一个定义了你的实体属性的哈希对象。请注意，对于每个实体，您必须指定 **PartitionKey** 和 **RowKey**。这些值是实体的唯一标识符，并且查询它们比查询其他属性快很多。Azure 存储服务使用 **PartitionKey** 自动将表的实体分发到多个存储节点上。具有相同的 **PartitionKey** 的实体存储在同一个节点上。**RowKey** 是实体在其所属分区内的唯一 ID。

	entity = { "content" => "test entity",
	  :PartitionKey => "test-partition-key", :RowKey => "1" }
	azure_table_service.insert_entity("testtable", entity)

## 更新实体

可使用多种方法来更新现有实体：

* **update_entity()：**通过替换现有实体来更新现有实体。
* **merge_entity()：**通过将新属性值合并到现有实体来更新现有实体。
* **insert_or_merge_entity()：**通过替换现有实体来更新现有实体。如果不存在实体，将插入一个新实体。
* **insert_or_replace_entity()：**通过将新属性值合并到现有实体来更新现有实体。如果不存在实体，将插入一个新实体。

以下示例演示了使用 **update_entity()** 更新实体：

	entity = { "content" => "test entity with updated content",
	  :PartitionKey => "test-partition-key", :RowKey => "1" }
	azure_table_service.update_entity("testtable", entity)

对于 **update_entity()** 和 **merge_entity()**，如果要更新的实体不存在，则更新操作将失败。因此，如果您希望存储某个实体而不考虑它是否已存在，则应改用 **insert_or_replace_entity()** 或 **insert_or_merge_entity()**。

## 使用实体组

有时，有必要成批地同时提交多项操作以确保通过服务器进行原子处理。若要完成此操作，首先要创建一个 **Batch** 对象，然后对 **TableService** 使用 **execute_batch()** 方法。下面的示例演示在一个批次中提交 RowKey 为 2 和 3 的两个实体。请注意，这仅适用于具有相同 PartitionKey 的实体。

	azure_table_service = Azure::TableService.new
	batch = Azure::Storage::Table::Batch.new("testtable",
	  "test-partition-key") do
	  insert "2", { "content" => "new content 2" }
	  insert "3", { "content" => "new content 3" }
	end
	results = azure_table_service.execute_batch(batch)

## 查询实体

若要查询表中的实体，请使用 **get_entity()** 方法并传递表名称 **PartitionKey** 和 **RowKey**。

	result = azure_table_service.get_entity("testtable", "test-partition-key",
	  "1")

## 查询实体集

若要查询表中的实体集，请创建一个查询哈希对象并使用 **query_entities()** 方法。下面的示例演示了如何获取具有相同 **PartitionKey** 的所有实体：

	query = { :filter => "PartitionKey eq 'test-partition-key'" }
	result, token = azure_table_service.query_entities("testtable", query)

> [AZURE.NOTE] 如果结果集太大，一个查询无法全部返回，将会返回一个继续标记，你可以使用该标记检索后续页面。

## 查询一部分实体属性

对表的查询可以只检索实体中的少数几个属性。此方法称为“投影”，可减少带宽并提高查询性能，尤其适用于大型实体。请使用 select 子句并传递你希望显示给客户端的属性的名称。

	query = { :filter => "PartitionKey eq 'test-partition-key'",
	  :select => ["content"] }
	result, token = azure_table_service.query_entities("testtable", query)

## <a id="how-to-delete-an-entity"></a>删除实体

若要删除实体，请使用 **delete_entity()** 方法。你需要传入包含该实体的表的名称、实体的 PartitionKey 和 RowKey。

		azure_table_service.delete_entity("testtable", "test-partition-key", "1")

## <a id="how-to-delete-a-table"></a>删除表

若要删除表，请使用 **delete_table()** 方法并传入要删除的表的名称。

		azure_table_service.delete_table("testtable")

## <a id="next-steps"></a>后续步骤

若要了解有关更复杂存储任务的信息，请访问下面的链接：

- [Azure 存储团队博客](http://blogs.msdn.com/b/windowsazurestorage/)
- GitHub 上的 [Azure SDK for Ruby](http://github.com/WindowsAzure/azure-sdk-for-ruby) 存储库

<!---HONumber=Mooncake_0606_2016-->