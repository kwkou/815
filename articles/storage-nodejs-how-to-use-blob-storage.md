<properties linkid="dev-nodejs-how-to-blob-storage" urlDisplayName="Blob Service" pageTitle="如何使用 Blob 存储 (Node.js) | Microsoft Azure" metaKeywords="Get started Azure blob, Azure unstructured data, Azure unstructured storage, Azure blob, Azure blob storage, Azure blob Node.js" description="了解如何使用 Azure Blob 服务上载、下载、列出和删除 Blob 内容。相关示例是使用 Node.js 编写的。" metaCanonical="" services="storage" documentationCenter="Node.js" title="How to Use the Blob Service from Node.js" authors="larryfr" solutions="" manager="" editor="" />





# 如何从 Node.js 使用 Blob 服务

本指南将演示如何使用 Azure 表存储服务
Azure Blob 服务。相关示例是使用
Node.js API 编写的。涉及的方案包括**上载**、**列出**、
**下载**和**删除** Blob。有关 Blob 的详细信息，请参阅[后续步骤][]部分。

## 目录

* [什么是 Blob 服务？][]    
* [概念][]    
* [创建 Azure 存储帐户][]    
* [创建 Node.js 应用程序][]  
* [配置应用程序以范围存储][]     
* [设置 Azure 存储连接字符串][]  
* [如何：创建容器][]  
* [如何：将 Blob 上载到容器][]  
* [如何：列出容器中的 Blob][]  
* [如何：下载 Blob][]  
* [如何：删除 Blob][]  
* [如何：并发访问][]     
* [如何：使用共享访问签名][]     
* [后续步骤][]

[WACOM.INCLUDE [howto-blob-storage](../includes/howto-blob-storage.md)]

##<a name="create-account"></a>创建 Azure 存储帐户
[WACOM.INCLUDE [create-storage-account](../includes/create-storage-account.md)]

## <a name="create-app"> </a>创建 Node.js 应用程序

创建一个空的 Node.js 应用程序。有关创建 Node.js 应用程序的说明，请参阅 [创建 Node.js 应用程序并将其部署到 Azure 网站]、[Node.js 云服务][Node.js 云服务]（使用 Windows PowerShell）或 [使用 WebMatrix 构建网站]。

## <a name="configure-access"> </a>配置应用程序以访问存储

若要使用 Azure 存储空间，你需要 Azure Storage SDK for Node.js，其中包括一组便于与存储 REST 服务进行通信的库。

### 使用 Node 包管理器 (NPM) 可获取该程序包

1.  使用 PowerShell (Windows)、Terminal (Mac) 或 Bash (Unix) 等命令行界面导航到你在其中创建了示例应用程序的文件夹。

2.  在命令窗口中键入 npm install azure-storage，这应该产生以下输出：

        azure-storage@0.1.0 node_modules\azure-storage
		├── extend@1.2.1
		├── xmlbuilder@0.4.3
		├── mime@1.2.11
		├── underscore@1.4.4	
		├── validator@3.1.0
		├── node-uuid@1.4.1
		├── xml2js@0.2.7 (sax@0.5.2)
		└── request@2.27.0 (json-stringify-safe@5.0.0, tunnel-agent@0.3.0, aws-sign@0.3.0, forever-agent@0.5.2, qs@0.6.6, oauth-sign@0.3.0, cookie-jar@0.3.0, hawk@1.0.0, form-data@0.1.3, http-signature@0.10.0)

3.  可以手动运行 **ls** 命令来验证是否创建了 **node\_modules** 文件夹。在该文件夹中，你将找到 **azure-storage** 包，其中包含你访问存储所需的库。

### 导入包

使用记事本或其他文本编辑器将以下内容添加到应用程序的
**server.js** 文件的顶部，以便在其中使用存储：

    var azure = require('azure-storage');

## <a name="setup-connection-string"> </a>设置 Azure 存储连接

