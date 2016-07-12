<properties 
	pageTitle="共享访问签名：了解 SAS 模型 | Azure" 
	description="了解使用共享访问签名 (SAS) 委派对 Azure 存储空间资源（包括 Blob、队列、表和文件）的访问权限。使用共享访问签名，你可以保护你的存储帐户密钥，同时授权其他用户访问你的帐户中的资源。你可以控制你授予的权限以及 SAS 的有效期。如果你还建立存储访问策略，则在你担心你的帐户安全受到威胁时可以撤消该 SAS。" 
	services="storage" 
	documentationCenter="" 
	authors="tamram" 
	manager="adinah" 
	editor=""/>

<tags 
	ms.service="storage" 
	ms.date="02/14/2016"
	wacn.date="04/11/2016"/>



# 共享访问签名，第 1 部分：了解 SAS 模型

## 概述

使用共享访问签名 (SAS) 是将对你的存储帐户中对象的受限访问权限授予其他客户端且不必公开帐户密钥的一种强大方法。在有关共享访问签名的教程的第 1 部分中，我们将概要介绍 SAS 模型并复习 SAS 最佳实践。本教程的[第 2 部分](/documentation/articles/storage-dotnet-shared-access-signature-part-2/)将逐步引导你完成使用 Blob 服务创建共享访问签名的过程。

## 什么是共享访问签名？

共享访问签名对存储帐户中的资源提供委托访问。这意味着你可以授权客户端在指定时间段内，以一组指定权限有限地访问你的存储帐户中的对象，而不必共享你的帐户访问密钥。SAS 是在其查询参数中包含对存储资源进行验证了身份的访问所需的所有信息的 URI。若要使用 SAS 访问存储资源，客户端只需将 SAS 传入到相应的构造函数或方法。

## 何时应使用共享访问签名？

需要将存储帐户中资源的访问权限提供给不能使用帐户密钥进行信任的客户端时，可以使用 SAS。你的存储帐户密钥包括主密钥和辅助密钥，这两种密钥都授予对你的帐户以及其中所有资源的管理访问权限。公开这两种帐户密钥的任何一种都会向可能的恶意或负面使用开放你的帐户。共享访问签名提供一种安全的方法，允许其他客户端根据你授予的权限读取、写入和删除你的存储帐户中的数据，而无需帐户密钥。

SAS 通常适用于用户需要在你的存储帐户中读取和写入其数据的服务情形。在存储帐户存储用户数据的情形中，有两种典型的设计模式：


1\.客户端通过执行身份验证的前端代理服务上载和下载数据。此前端代理服务的优势在于允许验证业务规则，但对于大量数据或大量事务，创建可扩展以匹配需求的服务可能成本高昂或十分困难。

![sas-storage-fe-proxy-service][sas-storage-fe-proxy-service]

2\.轻型服务按需对客户端进行身份验证，然后生成 SAS。在客户端接收 SAS 后，它们可以直接使用 SAS 定义的权限并且针对 SAS 允许的间隔访问存储帐户资源。SAS 减少了通过前端代理服务路由所有数据的需要。

![sas-storage-provider-service][sas-storage-provider-service]

许多实际服务可能会根据涉及的情形混合使用这两种方法，也就是说，通过前端代理对某些数据进行处理和验证，同时使用 SAS 直接保存和/或读取其他数据。

此外，在某些情况下，你需要使用 SAS 在复制操作中对源对象进行身份验证：

- 当你将一个 Blob 复制到驻留在不同存储帐户中的另一个 Blob 时，必须使用 SAS 对源 Blob 进行身份验证。使用版本 2015-04-05，你还可以选择使用 SAS 对目标 blob 进行身份验证。
- 当你将一个文件复制到驻留在不同存储帐户中的另一个文件时，必须使用 SAS 对源文件进行身份验证。使用版本 2015-04-05，你还可以选择使用 SAS 对目标文件进行身份验证。
- 当你将一个 Blob 复制到一个文件，或将一个文件复制到一个 Blob 时，必须使用 SAS 对源对象进行身份验证，即使源对象和目标对象驻留在同一存储帐户中。

## 共享访问签名的类型

Azure 存储空间的版本 2015-04-05 引入了一种新的共享访问签名类型，即帐户 SAS。现在可以创建两种类型的共享访问签名中的任一种：

