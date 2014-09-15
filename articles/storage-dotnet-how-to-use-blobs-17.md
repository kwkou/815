<properties linkid="dev-net-2-how-to-blob-storage" urlDisplayName="Blob Service (2.0)" pageTitle="How to use blob storage in .NET | Microsoft Azure" metaKeywords="Get started Azure blob, Azure unstructured data, Azure unstructured storage, Azure blob, Azure blob storage, Azure blob .NET, Azure blob C#, Azure blob C#" description="Learn how to use the Azure blob service to upload,,download, list, and delete blob content. Samples are written in C#." metaCanonical="" services="storage" documentationCenter=".NET" title="How to use the Azure Blob Storage Service in .NET" authors="" solutions="" manager="paulettm" editor="cgronlun" />

# 如何在 .NET 中使用 Azure Blob 存储服务

[1.7 版][] [2.0 版][]

本指南将演示如何使用 Azure Blob 存储服务执行常见方案。
示例是用 C\# 编写的且使用
了 .NET API。涉及的任务包括
**上载**、**列出**、**下载**和**删除** Blob。有关 Blob 的
详细信息，请参阅[后续步骤][]部分。

## 目录

-   [什么是 Blob 存储][]
-   [概念][]
-   [创建 Azure 存储帐户][]
-   [设置存储连接字符串][]
-   [如何：以编程方式访问 Blob 存储][]
-   [如何：创建容器][]
-   [如何：将 Blob 上载到容器中][]
-   [如何：列出容器中的 Blob][]
-   [如何：下载 Blob][]
-   [如何：删除 Blob][]
-   [后续步骤][]

[WACOM.INCLUDE [howto-blob-storage][]]

## 创建帐户创建 Azure 存储帐户

[WACOM.INCLUDE [create-storage-account][]]

## 设置连接字符串设置存储连接字符串

Azure .NET 存储 API 支持
使用存储连接字符串来配置用于访问存储
服务的终结点和凭据。你可以将存储连接字符串放置在一个配置文件中，而不是
在代码中对其进行硬编码：

-   当使用 Azure 云服务时，建议你使用 Azure 服务配置系统（`*.csdef` 和 `*.cscfg` 文件）来存储连接字符串。
-   在使用 Azure 网站或 Azure 虚拟机时，建议使用 .NET 配置系统（如 `web.config` 文件）来存储连接字符串。

在上述两种情况下，你都可以使用 `CloudConfigurationManager.GetSetting` 方法检索连接字符串，本指南稍后将对此进行介绍。

### 在使用云服务时配置连接字符串

该服务配置机制是 Azure 云服务
项目特有的，它使你能够从 Azure 管理
门户动态更改配置设置，而无需部署你的应用程序。

在 Azure 服务配置中配置连接字符串：

1.  在 Visual Studio 解决方案资源管理器内 Azure 部署项目的
    **“角色”**文件夹中，右键单击你的 Web 角色或
    辅助角色，然后单击**“属性”**。
    ![在 Visual Studio 中选择云服务角色的属性][]

2.  单击**“设置”**选项卡并按**“添加设置”**按钮。
    ![在 Visual Studio 中添加云服务设置][]

    新的 **Setting1** 条目稍后将显示在设置网格中。

3.  在新的 **Setting1** 条目的**“类型”**下拉列表中，选择
    **“连接字符串”**。
    ![Blob7][]

4.  单击 **Setting1** 条目最右侧的 **...** 按钮。
    此时将打开**“存储帐户连接字符串”**对话框。

5.  选择是要定位到存储模拟器（在本地计算机上模拟的 Windows
    Azure 存储空间），还是要定位到云中的实际存储帐户。
    本指南中的代码适用于其中任一方式。
    如果你希望使用我们之前在 Azure 中创建的存储帐户
    来存储 Blob 数据，请输入从本教程前面的步骤中
    复制的**“主访问密钥”**值。
    ![Blob8][]

6.  将条目**“名称”**从 **Setting1** 更改为更友好的名称，
    例如 **StorageConnectionString**。在本指南后面的
    代码中你将引用此连接字符串。
    ![Blob9][]