Azure 模块将读取环境变量 AZURE\_STORAGE\_ACCOUNT 和 AZURE\_STORAGE\_ACCESS\_KEY 或 AZURE\_STORAGE\_CONNECTION\_STRING，以便获取连接到 Azure 存储帐户所需的信息。如果未设置这些环境变量，则在调用 createBlobService 时必须指定帐户信息。

有关在管理门户中为 Azure 网站设置环境变量的示例，请参阅[使用存储构建 Node.js Web 应用程序]

## <a name="create-container"> </a>如何：创建容器

使用 BlobService 对象可以对容器和 Blob 进行操作。以下代码创建 BlobService 对象。将以下内容添加到 server.js 顶部附近。

    var blobSvc = azure.createBlobService();

> [WACOM.NOTE] 你可以匿名访问 Blob，只需使用 **createBlobServiceAnonymous** 并提供主机地址即可。例如，`var blobSvc = azure.createBlobService('https://myblob.blob.core.chinacloudapi.cn/');`.

All Blob 驻留在一个容器中。若要创建一个新的容器，请使用 **createContainerIfNotExists**。以下语句创建名为  'mycontainer' 的新容器

	blobSvc.createContainerIfNotExists('mycontainer', function(error, result, response){
      if(!error){
        // Container exists and allows 
        // anonymous read access to blob 
        // content and metadata within this container
      }
	});

