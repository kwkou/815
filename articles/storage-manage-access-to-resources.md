<properties 
	pageTitle="管理对 Azure 存储资源的访问" 
	description="了解可以通过哪些不同的方法来管理对 Azure 存储资源的访问。" 
	services="storage" 
	documentationCenter="" 
	authors="micurd,tamram" 
	manager="jahogg" 
	editor=""/>
	
<tags ms.service="storage" ms.date="02/20/2015" wacn.date="04/11/2015"/>


# 管理对 Azure 存储资源的访问

## 概述

默认情况下，只有存储帐户的所有者可访问该帐户中的 Blob、表和队列。如果服务或应用程序需要向其他客户端提供这些资源但不共享访问密钥，则有以下选项可允许访问：

- 可设置容器的权限以允许匿名读取该容器及其 Blob。表或队列不允许这样做。

- 可通过共享访问签名公开资源，这样即可通过指定提供资源的间隔以及客户端对于容器、Blob、表或队列资源将拥有的权限，允许对这些资源进行有限的访问。

- 可使用存储访问策略管理容器或其 Blob、队列或表的共享访问签名。存储访问策略对于共享访问签名提供了另一种控制措施，并提供了一种简单明了的撤消这些签名的方法。

## 限制对容器和 Blob 的访问

默认情况下，仅存储帐户的所有者能够访问容器以及其中的所有 Blob。若要授予匿名用户对容器及其 Blob 的读取权限，你可以设置容器权限以允许公共访问。匿名用户可以读取可公开访问的容器中的 Blob，而无需对请求进行身份验证。

容器提供了下列用于管理容器访问的选项：

- 完全公共读取访问：可通过匿名请求读取容器和 Blob 数据。客户端可以通过匿名请求枚举容器中的 Blob，但无法枚举存储帐户中的容器。

- 仅针对 Blob 的公共读取访问：可以通过匿名请求读取此容器中的 Blob 数据，但容器数据不可用。客户端无法通过匿名请求枚举容器中的 Blob。

- 无公共读取访问：仅帐户所有者可读取容器和 Blob 数据。

>[AZURE.NOTE]如果服务要求你对 Blob 资源进行更精确的控制，或者你希望提供针对读取操作以外的其他操作的权限，则可以使用"共享访问签名"来使资源对用户可访问。 

### 对匿名用户可用的功能
下表显示了在将容器的 ACL 设置为允许公共访问时匿名用户可调用的操作。

| REST 操作                                         | 具有完全公共读取访问权的权限 | 仅对于 Blob 具有公共读取访问权的权限 |
|--------------------------------------------------------|-----------------------------------------|---------------------------------------------------|
| 列出容器                                        | 仅所有者                              | 仅所有者                                        |
| 创建容器                                       | 仅所有者                              | 仅所有者                                        |
| 获取容器属性                               | 全部                                     | 仅所有者                                        |
| 获取容器元数据                                 | 全部                                     | 仅所有者                                        |
| 设置容器元数据                                 | 仅所有者                              | 仅所有者                                        |
| 获取容器 ACL                                      | 仅所有者                              | 仅所有者                                        |
| 设置容器 ACL                                      | 仅所有者                              | 仅所有者                                        |
| 删除容器                                       | 仅所有者                              | 仅所有者                                        |
| 列出 Blob                                             | 全部                                     | 仅所有者                                        |
| 放置 Blob                                               | 仅所有者                              | 仅所有者                                        |
| 获取 Blob                                               | 全部                                     | 全部                                               |
| 获取 Blob 属性                                    | 全部                                     | 全部                                               |
| 设置 Blob 属性                                    | 仅所有者                              | 仅所有者                                        |
| 获取 Blob 元数据                                      | 全部                                     | 全部                                               |
| 设置 Blob 元数据                                      | 仅所有者                              | 仅所有者                                        |
| 放置块                                              | 仅所有者                              | 仅所有者                                        |
| 获取块列表（仅限已提交的块）                 | 全部                                     | 全部                                               |
| 获取块列表（仅限未提交的块或所有块） | 仅所有者                              | 仅所有者                                        |
| 放置块列表                                         | 仅所有者                              | 仅所有者                                        |
| 删除 Blob                                            | 仅所有者                              | 仅所有者                                        |
| 复制 Blob                                              | 仅所有者                              | 仅所有者                                        |
| 为 Blob 创建快照                                         | 仅所有者                              | 仅所有者                                        |
| 租用 Blob                                             | 仅所有者                              | 仅所有者                                        |
| 放置页                                               | 仅所有者                              | 仅所有者                                        |
| 获取页范围                                        | 全部                                     | 全部                                                  |

