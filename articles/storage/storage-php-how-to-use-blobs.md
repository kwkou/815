<properties
    pageTitle="如何通过 PHP 使用 Blob 存储（对象存储）| Azure"
    description="使用 Azure Blob 存储（对象存储）将非结构化数据存储在云中。"
    documentationcenter="php"
    services="storage"
    author="mmacy"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="1af56b59-b3f0-4b46-8441-aab463ae088e"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="PHP"
    ms.topic="article"
    ms.date="12/08/2016"
    wacn.date="03/27/2017"
    ms.author="marsma" />

# 如何通过 PHP 使用 Blob 存储
[AZURE.INCLUDE [storage-selector-blob-include](../../includes/storage-selector-blob-include.md)]

## 概述
Azure Blob 存储是一种将非结构化数据作为对象/Blob 存储在云中的服务。Blob 存储可以存储任何类型的文本或二进制数据，例如文档、媒体文件或应用程序安装程序。Blob 存储也称为对象存储。

本指南将演示如何使用 Azure Blob 服务执行常见方案。示例是用 PHP 编写的并使用了 [Azure SDK for PHP][download]。涉及的任务包括“上传”、“列出”、“下载”和“删除” Blob。有关 Blob 的详细信息，请参阅[后续步骤](#next-steps)部分。

[AZURE.INCLUDE [storage-blob-concepts-include](../../includes/storage-blob-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

## 创建 PHP 应用程序
创建访问 Azure Blob 服务的 PHP 应用程序的唯一要求是从代码中引用 Azure SDK for PHP 中的类。可以使用任何开发工具（包括“记事本”）创建应用程序。

在本指南中，将使用服务功能，可在 PHP 应用程序中本地调用这些功能，或在 Azure Web 角色、辅助角色或网站内运行的代码中调用。

##<a id="GetClientLibrary"></a>获取 Azure 客户端库

[AZURE.INCLUDE [get-client-libraries](../../includes/get-client-libraries.md)]

##<a id="ConfigureStorage"></a>配置应用程序以访问 Blob 服务

若要使用 Azure Blob 服务 API，需要：

1. 使用 [require\_once] 语句引用 autoloader 文件，并
2. 引用所用的任意类。

下面的示例演示了如何包括 autoloader 文件并引用 **ServicesBuilder** 类。

> [AZURE.NOTE] 本示例（以及本文中的其他示例）假定已通过 Composer 安装了用于 Azure 的 PHP 客户端库。如果已手动安装这些库，则需要引用 `WindowsAzure.php` autoloader 文件。

	require_once 'vendor/autoload.php';
	use WindowsAzure\Common\ServicesBuilder;


在下面的示例中，将始终显示 `require_once` 语句，但只会引用执行该示例所需的类。

##<a id="ConnectionString"></a>设置 Azure 存储连接

若要实例化 Azure Blob 服务客户端，必须首先具有有效的连接字符串。Blob 服务连接字符串的格式为：

对于访问实时服务：

	$connectionString = 'BlobEndpoint=http://<storage account name>.blob.core.chinacloudapi.cn/;QueueEndpoint=http:// <storage account name>.queue.core.chinacloudapi.cn/;TableEndpoint=http:// <storage account name>.table.core.chinacloudapi.cn/;AccountName=<storage account name>;AccountKey=<storage account key>'

访问存储模拟器：

	UseDevelopmentStorage=true


若要创建任何 Azure 服务客户端，则需要使用 **ServicesBuilder** 类。方法：

* 将连接字符串直接传递给此类或
* 使用 **CloudConfigurationManager (CCM)** 检查多个外部源以获取连接字符串：
  * 默认情况下，它附带了对一个外部源的支持 - 环境变量。
  * 可通过扩展 **ConnectionStringSource** 类来添加新源。

在此处列出的示例中，将直接传递连接字符串。

	require_once 'vendor/autoload.php';

	use WindowsAzure\Common\ServicesBuilder;

	$blobRestProxy = ServicesBuilder::getInstance()->createBlobService($connectionString);

##<a id="CreateContainer"></a>创建容器
[AZURE.INCLUDE [storage-container-naming-rules-include](../../includes/storage-container-naming-rules-include.md)]

利用 **BlobRestProxy** 对象，可以使用 **createContainer** 方法创建 Blob 容器。创建容器时，可以在该容器上设置选项，但此操作不是必需的。（下面的示例演示了如何设置容器访问控制列表 (ACL) 和容器元数据。）

	require_once 'vendor\autoload.php';

	use MicrosoftAzure\Storage\Common\ServicesBuilder;
	use MicrosoftAzure\Storage\Blob\Models\CreateContainerOptions;
	use MicrosoftAzure\Storage\Blob\Models\PublicAccessType;
	use MicrosoftAzure\Storage\Common\ServiceException;

	// Create blob REST proxy.
	$blobRestProxy = ServicesBuilder::getInstance()->createBlobService($connectionString);


	// OPTIONAL: Set public access policy and metadata.
	// Create container options object.
	$createContainerOptions = new CreateContainerOptions();

	// Set public access policy. Possible values are
	// PublicAccessType::CONTAINER_AND_BLOBS and PublicAccessType::BLOBS_ONLY.
	// CONTAINER_AND_BLOBS:
	// Specifies full public read access for container and blob data.
    // proxys can enumerate blobs within the container via anonymous
	// request, but cannot enumerate containers within the storage account.
	//
	// BLOBS_ONLY:
	// Specifies public read access for blobs. Blob data within this
    // container can be read via anonymous request, but container data is not
    // available. proxys cannot enumerate blobs within the container via
	// anonymous request.
	// If this value is not specified in the request, container data is
	// private to the account owner.
	$createContainerOptions->setPublicAccess(PublicAccessType::CONTAINER_AND_BLOBS);

	// Set container metadata.
	$createContainerOptions->addMetaData("key1", "value1");
	$createContainerOptions->addMetaData("key2", "value2");

	try	{
		// Create container.
		$blobRestProxy->createContainer("mycontainer", $createContainerOptions);
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179439.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

调用 **setPublicAccess(PublicAccessType::CONTAINER\_AND\_BLOBS)** 将使容器和 Blob 数据可通过匿名请求访问。通过调用 **setPublicAccess(PublicAccessType::BLOBS\_ONLY)**，仅使 Blob 数据可通过匿名请求访问。有关容器 ACL 的详细信息，请参阅 [设置容器 ACL (REST API)][container-acl]。

有关 Blob 服务错误代码的详细信息，请参阅 [Blob 服务错误代码][error-codes]。

##<a id="UploadBlob"></a>将 Blob 上传到容器中

若要将文件作为 Blob 上传，请使用 **BlobRestProxy->createBlockBlob** 方法。此操作将创建 Blob（如果该 Blob 不存在），或者覆盖它（如果该 Blob 存在）。下面的代码示例假定已创建了容器，并使用 [fopen][fopen] 将文件作为流打开。

	require_once 'vendor/autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use MicrosoftAzure\Storage\Common\ServiceException;

	// Create blob REST proxy.
	$blobRestProxy = ServicesBuilder::getInstance()->createBlobService($connectionString);


	$content = fopen("c:\myfile.txt", "r");
	$blob_name = "myblob";

	try	{
		//Upload blob
		$blobRestProxy->createBlockBlob("mycontainer", $blob_name, $content);
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179439.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

请注意，上面的示例将 Blob 作为流上传。但是，也可使用 [file\_get\_contents][file_get_contents] 函数将 Blob 作为字符串上传。若要使用前面的示例执行此操作，请将 `$content = fopen("c:\myfile.txt", "r");` 更改为 `$content = file_get_contents("c:\myfile.txt");`。

##<a id="ListBlobs"></a>列出容器中的 Blob

若要列出容器中的 Blob，请将 **BlobRestProxy->listBlobs** 方法与 **foreach** 循环一起使用来循环访问结果。以下代码将容器中的每个 Blob 的名称作为容器中的输出并将其 URI 显示到浏览器。

	require_once 'vendor/autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use MicrosoftAzure\Storage\Common\ServiceException;

	// Create blob REST proxy.
	$blobRestProxy = ServicesBuilder::getInstance()->createBlobService($connectionString);


	try	{
		// List blobs.
		$blob_list = $blobRestProxy->listBlobs("mycontainer");
		$blobs = $blob_list->getBlobs();

		foreach($blobs as $blob)
		{
			echo $blob->getName().": ".$blob->getUrl()."<br />";
		}
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179439.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}


##<a id="DownloadBlob"></a>下载 Blob

若要下载 Blob，请调用 **BlobRestProxy->getBlob** 方法，然后对生成的 **GetBlobResult** 对象调用 **getContentStream** 方法。

	require_once 'vendor/autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use MicrosoftAzure\Storage\Common\ServiceException;

	// Create blob REST proxy.
	$blobRestProxy = ServicesBuilder::getInstance()->createBlobService($connectionString);


	try	{
		// Get blob.
		$blob = $blobRestProxy->getBlob("mycontainer", "myblob");
		fpassthru($blob->getContentStream());
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179439.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

请注意，上面的示例将 Blob 作为流资源获取（默认行为）。但是，可以使用 [stream\_get\_contents][stream-get-contents] 函数将返回的流转换为字符串。

##<a id="DeleteBlob"></a>删除 Blob

若要删除 Blob，请将容器名称和 Blob 名称传递到 **BlobRestProxy->deleteBlob**。

	require_once 'vendor/autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use MicrosoftAzure\Storage\Common\ServiceException;

	// Create blob REST proxy.
	$blobRestProxy = ServicesBuilder::getInstance()->createBlobService($connectionString);


	try	{
		// Delete blob.
		$blobRestProxy->deleteBlob("mycontainer", "myblob");
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179439.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

##<a id="DeleteContainer"></a>删除 Blob 容器

最后，若要删除 Blob 容器，请将容器名称传递到 **BlobRestProxy->deleteContainer**。

	require_once 'vendor/autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use MicrosoftAzure\Storage\Common\ServiceException;

	// Create blob REST proxy.
	$blobRestProxy = ServicesBuilder::getInstance()->createBlobService($connectionString);


	try	{
		// Delete container.
		$blobRestProxy->deleteContainer("mycontainer");
	}
	catch(ServiceException $e){
		// Handle exception based on error codes and messages.
		// Error codes and messages are here:
		// http://msdn.microsoft.com/zh-cn/library/azure/dd179439.aspx
		$code = $e->getCode();
		$error_message = $e->getMessage();
		echo $code.": ".$error_message."<br />";
	}

##<a id="next-steps"></a>后续步骤

在了解了 Azure Blob 服务的基础知识后，可单击下面的链接以了解有关更复杂的存储任务的详细信息。

- 访问 [Azure 存储团队博客](http://blogs.msdn.com/b/windowsazurestorage/)
- 参阅 [PHP 块 Blob 示例](https://github.com/WindowsAzure/azure-sdk-for-php-samples/blob/master/storage/BlockBlobExample.php)。
- 参阅 [PHP 页 Blob 示例](https://github.com/WindowsAzure/azure-sdk-for-php-samples/blob/master/storage/PageBlobExample.php)。
- [使用 AzCopy 命令行实用程序传输数据](/documentation/articles/storage-use-azcopy/)
 
有关详细信息，另请参阅 [PHP 开发人员中心](/develop/php/)。

[download]: /documentation/articles/php-download-sdk/
[container-acl]: http://msdn.microsoft.com/zh-cn/library/azure/dd179391.aspx
[error-codes]: http://msdn.microsoft.com/zh-cn/library/azure/dd179439.aspx
[file_get_contents]: http://php.net/file_get_contents
[require\_once]: http://php.net/require_once
[fopen]: http://www.php.net/fopen
[stream-get-contents]: http://www.php.net/stream_get_contents

<!---HONumber=Mooncake_0103_2017-->