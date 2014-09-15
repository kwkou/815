<properties linkid="dev-net-how-to-blob-storage" urlDisplayName="Blob Service" pageTitle="How to use blob storage from .NET | Microsoft Azure" metaKeywords="Get started Azure blob   Azure unstructured data   Azure unstructured storage   Azure blob   Azure blob storage   Azure blob .NET   Azure blob C#   Azure blob C#" description="Learn how to use Microsoft Azure Blob storage to upload,  download, list, and delete blob content. Samples are written in C#." metaCanonical="" disqusComments="1" umbracoNaviHide="1" services="storage" documentationCenter=".NET" title="How to use Microsoft Azure Blob storage in .NET" authors="tamram" manager="mbaldwin" editor="cgronlun" />

# 如何通过 .NET 使用 Blob 存储

本指南将演示如何使用 Azure Blob 存储服务执行常见方案。
示例是用 C\# 编写的并
使用了 Azure .NET 存储客户端库。涉及的任务包括
**上载**、**列出**、**下载**和**删除** Blob。有关 Blob 的
详细信息，请参阅[后续步骤][]部分。

> [WACOM.NOTE] 本指南适用于 Azure .NET 存储客户端库 2.x 及更高版本。建议使用的版本是存储客户端库 3.x，可通过 NuGet 或 Azure SDK for .NET 2.3 获得。有关如何获取存储客户端库的详细信息，请参阅[如何：以编程方式访问 Blob 存储][]。

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

Azure .NET 存储客户端库支持使用存储连接字符
串来配置终结点和用于访问存
储服务的凭据。你可以将存储连接字符串放置在一个配置文件中，而不是
在代码中对其进行硬编码：

-   当使用 Azure 云服务时，建议你使用 Azure 服务配置系统（`*.csdef` 和 `*.cscfg` 文件）来存储连接字符串。
-   在使用 Azure 网站、Azure 虚拟机时或者构建准备在 Azure 外部运行的 .NET 应用程序时，建议你使用 .NET 配置系统（如 `web.config` 或 `app.config` 文件）来存储连接字符串。

稍后将在本指南中介绍如何检索连接字符串。

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
    Azure 存储空间），还是要定位到云中的存储帐户。
    本指南中的代码适用于其中任一方式。
    如果你希望使用我们之前在 Azure 中创建的存储帐户
    来存储 Blob 数据，请输入从本教程前面的步骤中
    复制的**“主访问密钥”**值。
    ![Blob8][]

6.  将条目**“名称”**从 **Setting1** 更改为更友好的名称，
    例如 **StorageConnectionString**。在本指南后面的
    代码中你将引用此连接字符串。
    ![Blob9][]

### 使用 .NET 配置来配置连接字符串

