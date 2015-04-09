<properties linkid="develop-python-blob-service" urlDisplayName="Blob Service" pageTitle="如何使用 Blob 存储 (Python) | Microsoft Azure" metaKeywords="Azure blob service Python, Azure blobs Python" description="了解如何使用 Azure Blob 服务上载、列出、下载和删除 Blob。" metaCanonical="" disqusComments="1" umbracoNaviHide="0" services="storage" documentationCenter="Python" title="How to use the Blob service from Python" authors="" videoId="" scriptId="" />

# 如何从 Python 使用 Blob 存储服务
本指南将演示如何使用 Azure 表存储服务执行常见方案。相关示例是使用Python API 编写的。涉及的方案包括**上载**、**列出**、**下载**和**删除** Blob。有关 Blob 的详细信息，请参阅[后续步骤][]部分。

## 目录

[什么是 Blob 存储？][]   
 [概念][]   
 [创建 Azure 存储帐户][]   
 [如何：创建容器][]   
 [如何：将 Blob 上载到容器][]   
 [如何：列出容器中的 Blob][]   
 [如何：下载 Blob][]   
 [如何：删除 Blob][]   
 [如何：上载和下载大型 Blob][]   
 [后续步骤][]

[WACOM.INCLUDE [howto-blob-storage](../includes/howto-blob-storage.md)]

## <a name="create-account"> </a>创建 Azure 存储帐户

[WACOM.INCLUDE [create-storage-account](../includes/create-storage-account.md)]

## <a name="create-container"> </a>如何：创建容器

**注意：**如果你需要安装 Python 或客户端库，请参阅 [Python 安装指南](../python-how-to-install/)。


使用 **BlobService** 对象可以对容器和 Blob 进行操作。以下代码创建 **BlobService** 对象。在你希望在其中以编程方式访问 Azure 存储空间的任何 Python 文件中，将以下代码添加到文件的顶部附近：

	from azure.storage import *

以下代码使用存储帐户名称和帐户密钥创建一个 BlobService 对象。使用实际帐户和密钥替换  'myaccount' 和  'mykey'。

	blob_service = BlobService(account_name='myaccount', account_key='mykey')

所有存储 Blob 都驻留在一个容器中。如果该容器不存在，可以使用 BlobService 对象来创建它：

	blob_service.create_container('mycontainer')

默认情况下，新容器是专用容器，因此你必须指定存储访问密钥（如上面所做的那样）才能从该容器下载 Blob。如果你要让容器中的文件可供所有人使用，则可以使用以下代码创建容器并传递公共访问级别：

	blob_service.create_container('mycontainer', x_ms_blob_public_access='container') 

或者，也可以在创建容器后使用以下代码修改该容器：

	blob_service.set_container_acl('mycontainer', x_ms_blob_public_access='container')

在此更改后，Internet 上的任何人都可以查看公共容器中的 Blob，但只有您可以修改或删除它们。

## <a name="upload-blob"> </a>如何：将 Blob 上载到容器

若要将数据上载到 Blob，请使用 **put\_block\_blob\_from\_path**、**put\_block\_blob\_from\_file**、**put\_block\_blob\_from\_bytes** 或 **put\_block\_blob\_from\_text** 方法。这些方法属于高级方法，用于在数据大小超过 64 MB 时执行必要的分块。

**put\_block\_blob\_from\_path** 用于从指定路径上载文件的内容，**put\_block\_blob\_from\_file** 用于从已经打开的文件/系统上载内容。**put\_block\_blob\_from\_bytes** 用于上载一组字节，**put\_block\_blob\_from\_text** 可使用指定的编码（默认为 UTF-8）上载指定的文本值。

下面的示例将 test1.txt 文件的内容上载到"test1"Blob。

	blob_service.put_block_blob_from_path('mycontainer', 'myblob', 'task1.txt')

## <a name="list-blob"> </a>如何：列出容器中的 Blob

若要列出容器中的 Blob，请使用 **list\_blobs** 方法与
**for** 循环来显示容器中每个 Blob 的名称。以下代码将容器中每个 Blob 的名称和 url 输出到控制台。

	blobs = blob_service.list_blobs('mycontainer')
	for blob in blobs:
		print(blob.name)
		print(blob.url)

## <a name="download-blobs"> </a>如何：下载 Blob

若要从 Blob 下载数据，请使用 **get\_blob\_to\_path**、**get\_blob\_to\_file**、**get\_blob\_to\_bytes** 或 **get\_blob\_to\_text**。这些方法属于高级方法，用于在数据大小超过 64 MB 时执行必要的分块。

以下示例演示了如何使用 **get\_blob\_to\_path** 下载 **myblob** Blob 的内容，并将其存储到 **out-task1.txt** 文件：

	blob_service.get_blob_to_path('mycontainer', 'myblob', 'out-task1.txt')

## <a name="delete-blobs"> </a>如何：删除 Blob

最后，若要删除 Blob，请调用 delete_blob。

	blob_service.delete_blob('mycontainer', 'myblob') 

## <a name="next-steps"> </a>后续步骤

现在，你已了解有关 Blob 存储的基础知识，可单击下面的链接来了解如何执行更复杂的存储任务。

-   查看 MSDN 参考：[在 Azure 中存储和访问数据][]
-   访问 [Azure 存储团队博客][]

  [后续步骤]: #next-steps
  [什么是 Blob 存储？]: #what-is
  [概念]: #concepts
  [创建 Azure 存储帐户]: #create-account
  [如何：创建容器]: #create-container
  [如何：将 Blob 上载到容器]: #upload-blob
  [如何：列出容器中的 Blob]: #list-blob
  [如何：下载 Blob]: #download-blobs
  [如何：删除 Blob]: #delete-blobs
  [如何：上载和下载大型 Blob]: #large-blobs
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [Azure 存储团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
<!--HONumber=41-->
  
   
