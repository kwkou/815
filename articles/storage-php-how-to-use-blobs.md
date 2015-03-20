<properties title="How to use blob storage (PHP) - Azure feature guide" pageTitle="如何使用 Blob 存储 (PHP) | Microsoft Azure" metaKeywords="Azure blob service PHP, Azure blobs PHP" description="了解如何使用 Azure Blob 服务上载、列出、下载和删除 Blob。通过 PHP 编写代码示例。" documentationCenter="PHP" services="storage" videoId="" scriptId="" solutions="" authors="robmcm" manager="wpickett" editor="mollybos" />

#如何通过 PHP 使用 Blob 服务

本指南将演示如何使用 Azure Blob 服务执行常见方案。示例是用 PHP 编写的并使用了 [Azure SDK for PHP] [下载]。涉及的任务包括**上载**、**列出**、**下载**和**删除** Blob。有关 Blob 的详细信息，请参阅[后续步骤](#NextSteps) 节中添加以下代码。

##目录

* [什么是 Blob 存储](#what-is)
* [概念](#concepts)
* [创建 Azure 存储帐户](#CreateAccount)
* [创建 PHP 应用程序](#CreateApplication)
* [配置你的应用程序以访问 Blob 服务](#ConfigureStorage)
* [设置 Azure 存储连接](#ConnectionString)
* [如何：创建容器](#CreateContainer)
* [如何：将 Blob 上载到容器](#UploadBlob)
* [如何：列出容器中的 Blob](#ListBlobs)
* [如何：下载 Blob](#DownloadBlob)
* [如何：删除 Blob](#DeleteBlob)
* [如何：删除 Blob 容器](#DeleteContainer)
* [后续步骤](#NextSteps)

[WACOM.INCLUDE [howto-blob-storage](../includes/howto-blob-storage.md)]

<h2><a id="CreateAccount"></a>创建 Azure 存储帐户</h2>

[WACOM.INCLUDE [create-storage-account](../includes/create-storage-account.md)]

<h2><a id="CreateApplication"></a>创建 PHP 应用程序</h2>

创建访问 Azure Blob 服务的 PHP 应用程序的唯一要求是从代码中引用 Azure SDK for PHP 中的类。你可以使用任何开发工具（包括"记事本"）创建应用程序。

在本指南中，你将使用服务功能，这些功能可在 PHP 应用程序中本地调用，或通过在 Azure 的 Web 角色、辅助角色或网站中运行的代码调用。

<h2><a id="GetClientLibrary"></a>获取 Azure 客户端库</h2>

[WACOM.INCLUDE [get-client-libraries](../includes/get-client-libraries.md)]

<h2><a id="ConfigureStorage"></a>配置你的应用程序以访问 Blob 服务</h2>

若要使用 Azure Blob 服务 API，你需要：

1. 使用 [require_once][require_once] 语句引用 autoloader 文件，并
2. 引用可使用的所有类。

下面的示例演示了如何包括 autoloader 文件并引用 ServicesBuilder 类。

> [WACOM.NOTE]
> 本示例（以及本文中的其他示例）假定您已通过 Composer 安装用于 Azure 的 PHP 客户端库。如果你已手动安装这些库或将其作为 PEAR 包安装，则需要引用  `WindowsAzure.php` autoloader 文件。

	require_once 'vendor\autoload.php';
	use WindowsAzure\Common\ServicesBuilder;


在下面的示例中， `require_once` 语句将始终显示，但只会引用执行该示例所需的类。

<h2><a id="ConnectionString"></a>设置 Azure 存储连接</h2>

若要实例化 Azure Blob 服务客户端，你必须首先具有有效的连接字符串。Blob 服务连接字符串的格式为：

对于访问实时服务：

	DefaultEndpointsProtocol=[http|https];AccountName=[yourAccount];AccountKey=[yourKey]

For 访问模拟器存储：

	UseDevelopmentStorage=true


若要创建任何 Azure 服务客户端，你将需要使用 ServicesBuilder 类。您可以：

* 将连接字符串直接传递给此类或
* 使用 **CloudConfigurationManager (CCM)** 检查多个外部源以获取连接字符串：
	* 默认情况下，它附带了对一个外部源的支持 - 环境变量
	* 你可通过扩展 ConnectionStringSource 类来添加新源

在此处列出的示例中，将直接传递连接字符串。

	require_once 'vendor\autoload.php';

	use WindowsAzure\Common\ServicesBuilder;

	$blobRestProxy = ServicesBuilder::getInstance()->createBlobService($connectionString);

<h2><a id="CreateContainer"></a>如何：创建容器</h2>

利用 BlobRestProxy 对象，你可以使用 createContainer 方法创建 Blob 容器。创建容器时，可以在该容器上设置选项，但此操作不是必需的。（下面的示例演示了如何设置容器 ACL 和容器元数据。）

	require_once 'vendor\autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use WindowsAzure\Blob\Models\CreateContainerOptions;
	use WindowsAzure\Blob\Models\PublicAccessType;
	use WindowsAzure\Common\ServiceException;

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
	
	// Set container metadata
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

调用 setPublicAccess(PublicAccessType::CONTAINER\_AND\_BLOBS) 将使容器和 Blob 数据可通过匿名请求访问。通过调用 setPublicAccess(PublicAccessType::BLOBS_ONLY)，仅使 Blob 数据可通过匿名请求访问。有关容器 ACL 的详细信息，请参阅 [设置容器 ACL (REST API)][container-acl]。

有关 Blob 服务错误代码的详细信息，请参阅 [Blob 服务错误代码][error-codes]。

<h2><a id="UploadBlob"></a>如何：将 Blob 上载到容器</h2>

若要将文件作为 Blob 上载，请使用 BlobRestProxy->createBlockBlob 方法。此操作将创建 Blob（如果该 Blob 不存在），或者覆盖它（如果该 Blob 存在）。下面的代码示例假定已创建了容器，并使用 [fopen][fopen] 将文件作为流打开。

	require_once 'vendor\autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use WindowsAzure\Common\ServiceException;

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

请注意，上面的示例将 Blob 作为流上载。但是，也可使用 [file\_get\_contents][file_get_contents] 之类的函数将 Blob 作为字符串上载。为此，请将上面示例中的 `$content = fopen("c:\myfile.txt", "r");` 更改为 `$content = file_get_contents("c:\myfile.txt");`。

<h2><a id="ListBlobs"></a>如何：列出容器中的 Blob</h2>

若要列出容器中的 Blob，请将 BlobRestProxy->listBlobs 方法与 foreach 循环一起使用来循环访问结果。以下代码将容器中的每个 Blob 的名称及其 URI 输出到浏览器。

	require_once 'vendor\autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use WindowsAzure\Common\ServiceException;

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


<h2><a id="DownloadBlob"></a>如何：下载 Blob</h2>

若要下载 Blob，请调用 BlobRestProxy->getBlob 方法，然后对生成的 GetBlobResult 对象调用 getContentStream 方法。

	require_once 'vendor\autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use WindowsAzure\Common\ServiceException;

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

请注意，上面的示例将 Blob 作为流资源获取（默认行为）。但是，你可以使用 [stream\_get\_contents][stream-get-contents] 函数将返回的流转换为字符串。

<h2><a id="DeleteBlob"></a>如何：删除 Blob</h2>

若要删除 Blob，请将容器名称和 Blob 名称传递到 BlobRestProxy->deleteBlob。 

	require_once 'vendor\autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use WindowsAzure\Common\ServiceException;

	// Create blob REST proxy.
	$blobRestProxy = ServicesBuilder::getInstance()->createBlobService($connectionString);

	
	try	{
		// Delete container.
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

<h2><a id="DeleteContainer"></a>如何：删除 Blob 容器</h2>

最后，若要删除 Blob 容器，请将容器名称传递到 BlobRestProxy->deleteContainer。

	require_once 'vendor\autoload.php';

	use WindowsAzure\Common\ServicesBuilder;
	use WindowsAzure\Common\ServiceException;

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

<h2><a id="NextSteps"></a>后续步骤</h2>

现在，你已了解了 Azure Blob 服务的基础知识，单击下面的链接可了解如何执行更复杂的存储任务。

- 查看 MSDN 参考：[在 Azure 中存储和访问数据] []
- 访问 Azure 存储空间团队博客：<http://blogs.msdn.com/b/windowsazurestorage/>
- 参阅位于以下位置的 PHP 块 Blob 示例：<https://github.com/WindowsAzure/azure-sdk-for-php-samples/blob/master/storage/BlockBlobExample.php>.
- 参阅位于以下位置的 PHP 页 Blob 示例：<https://github.com/WindowsAzure/azure-sdk-for-php-samples/blob/master/storage/PageBlobExample.php>

[download]: /zh-cn/documentation/articles/php-download-sdk/
[在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
[container-acl]: http://msdn.microsoft.com/zh-cn/library/azure/dd179391.aspx
[error-codes]: http://msdn.microsoft.com/zh-cn/library/azure/dd179439.aspx
[file_get_contents]: http://php.net/file_get_contents
[require_once]: http://php.net/require_once
[fopen]: http://www.php.net/fopen
[stream-get-contents]: http://www.php.net/stream_get_contents
<!--HONumber=41-->