如果你正在编写不是 Azure 云服务的应用程序（参见上一部分），则建议你使用 .NET 配置系统（如 `web.config` 或 `app.config`）。这包括 Azure 网站或 Azure 虚拟机，以及设计为在 Azure 外部运行的应用程序。你可以使用 `<appSettings>` 元素存储连接字符串，如下所示：

    <configuration>
    <appSettings>
    <add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=[AccountName];AccountKey=[AccountKey" />
    </appSettings>
    </configuration>

阅读[配置连接字符串][]，了解有关存储连接字符串的详细信息。

你现在即可准备执行本指南中的操作任务。

## 以编程方式访问如何：以编程方式访问 Blob 存储

### 获得程序集

你可以使用 NuGet 来获得 `Microsoft.WindowsAzure.Storage.dll` 程序集。在**“解决方案资源管理器”**中，右键单击你的项目并选择**“管理 NuGet 包”**。在线搜索“WindowsAzure.Storage”，然后单击**“安装”**以安装 Azure 存储包和依赖项。

Azure SDK for .NET 中也包括了 `Microsoft.WindowsAzure.Storage.dll`，可从 [.NET 开发人员中心][]下载该版本。该程序集将安装到 `%Program Files%\Microsoft SDKs\Windows Azure\.NET SDK\<sdk-version>\ref\` 目录中。

### 命名空间声明

在你希望在其中以编程方式访问 Azure 存储空间的任何 C\# 文件中，
将以下命名空间声明添加到文件的顶部：

    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Auth;
    using Microsoft.WindowsAzure.Storage.Blob;

确保你引用了 `Microsoft.WindowsAzure.Storage.dll` 程序集。

### 检索连接字符串

可以使用 **CloudStorageAccount** 类型来表示你的存储
帐户信息。如果你使用的
是 Azure 项目模板并且/或者引用了
Microsoft.WindowsAzure.CloudConfigurationManager，
则可以使用 **CloudConfigurationManager** 类型
从 Azure 服务配置中检索你的存储连接字符串和
存储帐户信息：

    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

如果你要创建应用程序而不引用 Microsoft.WindowsAzure.CloudConfigurationManager，并且你的连接字符串位于 `web.config` 或 `app.config` 中（如上所示），则你可以使用 **ConfigurationManager** 来检索连接字符串。你需要将对 System.Configuration.dll 的引用添加到你的项目中，并为其添加另一个命名空间声明：

    using System.Configuration;
    ...
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    ConfigurationManager.ConnectionStrings["StorageConnectionString"].ConnectionString);

你可以使用 **CloudBlobClient** 类型来检索表示存储在 Blob 存储服务
中的容器和 Blob 的对象。以下代码
使用我们在上面检索到的存储帐户对象创建 **CloudBlobClient**
对象：

    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

### ODataLib 依赖项

.NET 存储客户端库中的 ODataLib 依赖项可通过在 NuGet （而非 WCF 数据服务）上获得的 ODataLib（5.0.2 版）包来解析。ODataLib 库可直接下载或者通过 NuGet 由代码项目引用。特定的 ODataLib 包为 [OData][]、[Edm][] 和 [Spatial][]。

## 创建容器如何：创建容器

所有存储 Blob 都驻留在一个容器中。你可以使用
**CloudBlobClient** 对象来获取对你要使用的容器的引用。
如果该容器不存在，你可以创建它：

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建 Blob 客户端。
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // 检索对容器的引用。 
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

    // 如果该容器不存在，则创建它。
    container.CreateIfNotExists();

默认情况下，新容器是专用容器，因此你必须指定存储访问密钥
才能从该容器下载 Blob。
如果你要让容器中
的文件可供所有人使用，则可以使用以下代码
将容器设置为公共容器：

    container.SetPermissions(
    new BlobContainerPermissions { PublicAccess = 
    BlobContainerPublicAccessType.Blob }); 

Internet 上的所有人都可以查看公共容器中的 Blob，但必须
提供相应的访问密钥时才能修改或删除它们。

## 上载到容器如何：将 Blob 上载到容器

Azure Blob 存储支持块 Blob 和页 Blob。大多数情况下，推荐使用块 Blob。

若要将文件上载到块 Blob，请获取容器引用，并使用它获取
块 Blob 引用。获取 Blob 引用后，可以通过调用 **UploadFromStream** 方法
将任何数据流上载到该 Blob。如果之前不存在 Blob，此操作将创建一个；
如果存在 Blob，此操作将覆盖它。下面的示例演示了如何将 Blob 上载到容器中，并假定已创建容器。

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建 Blob 客户端。
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // 检索对以前创建的容器的引用。
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

    // 检索对名为“myblob”的 Blob 的引用。
    CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob");

    // 使用来自本地文件的内容创建或覆盖“myblob”Blob。
    using (var fileStream = System.IO.File.OpenRead(@"path\myfile"))
    {
    blockBlob.UploadFromStream(fileStream);
    } 

## 列出容器中的 Blob如何：列出容器中的 Blob

若要列出容器中的 Blob，首先需要获取容器引用。然后，
你可以使用容器的 **ListBlobs** 方法来检索其中的 Blob 和/或目录。
若要访问返回的 **IListBlobItem** 的丰富属性
和方法，你必须将它转换为 **CloudBlockBlob**、
**CloudPageBlob** 或 **CloudBlobDirectory** 对象。如果类型未知，
你可以使用类型检查来确定要将其转换为哪种类型。以下代码演示了如何
检索和输出 `photos` 容器中每个项
的 URI：

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建 Blob 客户端。 
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // 检索对以前创建的容器的引用。
    CloudBlobContainer container = blobClient.GetContainerReference("photos");

    // 循环访问容器内的项并且输出长度和 URI。
    foreach (IListBlobItem item in container.ListBlobs(null, false))
    {
    if (item.GetType() == typeof(CloudBlockBlob))
        {
    CloudBlockBlob blob = (CloudBlockBlob)item;

    Console.WriteLine("Block blob of length {0}:{1}", blob.Properties.Length, blob.Uri);
                                        
        }
    else if (item.GetType() == typeof(CloudPageBlob))
        {
    CloudPageBlob pageBlob = (CloudPageBlob)item;

    Console.WriteLine("Page blob of length {0}:{1}", pageBlob.Properties.Length, pageBlob.Uri);

        }
    else if (item.GetType() == typeof(CloudBlobDirectory))
        {
    CloudBlobDirectory directory = (CloudBlobDirectory)item;
            
    Console.WriteLine("Directory:{0}", directory.Uri);
        }
    }

如上所示，还可将 Blob 服务视为容器中的目录。
这是为了让你能够以更类似于文件夹的结构来组织 Blob。
例如，请考虑名为 `photos` 的容器中包含的下面一组
块 Blob：

    photo1.jpg
    2010/architecture/description.txt
    2010/architecture/photo3.jpg
    2010/architecture/photo4.jpg
    2011/architecture/photo5.jpg
    2011/architecture/photo6.jpg
    2011/architecture/description.txt
    2011/photo7.jpg

当你对“photos”容器调用 **ListBlobs** 时（如上面的示例所示），返回的集合将
包含 **CloudBlobDirectory** 和 **CloudBlockBlob** 对象，分别表示最高层中
包含的目录和 Blob。下面是生成的输出：

    Directory:https://<accountname>.blob.core.chinacloudapi.cn/photos/2010/
    Directory:https://<accountname>.blob.core.chinacloudapi.cn/photos/2011/
    Block blob of length 505623:https://<accountname>.blob.core.chinacloudapi.cn/photos/photo1.jpg

另外，也可以将 **ListBlobs** 方法的 **UseFlatBlobListing** 参数设置
为 **true**。这将导致每个 Blob 将作为 **CloudBlockBlob**
 返回，而无论目录如何。下面是对 **ListBlobs** 的调用：

    // 循环访问容器内的项并且输出长度和 URI。
    foreach (IListBlobItem item in container.ListBlobs(null, true))
    {
       ...
    }

下面是结果：

    Block blob of length 4:https://<accountname>.blob.core.chinacloudapi.cn/photos/2010/architecture/description.txt
    Block blob of length 314618:https://<accountname>.blob.core.chinacloudapi.cn/photos/2010/architecture/photo3.jpg
    Block blob of length 522713:https://<accountname>.blob.core.chinacloudapi.cn/photos/2010/architecture/photo4.jpg
    Block blob of length 4:https://<accountname>.blob.core.chinacloudapi.cn/photos/2011/architecture/description.txt
    Block blob of length 419048:https://<accountname>.blob.core.chinacloudapi.cn/photos/2011/architecture/photo5.jpg
    Block blob of length 506388:https://<accountname>.blob.core.chinacloudapi.cn/photos/2011/architecture/photo6.jpg
    Block blob of length 399751:https://<accountname>.blob.core.chinacloudapi.cn/photos/2011/photo7.jpg
    Block blob of length 505623:https://<accountname>.blob.core.chinacloudapi.cn/photos/photo1.jpg

有关详细信息，请参阅 [CloudBlobContainer.ListBlobs][]。

## 下载 Blob如何：下载 Blob

若要下载 Blob，请首先检索 Blob 引用，然后调用 **DownloadToStream** 方法。以下示例
使用 **DownloadToStream** 方法将 Blob 内容传输到一个流对象，然后
你可以将该对象保存到本地文件。

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建 Blob 客户端。
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // 检索对以前创建的容器的引用。
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

    // 检索对名为“photo1.jpg”的 Blob 的引用。
    CloudBlockBlob blockBlob = container.GetBlockBlobReference("photo1.jpg");

    // 将 Blob 内容保存到一个文件。
    using (var fileStream = System.IO.File.OpenWrite(@"path\myfile"))
    {
    blockBlob.DownloadToStream(fileStream);
    } 

也可以使用 **DownloadToStream** 方法以文本字符串形式下载 Blob 的内容。

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建 Blob 客户端。
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // 检索对以前创建的容器的引用。
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

    // 检索对名为“myblob.txt”的 Blob 的引用
    CloudBlockBlob blockBlob2 = container.GetBlockBlobReference("myblob.txt");

    string text;
    using (var memoryStream = new MemoryStream())
    {
    blockBlob2.DownloadToStream(memoryStream);
    text = System.Text.Encoding.UTF8.GetString(memoryStream.ToArray());
    }

## 删除 Blob如何：删除 Blob

若要删除 Blob，请首先获取 Blob 引用，然后对其
调用 **Delete** 方法。

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建 Blob 客户端。
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // 检索对以前创建的容器的引用。
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

    // 检索对名为“myblob.txt”的 Blob 的引用。
    CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob.txt");

    // 删除 Blob。
    blockBlob.Delete(); 

## 后续步骤后续步骤

现在，你已了解有关 Blob 存储的基础知识，可单击下面的链接来了解
如何执行更复杂的存储任务。

-   查看 Blob 服务参考文档，了解有关可用 API 的完整详细信息：
    -   [.NET 存储客户端库参考][]
    -   [REST API 参考][]

-   在以下位置了解使用 Azure 存储空间能够执行的更高级任务：[在 Azure 中存储和访问数据][]。
-   查看更多功能指南，以了解在 Azure 中存储数据的其他方式。
    -   使用[表存储][]来存储结构化数据。
    -   使用[队列存储][]来存储非结构化数据。
    -   使用 [SQL Database][] 来存储关系数据。

  [后续步骤]: #next-steps
  [如何：以编程方式访问 Blob 存储]: #configure-access
  [什么是 Blob 存储]: #what-is
  [概念]: #concepts
  [创建 Azure 存储帐户]: #create-account
  [设置存储连接字符串]: #setup-connection-string
  [如何：创建容器]: #create-container
  [如何：将 Blob 上载到容器中]: #upload-blob
  [如何：列出容器中的 Blob]: #list-blob
  [如何：下载 Blob]: #download-blobs
  [如何：删除 Blob]: #delete-blobs
  [howto-blob-storage]: ../includes/howto-blob-storage.md
  [create-storage-account]: ../includes/create-storage-account.md
  [在 Visual Studio 中选择云服务角色的属性]: ./media/storage-dotnet-how-to-use-blobs/blob5.png
  [在 Visual Studio 中添加云服务设置]: ./media/storage-dotnet-how-to-use-blobs/blob6.png
  [Blob7]: ./media/storage-dotnet-how-to-use-blobs/blob7.png
  [Blob8]: ./media/storage-dotnet-how-to-use-blobs/blob8.png
  [Blob9]: ./media/storage-dotnet-how-to-use-blobs/blob9.png
  [配置连接字符串]: http://msdn.microsoft.com/zh-cn/library/azure/ee758697.aspx
  [.NET 开发人员中心]: http://azure.microsoft.com/zh-cn/develop/net/
  [OData]: http://nuget.org/packages/Microsoft.Data.OData/5.0.2
  [Edm]: http://nuget.org/packages/Microsoft.Data.Edm/5.0.2
  [Spatial]: http://nuget.org/packages/System.Spatial/5.0.2
  [.NET 存储客户端库参考]: http://msdn.microsoft.com/zh-cn/library/azure/dn261237.aspx
  [REST API 参考]: http://msdn.microsoft.com/zh-cn/library/azure/dd179355
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [表存储]: /en-us/documentation/articles/storage-dotnet-how-to-use-tables/
  [队列存储]: /en-us/documentation/articles/storage-dotnet-how-to-use-queues/
  [SQL Database]: /en-us/documentation/articles/sql-database-dotnet-how-to-use/
