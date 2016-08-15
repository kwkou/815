<properties
	pageTitle="如何通过 Python 使用 Azure Blob 存储 | Azure"
	description="使用 Azure Blob 存储（对象存储）将非结构化数据存储在云中。"
	services="storage"
	documentationCenter="python"
	authors="emgerner-msft"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="storage"
    ms.date="04/29/2016"
	wacn.date="06/13/2016"/>

# 如何通过 Python 使用 Azure Blob 存储

[AZURE.INCLUDE [storage-selector-blob-include](../includes/storage-selector-blob-include.md)]

## 概述

本指南将演示如何使用 Blob 存储执行常见方案。这些示例通过 Python 编写并使用 [Azure Storage SDK for Python][]。涉及的任务包括上载、列出、下载和删除 Blob。

[AZURE.INCLUDE [storage-blob-concepts-include](../includes/storage-blob-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

## 创建容器

根据要使用的 Blob 的类型，创建 **BlockBlobService**、**AppendBlobService** 或 **PageBlobService** 对象。以下代码使用 **BlockBlobService** 对象。在你希望在其中以编程方式访问 Azure 块 Blob 存储的任何 Python 文件中，将以下代码添加到文件的顶部附近。

	from azure.storage.blob import BlockBlobService

以下代码使用存储帐户名称和帐户密钥创建一个 **BlockBlobService** 对象。使用你的帐户名称和密钥替换“myaccount”和“mykey”。

	block_blob_service = BlockBlobService(account_name='myaccount', account_key='mykey', endpoint_suffix='core.chinacloudapi.cn')

[AZURE.INCLUDE [storage-container-naming-rules-include](../includes/storage-container-naming-rules-include.md)]

在以下代码示例中，如果容器不存在，可以使用 **BlockBlobService** 对象来创建它。

	block_blob_service.create_container('mycontainer')

默认情况下，新容器是专用容器，因此您必须指定存储访问密钥（如之前所做的那样）才能从该容器下载 Blob。如果你要让容器中的 Blob 可供所有人使用，则可以使用以下代码创建容器并传递公共访问级别。

	from azure.storage.blob import PublicAccess
	block_blob_service.create_container('mycontainer', public_access=PublicAccess.Container)

或者，也可以在创建容器后使用以下代码修改该容器。

	block_blob_service.set_container_acl('mycontainer', public_access=PublicAccess.Container)

在此更改后，Internet 上的任何人都可以查看公共容器中的 Blob，但只有您可以修改或删除它们。

## 将 Blob 上载到容器中

若要创建块 Blob 并上载数据，请使用 **create_blob_from_path**、**create_blob_from_stream**、**create_blob_from_bytes** 或 **create_blob_from_text** 方法。这些方法属于高级方法，用于在数据大小超过 64 MB 时执行必要的分块。

**create_blob_from_path** 上载指定路径文件中的内容，**create_blob_from_stream** 上载已打开的文件/流中的内容。**create_blob_from_bytes** 上载字节数组，**create_blob_from_text** 使用指定的编码（默认为 UTF-8）上载指定的文本值。

下面的示例将 **sunset.png** 文件的内容上载到 **myblob** Blob。

	from azure.storage.blob import ContentSettings
	block_blob_service.create_blob_from_path(
        'mycontainer',
        'myblockblob',
        'sunset.png',
        content_settings=ContentSettings(content_type='image/png')
    )

## 列出容器中的 Blob

若要列出容器中的 Blob，请使用 **list_blobs** 方法。此方法会返回一个生成器。以下代码将容器中每个 Blob 的“名称”输出到控制台。

	generator = block_blob_service.list_blobs('mycontainer')
	for blob in generator:
		print(blob.name)

## 下载 Blob

若要从 Blob 下载数据，请使用 **get_blob_to_path**、**get_blob_to_stream**、**get_blob_to_bytes** 或 **get_blob_to_text**。这些方法属于高级方法，用于在数据大小超过 64 MB 时执行必要的分块。

以下示例演示了如何使用 **get_blob_to_path** 下载 **myblob** Blob 的内容，并将其存储到 **out-sunset.png** 文件：

	block_blob_service.get_blob_to_path('mycontainer', 'myblockblob', 'out-sunset.png')

## 删除 Blob

最后，若要删除 Blob，请调用 **delete_blob**。

	block_blob_service.delete_blob('mycontainer', 'myblockblob')

## 写入追加 Blob

追加 Blob 针对追加操作（例如日志记录）进行了优化。类似于块 Blob，追加 Blob 由块组成，但是当你将新的块添加到追加 Blob 时，始终追加到该 Blob 的末尾。你不能更新或删除追加 Blob 中现有的块。追加 Blob 的块 ID 不公开，因为它们是用于一个块 Blob 的。

追加 Blob 中的每个块可以有不同的大小，最大为 4 MB，并且追加 Blob 最多可包含 50000 个块。因此，追加 Blob 的最大大小稍微大于 195 GB（4 MB X 50000 块）。

下面的示例创建一个新的追加 Blob 并向其追加某些数据，模拟一个简单的日志记录操作。

	from azure.storage.blob import AppendBlobService
	append_blob_service = AppendBlobService(account_name='myaccount', account_key='mykey', endpoint_suffix='core.chinacloudapi.cn')

	# The same containers can hold all types of blobs
	append_blob_service.create_container('mycontainer')

	# Append blobs must be created before they are appended to
	append_blob_service.create_blob('mycontainer', 'myappendblob')
	append_blob_service.append_blob_from_text('mycontainer', 'myappendblob', u'Hello, world!')

	append_blob = append_blob_service.get_blob_to_text('mycontainer', 'myappendblob')

## 后续步骤

现在，你已了解 Blob 存储的基础知识，可单击下面的链接了解详细信息。

- [Python 开发人员中心](/develop/python/)
- [Azure 存储空间服务 REST API](http://msdn.microsoft.com/zh-cn/library/azure/dd179355)
- [Azure 存储团队博客]


[Azure 存储空间团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
[Python Azure 包]: https://github.com/Azure/azure-storage-python
[Python Azure 存储服务包]: https://github.com/Azure/azure-storage-python

<!---HONumber=Mooncake_0606_2016-->