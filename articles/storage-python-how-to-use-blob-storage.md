<properties
	pageTitle="如何通过 Python 使用 Azure Blob 存储 | Windows Azure"
	description="了解如何通过 Python 使用 Azure Blob 服务上载、列出、下载和删除 Blob。"
	services="storage"
	documentationCenter="python"
	authors="huguesv"
	manager="wpickett"
	editor=""/>

<tags
	ms.service="storage"
	ms.date="05/11/2015"
	wacn.date="09/18/2015"/>

# 如何通过 Python 使用 Blob 存储服务

[AZURE.INCLUDE [storage-selector-blob-include](../includes/storage-selector-blob-include.md)]

## 概述

本指南将演示如何使用 Blob 存储执行常见方案。相关示例是使用 Python 编写的，并使用 [Python Azure 包][]。涉及的任务包括上载、列出、下载和删除 Blob。

[AZURE.INCLUDE [storage-blob-concepts-include](../includes/storage-blob-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

## 创建容器

> [AZURE.NOTE] 如果您需要安装 Python 或 [Python Azure 包][]，请参阅 [Python 安装指南](/documentation/articles/python-how-to-install)。

使用 **BlobService** 对象可以对容器和 Blob 进行操作。以下代码创建 **BlobService** 对象。在您希望在其中以编程方式访问 Azure 存储空间的任何 Python 文件中，将以下代码添加到文件的顶部附近。

	from azure.storage import BlobService

以下代码使用存储帐户名称和帐户密钥创建一个 **BlobService** 对象。使用实际帐户和密钥替换“myaccount”和“mykey”。

	blob_service = BlobService(account_name='myaccount', account_key='mykey')

[AZURE.INCLUDE [storage-container-naming-rules-include](../includes/storage-container-naming-rules-include.md)]

在以下代码示例中，如果该容器不存在，可以使用 **BlobService** 对象来创建它。

	blob_service.create_container('mycontainer')

默认情况下，新容器是专用容器，因此您必须指定存储访问密钥（如之前所做的那样）才能从该容器下载 Blob。如果您要让容器中的文件可供所有人使用，则可以使用以下代码创建容器并传递公共访问级别。

	blob_service.create_container('mycontainer', x_ms_blob_public_access='container') 

或者，也可以在创建容器后使用以下代码修改该容器。

	blob_service.set_container_acl('mycontainer', x_ms_blob_public_access='container')

在此更改后，Internet 上的任何人都可以查看公共容器中的 Blob，但只有您可以修改或删除它们。

## 将 Blob 上载到容器中

若要将数据上载到 Blob，请使用 **put_block_blob_from_path**、**put_block_blob_from_file**、**put_block_blob_from_bytes** 或 **put_block_blob_from_text** 方法。这些方法属于高级方法，用于在数据大小超过 64 MB 时执行必要的分块。

**put_block_blob_from_path** 上载指定路径中一个文件的内容，**put_block_blob_from_file** 上载已打开的文件/流中的内容。**put_block_blob_from_bytes** 上载一组字节，**put_block_blob_from_text** 使用使用指定的编码（默认为 UTF-8）上载指定的文本值。

下面的示例将 **sunset.png** 文件的内容上载到 **myblob** Blob。

	blob_service.put_block_blob_from_path(
        'mycontainer',
        'myblob',
        'sunset.png',
        x_ms_blob_content_type='image/png'
    )

## 列出容器中的 Blob

若要列出容器中的 Blob，请使用 **list_blobs** 方法。每次调用 **list_blobs** 将返回一段结果。若要获取所有结果，请检查结果的 **next_marker** 并在需要时再次调用 **list_blobs**。以下代码将容器中每个 Blob 的“名称”输出到控制台。

	blobs = []
	marker = None
	while True:
		batch = blob_service.list_blobs('mycontainer', marker=marker)
		blobs.extend(batch)
		if not batch.next_marker:
			break
		marker = batch.next_marker
	for blob in blobs:
		print(blob.name)

## 下载 Blob

每个结果段可以包含可变数目的 Blob，最大为 5000 个。如果存在特定数据段的 **next_marker**，容器中可能有多个 Blob。

若要从 Blob 下载数据，请使用 **get_blob_to_path**、**get_blob_to_file**、**get_blob_to_bytes** 或 **get_blob_to_text**。这些方法属于高级方法，用于在数据大小超过 64 MB 时执行必要的分块。

以下示例演示了如何使用 **get_blob_to_path** 下载 **myblob** Blob 的内容，并将其存储到 **out-sunset.png** 文件：

	blob_service.get_blob_to_path('mycontainer', 'myblob', 'out-sunset.png')

## 删除 Blob

最后，若要删除 Blob，请调用 delete_blob。

	blob_service.delete_blob('mycontainer', 'myblob') 

## 后续步骤

现在，您已了解有关 Blob 存储的基础知识，可单击下面的链接来了解更复杂的存储任务。

-   请参阅 MSDN 参考：[在 Azure 中存储和访问数据][]
-   访问 [Azure 存储空间团队博客][]

  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [Azure 存储空间团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
[Python Azure 包]: https://pypi.python.org/pypi/azure
<!---HONumber=70-->
  
   