## 创建和使用共享访问签名
共享访问签名是一个在特定时间间隔内授予对容器、Blob、队列和表的受限访问权限的 URI。通过为客户端提供共享访问签名，可使客户端能够访问存储帐户中的资源，而无需与客户端共享你的帐户密钥。

>[AZURE.NOTE] 有关深入的概念概述，以及有关共享访问签名的教程，请参阅[共享访问签名](/documentation/articles/storage-dotnet-shared-access-signature-part-1)

使用共享访问签名的受支持的操作包括：

- 读写页面或块 Blob 内容、块列表、属性和元数据

- 删除、租赁和创建 Blob 快照

- 列出容器中的 Blob

- 添加、移除、更新和删除队列消息（在存储客户端库 2.0 版和更新版本中）

- 获取队列元数据，包括消息计数（在存储客户端库 2.0 和更新版本中）

- 查询、添加、更新、删除和插入表实体（在存储客户端库 2.0 版和更新版本中）

共享访问签名 URI 查询参数合并了授予存储资源受控访问权限所需的所有必要信息。URI 查询参数指定共享访问签名的有效时间间隔、它所授予的权限、可用的资源以及存储服务用来对请求进行身份验证的签名。

此外，共享访问签名 URI 可以引用存储访问策略，对一组签名提供另一级别的控制，包括根据需要修改或撤消资源访问权限的功能。 

