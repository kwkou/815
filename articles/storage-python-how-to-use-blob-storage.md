<properties linkid="develop-python-blob-service" urlDisplayName="Blob Service" pageTitle="How to use blob storage (Python) | Microsoft Azure" metaKeywords="Azure blob service Python, Azure blobs Python" description="Learn how to use the Azure Blob service to upload, list, download, and delete blobs." metaCanonical="" disqusComments="1" umbracoNaviHide="0" services="storage" documentationCenter="Python" title="How to use the Blob service from Python" authors="" videoId="" scriptId="" />

# 如何从 Python 使用 Blob 存储服务

本指南将演示如何使用 Azure Blob 存储服务执行常见方案。
示例是用 Python API 编写的。
涉及的任务包括**上载**、**列出**、
**下载**和**删除** Blob。有关 Blob 的
详细信息，请参阅[后续步骤][]部分。

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

[WACOM.INCLUDE [howto-blob-storage][]]

## 创建 Azure 存储帐户

[WACOM.INCLUDE [create-storage-account][]]

## 如何：创建容器

**注意：**如果你需要安装 Python 或客户端库，请参阅 [Python 安装指南][]。

使用 **BlobService** 对象可以对容器和 Blob 进行操作。以下代码
将创建一个 **BlobService** 对象。在你希望在其中以
编程方式访问 Azure 存储空间的任何 Python 文件中，将以下代码添加到文件的顶部附近：

    from azure.storage import *

以下代码使用存储帐户名称和帐户密钥创建一个 **BlobService** 对象。使用实际帐户和密钥替换“myaccount”和“mykey”。

    blob_service = BlobService(account_name='myaccount', account_key='mykey')

所有存储 Blob 都驻留在一个容器中。如果该容器不存在，可以使用 **BlobService** 对象来创建它：

    blob_service.create_container('mycontainer')

默认情况下，新容器是专用容器，因此你必须指定存储访问密钥（如上面所做的那样）才能从该容器下载 Blob。如果你要让容器中的文件可供所有人使用，则可以使用以下代码创建容器并传递公共访问级别：

    blob_service.create_container('mycontainer', x_ms_blob_public_access='container') 

或者，也可以在创建容器后使用以下代码修改该容器：

    blob_service.set_container_acl('mycontainer', x_ms_blob_public_access='container')

在执行此更改后，Internet 上的任何人都可以看到公共容器中的 Blob，
但只有你可以修改或删除它们。

## 如何：将 Blob 上载到容器

若要将文件上载到 Blob，请使用 **put\_blob** 方法
来创建 Blob，将文件流用作 Blob 的内容。首先，
创建一个名为 **task1.txt** 的文件（包含任意内容都可以）
并将其与你的 Python 文件存储在同一目录中。

    myblob = open(r'task1.txt', 'r').read()
    blob_service.put_blob('mycontainer', 'myblob', myblob, x_ms_blob_type='BlockBlob')

## 如何：列出容器中的 Blob

若要列出容器中的 Blob，可使用带 **for** 循环的
**list\_blobs** 方法来显示容器中每个 Blob 的名称。以下
代码将容器中每个 Blob 的**名称**和 **url** 输出到
控制台。

    blobs = blob_service.list_blobs('mycontainer')
    for blob in blobs:
    print(blob.name)
    print(blob.url)

## 如何：下载 Blob

若要下载 Blob，请使用 **get\_blob** 方法将 Blob 内容
传输到一个流对象，稍后可以将该对象永久保存到一个
本地文件。

    blob = blob_service.get_blob('mycontainer', 'myblob')
    with open(r'out-task1.txt', 'w') as f:
    f.write(blob)

## 如何：删除 Blob

最后，若要删除 Blob，请调用 **delete\_blob**。

    blob_service.delete_blob('mycontainer', 'myblob') 

## 如何：上载和下载大型 Blob

块 Blob 的最大大小为 200 GB。对于小于 64 MB 的 Blob，可以通过调用一次 **put\_blob** 或 **get\_blob** 来上载或下载 Blob，如前所述。对于大于 64 MB 的 Blob，需要以 4 MB 或更小的块为单位上载或下载 Blob。

以下代码演示用于上载或下载任意大小的块 Blob 的函数示例。

    import base64

    chunk_size = 4 * 1024 * 1024

    def upload(blob_service, container_name, blob_name, file_path):
    blob_service.create_container(container_name, None, None, False)
    blob_service.put_blob(container_name, blob_name, '', 'BlockBlob')

    block_ids = []
    index = 0
    with open(file_path, 'rb') as f:
    while True:
    data = f.read(chunk_size)
    if data:
    length = len(data)
    block_id = base64.b64encode(str(index))
    blob_service.put_block(container_name, blob_name, data, block_id)
    block_ids.append(block_id)
    index += 1
    else:
    break

    blob_service.put_block_list(container_name, blob_name, block_ids)

    def download(blob_service, container_name, blob_name, file_path):
    props = blob_service.get_blob_properties(container_name, blob_name)
    blob_size = int(props['content-length'])

    index = 0
    with open(file_path, 'wb') as f:
    while index < blob_size:
    chunk_range = 'bytes={}-{}'.format(index, index + chunk_size - 1)
    data = blob_service.get_blob(container_name, blob_name, x_ms_range=chunk_range)
    length = len(data)
    index += length
    if length > 0:
    f.write(data)
    if length < chunk_size:
    break
    else:
    break

如果你需要大于 200 GB 的 Blob，则可以使用页 Blob 而非块 Blob。页 Blob 的最大大小为 1 TB，其中的页与 512 字节页边界对齐。使用 **put\_blob** 可创建页 Blob，使用 **put\_page** 可向其中写入内容，而使用 **get\_blob** 可从中读取内容。

## 后续步骤

现在，你已了解了有关 Blob 存储的基础知识，可单击下面的链接来了解
如何执行更复杂的存储任务。

-   查看 MSDN 参考：[在 Azure 中存储和访问数据][]
-   访问 [Azure 存储空间团队博客][]

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
  [howto-blob-storage]: ../includes/howto-blob-storage.md
  [create-storage-account]: ../includes/create-storage-account.md
  [Python 安装指南]: ../python-how-to-install/
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [Azure 存储空间团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
