<properties 
	pageTitle="如何通过 Java 使用文件存储 | Azure" 
	description="了解如何使用 Azure 文件服务上载、下载、列出和删除文件。用 Java 编写的示例。" 
	services="storage" 
	documentationCenter="java" 
	authors="rmcmurray" 
	manager="wpickett" 
	editor="jimbe" />

<tags 
	ms.service="storage" 

	ms.date="05/04/2016"
	wacn.date="06/06/2016"/>

# 如何通过 Java 使用文件存储

[AZURE.INCLUDE [storage-selector-file-include](../includes/storage-selector-file-include.md)]

## 概述

在本指南中，你将学习如何针对 Azure 文件存储服务执行基本的操作。通过以 Java 编写的示例，你将学习如何创建共享和目录，以及如何上载、列出和删除文件。如果你不熟悉 Azure 的文件存储服务，则若要了解这些示例，你需要学习后续部分的概念。

[AZURE.INCLUDE [storage-file-concepts-include](../includes/storage-file-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

## 创建 Java 应用程序

若要生成示例，你需要 Java 开发工具包 (JDK) 和 [Azure Storage SDK for Java][]。此外，你应该已经创建了一个 Azure 存储帐户。

## 设置你的应用程序以使用文件存储

若要使用 Azure 存储 API，请将下列语句添加到要通过其来访问存储服务的 Java 文件的顶部：

	// Include the following imports to use blob APIs.
    import com.microsoft.azure.storage.*;
	import com.microsoft.azure.storage.file.*;

## 设置 Azure 存储连接字符串

若要使用文件存储，你需要连接到你的 Azure 存储帐户。第一步是配置连接字符串，我们将使用该字符串连接到你的存储帐户。为此，我们需要定义一个静态变量。

	// Configure the connection-string with your values
	public static final String storageConnectionString =
	    "DefaultEndpointsProtocol=http;" +
	    "AccountName=your_storage_account_name;" +
	    "AccountKey=your_storage_account_key;" +
	    "EndpointSuffix=core.Chinacloudapi.cn";

> [AZURE.NOTE]将 your_storage_account_name 和 your_storage_account_key 替换为你的存储帐户的实际值。

## 连接到 Azure 存储帐户

若要连接到你的存储帐户，你需要使用 **CloudStorageAccount** 对象，以便将连接字符串传递到 **parse** 方法。

	// Use the CloudStorageAccount object to connect to your storage account
	try {
		CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);
	} catch (InvalidKeyException invalidKey) {
		// Handle the exception
	}

**CloudStorageAccount.parse** 会引发 InvalidKeyException，因此须将其置于 try/catch 块内。

## 如何：创建共享

文件存储中的所有文件和目录都位于名为 **Share** 的容器内。你的存储帐户可以拥有无数的共享，只要你的帐户容量允许。若要获得共享及其内容的访问权限，你需要使用文件存储客户端。

	// Create the file storage client.
	CloudFileClient fileClient = storageAccount.createCloudFileClient();

使用文件存储客户端以后，你就可以获得对共享的引用。

	// Get a reference to the file share
	CloudFileShare share = fileClient.getShareReference("sampleshare");

实际创建共享时，请使用 CloudFileShare 对象的 **createIfNotExists** 方法。

	if (share.createIfNotExists()) {
		System.out.println("New share created");
	}

而在目前，**share** 保留对名为 **sampleshare** 的共享的引用。

## 如何：上载文件

Azure 文件存储共享至少包含文件所在的根目录。在本部分，你将学习如何将文件从本地存储上载到共享所在的根目录。

上载文件的第一步是获取对文件所在的目录的引用。为此，你需要调用共享对象的 **getRootDirectoryReference** 方法。

	//Get a reference to the root directory for the share.
	CloudFileDirectory rootDir = share.getRootDirectoryReference();

现在，你已经有了共享所在的根目录的引用，因此可以使用以下代码来上载文件。

	// Define the path to a local file.
	final String filePath = "C:\\temp\\Readme.txt";

	CloudFile cloudFile = rootDir.getFileReference("Readme.txt");
	cloudFile.uploadFromFile(filePath);

## 如何：创建目录

你也可以将文件置于子目录中，不必将其全部置于根目录中，以便对存储进行有效的组织。Azure 文件存储服务可以创建任意数目的目录，只要你的帐户允许。以下代码将在根目录下创建名为 **sampledir** 的子目录。

	//Get a reference to the root directory for the share.
	CloudFileDirectory rootDir = share.getRootDirectoryReference();

	//Get a reference to the sampledir directory
	CloudFileDirectory sampleDir = rootDir.getDirectoryReference("sampledir");

	if (sampleDir.createIfNotExists()) {
		System.out.println("sampledir created");
	} else {
		System.out.println("sampledir already exists");
	}

## 如何：列出共享中的文件和目录

可以轻松获取共享中文件和目录的列表，只需针对 CloudFileDirectory 引用调用 **listFilesAndDirectories** 即可。该方法将返回你可以对其进行循环访问的 ListFileItem 对象的列表。例如，下面的代码将列出根目录中的文件和目录。

	//Get a reference to the root directory for the share.
	CloudFileDirectory rootDir = share.getRootDirectoryReference();

	for ( ListFileItem fileItem : rootDir.listFilesAndDirectories() ) {
		System.out.println(fileItem.getUri());
	}


## 如何：下载文件

对于文件存储，另一项需要更频繁执行的操作是下载文件。在下面的示例中，代码会下载 SampleFile.txt 并显示其内容。

	//Get a reference to the root directory for the share.
	CloudFileDirectory rootDir = share.getRootDirectoryReference();

	//Get a reference to the directory that contains the file
	CloudFileDirectory sampleDir = rootDir.getDirectoryReference("sampledir");

	//Get a reference to the file you want to download
	CloudFile file = sampleDir.getFileReference("SampleFile.txt");

	//Write the contents of the file to the console.
	System.out.println(file.downloadText());

## 如何：删除文件

另一项常见的文件存储操作是删除文件。下面的代码会删除名为 SampleFile.txt 的文件，该文件存储在名为 **sampledir** 的目录中。


	// Get a reference to the root directory for the share.
	CloudFileDirectory rootDir = share.getRootDirectoryReference();

	// Get a reference to the directory where the file to be deleted is in
	CloudFileDirectory containerDir = rootDir.getDirectoryReference("sampledir");

	String filename = "SampleFile.txt"
	CloudFile file;

	file = containerDir.getFileReference(filename)
	if ( file.deleteIfExists() ) {
		System.out.println(filename + " was deleted");
	}


## 如何：删除目录

删除目录相当简单，但需注意的是，你不能删除仍然包含有文件或其他目录的目录。

	// Get a reference to the root directory for the share.
	CloudFileDirectory rootDir = share.getRootDirectoryReference();

	// Get a reference to the directory you want to delete
	CloudFileDirectory containerDir = rootDir.getDirectoryReference("sampledir");

	// Delete the directory
	if ( containerDir.deleteIfExists() ) {
		System.out.println("Directory deleted");
	}


## 如何：删除共享

删除共享时，可针对 CloudFileShare 对象调用 **deleteIfExists** 方法。以下是具有此类功能的示例代码。

	try
	{
	    // Retrieve storage account from connection-string.
	    CloudStorageAccount storageAccount = CloudStorageAccount.parse(storageConnectionString);

	    // Create the file client.
	   CloudFileClient fileClient = storageAccount.createCloudFileClient();

	   // Get a reference to the file share
	   CloudFileShare share = fileClient.getShareReference("sampleshare");

	   if (share.deleteIfExists()) {
		   System.out.println("sampleshare deleted");
	   }
	} catch (Exception e) {
		e.printStackTrace();
	}

## 后续步骤

如果你还想更多地了解其他 Azure 存储 API，请点击以下链接。

- [Java 开发中心](/develop/java/)
- [Azure Storage SDK for Java](https://github.com/azure/azure-storage-java)
- [Azure Storage SDK for Android](https://github.com/azure/azure-storage-android)
- [Azure 存储客户端 SDK 参考](http://azure.github.io/azure-storage-java/)
- [Azure 存储空间服务 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dd179355.aspx)
- [Azure 存储团队博客](http://blogs.msdn.com/b/windowsazurestorage/)
- [使用 AzCopy 命令行实用程序传输数据](/documentation/articles/storage-use-azcopy/)

<!---HONumber=Mooncake_0530_2016-->