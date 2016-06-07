<properties
	pageTitle="如何通过 Java 使用 Azure Blob 存储 | Azure"
	description="使用 Azure Blob 存储（对象存储）将非结构化数据存储在云中。"
	services="storage"
	documentationCenter="java"
	authors="rmcmurray"
	manager="wpickett"
	editor="jimbe"/>

<tags
	ms.service="storage"
	ms.date="05/04/2016"
	wacn.date="06/06/2016"/>

# 如何通过 Java 使用 Blob 存储

[AZURE.INCLUDE [storage-selector-blob-include](../includes/storage-selector-blob-include.md)]

## 概述

本文将演示如何使用 Azure Blob 存储执行常见任务。这些示例用 Java 编写并使用 [Azure Storage SDK for Java][]。涉及的任务包括**上传**、**列出**、**下载**和**删除** Blob。有关 Blob 的详细信息，请参阅[后续步骤](#NextSteps)部分。

> [AZURE.NOTE] SDK 提供给在 Android 设备上使用 Azure 存储空间的开发人员。有关详细信息，请参阅 [Azure Storage SDK for Android][]。

[AZURE.INCLUDE [storage-blob-concepts-include](../includes/storage-blob-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

## <a name="CreateApplication"> </a>创建 Java 应用程序

在本文中，你将使用存储功能，这些功能可在本地 Java 应用程序中运行，或在 Azure 的 Web 角色或辅助角色中通过运行的代码来运行。

为此，你将需要安装 Java 开发工具包 (JDK)，并在你的 Azure 订阅中创建一个 Azure 存储帐户。完成此操作后，你将需要验证开发系统满足最低要求和 GitHub 上的 [Azure Storage SDK for Java][] 存储库中列出的依赖项。如果你的系统满足这些要求，你可以按照说明下载和安装系统中该存储库的 Azure Storage Libraries for Java。完成这些任务后，您将能够创建一个 Java 应用程序，以便使用本文中的示例。

## 配置你的应用程序以访问 Blob 存储

将下列导入语句添加到要在其中使用 Azure 存储 API 以访问 Blob 的 Java 文件的顶部：

    // Include the following imports to use blob APIs.
    import com.microsoft.azure.storage.*;
    import com.microsoft.azure.storage.blob.*;

## 设置 Azure 存储连接字符串

Azure 存储客户端使用存储连接字符串来存储用于访问数据管理服务的终结点和凭据。在客户端应用程序中运行时，必须提供以下格式的存储连接字符串，并对 AccountName 和 AccountKey 值使用[管理门户](https://manage.windowsazure.cn)中列出的存储帐户的名称和存储帐户的主访问密钥。下面的示例演示如何声明一个静态字段以保存连接字符串。

    // Define the connection-string with your values
    public static final String storageConnectionString =
        "DefaultEndpointsProtocol=http;" +
        "AccountName=your_storage_account;" +
        "AccountKey=your_storage_account_key;" +
	"EndpointSuffix=core.chinacloudapi.cn";

在 Azure 的角色中运行的应用程序中，此字符串可存储在服务配置文件  *ServiceConfiguration.cscfg* 中，并可通过调用 **RoleEnvironment.getConfigurationSettings** 方法进行访问。下面的示例从服务配置文件中名为  *StorageConnectionString* 的 **Setting** 元素获取连接字符串

    // Retrieve storage account from connection-string.
    String storageConnectionString =
        RoleEnvironment.getConfigurationSettings().get("StorageConnectionString");

下面的示例假定你使用了这两个方法之一来获取存储连接字符串。

## 创建容器

利用 **CloudBlobClient** 对象，可以获得容器和 Blob 的引用对象。以下代码将创建 **CloudBlobClient** 对象。

> [AZURE.NOTE] 还有其他方式来创建 **CloudStorageAccount** 对象；有关详细信息，请参阅 [Azure 存储空间客户端 SDK 参考]中的 **CloudStorageAccount**。

[AZURE.INCLUDE [storage-container-naming-rules-include](../includes/storage-container-naming-rules-include.md)]

使用 **CloudBlobClient** 对象获取对你要使用的容器的引用。可使用 **createIfNotExists** 方法创建容器（如果不存在），否则将返回现有容器。默认情况下，新容器是专用容器，因此您必须指定存储访问密钥（如之前所做的那样）才能从该容器下载 Blob。

	try
    {
        // Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

    	// Create the blob client.
	   CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

	   // Get a reference to a container.
	   // The container name must be lower case
	   CloudBlobContainer container = blobClient.getContainerReference("mycontainer");

	   // Create the container if it does not exist.
    	container.createIfNotExists();
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

### 可选：配置进行公共访问的容器

默认情况下，容器的权限已配置为允许进行私有访问，但你也可以轻松地将容器的权限配置为允许 Internet 上的用户进行公开的、只读的访问：

    // Create a permissions object.
    BlobContainerPermissions containerPermissions = new BlobContainerPermissions();

    // Include public access in the permissions object.
    containerPermissions.setPublicAccess(BlobContainerPublicAccessType.CONTAINER);

    // Set the permissions on the container.
    container.uploadPermissions(containerPermissions);

## 将 Blob 上传到容器中

若要将文件上传到 Blob，请获取容器引用，并使用它获取 Blob 引用。获取 Blob 引用后，可以通过对该 Blob 引用调用 upload 来上传任何数据流。此操作将创建 Blob（如果该 Blob 不存在），或者覆盖它（如果该 Blob 存在）。下面的代码示例演示了这一点，并假定已创建容器。

	try
    {
        // Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

    	// Create the blob client.
    	CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

	   // Retrieve reference to a previously created container.
    	CloudBlobContainer container = blobClient.getContainerReference("mycontainer");

        // Define the path to a local file.
        final String filePath = "C:\\myimages\\myimage.jpg";

    	// Create or overwrite the "myimage.jpg" blob with contents from a local file.
    	CloudBlockBlob blob = container.getBlockBlobReference("myimage.jpg");
    	File source = new File(filePath);
    	blob.upload(new FileInputStream(source), source.length());
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 列出容器中的 Blob

若要列出容器中的 Blob，请先获取容器引用，就像上传 Blob 时执行的操作一样。可将容器的 **listBlobs** 方法用于 **for** 循环。以下代码将容器中每个 Blob 的 URI 输出到控制台。

	try
    {
        // Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

    	// Create the blob client.
    	CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

    	// Retrieve reference to a previously created container.
    	CloudBlobContainer container = blobClient.getContainerReference("mycontainer");

    	// Loop over blobs within the container and output the URI to each of them.
    	for (ListBlobItem blobItem : container.listBlobs()) {
	       System.out.println(blobItem.getUri());
	   }
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

请注意，你可以命名 Blob，并在其名称中包含路径信息。这将创建一个虚拟目录结构，你可以像传统文件系统一样组织和遍历。注意，该目录结构仅仅是虚拟的 - Blob 存储中唯一可用的资源是容器和 Blob。但是，客户端库提供 **CloudBlobDirectory** 对象来引用虚拟目录，并简化了以这种方式组织的 Blob 的使用过程。

例如，你可以创建一个名为“photos”的容器，你可以在其中上传名为“rootphoto1”、“2010/photo1”、“2010/photo2”和“2011/photo1”的 Blob。这将在“photos”容器中创建虚拟目录“2010”和“2011”。当你对“photos”容器调用 **listBlobs** 时，返回的集合将包含表示最高层中所含目录和 Blob 的 **CloudBlobDirectory** 和 **CloudBlob** 对象。在本例中，将返回目录“2010”和“2011”以及照片“rootphoto1”。可使用 **instanceof** 运算符来区分这些对象。

还可以向 **listBlobs** 方法传入参数，并将 **useFlatBlobListing** 参数设置为 true。这将导致返回每个 Blob，而无论目录如何。有关详细信息，请参阅 [Azure 存储空间客户端 SDK 参考]中的 **CloudBlobContainer.listBlobs**。

## 下载 Blob

若要下载 Blob，请执行之前用于上传 Blob 的相同步骤以获取 Blob 引用。在上传示例中，您对 Blob 对象调用了 upload。在下面的示例中，调用 download 以将 Blob 内容传输到可用于将 Blob 保存到本地文件的流对象（如 **FileOutputStream**）。

    try
    {
    	// Retrieve storage account from connection-string.
	   CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

	   // Create the blob client.
	   CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

	   // Retrieve reference to a previously created container.
	   CloudBlobContainer container = blobClient.getContainerReference("mycontainer");

	   // Loop through each blob item in the container.
	   for (ListBlobItem blobItem : container.listBlobs()) {
	       // If the item is a blob, not a virtual directory.
	       if (blobItem instanceof CloudBlob) {
	           // Download the item and save it to a file with the same name.
    	        CloudBlob blob = (CloudBlob) blobItem;
    	        blob.download(new FileOutputStream("C:\\mydownloads\\" + blob.getName()));
    	    }
    	}
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 删除 Blob

若要删除 Blob，请获取 Blob 引用，然后调用 **deleteIfExists**。

    try
    {
	   // Retrieve storage account from connection-string.
	   CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

	   // Create the blob client.
	   CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

	   // Retrieve reference to a previously created container.
	   CloudBlobContainer container = blobClient.getContainerReference("mycontainer");

	   // Retrieve reference to a blob named "myimage.jpg".
	   CloudBlockBlob blob = container.getBlockBlobReference("myimage.jpg");

	   // Delete the blob.
	   blob.deleteIfExists();
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 删除 Blob 容器

最后，若要删除 Blob 容器，请获取 Blob 容器引用，然后调用 **deleteIfExists**。

    try
    {
	   // Retrieve storage account from connection-string.
	   CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

	   // Create the blob client.
	   CloudBlobClient blobClient = storageAccount.createCloudBlobClient();

	   // Retrieve reference to a previously created container.
	   CloudBlobContainer container = blobClient.getContainerReference("mycontainer");

	   // Delete the blob container.
	   container.deleteIfExists();
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 后续步骤

现在，你已了解有关 Blob 存储的基础知识，可单击下面的链接来了解更复杂的存储任务。

- [Azure Storage SDK for Java][]
- [Azure 存储客户端 SDK 参考][]
- [Azure 存储 REST API][]
- [Azure 存储团队博客][]

有关详细信息，另请参阅 [Java 开发人员中心](/develop/java/)。

[Azure SDK for Java]: /develop/java/
[Azure Storage SDK for Java]: https://github.com/azure/azure-storage-java
[Azure Storage SDK for Android]: https://github.com/azure/azure-storage-android
[Azure 存储客户端 SDK 参考]: http://dl.windowsazure.com/storage/javadoc/
[Azure 存储空间客户端 SDK 参考]: http://dl.windowsazure.com/storage/javadoc/
[Azure 存储 REST API]: https://msdn.microsoft.com/zh-cn/library/azure/dd179355.aspx
[Azure 存储团队博客]: http://blogs.msdn.com/b/windowsazurestorage/

<!---HONumber=Mooncake_0530_2016-->