### 在使用网站或虚拟机时配置连接字符串

在使用网站或虚拟机时，建议你使用 .NET 配置系统（如 `web.config`）。你可以使用 `<appSettings>` 元素存储连接字符串：

    <configuration>
    <appSettings>
    <add key="StorageConnectionString"
    value="DefaultEndpointsProtocol=https;AccountName=[AccountName];AccountKey=[AccountKey]" />
    </appSettings>
    </configuration>

阅读[配置连接字符串][]，了解有关存储连接字符串的详细信息。

你现在即可准备执行本指南中的操作任务。

## 以编程方式访问如何：以编程方式访问 Blob 存储

在你希望在其中以编程方式访问 Azure 存储空间的任何 C\# 文件中，
将以下代码命名空间声明添加到文件的顶部：

    using Microsoft.WindowsAzure;
    using Microsoft.WindowsAzure.StorageClient;

你可以使用 **CloudStorageAccount** 类型和
**CloudConfigurationManager** 类型从
Azure 服务配置中检索你的存储连接字符串和存储帐户信息：

    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

你可以使用 **CloudBlobClient** 类型来检索表示存储在 Blob 存储服务
中的容器和 Blob 的对象。以下代码
使用我们在上面检索到的存储帐户对象创建 **CloudBlobClient**
对象：

    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

## 创建容器如何：创建容器

所有存储 Blob 都驻留在一个容器中。你可以使用
**CloudBlobClient** 对象来获取对你要使用的容器的引用。
如果该容器不存在，你可以创建它：

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建 Blob 客户端 
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // 检索对容器的引用 
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

    // 如果该容器不存在，则创建它
    container.CreateIfNotExist();

默认情况下，新容器是专用容器，因此你必须指定存储访问密钥
（如上面所做的那样）才能从该容器下载 Blob。
如果你要让容器中
的文件可供所有人使用，则可以使用以下代码
将容器设置为公共容器：

    container.SetPermissions(
    new BlobContainerPermissions { PublicAccess = 
    BlobContainerPublicAccessType.Blob }); 

Internet 上的所有人都可以查看公共容器中的 Blob，但必须
提供相应的访问密钥时才能修改或删除它们。

## 上载到容器如何：将 Blob 上载到容器中

若要将文件上载到 Blob，请获取容器引用，并使用它获取 Blob 引用。
获取 Blob 引用后，可以通过对该 Blob 引用调用
**UploadFromStream** 方法，将任何数据流上载
到该引用。此操作将创建 Blob（如果该 Blob 不存在），
或者覆盖它（如果该 Blob 存在）。下面的代码示例演示了这一点，并假定已创建容器。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建 Blob 客户端
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // 检索对以前创建的容器的引用
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

    // 检索对名为“myblob”的 Blob 的引用
    CloudBlob blob = container.GetBlobReference("myblob");

    // 使用来自本地文件的内容创建或覆盖“myblob”Blob
    using (var fileStream = System.IO.File.OpenRead(@"path\myfile"))
    {
    blob.UploadFromStream(fileStream);
    } 

## 列出容器中的 Blob如何：列出容器中的 Blob

若要列出容器中的 Blob，首先需要获取容器引用。然后，
你可以使用容器的 **ListBlobs** 方法检索其中的 Blob。
以下代码演示了如何检索和输出容器中每个 Blob 的
**Uri：**

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建 Blob 客户端
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // 检索对以前创建的容器的引用
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

    // 循环访问容器内的 Blob 并且输出每个的 URI
    foreach (var blobItem in container.ListBlobs())
    {
    Console.WriteLine(blobItem.Uri);
    } 

还可将 Blob 服务视为容器中的目录。
这是为了让你能够以更类似于文件夹的结构来组织 Blob。
例如，你可以创建一个名为“photos”的容器，在其中
上载名为“rootphoto1”、“2010/photo1”、“2010/photo2”和
“2011/photo1”的 Blob。这实际上将在“photos”容器中创建
目录“2010”和“2011”。当你对“photos”容器
调用 **ListBlobs** 时，返回的集合将包含表示最高层中
所含目录和 Blob 的 **CloudBlobDirectory** 和 **CloudBlob**
对象。在本例中，将返回目录
“2010”和“2011”以及照片“rootphoto1”。
或者，也可以通过将
**UseFlatBlobListing** 设置为
**true**，传入新的
**BlobRequestOptions** 类。这将导致返回每个 Blob，而无论目录如何。
有关详细信息，请参阅 [CloudBlobContainer.ListBlobs][]。