- **帐户 SAS。** 帐户 SAS 可委派对一个或多个存储服务中的资源的访问权限。通过服务 SAS 提供的所有操作也可以通过帐户 SAS 提供。此外，使用帐户 SAS，你还可以委派对适用于给定服务的操作（例如，**获取/设置服务属性**和**获取服务统计信息**）的访问权限。你还可以委派对 blob 容器、表、队列和文件共享执行读取、写入和删除操作的访问权限，而这是服务 SAS 所不允许的。有关构造帐户 SAS 令牌的深入信息，请参阅[构造帐户 SAS](https://msdn.microsoft.com/zh-cn/library/mt584140.aspx)。

- **服务 SAS。** 服务 SAS 只能委派对以下一个存储服务中的资源的访问权限：Blob、队列、表或文件服务。有关构造服务 SAS 令牌的深入信息，请参阅[构造服务 SAS](https://msdn.microsoft.com/zh-cn/library/dn140255.aspx) 和[服务 SAS 示例](https://msdn.microsoft.com/zh-cn/library/dn140256.aspx)。

## 共享访问签名的工作方式

共享访问签名是一种 URI，它指向一个或多个存储资源并且包括包含一组特殊查询参数的令牌。该令牌指示客户端可以如何访问资源。签名是其中一个查询参数，它是由 SAS 参数构造的并且使用帐户密钥进行签名。Azure 存储空间使用该签名对 SAS 进行身份验证。

帐户 SAS 令牌和服务 SAS 令牌包括一些公用参数，但所采用的参数也有几个不同。

### 帐户 SAS 令牌和服务 SAS 令牌共有的参数

- **Api 版本** 一个可选参数，它指定要用于执行请求的存储服务版本。 
- **服务版本** 一个必需参数，它指定要用于对请求进行身份验证的存储服务版本。
- **开始时间。** 这是 SAS 生效的时间。共享访问签名的开始时间是可选的；如果省略，SAS 将立即生效。 
- **到期时间。** 这是之后 SAS 将不再有效的时间。最佳实践建议你或者为 SAS 指定到期时间，或者将其与某一存储访问策略相关联（有关更多信息，请参见下文）。
- **权限。** 对 SAS 指定的权限指示客户端可使用 SAS 对存储资源执行哪些操作。帐户 SAS 和服务 SAS 提供的权限不同。
- **IP。** 一个可选参数，它指定 Azure 外部要从中接受请求的一个 IP 地址或 IP 地址范围（有关 Express Route，请参阅[路由会话配置状态](/documentation/articles/expressroute-workflows/#routing-session-configuration-state)部分）。 
- **协议。** 一个可选参数，它指定请求允许的协议。可能的值包括“HTTPS 和 HTTP”(https,http)（它是默认值）或者“仅限 HTTPS”(https)。请注意，“仅限 HTTP”是不允许的值。
- **签名。** 签名由指定为部分令牌的其他参数构造，然后进行加密。它用于对 SAS 进行身份验证。

### 帐户 SAS 令牌的参数

- **一个服务或多个服务。** 帐户 SAS 可委派对一个或多个存储服务的访问权限。例如，可以创建一个帐户 SAS 以委派对 Blob 和文件服务的访问权限。也可以创建一个 SAS 以委派对所有四种服务（Blob、队列、表和文件）的访问权限。
- **存储资源类型。** 帐户 SAS 适用于一个或多个类别的存储资源，而不是特定资源。你可以创建帐户 SAS 以委派对以下项的访问权限：
	- 服务级别 API，将针对存储帐户资源调用它。示例包括**获取/设置服务属性**、**获取服务统计信息**和**列出容器/队列/表/共享**。
	- 容器级别 API，将针对每个服务的容器对象调用它：blob 容器、队列、表和文件共享。示例包括**创建/删除容器**、**创建/删除队列**、**创建/删除表**、**创建/删除共享**和**列出 Blob/文件和目录**。
	- 对象级别 API，将针对 blob、队列消息、表实体和文件调用它。例如，**放置 Blob**、**查询实体**、**获取消息**和**创建文件**。

### 服务 SAS 令牌的参数

- **存储资源。** 你可以使用服务 SAS 为其委派访问权限的存储资源包括：
	- 容器和 Blob
	- 文件共享和文件
	- 队列
	- 表和表实体范围。

## SAS URI 的示例

下面是服务 SAS URI 的一个示例，它提供对某一 Blob 的读写权限。该表分解了 URI 的每个部分，以便理解它是如何影响 SAS 的：

	https://myaccount.blob.core.chinacloudapi.cn/sascontainer/sasblob.txt?sv=2015-04-05&st=2015-04-29T22%3A18%3A26Z&se=2015-04-30T02%3A23%3A26Z&sr=b&sp=rw&sip=168.1.5.60-168.1.5.70&spr=https&sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D

Name|SAS 部分|说明
---|---|---
Blob URI|https://myaccount.blob.core.chinacloudapi.cn/sascontainer/sasblob.txt | Blob 的地址。请注意，强烈建议使用 HTTPS。
存储服务版本|sv=2015-04-05|对于存储服务版本 2012-02-12 和更高版本，此参数指示要使用的版本。
开始时间|st=2015-04-29T22%3A18%3A26Z|以 ISO 8601 格式指定。如果你想要 SAS 立即生效，则省略开始时间。
到期时间|se=2015-04-30T02%3A23%3A26Z|以 ISO 8601 格式指定。
资源|sr=b|资源是 Blob。
权限|sp=rw|SAS 授予的权限包括读取 (r) 和写入 (w)。
IP 范围|sip=168.1.5.60-168.1.5.70|将从中接受请求的 IP 地址范围。
协议|spr=https|仅允许使用 HTTPS 的请求。
签名|sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D|用于对 Blob 的访问权限进行身份验证。该签名是利用 SHA256 算法通过“字符串到签名”和密钥进行计算，然后使用 Base64 编码进行编码的 HMAC。

下面是在令牌中使用相同的公用参数的帐户 SAS 的一个示例。由于这些参数已在前面说明，因此不在此处对其进行说明。下表中仅说明了特定于帐户 SAS 的参数。

	https://myaccount.blob.core.chinacloudapi.cn/?restype=service&comp=properties&sv=2015-04-05&ss=bf&srt=s&st=2015-04-29T22%3A18%3A26Z&se=2015-04-30T02%3A23%3A26Z&sr=b&sp=rw&sip=168.1.5.60-168.1.5.70&spr=https&sig=F%6GRVAZ5Cdj2Pw4tgU7IlSTkWgn7bUkkAg8P6HESXwmf%4B

Name|SAS 部分|说明
---|---|---
资源 URI|https://myaccount.blob.core.chinacloudapi.cn/?restype=service&comp=properties|The Blob 服务终结点，包含用于获取服务属性（使用 GET 调用时）或设置服务属性（使用 SET 调用时）的参数。
服务|ss=bf|该 SAS 适用于 Blob 和文件服务
资源类型|srt=s|该 SAS 适用于服务级别操作。
权限|sp=rw|这些权限向读取和写入操作授予访问权限。  

鉴于权限仅限于服务级别，使用此 SAS 的可访问操作包括 **获取 Blob 服务属性**（读取）和**设置 Blob 服务属性**（写入）。但是，使用其他资源 URI，同一个 SAS 令牌还可用于委派对**获取 Blob 服务统计信息**（读取）的访问权限。

## 使用存储访问策略控制 SAS ##

共享访问签名可以采取以下两种形式的一种：

- **临时 SAS：**在你创建一个临时 SAS 时，针对该 SAS 的开始时间、到期时间和权限全都在 SAS URI 上指定（在省略开始时间的情况下，也可以是暗示的）。这种类型的 SAS 可以创建为帐户 SAS 或服务 SAS。 

- **具有存储访问策略的 SAS：**存储访问策略是对资源容器（Blob 容器、表、队列或文件共享）定义的，可用于管理针对一个或多个共享访问签名的约束。在你将某一 SAS 与一个存储访问策略相关联时，该 SAS 将继承对该存储访问策略定义的约束：开始时间、到期时间和权限。

>[AZURE.NOTE] 目前，帐户 SAS 必须是一个临时 SAS。帐户 SAS 尚不支持存储访问策略。

这两种形式之间的差异对于一个关键情形而言十分重要：吊销。一个 SAS 就是一个 URL，因此，获取该 SAS 的任何人都可以使用它，而与是谁请求它以便开始的无关。如果某一 SAS 是公开发布的，则世界上的任何人都可以使用它。在发生以下四种情况之一前分发的 SAS 是有效的：

1.	达到了对该 SAS 指定的到期时间。
2.	达到了对该 SAS 引用的存储访问策略指定的到期时间（如果引用某一存储访问策略并且该存储访问策略指定一个到期时间）。这可能是因为经过了该间隔而发生，或者是因为你修改了该存储访问策略以使到期时间已经是过去时间而发生（这是用于吊销该 SAS 的一种方法）。
3.	删除了该 SAS 引用的存储访问策略，这是用于吊销 SAS 的另一种方法。请注意，如果你使用完全相同的名称重新创建该存储访问策略，则根据与该存储访问策略相关联的权限，所有现有 SAS 标记都将再次有效（假定尚未经过该 SAS 的到期时间）。如果你想要吊销该 SAS，请确保使用不同时间（如果你使用将来的到期时间重新创建该访问策略）。
4.	将重新生成用于创建 SAS 的帐户密钥。请注意，这样做将导致使用该帐户密钥的所有应用程序组件身份验证失败，直到这些组件更新为使用其他有效帐户密钥或者重新生成的新帐户密钥。

>[AZURE.IMPORTANT] 共享访问签名 URI 与用于创建签名的帐户密钥和关联的存储访问策略（如果有）相关联。如果未指定存储访问策略，则吊销共享访问签名的唯一方法是更改帐户密钥。

## 示例：创建和使用共享访问签名

下面是两种类型的共享访问签名（帐户 SAS 和服务 SAS）的一些示例。

若要运行这些示例，需下载和引用以下包：

- [适用于 .NET 的 Azure 存储客户端库](http://www.nuget.org/packages/WindowsAzure.Storage) 6.x 或更高版本（以便使用帐户 SAS）。
- [Azure 配置管理器](http://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager) 

### 示例：帐户 SAS

以下代码示例将创建一个帐户 SAS，该 SAS 对 Blob 和文件服务是有效的，并授予客户端读取、写入和列表权限，使其能够访问服务级别 API。帐户 SAS 将协议限制为 HTTPS，因此请求必须使用 HTTPS 发出。

    static string GetAccountSASToken()
    {
        // To create the account SAS, you need to use your shared key credentials. Modify for your account.
        CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
            Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

        // Create a new access policy for the account.
        SharedAccessAccountPolicy policy = new SharedAccessAccountPolicy()
            {
                Permissions = SharedAccessAccountPermissions.Read | SharedAccessAccountPermissions.Write | SharedAccessAccountPermissions.List,
                Services = SharedAccessAccountServices.Blob | SharedAccessAccountServices.File,
                ResourceTypes = SharedAccessAccountResourceTypes.Service,
                SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
                Protocols = SharedAccessProtocol.HttpsOnly
            };

        // Return the SAS token.
        return storageAccount.GetSharedAccessSignature(policy);
    }

若要使用帐户 SAS 访问 Blob 服务的服务级别 API，请使用 SAS 和存储帐户的 Blob 存储终结点构造 Blob 客户端对象。

    static void UseAccountSAS(string sasToken)
    {
        // In this case, we have access to the shared key credentials, so we'll use them
        // to get the Blob service endpoint.
        CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
            Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));
        CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

        // Create new storage credentials using the SAS token.
        StorageCredentials accountSAS = new StorageCredentials(sasToken);
        // Use these credentials and the Blob storage endpoint to create a new Blob service client.
        CloudStorageAccount accountWithSAS = new CloudStorageAccount(accountSAS, blobClient.StorageUri, null, null, null);
        CloudBlobClient blobClientWithSAS = accountWithSAS.CreateCloudBlobClient();

        // Now set the service properties for the Blob client created with the SAS.
        blobClientWithSAS.SetServiceProperties(new ServiceProperties()
        {
            HourMetrics = new MetricsProperties()
            {
                MetricsLevel = MetricsLevel.ServiceAndApi,
                RetentionDays = 7,
                Version = "1.0"
            },
            MinuteMetrics = new MetricsProperties()
            {
                MetricsLevel = MetricsLevel.ServiceAndApi,
                RetentionDays = 7,
                Version = "1.0"
            },
            Logging = new LoggingProperties()
            {
                LoggingOperations = LoggingOperations.All,
                RetentionDays = 14,
                Version = "1.0"
            }
        });

        // The permissions granted by the account SAS also permit you to retrieve service properties.
        ServiceProperties serviceProperties = blobClientWithSAS.GetServiceProperties();
        Console.WriteLine(serviceProperties.HourMetrics.MetricsLevel);
        Console.WriteLine(serviceProperties.HourMetrics.RetentionDays);
        Console.WriteLine(serviceProperties.HourMetrics.Version);
    }

### 示例：具有存储访问策略的服务 SAS

以下代码示例在容器上创建存储访问策略，然后为该容器生成服务 SAS。然后可以将此 SAS 提供给客户端，使其获得对容器的读/写权限。更改代码以使用你自己的帐户名：

    // Parse the connection string for the storage account.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));
    
    // Create the storage account with the connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(storageConnectionString);
       
    // Create the blob client object.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
    
    // Get a reference to the container for which shared access signature will be created.
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");
    container.CreateIfNotExists();
    
    // Get the current permissions for the blob container.
    BlobContainerPermissions blobPermissions = container.GetPermissions();

    // Clear the container's shared access policies to avoid naming conflicts.
    blobPermissions.SharedAccessPolicies.Clear();
    
    // The new shared access policy provides read/write access to the container for 24 hours.
    blobPermissions.SharedAccessPolicies.Add("mypolicy", new SharedAccessBlobPolicy()
    {
       // To ensure SAS is valid immediately, don’t set the start time.
       // This way, you can avoid failures caused by small clock differences.
       SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
       Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.Create | SharedAccessBlobPermissions.Add
    });
    
    // The public access setting explicitly specifies that 
    // the container is private, so that it can't be accessed anonymously.
    blobPermissions.PublicAccess = BlobContainerPublicAccessType.Off;
    
    // Set the new stored access policy on the container.
    container.SetPermissions(blobPermissions);
    
    // Get the shared access signature token to share with users.
    string sasToken =
       container.GetSharedAccessSignature(new SharedAccessBlobPolicy(), "mypolicy");

拥有服务 SAS 的客户端可以使用它从其代码对请求进行身份验证，以便读取或写入到容器中的 blob。例如，下面的代码使用 SAS 令牌在容器中创建新的块 blob。更改代码以使用你自己的帐户名：

    Uri blobUri = new Uri("https://myaccount.blob.core.chinacloudapi.cn/mycontainer/myblob.txt");
    
    // Create credentials with the SAS token. The SAS token was created in previous example.
    StorageCredentials credentials = new StorageCredentials(sasToken);
    
    // Create a new blob.
    CloudBlockBlob blob = new CloudBlockBlob(blobUri, credentials);
    
    // Upload the blob. 
    // If the blob does not yet exist, it will be created. 
    // If the blob does exist, its existing content will be overwritten.
    using (var fileStream = System.IO.File.OpenRead(@"c:\Temp\myblob.txt"))
    {
    	blob.UploadFromStream(fileStream);
    }


## 使用共享访问签名的最佳实践

当你在应用程序中使用共享访问签名时，需要知道以下两个可能的风险：

- 如果 SAS 泄露，则获取它的任何人都可以使用它，这可能会损害你的存储帐户。
- 如果提供给客户端应用程序的 SAS 到期并且应用程序无法从你的服务检索新 SAS，则可能会影响该应用程序的功能。  

下面这些针对使用共享访问签名的建议将帮助化解这些风险：

1. **始终使用 HTTPS** 创建 SAS 或分发 SAS。如果某一 SAS 通过 HTTP 传递并且被截取，则执行中间人攻击的攻击者将能够读取 SAS、然后使用它，就像目标用户可能具有一样，这可能会暴露敏感数据或者使恶意用户能够损坏数据。
2. **尽可能参照存储访问策略。** 存储访问策略使你可以选择撤消权限而不必重新生成存储帐户密钥。将针对 SAS 的到期时间设置为非常长的时间（或者无限远），并且确保定期对其进行更新以便将到期时间移到将来的更远时间。
3. **对临时 SAS 使用近期的到期时间。** 这样，即使某一 SAS 不知不觉地泄露，它也只是短期有效。如果你无法参照某一存储访问策略，该行为尤其重要。该行为还通过限制可用于上载到它的时间，帮助限制可以写入 Blob 的数据量。
4. **如果需要，让客户端自动续订 SAS。** 客户端应在预期到期时间之前很久就续订 SAS，这样，即使提供 SAS 的服务不可用，客户端也有时间重试。如果你的 SAS 旨在用于少量即时的、短期操作，这些操作应该在给定的到期时间内完成，则上述做法可能是不必要的，因为不应续订 SAS。但是，如果你的客户端定期通过 SAS 发出请求，则有效期可能就会起作用。需要考虑的主要方面就是在以下两者间进行权衡：要短期的 SAS 的需要（如上所述）以及确保客户端尽早续订以免在成功续订前因 SAS 到期而中断。
5. **要注意 SAS 开始时间。** 如果你将 SAS 的开始时间设置为“现在”，则由于时钟偏移（根据不同计算机，当前时间中的差异），在前几分钟将会暂时观察到失败。通常，将开始时间设置为至少 15 分钟前，或者根本不设置，这会使它在所有情况下都立即生效。同样原则也适用于到期时间 – 请记住，对于任何请求，在任一方向你可能会观察到最多 15 分钟的时钟偏移。请注意，对于使用 2012-02-12 之前的 REST 版本的客户端，对于未参照某一存储访问策略的 SAS 的最大持续时间是 1 小时，指定超过 1 小时的更长期间的任何策略都将失败。
6.	**对要访问的资源要具体。** 一个典型的安全性最佳实践是向用户提供所需最小权限。如果某一用户仅需要对单个实体的读取访问权限，则向该用户授予对该单个实体的读取访问权限，而不要授予针对所有实体的读取/写入/删除访问权限。这也有助于化解泄露 SAS 的威胁，因为在攻击者手中掌握的 SAS 的权限较为有限。
7.	**了解对任何使用都将向你的帐户收费，包括使用 SAS 所做的工作。** 如果向你提供了针对某一 Blob 的写访问权限，用户可以选择上载 200GB Blob。如果你还向用户提供了对 Blob 的读访问权限，他们可能会选择下载 Blob 10 次，对你产生 2TB 的传出费用。此外，提供受限权限，帮助降低恶意用户的潜在威胁。使用短期 SAS 以便减少这一威胁（但要注意结束时间上的时钟偏移）。
8.	**验证使用 SAS 写入的数据。** 在某一客户端应用程序将数据写入你的存储帐户时，请记住对于这些数据可能存在问题。如果你的应用程序要求在数据可供使用前对数据进行验证或授权，你应该在写入数据后、但在你的应用程序使用这些数据前执行此验证。这一实践还有助于防止损坏的数据或恶意数据写入你的帐户，这些数据可能是正常要求 SAS 的用户写入的，也可能是利用泄露的 SAS 的用户写入的。
9. **不要总是使用 SAS。** 有时候，与针对你的存储帐户的特定操作相关联的风险要超过 SAS 所带来的好处。对于此类操作，应创建一个中间层服务，该服务在执行业务规则验证、身份验证和审核后写入你的存储帐户。此外，有时候以其他方式管理访问会更简单。例如，如果你想要使某一容器中的所有 Blob 都可以公开读取，则可以使该容器成为公共的，而不是为每个客户端都提供 SAS 以便进行访问。
10.	**使用存储分析监视你的应用程序。** 你可以使用日志记录和度量来观察由于你的 SAS 提供程序服务中的中断或者由于某一存储访问策略的无意中删除而导致的身份验证失败中的任何峰值情形。有关其他信息，请参阅 [Azure 存储空间团队博客](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/08/03/windows-azure-storage-logging-using-logs-to-track-storage-requests.aspx)。

## 结束语 ##

共享访问签名用于将存储帐户的受限权限提供给不应具有帐户密钥的客户端。因此，它们是安全模型的重要环节，适合使用 Azure 存储空间的任何应用程序。如果你按照本文中介绍的最佳实践执行，则可以使用 SAS 更灵活地访问你的存储帐户中的资源，且不会影响应用程序的安全性。

## 后续步骤 ##

- [共享访问签名，第 2 部分：创建 SAS 并将 SAS 用于 Blob 存储](/documentation/articles/storage-dotnet-shared-access-signature-part-2/)
- [在 Windows 上开始使用 Azure 文件存储](/documentation/articles/storage-dotnet-how-to-use-files/)
- [管理对容器和 blob 的匿名读取访问](/documentation/articles/storage-manage-access-to-resources/)
- [使用共享的访问签名委托访问](http://msdn.microsoft.com/zh-cn/library/azure/ee395415.aspx)
- [介绍表和队列 SAS](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas.aspx)
[sas-storage-fe-proxy-service]: ./media/storage-dotnet-shared-access-signature-part-1/sas-storage-fe-proxy-service.png
[sas-storage-provider-service]: ./media/storage-dotnet-shared-access-signature-part-1/sas-storage-provider-service.png


 

<!---HONumber=Mooncake_0405_2016-->