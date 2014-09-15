<properties linkid="dev-net-how-to-use-blog-storage-service-java" urlDisplayName="Blob Service" pageTitle="How to use blob storage (Java) | Microsoft Azure" metaKeywords="Get started Azure blob, Azure unstructured data, Azure unstructured storage, Azure blob, Azure blob storage, Azure blob Java" description="Learn how to use the Azure blob service to upload, download, list, and delete blob content. Samples written in Java." metaCanonical="" services="storage" documentationCenter="Java" title="How to use Blob Storage from Java" authors="" solutions="" manager="" editor="" />

# 如何通过 Java 使用 Blob 存储

本指南将演示如何使用 Azure Blob 存储服务执行常见方案。
示例是用 Java 编写的并且使用了
[Azure SDK for Java][]。涉及的任务包括
**上载**、**列出**、**下载**和**删除** Blob。有关 Blob 的
详细信息，请参阅[后续步骤][]部分。

## 目录

-   [什么是 Blob 存储][]
-   [概念][]
-   [创建 Azure 存储帐户][]
-   [创建 Java 应用程序][]
-   [配置你的应用程序以访问 Blob 存储][]
-   [设置 Azure 存储连接字符串][]
-   [如何：创建容器][]
-   [如何：将 Blob 上载到容器中][]
-   [如何：列出容器中的 Blob][]
-   [如何：下载 Blob][]
-   [如何：删除 Blob][]
-   [如何：删除 Blob 容器][]
-   [后续步骤][]

[WACOM.INCLUDE [howto-blob-storage][]]

## 创建 Azure 存储帐户

[WACOM.INCLUDE [create-storage-account][]]

## 创建 Java 应用程序

在本指南中，你将使用存储功能，这些功能可在本地 Java 应用
程序中运行，或在 Azure 的 Web 角色或辅助角色中通过运行的代码
来运行。我们假定你已下载并安装了
Java 开发工具包 (JDK)，已按照
[下载 Azure SDK for Java][Azure SDK for Java] 中的说明安装了
Azure Libraries for Java 和 Azure SDK，
并已在你的 Azure 订阅中创建了一个
Azure 存储帐户。

你可以使用任何开发工具（包括“记事本”）创建应用程序。
你只要能够编译 Java 项目并引用 Azure Libraries
for Java 即可。

## 配置你的应用程序以访问 Blob 存储

将下列 import 语句添加到需要在其中使用 Azure 存储 API 来
访问 Blob 的 Java 文件的顶部：

    // 包括下列 import 语句以使用 Blob API
    import com.microsoft.windowsazure.services.core.storage.*;
    import com.microsoft.windowsazure.services.blob.client.*;

## 设置 Azure 存储连接字符串

Azure 存储客户端使用存储连接字符串来存储用于访问数据管理服务
的终结点和凭据。在客户端应用程序中运行时，必须提供以下格式的存储连接字符串，并对 *AccountName* 和 *AccountKey* 值使用管理门户中列出的存储帐户的名称和存储帐户的主访问密钥。此示例演示了如何声明一个静态字段来保存连接字符串：

    // 使用你的值定义连接字符串
    public static final String storageConnectionString = 
    "DefaultEndpointsProtocol=http;" + 
    "AccountName=your_storage_account;" + 
    "AccountKey=your_storage_account_key";

在 Azure 中的某个角色内运行的应用程序中，此字符串可
存储在服务配置文件 ServiceConfiguration.cscfg 中，
并可通过调用 **RoleEnvironment.getConfigurationSettings**
方法进行访问。下面是从服务配置文件中
名为 StorageConnectionString 的**设置**
元素中获取连接字符串的示例**：

    // 通过连接字符串检索存储帐户
    String storageConnectionString = 
    RoleEnvironment.getConfigurationSettings().get("StorageConnectionString");

## 如何：创建容器

利用 CloudBlobClient 对象，可以获取容器和 Blob 的引用对象。
以下代码将创建 **CloudBlobClient** 对象。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount =
    CloudStorageAccount.parse(storageConnectionString);

    // 创建 Blob 客户端
    CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

所有 Blob 都驻留在一个容器中。使用 **CloudBlobClient** 对象
获取对你要使用的容器的引用。可使用
**createIfNotExist** 方法创建容器（如果容器不存在），
如果容器存在，则将返回现有容器。默认情况下，新容器
是专用容器，因此你必须指定存储访问密钥（如上面所做的那样）
才能从该容器下载 Blob。

    // 获取对容器的引用
    // 容器名称必须小写
    CloudBlobContainer container = blobClient.getContainerReference("mycontainer");

    // 如果容器不存在，则创建容器
    container.createIfNotExist();

若要使文件成为公共文件，你可以设置容器的权限。

    // 创建一个权限对象
    BlobContainerPermissions containerPermissions = new BlobContainerPermissions();

    // 将公共访问权限包括在权限对象中
    containerPermissions.setPublicAccess(BlobContainerPublicAccessType.CONTAINER);

    // 设置容器上的权限
    container.uploadPermissions(containerPermissions);

Internet 上的所有人都能查看公共容器中的 Blob，但公共访问权限
仅限于读取。

## 如何：将 Blob 上载到容器中

