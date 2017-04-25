<properties
    pageTitle="如何通过 Python 使用 Azure Blob 存储（对象存储）| Azure"
    description="使用 Azure Blob 存储（对象存储）将非结构化数据存储在云中。"
    services="storage"
    documentationcenter="python"
    author="mmacy"
    manager="timlt"
    editor="tysonn"
    translationtype="Human Translation" />
<tags
    ms.assetid="0348e360-b24d-41fa-bb12-b8f18990d8bc"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="python"
    ms.topic="article"
    ms.date="2/24/2017"
    wacn.date="04/24/2017"
    ms.author="marsma"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="1f4255117ae4cbbf51f332e1b7a915cc8e1de7a1"
    ms.lasthandoff="04/14/2017" />

# <a name="how-to-use-azure-blob-storage-from-python"></a>如何通过 Python 使用 Azure Blob 存储
[AZURE.INCLUDE [storage-selector-blob-include](../../includes/storage-selector-blob-include.md)]


## <a name="overview"></a>概述
Azure Blob 存储是一种将非结构化数据作为对象/Blob 存储在云中的服务。 Blob 存储可以存储任何类型的文本或二进制数据，例如文档、媒体文件或应用程序安装程序。 Blob 存储也称为对象存储。

本指南将演示如何使用 Blob 存储执行常见方案。 这些示例用 Python 编写并使用 [Azure Storage SDK for Python]。 涉及的任务包括上传、列出、下载和删除 Blob。

[AZURE.INCLUDE [storage-blob-concepts-include](../../includes/storage-blob-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

## <a name="create-a-container"></a>创建容器
根据要使用的 Blob 的类型，创建 **BlockBlobService**、**AppendBlobService** 或 **PageBlobService** 对象。 以下代码使用 **BlockBlobService** 对象。 在希望在其中以编程方式访问 Azure 块 Blob 存储的任何 Python 文件中，将以下代码添加到文件的顶部附近。

    from azure.storage.blob import BlockBlobService

以下代码使用存储帐户名称和帐户密钥创建一个 **BlockBlobService** 对象。  使用帐户名称和密钥替换“myaccount”和“mykey”。

    block_blob_service = BlockBlobService(account_name='myaccount', account_key='mykey', endpoint_suffix='core.chinacloudapi.cn')

[AZURE.INCLUDE [storage-container-naming-rules-include](../../includes/storage-container-naming-rules-include.md)]

在以下代码示例中，如果容器不存在，可以使用 **BlockBlobService** 对象进行创建。

    block_blob_service.create_container('mycontainer')

默认情况下，新容器是专用容器，因此必须指定存储访问密钥（如之前所做的那样）才能从该容器下载 Blob。 如果要让容器中的 Blob 可供所有人使用，则可以使用以下代码创建容器并传递公共访问级别。

    from azure.storage.blob import PublicAccess
    block_blob_service.create_container('mycontainer', public_access=PublicAccess.Container)

或者，也可以在创建容器后使用以下代码修改该容器。

    block_blob_service.set_container_acl('mycontainer', public_access=PublicAccess.Container)

在此更改后，Internet 上的任何人都可以查看公共容器中的 Blob，但只有你可以修改或删除它们。

## <a name="upload-a-blob-into-a-container"></a>将 Blob 上传到容器中
若要创建块 blob 和上传数据，请使用 **create\_blob\_from\_path**、**create\_blob\_from\_stream**、**create\_blob\_from\_bytes** 或 **create\_blob\_from\_text** 方法。 这些方法属于高级方法，用于在数据大小超过 64 MB 时执行必要的分块。

**create\_blob\_from\_path** 用于从指定位置上传文件内容，**create\_blob\_from\_stream** 用于从已经打开的文件/流上传内容。 **create\_blob\_from\_bytes** 用于上传一组字节，**create\_blob\_from\_text** 使用指定的编码（默认为 UTF-8）上传指定的文本值。

下面的示例将 **sunset.png** 文件的内容上传到 **myblob** Blob。

    from azure.storage.blob import ContentSettings
    block_blob_service.create_blob_from_path(
        'mycontainer',
        'myblockblob',
        'sunset.png',
        content_settings=ContentSettings(content_type='image/png')
                )

## <a name="list-the-blobs-in-a-container"></a>列出容器中的 Blob
若要列出容器中的 blob，请使用 **list\_blobs** 方法。 此方法会返回一个生成器。 以下代码将容器中每个 Blob 的“名称”  输出到控制台。

    generator = block_blob_service.list_blobs('mycontainer')
    for blob in generator:
        print(blob.name)

## <a name="download-blobs"></a>下载 Blob
若要从 blob 下载数据，请使用 **get\_blob\_to\_path**、**get\_blob\_to\_stream**、**get\_blob\_to\_bytes** 或 **get\_blob\_to\_text**。 这些方法属于高级方法，用于在数据大小超过 64 MB 时执行必要的分块。

下面的示例演示了如何使用 **get\_blob\_to\_path** 下载 **myblob** blob 的内容，并将其存储到 **out-sunset.png** 文件。

    block_blob_service.get_blob_to_path('mycontainer', 'myblockblob', 'out-sunset.png')

## <a name="delete-a-blob"></a>删除 Blob
最后，若要删除 Blob，请调用 **delete_blob**。

    block_blob_service.delete_blob('mycontainer', 'myblockblob')

## <a name="writing-to-an-append-blob"></a>写入追加 Blob
追加 Blob 针对追加操作（例如日志记录）进行了优化。 类似于块 Blob，追加 Blob 由块组成，但是当你将新的块添加到追加 Blob 时，始终追加到该 Blob 的末尾。 你不能更新或删除追加 Blob 中现有的块。 追加 Blob 的块 ID 不公开，因为它们是用于一个块 Blob 的。

追加 Blob 中的每个块可以有不同的大小，最大为 4 MB，并且追加 Blob 最多可包含 50000 个块。 因此，追加 Blob 的最大大小稍微大于 195 GB（4 MB X 50000 块）。

下面的示例创建一个新的追加 Blob 并向其追加某些数据，模拟一个简单的日志记录操作。

    from azure.storage.blob import AppendBlobService
    append_blob_service = AppendBlobService(account_name='myaccount', account_key='mykey', endpoint_suffix='core.chinacloudapi.cn')

    # The same containers can hold all types of blobs
    append_blob_service.create_container('mycontainer')

    # Append blobs must be created before they are appended to
    append_blob_service.create_blob('mycontainer', 'myappendblob')
    append_blob_service.append_blob_from_text('mycontainer', 'myappendblob', u'Hello, world!')

    append_blob = append_blob_service.get_blob_to_text('mycontainer', 'myappendblob')

## <a name="next-steps"></a>后续步骤
在了解了 Blob 存储的基础知识后，可单击下面的链接了解详细信息。

- [Python 开发人员中心](/develop/python/)
- [Azure 存储服务 REST API](http://msdn.microsoft.com/zh-cn/library/azure/dd179355)
- [Azure 存储团队博客]
- [Azure Storage SDK for Python]

[Azure 存储团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
[Azure Storage SDK for Python]: https://github.com/Azure/azure-storage-python
<!--Update_Description: wording update;add anchors to H2 titles-->
