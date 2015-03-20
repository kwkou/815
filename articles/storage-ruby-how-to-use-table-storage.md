<properties linkid="dev-ruby-how-to-table-services" urlDisplayName="Table Service" pageTitle="如何使用表存储 (Ruby) | Microsoft Azure" metaKeywords="Azure table storage service, Azure table service Ruby, table storage Ruby" description="了解如何在 Azure 中使用表存储服务。相关代码示例是使用 Ruby API 编写的。" metaCanonical="" services="storage" documentationCenter="Ruby" title="How to Use the Table Service from Ruby" authors="guayan" solutions="" manager="" editor="" />





# 如何通过 Ruby 使用表服务

本指南将演示如何使用 Microsoft Azure 表服务执行常见方案。相关示例是使用Ruby API 编写的。涉及的方案包括**创建和删除表、插入和查询表中的实体**。有关表的详细信息，请参阅[后续步骤](#next-steps) 部分。

## 目录

* [什么是表服务？](#what-is)
* [概念](#concepts)
* [创建 Azure 存储帐户](#create-a-windows-azure-storage-account)
* [创建 Ruby 应用程序](#create-a-ruby-application)
* [配置应用程序以访问存储](#configure-your-application-to-access-storage)
* [设置 Azure 存储连接](#setup-a-windows-azure-storage-connection)
* [如何：创建表](#how-to-create-a-table)
* [如何：向表中添加实体](#how-to-add-an-entity-to-a-table)
* [如何：更新实体](#how-to-update-an-entity)
* [如何：使用实体组](#how-to-work-with-groups-of-entities)
* [如何：查询实体](#how-to-query-for-an-entity)
* [如何：查询实体集](#how-to-query-a-set-of-entities)
* [如何：查询一部分实体属性](#how-to-query-a-subset-of-entity-properties)
* [如何：删除实体](#how-to-delete-an-entity)
* [如何：删除表](#how-to-delete-a-table)
* [后续步骤](#next-steps)

[WACOM.INCLUDE [howto-table-storage](../includes/howto-table-storage.md)]

## <a id="create-a-windows-azure-storage-account"></a>创建 Azure 存储帐户

[WACOM.INCLUDE [create-storage-account](../includes/create-storage-account.md)]

## <a id="create-a-ruby-application"></a>创建 Ruby 应用程序

创建 Ruby 应用程序。有关说明， 
请参阅 [在 Azure 上创建 Ruby 应用程序](/zh-cn/develop/ruby/tutorials/web-app-with-linux-vm/)。

## <a id="configure-your-application-to-access-storage"></a>配置应用程序以访问存储

若要使用 Azure 存储空间，需要下载和使用 Ruby azure 包， 其中包括一组便于与存储 REST 服务进行通信的库。

### 使用 RubyGems 获取该程序包

1. 使用命令行接口，例如 PowerShell (Windows)、Terminal (Mac) 或 Bash (Unix)。

2. 在命令窗口中键入"gem install azure"以安装 gem 和依赖项。

### 导入包

使用常用的文本编辑器将以下内容添加到你要在其中使用存储的 Ruby 文件的顶部：

	require "azure"

## <a id="setup-a-windows-azure-storage-connection"></a>设置 Azure 存储连接

Azure 模块将读取环境变量 **AZURE\_STORAGE\_ACCOUNT** 和 **AZURE\_STORAGE\_ACCESS\_KEY**，以便获取连接到 Azure 存储帐户所需的信息。如果未设置这些环境变量，则在使用 Azure::TableService 之前必须通过以下代码指定帐户信息：

	Azure.config.storage_account_name = "<your azure storage account>"
	Azure.config.storage_access_key = "<your azure storage access key>"

获取这些值：

1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn/).

2. 导航到要使用的存储帐户。

3. 单击导航窗格底部的"管理密钥"。

4. 在弹出对话框中，将会看到存储帐户名称、主访问密钥和辅助访问密钥。对于访问密钥，你可以使用主访问密钥，也可以使用辅助访问密钥。

## <a id="how-to-create-a-table"></a>如何创建表

使用 Azure::TableService 对象可以对表和实体进行操作。若要创建表，请使用 **create\_table()** 方法。以下示例将创建一个表或输出存在的错误。

	azure_table_service = Azure::TableService.new
	begin
	  azure_table_service.create_table("testtable")
	rescue
	  puts $!
	end

## <a id="how-to-add-an-entity-to-a-table"></a>如何向表中添加实体

若要添加实体，应首先创建一个定义了你的实体属性的哈希对象。请注意，对于每个实体，你必须指定 PartitionKey 和 RowKey。这些值是实体的唯一标识符，并且查询它们比查询其他属性快很多。Azure 存储服务使用 PartitionKey 自动将表的实体分发到多个存储节点上。具有相同的 PartitionKey 的实体存储在同一个节点上。RowKey 是实体在其所属分区内的唯一 ID。** 

	entity = { "content" => "test entity", 
	  :PartitionKey => "test-partition-key", :RowKey => "1" }
	azure_table_service.insert_entity("testtable", entity)

## <a id="how-to-update-an-entity"></a>如何：更新实体

可使用多种方法来更新现有实体：

* **update\_entity()：**通过替换现有实体来更新现有实体。
* **merge\_entity()：**通过将新属性值合并到现有实体来更新现有实体。
* **insert\_or\_merge\_entity()：**通过替换现有实体来更新现有实体。如果不存在实体，则会插入新实体：
* **insert\_or\_replace\_entity()：**通过将新属性值合并到现有实体中来更新现有实体。如果不存在实体，则会插入新实体。

以下示例演示了使用 **update\_entity()** 更新实体：

	entity = { "content" => "test entity with updated content", 
	  :PartitionKey => "test-partition-key", :RowKey => "1" }
	azure_table_service.update_entity("testtable", entity)

对于 **update\_entity()** 和 **merge\_entity()**，如果待更新的实体不存在，则更新操作将失败。因此，如果你希望存储某个实体而不考虑它是否已存在，则应改用 **insert\_or\_replace\_entity()** 或 **insert\_or\_merge\_entity()**。

## <a id="how-to-work-with-groups-of-entities"></a>如何：使用实体组

有时，有必要成批地同时提交多项操作以确保通过服务器进行原子处理。若要完成此操作，首先要创建一个 **Batch** 对象，然后对 **TableService** 使用 **execute\_batch()** 方法。下面的示例演示在一个批次中提交 RowKey 为 2 和 3 的两个实体。请注意，这仅适用于具有相同 PartitionKey 的实体。

	azure_table_service = Azure::TableService.new
	batch = Azure::Storage::Table::Batch.new("testtable", 
	  "test-partition-key") do
	  insert "2", { "content" => "new content 2" }
	  insert "3", { "content" => "new content 3" }
	end
	results = azure_table_service.execute_batch(batch)

## <a id="how-to-query-for-an-entity"></a>如何：查询实体

若要查询表中的实体，请使用 **get\_entity()** 方法并传递表名称 **PartitionKey** 和 **RowKey**。

	result = azure_table_service.get_entity("testtable", "test-partition-key", 
	  "1")

## <a id="how-to-query-a-set-of-entities"></a>如何：查询实体集

若要查询表中的实体集，请创建一个查询哈希对象并使用 **query\_entities()** 方法。下面的示例演示了如何获取具有相同 PartitionKey 的所有实体：

	query = { :filter => "PartitionKey eq 'test-partition-key'" }
	result, token = azure_table_service.query_entities("testtable", query)

注意，如果结果集太大，一个查询无法全部返回，将会返回一个继续标记，你可以使用该标记检索后续页面。

## <a id="how-to-query-a-subset-of-entity-properties"></a>如何：查询一部分实体属性

对表的查询可以只检索实体中的少数几个属性。此方法称为"投影"，可减少带宽并提高查询性能，尤其适用于大型实体。请使用 select 子句并传递你希望显示给客户端的属性的名称。

	query = { :filter => "PartitionKey eq 'test-partition-key'", 
	  :select => ["content"] }
	result, token = azure_table_service.query_entities("testtable", query)

## <a id="how-to-delete-an-entity"></a>如何：删除实体

若要删除实体，请使用 **delete\_entity()** 方法。你需要传入包含该实体的表的名称、实体的 PartitionKey 和 RowKey。

		azure_table_service.delete_entity("testtable", "test-partition-key", "1")

## <a id="how-to-delete-a-table"></a>如何：删除表

若要删除表，请使用 **delete\_table()** 方法并传入要删除的表的名称。

		azure_table_service.delete_table("testtable")

## <a id="next-steps"></a>后续步骤

现在，你已了解有关表存储的基础知识，可单击下面的链接来了解如何执行更复杂的存储任务。

* 查看 MSDN 参考：[在 Azure 中存储和访问数据](http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx)
* 访问 [Azure 存储空间团队博客](http://blogs.msdn.com/b/windowsazurestorage/)
* 访问 [Azure SDK for Ruby](http://github.com/WindowsAzure/azure-sdk-for-ruby) repository on GitHub
<!--HONumber=41-->