有关共享访问签名的 URI 格式的信息，请参阅[使用共享访问签名委托访问权限](https://msdn.microsoft.com/zh-CN/library/ee395415.aspx)。

### 共享访问签名的安全使用
共享访问签名授予对由 URI 授予的权限指定的资源的访问权。你应始终使用 HTTPS 构造共享访问签名 URI。如果将 HTTP 与共享访问签名一起使用，则会导致你的存储帐户易于受到恶意使用。

如果共享访问签名授予非公用的访问权限，则应使用尽可能少的权限构造该签名。此外，共享访问签名应通过安全连接安全地分发到客户端，应出于吊销目的而将该签名与存储访问策略关联，并且应为签名指定尽可能短的生存期。

>[AZURE.NOTE] 共享访问签名 URI 与用于创建签名的帐户密钥和关联的存储访问策略（如果有）相关联。如果未指定存储访问策略，则吊销共享访问签名的唯一方法是更改帐户密钥。 

### 创建共享访问签名
以下代码示例在容器上创建访问策略，然后为该容器生成共享访问签名。随后，可将此共享访问签名提供给客户端：

    // The connection string for the storage account.  Modify for your account.
    string storageConnectionString =
       "DefaultEndpointsProtocol=https;" +
       "AccountName=myaccount;" +
       "AccountKey=<account-key>";
    
    // As an alternative, you can retrieve storage account information from an app.config file. 
    // This is one way to store and retrieve a connection string if you are 
    // writing an application that will run locally, rather than in Microsoft Azure.
    
    // string storageConnectionString = ConfigurationManager.AppSettings["StorageAccountConnectionString"];
    
    // Create the storage account with the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(storageConnectionString);
       
    // Create the blob client object.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
    
    // Get a reference to the container for which shared access signature will be created.
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");
    container.CreateIfNotExists();
    
    // Create blob container permissions, consisting of a shared access policy 
    // and a public access setting. 
    BlobContainerPermissions blobPermissions = new BlobContainerPermissions();
    
    // The shared access policy provides 
    // read/write access to the container for 10 hours.
    blobPermissions.SharedAccessPolicies.Add("mypolicy", new SharedAccessBlobPolicy()
    {
       // To ensure SAS is valid immediately, don't set start time.
       // This way, you can avoid failures caused by small clock differences.
       SharedAccessExpiryTime = DateTime.UtcNow.AddHours(10),
       Permissions = SharedAccessBlobPermissions.Write |
      SharedAccessBlobPermissions.Read
    });
    
    // The public access setting explicitly specifies that 
    // the container is private, so that it can't be accessed anonymously.
    blobPermissions.PublicAccess = BlobContainerPublicAccessType.Off;
    
    // Set the permission policy on the container.
    container.SetPermissions(blobPermissions);
    
    // Get the shared access signature to share with users.
    string sasToken =
       container.GetSharedAccessSignature(new SharedAccessBlobPolicy(), "mypolicy");

### 使用共享访问签名
接收共享访问签名的客户端可从其代码使用此签名以构造类型为 [StorageCredentials](https://msdn.microsoft.com/zh-CN/library/microsoft.windowsazure.storage.auth.storagecredentials.aspx) 的对象。随后，可使用这些凭据构造 [CloudStorageAccount](https://msdn.microsoft.com/zh-CN/library/microsoft.windowsazure.storage.cloudstorageaccount.aspx) 或 [CloudBlobClient](https://msdn.microsoft.com/zh-CN/library/microsoft.windowsazure.storage.blob.cloudblobclient.aspx) 对象以便使用资源，如此示例中所示：

    Uri blobUri = new Uri("https://myaccount.blob.core.chinacloudapi.cn/mycontainer/myblob.txt");
    
    // Create credentials with the SAS token. The SAS token was created in previous example.
    StorageCredentials credentials = new StorageCredentials(sasToken);
    
    // Create a new blob.
    CloudBlockBlob blob = new CloudBlockBlob(blobUri, credentials);
    
    // Upload the blob. 
    // If the blob does not yet exist, it will be created. 
    // If the blob does exist, its existing content will be overwritten.
    using (var fileStream = System.IO.File.OpenRead(@"c:\Test\myblob.txt"))
    {
    blob.UploadFromStream(fileStream);
    }

## 使用存储访问策略
存储访问策略额外增加了一层对服务器端共享访问签名的控制。建立存储访问策略可将共享访问签名分组以及对该策略绑定的签名施加其他限制。可使用存储访问策略更改签名的开始时间、到期时间或权限，还可在颁发签名后将其撤消。

通过存储访问策略，可对已颁发的共享访问签名加强控制。可在存储在所共享的 Blob、容器、队列或表上的存储访问策略中指定签名的生存期和权限，而不能在 URL 上指定这些内容。若要为一个或多个签名更改这些参数，可修改存储访问策略而非重新颁发这些签名。还可通过修改存储访问策略，快速撤消签名。

例如，假定已颁发了与存储访问策略关联的共享访问签名。如果已在存储访问策略中指定了到期时间，则可修改访问策略以延长签名的生存期，而不必重新颁发新的签名。

最佳做法建议对于为其颁发共享访问签名的任何已签名资源指定存储访问策略，因为可使用该存储策略在颁发签名后修改或撤消签名。如果未指定存储策略，则建议限制签名的生存期，以使存储帐户资源的风险降至最低。 

### 将共享访问签名与存储访问策略关联
存储访问策略包括一个在容器、队列或表中独一无二的名称，它最长为 64 个字符。要将一个共享访问签名与一个存储访问策略关联，请在创建该共享访问签名时指定此标识符。在共享访问签名 URI 上， *signedidentifier* 字段指定该存储访问策略的标识符。

容器、队列或表最多可包括 5 个存储访问策略。每个策略均可供任意数量的共享访问签名使用。

>[AZURE.NOTE]建立容器、队列或表的存储访问策略时，它可能最多需要 30 秒才能生效。在此期间，与该存储访问策略关联的共享访问签名将失败，状态代码为 403（禁止访问），直到访问策略成为活动的。

### 指定共享访问策略的访问策略参数
存储访问策略可指定与之关联的签名的以下访问策略参数：

- 开始时间

- 到期时间

- 权限

根据希望如何控制对存储资源的访问，可在存储访问策略中指定所有这些参数，并从共享访问签名的 URL 中忽略这些参数。这样一来，你可以随时修改关联的签名的行为以及撤消签名。或者，还可在存储访问策略中指定一个或多个访问策略参数，并在 URL 上指定其他参数。最后，可以在 URL 上指定所有参数。在这种情况下，你可以使用存储访问策略来撤消签名，但不修改其行为。

共享访问签名和存储访问策略一同必须包括对签名进行身份验证所需的所有字段。如果缺少任何必需的字段，则请求将失败，状态代码为 403（已禁止）。同样地，如果在共享访问签名 URL 中和存储访问策略中都指定了字段，则请求将失败，状态代码为 403（错误请求）。有关构成共享访问签名的字段的详细信息，请参阅"创建和使用共享访问签名"。

### 修改或撤消存储访问策略

若要撤消对使用同一存储访问策略的共享访问签名的访问，请通过用不含策略名称的新列表改写存储策略列表，从存储资源中删除该存储策略。若要更改存储访问策略的访问设置，请用包含具有新访问控制详细信息的同名策略的新列表改写存储策略列表。


<!--HONumber=51-->