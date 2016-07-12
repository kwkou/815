<properties 
    pageTitle="如何使用 Blob 存储 (C++) | Azure" 
	description="使用 Azure Blob 存储（对象存储）将非结构化数据存储在云中。"
    services="storage" 
    documentationCenter=".net" 
    authors="tamram" 
    manager="adinah" 
    editor=""/>

<tags 
    ms.service="storage" 
    ms.date="04/29/2016"
    wacn.date="06/06/2016"/>

# 如何通过 C++ 使用 Blob 存储  

[AZURE.INCLUDE [storage-selector-blob-include](../includes/storage-selector-blob-include.md)]

## 概述
本指南将演示如何使用 Azure Blob 存储服务执行常见方案。示例用 C++ 编写，并使用[适用于 C++ 的 Azure 存储空间客户端库](http://github.com/Azure/azure-storage-cpp/blob/master/README.md)。涉及的任务包括**上载**、**列出**、**下载**和**删除** Blob。

>[AZURE.NOTE] 本指南主要面向适用于 C++ 的 Azure 存储空间客户端库 1.0.0 版及更高版本。建议的版本是存储空间客户端库 2.2.0，它可以通过 [NuGet](http://www.nuget.org/packages/wastorage) 或 [GitHub](https://github.com/Azure/azure-storage-cpp) 获得。

[AZURE.INCLUDE [storage-blob-concepts-include](../includes/storage-blob-concepts-include.md)]
[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

## 创建 C++ 应用程序
在本指南中，你将使用存储功能，这些功能可以在 C++ 应用程序中运行。

为此，你将需要安装适用于 C++ 的 Azure 存储客户端库，并在你的 Azure 订阅中创建 Azure 存储帐户。

若要安装适用于 C++ 的 Azure 存储客户端库，你可以使用以下方法：

-	**Linux：**按照[适用于 C++ 的 Azure 存储空间客户端库自述文件](https://github.com/Azure/azure-storage-cpp/blob/master/README.md)页中提供的说明操作。  
-	**Windows：**在 Visual Studio 主菜单中，单击“工具”->“NuGet 程序包管理器”->“程序包管理器控制台”。在 [NuGet 程序包管理器控制台](http://docs.nuget.org/docs/start-here/using-the-package-manager-console)窗口中输入以下命令，然后按 **ENTER**。  

		Install-Package wastorage

## 配置你的应用程序以访问 Blob 存储  
将以下 include 语句添加到 C++ 文件的顶部，你要在此使用 Azure 存储 API 来访问 blob：

	#include "was/storage_account.h"
	#include "was/blob.h"

## 设置 Azure 存储连接字符串
Azure 存储客户端使用存储连接字符串来存储用于访问数据管理服务的终结点和凭据。在客户端应用程序中运行时，必须提供以下格式的存储连接字符串，并使用[管理门户](https://manage.windowsazure.cn)中列出的存储帐户的存储帐户名称和存储访问密钥作为 *AccountName* 和 *AccountKey* 值。有关存储帐户和访问密钥的信息，请参阅[关于 Azure 存储帐户](/documentation/articles/storage-create-storage-account/)。此示例演示如何声明一个静态字段以保存连接字符串：

	// Define the connection-string with your values.
	const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;AccountName=your_storage_account;AccountKey=your_storage_account_key;EndpointSuffix=core.chinacloudapi.cn"));

若要在本地 Windows 计算机中测试您的应用程序，可以使用随同 [Azure SDK](/downloads/) 一起安装的 Azure [存储模拟器](/documentation/articles/storage-use-emulator/)。存储模拟器是一种用于模拟本地开发计算机上 Azure 中可用的 Blob、队列和表服务的实用程序。以下示例演示如何声明一个静态字段以将连接字符串保存到你的本地存储模拟器：

	// Define the connection-string with Azure Storage Emulator.
	const utility::string_t storage_connection_string(U("UseDevelopmentStorage=true;"));  

若要启动 Azure 存储模拟器，请选择“开始”按钮或按 **Windows** 键。开始键入“Azure 存储模拟器”，然后从应用程序列表中选择“Azure 存储模拟器”。

下面的示例假定你使用了这两个方法之一来获取存储连接字符串。

## 检索你的连接字符串
可以使用 **cloud_storage_account** 类来表示您的存储帐户信息。若要从存储连接字符串中检索您的存储帐户信息，您可以使用 **parse** 方法。

	// Retrieve storage account from connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

其次，获取对 **cloud_blob_client** 类的引用，因为它允许您检索表示存储在 Blob 存储服务中的容器和 Blob 的对象。以下代码使用我们在上面检索到的存储帐户对象创建 **cloud_blob_client** 对象：

	// Create the blob client.
	azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();  

## 如何：创建容器

[AZURE.INCLUDE [storage-container-naming-rules-include](../includes/storage-container-naming-rules-include.md)]

此示例演示如何创建一个容器（如果该容器不存在）：

	try
	{
		// Retrieve storage account from connection string.
		azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

		// Create the blob client.
		azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();

		// Retrieve a reference to a container.
		azure::storage::cloud_blob_container container = blob_client.get_container_reference(U("my-sample-container"));

		// Create the container if it doesn't already exist.
		container.create_if_not_exists();
	}
	catch (const std::exception& e)
	{
		std::wcout << U("Error: ") << e.what() << std::endl;
	}  

默认情况下，新容器是专用容器，因此你必须指定存储访问密钥才能从该容器下载 Blob。如果要使容器中的文件 (blob) 对任何人都可用，则可以使用以下代码将容器设置为公用：

	// Make the blob container publicly accessible.
	azure::storage::blob_container_permissions permissions;
	permissions.set_public_access(azure::storage::blob_container_public_access_type::blob);
	container.upload_permissions(permissions);  

Internet 中的所有人都可以查看公共容器中的 Blob，但是，仅在你具有相应的访问密钥时，才能修改或删除它们。

## 如何：将 Blob 上载到容器
Azure Blob 存储支持块 Blob 和页 Blob。大多数情况下，推荐使用块 Blob。

若要将文件上载到块 Blob，请获取容器引用，并使用它获取块 Blob 引用。拥有 Blob 引用后，您可以通过调用 **upload_from_stream** 方法将任何数据流上载到其中。如果之前不存在 Blob，此操作将创建一个；如果存在 Blob，此操作将覆盖它。下面的示例演示了如何将 Blob 上载到容器中，并假定已创建容器。  

	// Retrieve storage account from connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the blob client.
	azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();

	// Retrieve a reference to a previously created container.
	azure::storage::cloud_blob_container container = blob_client.get_container_reference(U("my-sample-container"));

	// Retrieve reference to a blob named "my-blob-1".
	azure::storage::cloud_block_blob blockBlob = container.get_block_blob_reference(U("my-blob-1"));

	// Create or overwrite the "my-blob-1" blob with contents from a local file.
	concurrency::streams::istream input_stream = concurrency::streams::file_stream<uint8_t>::open_istream(U("DataFile.txt")).get();
	blockBlob.upload_from_stream(input_stream);
	input_stream.close().wait();

	// Create or overwrite the "my-blob-2" and "my-blob-3" blobs with contents from text.
	// Retrieve a reference to a blob named "my-blob-2".
	azure::storage::cloud_block_blob blob2 = container.get_block_blob_reference(U("my-blob-2"));
	blob2.upload_text(U("more text"));

	// Retrieve a reference to a blob named "my-blob-3".
	azure::storage::cloud_block_blob blob3 = container.get_block_blob_reference(U("my-directory/my-sub-directory/my-blob-3"));
	blob3.upload_text(U("other text"));  

你还可以使用 **upload_from_file** 方法将文件上载到块 blob。

## 如何：列出容器中的 Blob
若要列出容器中的 Blob，首先需要获取容器引用。然后，您可以使用容器的 **list_blobs** 方法来检索其中的 blob 和/或目录。若要针对一个返回的 **list_blob_item** 访问其丰富的属性和方法，您必须调用 **list_blob_item.as_blob** 方法以获取一个 **cloud_blob** 对象，或调用 **list_blob.as_directory** 方法以获取 cloud_blob_directory 对象。以下代码演示如何检索和输出 **my-sample-container** 容器中每一项的 URI：  

	// Retrieve storage account from connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the blob client.
	azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();

	// Retrieve a reference to a previously created container.
	azure::storage::cloud_blob_container container = blob_client.get_container_reference(U("my-sample-container"));

	// Output URI of each item.
	azure::storage::list_blob_item_iterator end_of_results;
	for (auto it = container.list_blobs(); it != end_of_results; ++it)
	{
		if (it->is_blob())
		{
			std::wcout << U("Blob: ") << it->as_blob().uri().primary_uri().to_string() << std::endl;
		}
		else
		{
			std::wcout << U("Directory: ") << it->as_directory().uri().primary_uri().to_string() << std::endl;
		}
	}

有关列出操作的更多详细信息，请参阅[使用 C++ 列出 Azure 存储资源](/documentation/articles/storage-c-plus-plus-enumeration/)。

## 如何：下载 Blob
若要下载 Blob，请首先检索 Blob 引用，然后调用 **download_to_stream** 方法。以下示例使用 **download_to_stream** 方法将 Blob 内容传输到一个流对象，然后您可以将该对象保存到本地文件。  

	// Retrieve storage account from connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the blob client.
	azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();

	// Retrieve a reference to a previously created container.
	azure::storage::cloud_blob_container container = blob_client.get_container_reference(U("my-sample-container"));

	// Retrieve reference to a blob named "my-blob-1".
	azure::storage::cloud_block_blob blockBlob = container.get_block_blob_reference(U("my-blob-1"));

	// Save blob contents to a file.
	concurrency::streams::container_buffer<std::vector<uint8_t>> buffer;
	concurrency::streams::ostream output_stream(buffer);
	blockBlob.download_to_stream(output_stream);

	std::ofstream outfile("DownloadBlobFile.txt", std::ofstream::binary);
	std::vector<unsigned char>& data = buffer.collection();

	outfile.write((char *)&data[0], buffer.size());
	outfile.close();  

或者，可以使用 **download_to_file** 方法将 blob 的内容下载到文件。
此外，也可以使用 **download_text** 方法以文本字符串形式下载 Blob 的内容。  

	// Retrieve storage account from connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the blob client.
	azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();

	// Retrieve a reference to a previously created container.
	azure::storage::cloud_blob_container container = blob_client.get_container_reference(U("my-sample-container"));

	// Retrieve reference to a blob named "my-blob-2".
	azure::storage::cloud_block_blob text_blob = container.get_block_blob_reference(U("my-blob-2"));

	// Download the contents of a blog as a text string.
	utility::string_t text = text_blob.download_text();

## 如何：删除 Blob
若要删除 Blob，请先获取 Blob 引用，然后对其调用 **delete_blob** 方法。  

	// Retrieve storage account from connection string.
	azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

	// Create the blob client.
	azure::storage::cloud_blob_client blob_client = storage_account.create_cloud_blob_client();

	// Retrieve a reference to a previously created container.
	azure::storage::cloud_blob_container container = blob_client.get_container_reference(U("my-sample-container"));

	// Retrieve reference to a blob named "my-blob-1".
	azure::storage::cloud_block_blob blockBlob = container.get_block_blob_reference(U("my-blob-1"));

	// Delete the blob.
	blockBlob.delete_blob();

## 后续步骤
既然你已了解 blob 存储的基础知识，请按照下面的链接了解有关 Azure 存储的详细信息。  

-	[如何通过 C++ 使用队列存储](/documentation/articles/storage-c-plus-plus-how-to-use-queues/)
-	[如何通过 C++ 使用表存储](/documentation/articles/storage-c-plus-plus-how-to-use-tables/)
-	[使用 C++ 列出 Azure 存储资源](/documentation/articles/storage-c-plus-plus-enumeration/)
-	[适用于 C++ 的存储空间客户端库参考](http://azure.github.io/azure-storage-cpp)
-	[Azure 存档文档](/documentation/services/storage/)
- [使用 AzCopy 命令行实用程序传输数据](/documentation/articles/storage-use-azcopy/)




<!---HONumber=Mooncake_0530_2016-->