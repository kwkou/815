<properties 
	pageTitle="如何通过 Node.js 使用 Blob 存储 | Azure" 
	description="了解如何使用 Blob 存储上载、下载、列出和删除 Blob 内容。示例用 Node.js 编写。"
	services="storage" 
	documentationCenter="nodejs" 
	authors="MikeWasson" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="storage" 
	ms.date="04/08/2016"
	wacn.date="05/23/2016"/>



# 如何通过 Node.js 使用 Blob 存储

[AZURE.INCLUDE [storage-selector-blob-include](../includes/storage-selector-blob-include.md)]

## 概述

本文介绍如何使用 Blob 存储执行常见方案。相关示例是通过 Node.js API 编写的。涉及的方案包括如何上载、列出、下载和删除 Blob。

[AZURE.INCLUDE [storage-blob-concepts-include](../includes/storage-blob-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

## 创建 Node.js 应用程序

有关创建 Node.js 应用程序的说明，请参阅[在 Azure App Service 中创建 Node.js Web 应用]，使用 Windows PowerShell [生成 Node.js 应用程序并将其部署到 Azure 云服务]，或[使用 Web Matrix 生成 Node.js Web 应用并将其部署到 Azure]。

## 配置应用程序以访问存储

若要使用 Azure 存储空间，你需要 Azure Storage SDK for Node.js，其中包括一组便于与存储 REST 服务进行通信的库。

### 使用 Node 包管理器 (NPM) 可获取该程序包

1.  使用 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (Unix) 等命令行界面导航到你在其中创建了示例应用程序的文件夹。

2.  在命令窗口中键入 **npm install azure-storage**。该命令的输出类似于以下代码示例。

		azure-storage@0.5.0 node_modules\azure-storage
		+-- extend@1.2.1
		+-- xmlbuilder@0.4.3
		+-- mime@1.2.11
		+-- node-uuid@1.4.3
		+-- validator@3.22.2
		+-- underscore@1.4.4
		+-- readable-stream@1.0.33 (string_decoder@0.10.31, isarray@0.0.1, inherits@2.0.1, core-util-is@1.0.1)
		+-- xml2js@0.2.7 (sax@0.5.2)
		+-- request@2.57.0 (caseless@0.10.0, aws-sign2@0.5.0, forever-agent@0.6.1, stringstream@0.0.4, oauth-sign@0.8.0, tunnel-agent@0.4.1, isstream@0.1.2, json-stringify-safe@5.0.1, bl@0.9.4, combined-stream@1.0.5, qs@3.1.0, mime-types@2.0.14, form-data@0.2.0, http-signature@0.11.0, tough-cookie@2.0.0, hawk@2.3.1, har-validator@1.8.0)

3.  可以手动运行 **ls** 命令来验证是否创建了 **node\_modules** 文件夹。在该文件夹中，你将找到 **azure-storage** 包，其中包含你访问存储所需的库。

### 导入包

使用记事本或其他文本编辑器将以下内容添加到你要在其中使用存储的应用程序的 **server.js** 文件的顶部：

    var azure = require('azure-storage');

## 设置 Azure 存储连接

Azure 模块将读取环境变量 `AZURE_STORAGE_ACCOUNT`、`AZURE_STORAGE_ACCESS_KEY` 或 `AZURE_STORAGE_CONNECTION_STRING`，以便获取连接到 Azure 存储帐户所需的信息。如果未设置这些环境变量，则在调用 **createBlobService** 时必须指定帐户信息。

有关在管理门户中为 Azure Web 应用设置环境变量的示例，请参阅[使用 Azure 表服务的 Node.js Web 应用]。

## 创建容器

使用 **BlobService** 对象可以对容器和 Blob 进行操作。以下代码创建 **BlobService** 对象。将以下内容添加到 **server.js** 顶部附近。

    var blobSvc = azure.createBlobService();

> [AZURE.NOTE] 您可以匿名访问 Blob，只需使用 **createBlobServiceAnonymous** 并提供主机地址即可。例如，使用 `var blobSvc = azure.createBlobServiceAnonymous('https://myblob.blob.core.chinacloudapi.cn/');`。

[AZURE.INCLUDE [storage-container-naming-rules-include](../includes/storage-container-naming-rules-include.md)]

若要创建一个新的容器，请使用 **createContainerIfNotExists**。以下代码示例将创建名为“mycontainer”的新容器：

	blobSvc.createContainerIfNotExists('mycontainer', function(error, result, response){
	    if(!error){
	      // Container exists and allows
	      // anonymous read access to blob
	      // content and metadata within this container
	    }
	});

如果该容器是新建的，则 `result.created` 为 true。如果该容器已存在，则 `result.created` 为 false。`response` 包含有关操作的信息，包括容器的 ETag 信息。

### 容器安全性

默认情况下，新容器是私有的，不能匿名访问。若要使容器公开，以便能够对其进行匿名访问，可将容器的访问级别设置为“Blob”或“容器”。

* **Blob** - 可匿名读取此容器中的 Blob 内容和元数据，但无法匿名读取容器元数据（如列出容器中的所有 Blob）

* **容器** - 可匿名读取 Blob 内容和元数据，以及容器元数据

以下代码示例演示了如何将访问级别设置为“Blob”：

	blobSvc.createContainerIfNotExists('mycontainer', {publicAccessLevel : 'blob'}, function(error, result, response){
	    if(!error){
	      // Container exists and is private
	    }
	});

另外，您可以通过使用 **setContainerAcl** 指定访问级别来修改容器的访问级别。以下代码示例将访问级别更改为“容器”：

	blobSvc.setContainerAcl('mycontainer', null /* signedIdentifiers */, {publicAccessLevel : 'container'} /* publicAccessLevel*/, function(error, result, response){
	  if(!error){
	    // Container access level set to 'container'
	  }
	});

结果将包含有关操作的信息，包括容器的当前 **ETag**。

### 筛选器

你可以向使用 **BlobService** 执行的操作应用可选的筛选操作。筛选操作可包括日志记录、自动重试等。筛选器是实现具有签名的方法的对象：

	function handle (requestOptions, next)

在对请求选项执行预处理后，该方法需要调用“next”并且传递具有以下签名的回调：

	function (returnObject, finalCallback, next)

在此回调中并且在处理 returnObject（来自对服务器请求的响应）后，回调需要调用 next（如果它存在以便继续处理其他筛选器）或只调用 finalCallback 以便结束服务调用。

Azure SDK for Node.js 中附带了两个实现了重试逻辑的筛选器，分别是 **ExponentialRetryPolicyFilter** 和 **LinearRetryPolicyFilter**。下面的代码将创建一个 **BlobService** 对象，该对象使用 **ExponentialRetryPolicyFilter**：

	var retryOperations = new azure.ExponentialRetryPolicyFilter();
	var blobSvc = azure.createBlobService().withFilter(retryOperations);

## 将 Blob 上载到容器中

有三种类型的 Blob：块 Blob、页 Blob 和追加 Blob。块 Blob 使你能够更高效地上传大型数据。追加 Blob 针对追加操作进行了优化。页 Blob 针对读取/写入操作进行了优化。有关详细信息，请参阅[了解块 Blob、追加 Blob 和页 Blob](http://msdn.microsoft.com/zh-cn/library/azure/ee691964.aspx)。

### 块 Blob

若要将数据上传到块 Blob，可使用以下方法：

* **createBlockBlobFromLocalFile** - 创建新的块 Blob 和上传文件的内容。

* **createBlockBlobFromStream** - 创建新的块 Blob 和上传流的内容。

* **createBlockBlobFromText** - 创建新的块 Blob 和上传字符串的内容。

* **createWriteStreamToBlockBlob** - 向块 Blob 提供写入流。

以下代码示例将 **test.txt** 文件的内容上传到 **myblob** 中。

	blobSvc.createBlockBlobFromLocalFile('mycontainer', 'myblob', 'test.txt', function(error, result, response){
	  if(!error){
	    // file uploaded
	  }
	});

这些方法返回的 `result` 将包含有关操作的信息，例如 Blob 的 **ETag**。

### 追加 Blob

若要将数据上传到新的追加 Blob，可使用以下方法：

* **createAppendBlobFromLocalFile** - 创建新的追加 Blob 并上传文件的内容

* **createAppendBlobFromStream** - 创建新的追加 Blob 并上传流的内容

* **createAppendBlobFromText** - 创建新的追加 Blob 并上传字符串的内容

* **createWriteStreamToNewAppendBlob** - 创建新的 Blob，然后向其提供要写入的流

以下代码示例将 **test.txt** 文件的内容上传到 **myappendblob** 中。

	blobSvc.createAppendBlobFromLocalFile('mycontainer', 'myappendblob', 'test.txt', function(error, result, response){
	  if(!error){
	    // file uploaded
	  }
	});

若要将块追加到现有追加 Blob，请使用以下方法：

* **appendFromLocalFile** - 将文件的内容追加到现有追加 Blob

* **appendFromStream** - 将流的内容追加到现有追加 Blob

* **appendFromText** - 将字符串的内容追加到现有追加 Blob

* **appendBlockFromStream** - 将流的内容追加到现有追加 Blob

* **appendBlockFromText** - 将字符串的内容追加到现有追加 Blob

> [AZURE.NOTE] appendFromXXX API 将会执行某些客户端验证以快速失败，从而避免不必要的服务器调用。而 appendBlockFromXXX 则不会如此。

以下代码示例将 **test.txt** 文件的内容上传到 **myappendblob** 中。

	blobSvc.appendFromText('mycontainer', 'myappendblob', 'text to be appended', function(error, result, response){
	  if(!error){
	    // text appended
	  }
	});


### 页 Blob

若要将数据上传到页 Blob，可使用以下方法：

* **createPageBlob** - 创建新的特定长度的页 Blob。

* **createPageBlobFromLocalFile** - 创建新的页 Blob 并上传文件的内容。

* **createPageBlobFromStream** - 创建新的页 Blob 并上传流的内容。

* **createWriteStreamToExistingPageBlob** - 向现有页 Blob 提供写入流。

* **createWriteStreamToNewPageBlob** - 创建新的 blob，然后向其提供写入流。

以下代码示例将 **test.txt** 文件的内容上传到 **mypageblob** 中。

	blobSvc.createPageBlobFromLocalFile('mycontainer', 'mypageblob', 'test.txt', function(error, result, response){
	  if(!error){
	    // file uploaded
	  }
	});

> [AZURE.NOTE] 页 Blob 包含 512 字节的“页面”。当你上传大小不是 512 倍数的数据时，将会收到错误。

## 列出容器中的 Blob

若要列出容器中的 Blob，请使用 **listBlobsSegmented** 方法。如果你想要返回带特定前缀的 Blob，请使用 **listBlobsSegmentedWithPrefix**。

	blobSvc.listBlobsSegmented('mycontainer', null, function(error, result, response){
	  if(!error){
	      // result.entries contains the entries
	      // If not all blobs were returned, result.continuationToken has the continuation token.
	  }
	});

`result` 包含一个 `entries` 集合，该集合是一组用于描述每个 Blob 的对象。如果不能返回所有的 Blob，`result` 还将提供 `continuationToken`，这可用作第二个参数来检索其他条目。

## 下载 Blob

若要从 Blob 下载数据，可使用以下方法：

* **getBlobToLocalFile** - 将 Blob 内容写入文件

* **getBlobToStream** - 将 Blob 内容写入流

* **getBlobToText** - 将 Blob 内容写入字符串

* **createReadStream** - 提供可从 Blob 读取内容的流

以下代码示例演示了如何使用 **getBlobToStream** 下载 **myblob** Blob 的内容，并使用一个流将其存储到 **output.txt** 文件：

	var fs = require('fs');
	blobSvc.getBlobToStream('mycontainer', 'myblob', fs.createWriteStream('output.txt'), function(error, result, response){
	  if(!error){
	    // blob retrieved
	  }
	});

`result` 包含有关 Blob 的信息，包括 **ETag** 信息。

## 删除 Blob

最后，若要删除 Blob，请调用 **deleteBlob**。以下代码示例将删除名为 **myblob** 的 Blob。

	blobSvc.deleteBlob(containerName, 'myblob', function(error, response){
	  if(!error){
		// Blob has been deleted
	  }
	});

## 并发访问

若要允许从多个客户端或多个进程实例并发访问某个 Blob，您可以使用 **ETag** 或**租约**。

* **Etag** - 用于检测 Blob 或容器是否已被其他进程修改

* **租约** - 用于在某个时段内获取对 Blob 的独占式可续订写入或删除访问

### ETag

如果你需要允许多个客户端或实例同时写入块 Blob 或页 Blob，请使用 ETag。ETag 用于确定自从你第一次读取或创建某个容器或 Blob 以来，该容器或 Blob 是否被修改，这样就可以避免覆盖其他客户端或进程提交的更改。

可以使用可选的 `options.accessConditions` 参数设置 ETag 条件。如果 Blob 已存在且具有 `etagToMatch` 所包含的 ETag 值，则以下代码示例将仅上传 **test.txt** 文件。

	blobSvc.createBlockBlobFromLocalFile('mycontainer', 'myblob', 'test.txt', { accessConditions: { EtagMatch: etagToMatch} }, function(error, result, response){
	    if(!error){
	    // file uploaded
	  }
	});

当你使用 ETag 时，常规模式为：

1. 通过创建、列出或获取操作来获取 ETag。

2. 执行一个操作，查看 ETag 值是否尚未修改。

如果值已修改，则表明在你获得 ETag 值后，其他客户端或实例已修改该 Blob 或容器。

### 租约

新的租约可使用 **acquireLease** 方法获取，只需指定你希望获取其租约的 Blob 或容器即可。例如，以下代码将获取 **myblob** 的租约。

	blobSvc.acquireLease('mycontainer', 'myblob', function(error, result, response){
	  if(!error) {
	    console.log('leaseId: ' + result.id);
	  }
	});

对 **myblob** 的后续操作必须提供 `options.leaseId` 参数。租约 ID 作为 `result.id` 从 **acquireLease** 返回。

> [AZURE.NOTE] 默认情况下，租约期限为无期。你可以指定一个有限的租期（15 到 60 秒），只需提供 `options.leaseDuration` 参数即可。

若要删除租约，请使用 **releaseLease**。若要中断租约，但又要防止其他人在您的原始租约到期之前获得新租约，则可使用 **breakLease**。

## 使用共享访问签名

共享访问签名 (SAS) 是一种安全的方法，用于对 Blob 和容器进行细致访问而无需提供你的存储帐户名或密钥。通常使用共享访问签名来提供对你的数据的有限访问权限，例如允许移动应用程序访问 Blob。

> [AZURE.NOTE] 虽然你也可以允许匿名访问 Blob，但共享访问签名可以让你提供更受控制的访问，因为你必须生成 SAS。

受信任的应用程序（例如基于云的服务）可使用 **BlobService** 的 **generateSharedAccessSignature** 生成共享访问签名，然后将其提供给不受信任的或不完全受信任的应用程序，例如移动应用。共享访问签名可使用策略生成，该策略描述了共享访问签名的生效日期和失效日期，以及授予共享访问签名持有者的访问级别。

以下代码示例生成了一个新的共享访问策略，该策略将允许共享访问签名持有者对 **myblob** Blob 执行读取操作，并且在创建后 100 分钟过期。

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

请注意，还必须提供主机信息，因为共享访问签名持有者尝试访问容器时，必须提供该信息。

然后，客户端应用程序将共享访问签名用于 **BlobServiceWithSAS**，以便针对 Blob 执行操作。以下语句获取有关 **myblob** 的信息。

	var sharedBlobSvc = azure.createBlobServiceWithSas(host, blobSAS);
	sharedBlobSvc.getBlobProperties('mycontainer', 'myblob', function (error, result, response) {
	  if(!error) {
	    // retrieved info
	  }
	});

由于共享访问签名在生成时具有只读访问权限，因此如果尝试修改 Blob，则会返回错误。

### 访问控制列表

你还可以使用访问控制列表 (ACL) 为 SAS 设置访问策略。如果你希望允许多个客户端访问某个容器，但为每个客户端提供了不同的访问策略，则访问控制列表会很有用。

ACL 是使用一组访问策略实施的，每个策略都有一个关联的 ID。以下代码示例定义了两个策略，一个用于“user1”，一个用于“user2”：

	var sharedAccessPolicy = {
	  user1: {
	    Permissions: azure.BlobUtilities.SharedAccessPermissions.READ,
	    Start: startDate,
	    Expiry: expiryDate
	  },
	  user2: {
	    Permissions: azure.BlobUtilities.SharedAccessPermissions.WRITE,
	    Start: startDate,
	    Expiry: expiryDate
	  }
	};

以下代码示例将获取 **mycontainer** 的当前 ACL，然后使用 **setBlobAcl** 添加新策略。此方法具有以下用途：

	var extend = require('extend');
	blobSvc.getBlobAcl('mycontainer', function(error, result, response) {
	  if(!error){
	    var newSignedIdentifiers = extend(true, result.signedIdentifiers, sharedAccessPolicy);
	    blobSvc.setBlobAcl('mycontainer', newSignedIdentifiers, function(error, result, response){
	      if(!error){
	        // ACL set
	      }
	    });
	  }
	});

设置 ACL 后，你可以根据某个策略的 ID 创建共享访问签名。以下代码示例将为“user2”创建新的共享访问签名：

	blobSAS = blobSvc.generateSharedAccessSignature('mycontainer', { Id: 'user2' });

## 后续步骤

有关详细信息，请参阅以下资源。

-   [Azure Storage SDK for Node API 参考][]
-   [Azure 存储团队博客][]
-   GitHub 上的 [Azure Storage SDK for Node][] 存储库
-   [Node.js 开发人员中心](/develop/nodejs/)
-   [使用 AzCopy 命令行实用程序传输数据](/documentation/articles/storage-use-azcopy/)

[Azure Storage SDK for Node]: https://github.com/Azure/azure-storage-node

[在 Azure App Service 中创建 Node.js Web 应用]: /documentation/articles/web-sites-nodejs-develop-deploy-mac/
[Node.js Cloud Service with Storage]: /documentation/articles/storage-nodejs-use-table-storage-cloud-service-app/
[使用 Azure 表服务的 Node.js Web 应用]: /documentation/articles/storage-nodejs-use-table-storage-web-site/
[使用 Web Matrix 生成 Node.js Web 应用并将其部署到 Azure]: /documentation/articles/web-sites-nodejs-use-webmatrix/
[Using the REST API]: http://msdn.microsoft.com/zh-cn/library/azure/hh264518.aspx
[Azure Portal]: https://portal.azure.cn
[生成 Node.js 应用程序并将其部署到 Azure 云服务]: /documentation/articles/cloud-services-nodejs-develop-deploy-app/
[Azure 存储团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
[Azure Storage SDK for Node API 参考]: http://azure.github.io/azure-storage-node/
 
<!---HONumber=Mooncake_0516_2016-->