如果创建了容器， `result` 将为 true。如果容器已存在， `result` 将为 false。 `response` 将包含有关操作的信息，包括 [ETag](http://en.wikipedia.org/wiki/HTTP_ETag) information for the container.

###容器安全性

默认情况下，新容器是私有的，不能匿名访问。若要使容器公开，以便能够对其进行匿名访问，可将容器的访问级别设置为 **Blob** 或"容器"。

* **Blob** - 可匿名读取此容器中的 Blob 内容和元数据，但无法匿名读取容器元数据（如列出容器中的所有 Blob）。

* **容器** - 可匿名读取 Blob 内容和元数据，以及容器元数据。

下面的示例演示了如何将访问级别设置为"Blob"： 

    blobSvc.createContainerIfNotExists('mycontainer', {publicAccessLevel : 'blob'}, function(error, result, response){
      if(!error){
        // Container exists and is private
      }
	});

Alternatively, you can modify the access level of a container by using **setContainerAcl** to specify the access level. The following example changes the access level to container:

    blobSvc.setContainerAcl('mycontainer', null, 'container', function(error, result, response){
	  if(!error){
		// Container access level set to 'container'
	  }
	});

结果将包含有关操作的信息，包括容器的当前 **ETag**。

###筛选器

可以向使用 BlobService 执行的操作应用可选的筛选操作。筛选操作可以包括日志记录、自动重试等。筛选器是实现了具有签名的方法的对象：

		function handle (requestOptions, next)

在对请求选项执行预处理后，该方法需要调用"next"并且传递具有以下签名的回调：

		function (returnObject, finalCallback, next)

在此回调中并且在处理 returnObject（来自对服务器请求的响应）后，回调需要调用 next（如果它存在以便继续处理其他筛选器）或只调用 finalCallback 以便结束服务调用。

Azure SDK for Node.js 中附带了两个实现了重试逻辑的筛选器，分别是 ExponentialRetryPolicyFilter 和 LinearRetryPolicyFilter。下面的代码将创建一个 BlobService 对象，该对象使用 ExponentialRetryPolicyFilter：

	var retryOperations = new azure.ExponentialRetryPolicyFilter();
	var blobSvc = azure.createBlobService().withFilter(retryOperations);

## <a name="upload-blob"> </a>如何：将 Blob 上载到容器

Blob 可能基于块，也可能基于页。块 Blob 可以让你更高效地上载大型数据，而页 Blob 则针对读/写操作进行了优化。有关详细信息，请参阅[了解块 Blob 和页 Blob](http://msdn.microsoft.com/zh-cn/library/azure/ee691964.aspx).

###块 Blob

若要将数据上载到块 Blob，可使用以下方法：

* **createBlockBlobFromLocalFile** - 创建新的块 Blob 和上载文件的内容。

* **createBlockBlobFromStream** - 创建新的块 Blob 和上载流的内容。

* **createBlockBlobFromText** - 创建新的块 Blob 和上载字符串的内容。

* **createWriteStreamToBlockBlob** - 向块 Blob 提供写入流。

下面的示例将 **test.txt** 文件的内容上载到 **myblob** 中。

	blobSvc.createBlockBlobFromLocalFile('mycontainer', 'myblob', 'test.txt', function(error, result, response){
	  if(!error){
	    // file uploaded
	  }
	});

这些方法返回的  `result` 将包含有关操作的信息，例如 Blob 的 **ETag**。

###页 Blob

若要将数据上载到页 Blob，可使用以下方法：

* **createPageBlob** - 创建新的特定长度的页 Blob。

* **createPageBlobFromLocalFile** - 创建新的页 Blob 并上载文件的内容。

* **createPageBlobFromStream** - 创建新的页 Blob 并上载流的内容。

* **createWriteStreamToExistingPageBlob** - 向现有页 Blob 提供写入流。

* **createWriteStreamToNewPageBlob** - 创建新的 blob，然后向其提供写入流。

下面的示例将 **test.txt** 文件的内容上载到 **mypageblob** 中。

	blobSvc.createPageBlobFromLocalFile('mycontainer', 'mypageblob', 'test.txt', function(error, result, response){
	  if(!error){
	    // file uploaded
	  }
	});

> [WACOM.NOTE] 页 Blob 包含 512 字节的  'pages'。当你上载大小不是 512 倍数的数据时，可能会收到错误。

## <a name="list-blob"> </a>如何：列出容器中的 Blob

若要列出容器中的 Blob，请使用 **listBlobsSegmented** 方法。如果你想要返回带特定前缀的 Blob，请使用 **listBlobsSegmentedWithPrefix**。

    blobSvc.listBlobsSegmented('mycontainer', null, function(error, result, response){
      if(!error){
        // result contains the entries
	  }
	});

 `result` 将包含一个  `entries` 集合，该集合是一组用于描述每个 Blob 的对象。如果不能返回所有 Blob， `result` 还将提供  `continuationToken`，这可用作第二个参数来检索其他条目。

## <a name="download-blob"> </a>如何：下载 Blob

若要从 Blob 下载数据，可使用以下方法：

* **getBlobToFile** - 将 Blob 内容写入文件

* **getBlobToStream** - 将 Blob 内容写入流。

* **getBlobToText** - 将 Blob 内容写入字符串。

* **createReadStream** - 提供可从 Blob 读取内容的流

以下示例演示了如何使用 getBlobToStream 下载 myblob Blob 的内容，并使用一个流将其存储到 output.txt 文件：

    var fs=require('fs');
	blobSvc.getBlobToStream('mycontainer', 'myblob', fs.createWriteStream('output.txt'), function(error, result, response){
	  if(!error){
	    // blob retrieved
	  }
	});

 `result` 将包含有关 Blob 的信息，包括 **ETag** 信息。

## <a name="delete-blob"> </a>如何：删除 Blob

最后，若要删除 Blob，请调用 deleteBlob。下面的示例将删除名为"myblob"的 Blob。

    blobSvc.deleteBlob(containerName, 'myblob', function(error, response){
	  if(!error){
		// Blob has been deleted
	  }
	});

##<a name="concurrent-access"></a>如何：并发访问

若要允许从多个客户端或多个进程实例并发访问某个 Blob，你可以使用 **ETag** 或**租约**。

* **Etag** - 用于检测 Blob 或容器是否已被其他进程修改。

* **租约** - 用于在某个时段内获取对 Blob 的独占式可续订写入或删除访问。

###ETag

如果你需要允许多个客户端或实例同时写入该 Blob，则应使用 ETag。ETag 用于确定自从你第一次读取或创建某个容器或 Blob 以来，该容器或 Blob 是否被修改，这样就可以避免覆盖其他客户端或进程提交的更改。

ETag 条件能够使用可选的  `options.accessConditions` 参数进行设置。如果 Blob 已存在且具有  `etagToMatch` 所包含的 ETag 值，则以下示例将仅上载 **test.txt** 文件。

	blobSvc.createBlockBlobFromLocalFile('mycontainer', 'myblob', 'test.txt', { accessConditions: { 'if-match': etagToMatch} }, function(error, result, response){
      if(!error){
	    // file uploaded
	  }
	});

ETag 的常规使用模式是：

1. 通过创建、列出或获取操作来获取 ETag。

2. 执行一个操作，查看 ETag 值是否尚未修改。

如果值已修改，则表明在你获得 ETag 值后，其他客户端或实例已修改该 blob 或容器。

###租约

新的租约可使用 **acquireLease** 方法获取，只需指定你希望获取其租约的 Blob 或容器即可。例如，以下语句将获取 **myblob** 的租约。

	blobSvc.acquireLease('mycontainer', 'myblob', function(error, result, response){
	  if(!error) {
	    console.log(result);
	  }
	});

对 **myblob** 的后续操作必须提供  `options.leaseId` 参数。将从 **acquireLease** 返回  `result.id` 形式的租约 ID。

> [WACOM.NOTE] 默认情况下，租约期限为无期。你可以指定一个无限的租期（15 到 60 秒），只需提供  `options.leaseDuration` 参数即可。

若要删除租约，请使用 **releaseLease**。若要中断租约，但又要防止其他人在你的原始租约到期之前获得新租约，则可使用 **breakLease**。

## <a name="sas"></a>如何：使用共享访问签名

共享访问签名 (SAS) 是一种安全的方法，用于对 Blob 和容器进行细致访问而无需提供你的存储帐户名或密钥。通常使用 SAS 来提供对你的数据的有限访问权限，例如允许移动应用程序访问 Blob。

> [WACOM.NOTE] 虽然你也可以允许匿名访问 Blob，但 SAS 可以让你提供更受控制的访问，因为你必须生成 SAS。

受信任的应用程序（例如基于云的服务）可使用 **BlobService** 的 **generateSharedAccessSignature** 生成 SAS，然后将其提供给不受信任的或不完全受信任的应用程序。例如，移动应用程序。SAS 可使用策略生成，该策略描述了 SAS 的生效日期和失效日期，以及授予 SAS 持有者的访问级别。

下面的示例生成了一个新的共享访问策略，该策略将允许 SAS 持有者对 **myblob** Blob 执行读取操作，在创建后 100 分钟过期。

	var startDate = new Date();
	var expiryDate = new Date(startDate);
	expiryDate.setMinutes(startDate.getMinutes() + 100);
	startDate.setMinutes(startDate.getMinutes() - 100);
	    
	var sharedAccessPolicy = {
	  AccessPolicy: {
	    Permissions: azure.BlobUtilities.SharedAccessPermissions.READ,
	    Start: startDate,
	    Expiry: expiryDate
	  },
	};
	
	var blobSAS = blobSvc.generateSharedAccessSignature('mycontainer', 'myblob', sharedAccessPolicy);
	var host = blobSvc.host;

请注意，还必须提供主机信息，因为 SAS 持有者尝试访问容器时，必须提供该信息。

然后，客户端应用程序将 SAS 用于 **BlobServiceWithSAS**，以便针对 Blob 执行操作。以下语句获取有关 **myblob** 的信息。

	var sharedBlobSvc = azure.createBlobServiceWithSas(host, blobSAS);
	sharedBlobSvc.getBlobProperties('mycontainer', 'myblob', function (error, result, response) {
	  if(!error) {
	    // retrieved info
	  }
	});

由于 SAS 在生成时只具有读取访问权限，因此如果尝试修改 Blob，则会返回错误。

###访问控制列表

你还可以使用访问控制列表 (ACL) 为 SAS 设置访问策略。如果你希望允许多个客户端访问某个容器，但为每个客户端提供了不同的访问策略，则访问控制列表会很有用。

ACL 是使用一组访问策略实施的，每个策略都有一个关联的 ID。下面的示例定义了两个策略，一个用于"user1"，一个用于"user2"：

	var sharedAccessPolicy = [
	  {
	    AccessPolicy: {
	      Permissions: azure.BlobUtilities.SharedAccessPermissions.READ,
	      Start: startDate,
	      Expiry: expiryDate
	    },
	    Id: 'user1'
	  },
	  {
	    AccessPolicy: {
	      Permissions: azure.BlobUtilities.SharedAccessPermissions.WRITE,
	      Start: startDate,
	      Expiry: expiryDate
	    },
	    Id: 'user2'
	  }
	];

下面的示例获取 **mycontainer** 的当前 ACL，，然后使用 **setBlobAcl** 添加新策略。此方法具有以下用途：

	blobSvc.getBlobAcl('mycontainer', function(error, result, response) {
      if(!error){
		//push the new policy into signedIdentifiers
		result.signedIdentifiers.push(sharedAccessPolicy);
		blobSvc.setBlobAcl('mycontainer', result, function(error, result, response){
	  	  if(!error){
	    	// ACL set
	  	  }
		});
	  }
	});

设置 ACL 后，你可以根据某个策略的 ID 创建 SAS。以下示例为  'user2': 创建新的 SAS

	blobSAS = blobSvc.generateSharedAccessSignature('mycontainer', { Id: 'user2' });

## <a name="next-steps"> </a>后续步骤

现在，你已了解有关 Blob 存储的基础知识，可单击下面的链接来了解如何执行更复杂的存储任务。

-   查看 MSDN 参考：[在 Azure 中存储和访问数据][]。
-   访问 [Azure 存储空间团队博客][]。
-   访问 GitHub 上的 [Azure Storage SDK for Node][] 存储库。

  [Azure Storage SDK for Node]: https://github.com/Azure/azure-storage-node
  [后续步骤]: #next-steps
  [什么是 Blob 服务？]: #what-is
  [概念]: #concepts
  [创建 Azure 存储帐户]: #create-account
  [创建 Node.js 应用程序]: #create-app
  [配置应用程序以范围存储]: #configure-access
  [设置 Azure 存储连接字符串]: #setup-connection-string
  [如何：创建容器]: #create-container
  [如何：将 Blob 上载到容器]: #upload-blob
  [如何：列出容器中的 Blob]: #list-blob
  [如何：下载 Blob]: #download-blobs
  [如何：删除 Blob]: #delete-blobs
  [如何：并发访问]: #concurrent-access
  [如何：使用共享访问签名]: #sas
[创建 Node.js 应用程序并将其部署到 Azure 网站]: /zh-cn/develop/nodejs/tutorials/create-a- Website-(mac)/
  [使用存储构建 Node.js 云服务]: /zh-cn/documentation/articles/storage-nodejs-use-table-storage-cloud-service-app/
  [使用存储构建 Node.js Web 应用程序]: /zh-cn/documentation/articles/storage-nodejs-use-table-storage-web-site/
 [使用 WebMatrix 构建网站]: /zh-cn/documentation/articles/web-sites-nodejs-use-webmatrix/
  [使用 REST API]: http://msdn.microsoft.com/zh-cn/library/azure/hh264518.aspx
  [Azure 管理门户]: http://manage.windowsazure.cn
  [Node.js 云服务]: /zh-cn/documentation/articles/cloud-services-nodejs-develop-deploy-app/
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [Azure 存储空间团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
<!--HONumber=41-->