## 下载 Blob如何：下载 Blob

若要下载 Blob，请首先检索 Blob 引用。以下示例
使用 **DownloadToStream** 方法将 Blob 内容传输到一个流对象，然后
你可以将该对象保存到本地文件。你还可以调用
Blob 的 **DownloadToFile**、**DownloadByteArray** 或
 **DownloadText** 方法。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建 Blob 客户端
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // 检索对以前创建的容器的引用
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

    // 检索对名为“myblob”的 Blob 的引用
    CloudBlob blob = container.GetBlobReference("myblob");

    // 将 Blob 内容保存到磁盘
    using (var fileStream = System.IO.File.OpenWrite(@"path\myfile"))
    {
    blob.DownloadToStream(fileStream);
    } 

## 删除 Blob如何：删除 Blob

最后，若要删除 Blob，请获取 Blob 引用，然后对其调用
**Delete** 方法。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建 Blob 客户端
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // 检索对以前创建的容器的引用
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

    // 检索对名为“myblob”的 Blob 的引用
    CloudBlob blob = container.GetBlobReference("myblob");

    // 删除 Blob
    blob.Delete(); 

## 后续步骤后续步骤

现在，你已了解有关 Blob 存储的基础知识，可单击下面的链接来了解
如何执行更复杂的存储任务。

-   查看 Blob 服务参考文档，了解有关可用 API 的完整详细信息：
    -   [.NET 客户端库引用][]
    -   [REST API 参考][]

-   在以下位置了解使用 Azure 存储空间能够执行的更高级任务：[在 Azure 中存储和访问数据][]。
-   查看更多功能指南，以了解在 Azure 中存储数据的其他方式。
    -   使用[表存储][]来存储结构化数据。
    -   使用 [SQL Database][] 来存储关系数据。

  [1.7 版]: /en-us/develop/net/how-to-guides/blob-storage-v17/ "1.7 版"
  [2.0 版]: /en-us/develop/net/how-to-guides/blob-storage/ "2.0 版"
  [后续步骤]: #next-steps
  [什么是 Blob 存储]: #what-is
  [概念]: #concepts
  [创建 Azure 存储帐户]: #create-account
  [设置存储连接字符串]: #setup-connection-string
  [如何：以编程方式访问 Blob 存储]: #configure-access
  [如何：创建容器]: #create-container
  [如何：将 Blob 上载到容器中]: #upload-blob
  [如何：列出容器中的 Blob]: #list-blob
  [如何：下载 Blob]: #download-blobs
  [如何：删除 Blob]: #delete-blobs
  [howto-blob-storage]: ../includes/howto-blob-storage.md
  [create-storage-account]: ../includes/create-storage-account.md
  [在 Visual Studio 中选择云服务角色的属性]: ./media/storage-dotnet-how-to-use-blobs-17/blob5.png
  [在 Visual Studio 中添加云服务设置]: ./media/storage-dotnet-how-to-use-blobs-17/blob6.png
  [Blob7]: ./media/storage-dotnet-how-to-use-blobs-20/blob7.png
  [Blob8]: ./media/storage-dotnet-how-to-use-blobs-20/blob8.png
  [Blob9]: ./media/storage-dotnet-how-to-use-blobs-20/blob9.png
  [配置连接字符串]: http://msdn.microsoft.com/zh-cn/library/azure/ee758697.aspx
  [.NET 客户端库引用]: http://msdn.microsoft.com/zh-cn/library/azure/wl_svchosting_mref_reference_home
  [REST API 参考]: http://msdn.microsoft.com/zh-cn/library/azure/dd179355
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [表存储]: /en-us/develop/net/how-to-guides/table-services/
  [SQL Database]: /en-us/develop/net/how-to-guides/sql-database/