若要将文件上载到 Blob，请获取容器引用，并使用它获取 Blob 引用。
获取 Blob 引用后，可以通过对该 Blob 引用
调用 upload 来上载任何流。此操作将创建 Blob
（如果该 Blob 不存在），或者覆盖它（如果该 Blob 存在）。此代码
示例演示了这一点，并假定已创建容器。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount =
    CloudStorageAccount.parse(storageConnectionString);

    // 创建 Blob 客户端
    CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

    // 检索对以前创建的容器的引用
    CloudBlobContainer container = blobClient.getContainerReference("mycontainer");

    // 使用来自本地文件的内容创建或覆盖“myimage.jpg”Blob
    CloudBlockBlob blob = container.getBlockBlobReference("myimage.jpg");
    File source = new File("c:\\myimages\\myimage.jpg");
    blob.upload(new FileInputStream(source), source.length());

## 如何：列出容器中的 Blob

若要列出容器中的 Blob，请先获取容器引用，就像上载 Blob 时
执行的操作一样。可将容器的 **listBlobs**
 方法用于 for 循环。以下代码将容器中每个 Blob 的 URI 输出
到控制台。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount =
    CloudStorageAccount.parse(storageConnectionString);

    // 创建 Blob 客户端
    CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

    // 检索对以前创建的容器的引用
    CloudBlobContainer container = blobClient.getContainerReference("mycontainer");

    // 循环访问容器内的 Blob 并且输出每个的 URI
    for (ListBlobItem blobItem :container.listBlobs()) {
    System.out.println(blobItem.getUri());
    }

还可将 Blob 服务视为容器中的目录。
这是为了让你能够以更类似于文件夹的结构来组织 Blob。

例如，你可以创建一个名为“photos”的容器，在其中上载名为
“rootphoto1”、“2010/photo1”、“2010/photo2”和
“2011/photo1”的 Blob。这实际上将在“photos”容器中创建
虚拟目录“2010”和“2011”。当你对“photos”容器调用 **listBlobs** 时，
返回的集合将包含表示最高层中所含目录
和 Blob 的 **CloudBlobDirectory**
和 **CloudBlob** 对象。在本例中，
将返回目录“2010”和“2011”以及照片“rootphoto1”。
可使用 **instanceof** 运算符来区分这些对象。

还可以向 **listBlobs** 方法传入参数，并将
**useFlatBlobListing** 参数设置为 true。这将导致返回每个 Blob，
而无论目录如何。有关详细信息，
请参阅 Javadocs 文档中的 CloudBlobContainer.listBlobs。

## 如何：下载 Blob

若要下载 Blob，请执行之前用于上载 Blob 的相同步骤以
获取 Blob 引用。在上载示例中，你对 Blob 对象
调用了 upload。在下面的示例中，调用 download 以
将 Blob 内容传输到可用于将 Blob 保存到
本地文件的流对象（如 **FileOutputStream**）。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount =
    CloudStorageAccount.parse(storageConnectionString);

    // 创建 Blob 客户端
    CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

    // 检索对以前创建的容器的引用
    CloudBlobContainer container = blobClient.getContainerReference("mycontainer");

    // 对于容器中的每个项
    for (ListBlobItem blobItem :container.listBlobs()) {
    // 如果该项为 Blob 而不是虚拟目录
    if (blobItem instanceof CloudBlob) {
    // 下载该项并将其保存到一个同名的文件中
    CloudBlob blob = (CloudBlob) blobItem;
    blob.download(new FileOutputStream(blob.getName()));
        }
    }

## 如何：删除 Blob

若要删除 Blob，请获取 Blob 引用，然后调用 **delete**。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount =
    CloudStorageAccount.parse(storageConnectionString);

    // 创建 Blob 客户端
    CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

    // 检索对以前创建的容器的引用
    CloudBlobContainer container = blobClient.getContainerReference("mycontainer");

    // 检索对名为“myimage.jpg”的 Blob 的引用
    CloudBlockBlob blob = container.getBlockBlobReference("myimage.jpg");

    // 删除 Blob
    blob.delete();

## 如何：删除 Blob 容器

最后，若要删除 Blob 容器，请获取 Blob 容器引用，然后
调用 delete。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount =
    CloudStorageAccount.parse(storageConnectionString);

    // 创建 Blob 客户端
    CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

    // 检索对以前创建的容器的引用
    CloudBlobContainer container = blobClient.getContainerReference("mycontainer");

    // 删除 Blob 容器
    container.delete();

## 后续步骤

现在，你已了解有关 Blob 存储的基础知识，可单击下面的链接来了解
如何执行更复杂的存储任务。

-   查看 MSDN 参考：[在 Windows Azure 中存储和访问
    数据]
-   访问 Azure 存储空间团队博客：<http://blogs.msdn.com/b/windowsazurestorage/>

  [Azure SDK for Java]: http://azure.microsoft.com/zh-cn/develop/java/
  [后续步骤]: #NextSteps
  [什么是 Blob 存储]: #what-is
  [概念]: #Concepts
  [创建 Azure 存储帐户]: #CreateAccount
  [创建 Java 应用程序]: #CreateApplication
  [配置你的应用程序以访问 Blob 存储]: #ConfigureStorage
  [设置 Azure 存储连接字符串]: #ConnectionString
  [如何：创建容器]: #CreateContainer
  [如何：将 Blob 上载到容器中]: #UploadBlob
  [如何：列出容器中的 Blob]: #ListBlobs
  [如何：下载 Blob]: #DownloadBlob
  [如何：删除 Blob]: #DeleteBlob
  [如何：删除 Blob 容器]: #DeleteContainer
  [howto-blob-storage]: ../includes/howto-blob-storage.md
  [create-storage-account]: ../includes/create-storage-account.md
