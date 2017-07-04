<properties
    pageTitle="如何通过 Python 使用 Azure 文件存储 | Azure"
    description="了解如何通过 Python 使用 Azure 文件存储上传、列出、下载和删除文件。"
    services="storage"
    documentationcenter="python"
    author="robinsh"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="297f3a14-6b3a-48b0-9da4-db5907827fb5"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="python"
    ms.topic="article"
    ms.date="12/08/2016"
    wacn.date="01/06/2017"
    ms.author="robinsh" />

# 如何通过 Python 使用 Azure 文件存储

[AZURE.INCLUDE [storage-selector-file-include](../../includes/storage-selector-file-include.md)]

## 概述

本文将演示如何使用文件存储执行常见方案。这些示例用 Python 编写并使用 [Azure Storage SDK for Python]。涉及的方案包括上传、列出、下载以及删除文件。

[AZURE.INCLUDE [storage-file-concepts-include](../../includes/storage-file-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

## 创建共享
通过 **FileService** 对象，可使用共享、目录和文件。以下代码创建 **FileService** 对象。在要在其中以编程方式访问 Azure 存储的 Python 文件中，在顶部附近添加以下内容：

	from azure.storage.file import FileService

以下代码使用存储帐户名称和帐户密钥创建 **FileService** 对象。将“myaccount”和“mykey”替换为你的帐户名称和密钥。

	file_service = **FileService** (account_name='myaccount', account_key='mykey',endpoint_suffix='core.chinacloudapi.cn')

在以下代码示例中，如果共享不存在，可以使用 **FileService** 对象来创建它。

	file_service.create_share('myshare')

## 将文件上传到共享

Azure 文件存储共享至少包含文件所在的根目录。本部分介绍如何将文件从本地存储上传到共享的根目录。

若要创建文件并上传数据，请使用 **create\_file\_from\_path**、**create\_file\_from\_stream**、**create\_file\_from\_bytes** 或 **create\_file\_from\_text** 方法。这些方法属于高级方法，将在数据大小超过 64 MB 时执行必要的分块。

**create\_file\_from\_path** 从指定的路径上传文件的内容，而 **create\_file\_from\_stream** 上传已打开的文件/流中的内容。**create\_file\_from\_bytes** 上传字节数组，而 **create\_file\_from\_text** 使用指定的编码（默认为 UTF-8）上传指定的文本值。

下面的示例将 **sunset.png** 文件的内容上传到 **myfile** 文件中。

	from azure.storage.file import ContentSettings
	file_service.create_file_from_path(
        'myshare',
        None, # We want to create this blob in the root directory, so we specify None for the directory_name
        'myfile',
        'sunset.png',
        content_settings=ContentSettings(content_type='image/png'))

## 如何：创建目录

也可将文件置于子目录中，而不将其全部置于根目录中，以便对存储进行有效的组织。Azure 文件存储服务允许创建帐户所允许任意数目的目录。以下代码将在根目录下创建名为 **sampledir** 的子目录。

	file_service.create_directory('myshare', 'sampledir')

## 如何：列出共享中的文件和目录

若要列出共享中的文件和目录，请使用 **list\_directories\_and\_files** 方法。此方法会返回一个生成器。以下代码将共享中每个文件和目录的**名称**输出到控制台。

	generator = file_service.list_directories_and_files('myshare')
	for file_or_dir in generator:
		print(file_or_dir.name)

## 下载文件

若要从文件中下载数据，请使用 **get\_file\_to\_path**、**get\_file\_to\_stream**、**get\_file\_to\_bytes** 或 **get\_file\_to\_text**。这些方法属于高级方法，将在数据大小超过 64 MB 时执行必要的分块。

以下示例演示了如何使用 **get\_file\_to\_path** 下载 **myfile** 文件的内容，并将其存储到 **out-sunset.png** 文件。

	file_service.get_file_to_path('myshare', None, 'myfile', 'out-sunset.png')

## 删除文件

最后，若要删除文件，请调用 **delete\_file**。

	file_service.delete_file('myshare', None, 'myfile')

## 后续步骤

现在，你已了解文件存储的基础知识，可单击下面的链接了解详细信息。

- [Python 开发人员中心](/develop/python/)
- [Azure 存储服务 REST API](http://msdn.microsoft.com/zh-cn/library/azure/dd179355)
- [Azure 存储团队博客]
- [Azure Storage SDK for Python]

[Azure 存储团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
[Azure Storage SDK for Python]: https://github.com/Azure/azure-storage-python

<!---HONumber=Mooncake_0103_2017-->