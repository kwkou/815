<properties
    pageTitle="如何通过 iOS 使用 Azure Blob 存储 | Azure"
    description="了解如何使用 Azure Blob 存储上载、下载、列出和删除 Blob 内容。示例是用 Objective-C 编写的。"
    services="storage"
    documentationCenter="ios"
    authors="micurd"
    manager="jahogg"/>

<tags
    ms.service="storage"
    ms.date="04/11/2016"
    wacn.date="05/23/2016"/>

# 如何通过 iOS 使用 Blob 存储

[AZURE.INCLUDE [storage-selector-blob-include](../includes/storage-selector-blob-include.md)]

## 概述

本文将演示如何使用 Azure Blob 存储执行常见任务。示例是用 Objective-C 编写的，并使用了 [Azure Storage Client Library for iOS（适用于 iOS 的 Azure 存储客户端库）](https://github.com/Azure/azure-storage-ios)。涉及的任务包括**上载**、**列出**、**下载**和**删除** Blob。有关 Blob 的详细信息，请参阅[后续步骤](#next-steps)部分。你也可以下载[示例应用](https://github.com/Azure/azure-storage-ios/tree/master/BlobSample)以快速了解如何在 iOS 应用程序中使用 Azure 存储空间。

[AZURE.INCLUDE [storage-blob-concepts-include](../includes/storage-blob-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

## 生成框架文件
若要使用 Azure 存储空间 iOS 库，首先需要生成框架文件。

1. 首先，下载或克隆 [azure-storage-ios repo](https://github.com/azure/azure-storage-ios)。

2. 转到 azure-storage-ios -> Lib -> Azure Storage Client Library，并在 Xcode 中打开 `Azure Storage Client Library.xcodeproj`。

3. 在 Xcode 的左上方，将活动方案从“Azure Storage Client Library”更改为“Framework”。

4. 生成项目 (⌘+B)。这将在桌面上创建 `Azure Storage Client Library.framework` 文件。

## 将 Azure 存储空间 iOS 库导入到应用程序中

可以通过执行以下操作将 Azure 存储空间 iOS 库导入到应用程序中：

1. 在 Xcode 中创建一个新项目或打开现有项目。

2. 单击左侧导航栏中的项目，然后单击项目编辑器顶部的“常规”选项卡。

3. 在“链接的框架和库”部分下，单击“添加”按钮 (+)。

4. 单击“添加其他...”。导航到刚创建的 `Azure Storage Client Library.framework` 文件并添加它。

5. 在“链接的框架和库”部分下，再次单击“添加”按钮 (+)。

6. 在已提供的库的列表中，搜索 `libxml2.2.dylib` 并将其添加到你的项目。

##Import 语句
你需要在要调用 Azure 存储空间 API 的文件中添加以下 import 语句。

    // Include the following import statement to use blob APIs.
    #import <Azure Storage Client Library/Azure_Storage_Client_Library.h>

## 配置你的应用程序以访问 Blob 存储

有两种方法可以对要访问存储服务的应用程序进行身份验证：

- 共享密钥：使用共享密钥仅用于测试目的
- 共享访问签名 (SAS)：对于生产应用程序使用 SAS

###共享密钥
共享密钥身份验证意味着你的应用程序将使用帐户名和帐户密钥访问存储服务。为了快速说明如何通过 iOS 使用 Blob 存储，我们将在此入门指南中使用共享密钥身份验证。

> [AZURE.WARNING (仅将“共享密钥”身份验证用于测试目的！) ] 为关联的存储帐户提供完全读/写访问权限的帐户名和帐户密钥将分发给下载你的应用的每个人。这**不**是好的做法，因为你会冒着不受信任的客户端泄露你的密钥的风险。

使用共享密钥身份验证时，你将创建一个连接字符串。连接字符串由以下部分组成：

- **DefaultEndpointsProtocol** - 你可以选择 HTTP 或 HTTPS。但是，强烈建议使用 HTTPS。
- **帐户名** - 存储帐户的名称
- **帐户密钥** - 如果你使用 [Azure 管理门户](https://manage.windowsazure.cn)，请在该门户中导航到你的存储帐户，然后单击“管理访问密钥”。 

下面是该密钥在你的应用程序中的显示方式：

    // Create a storage account object from a connection string.
    AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=account_name;AccountKey=account_key;EndpointSuffix=core.chinacloudapi.cn"];

###共享访问签名 (SAS)
对于 iOS 应用程序，客户端针对 Blob 存储对请求进行身份验证的建议方法是使用共享访问签名 (SAS)。SAS 允许你使用指定的权限集向客户端授予在指定的时间内对资源的访问权限。
作为存储帐户所有者，你需要为 iOS 客户端生成要使用的 SAS。若要生成 SAS，你可能需要编写单独的服务，该服务生成要分发给客户端的 SAS。出于测试目的，你还可以使用 Azure CLI 来生成 SAS。请注意，共享密钥凭据用于生成 SAS，但客户端随后可以通过封装在 SAS URL 中的身份验证信息来使用 SAS。
创建 SAS 时，可以指定 SAS 有效的时间间隔，以及 SAS 授予客户端的权限。例如，对于 blob 容器，SAS 可以授予对容器中的 blob 的读取、写入或删除权限，以及列出容器中的 blob 的列出权限。

以下示例演示如何使用 Azure CLI 来生成 SAS 令牌，该令牌授予对容器 sascontainer 的读取和写入权限，这些权限截止于 2015 年 9 月 5 日上午 12:00 (UTC)。

1. 首先，请参阅[安装 Azure CLI](/documentation/articles/xplat-cli-install/) 以了解如何安装 Azure CLI 并连接到 Azure 订阅。

2. 接下来，在 Azure CLI 中键入以下命令以获得帐户的连接字符串：

		azure storage account connectionstring show youraccountname

3. 使用刚生成的连接字符串创建一个环境变量：

		export AZURE_STORAGE_CONNECTION_STRING=”your connection string”

4. 生成 SAS URL：

		azure storage container sas create --container sascontainer --permissions rw --expiry 2015-09-05T00:00:00

5. SAS URL 应类似于如下内容：

		https://myaccount.blob.core.chinacloudapi.cn/sascontainer/sasblob.txt?sv=2012-02-12&st=2013-04-29T22%3A18%3A26Z&se=2013-04-30T02%3A23%3A26Z&sr=b&sp=rw&sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D

6. 在 iOS 应用程序中，现在可以通过以下方式使用 SAS URL 获取对你的容器的引用：

		// Get a reference to a container in your Storage account
    	AZSCloudBlobContainer *blobContainer = [[AZSCloudBlobContainer alloc] initWithUrl:[NSURL URLWithString:@" your SAS URL"]];

如你所见，使用 SAS 令牌时，不会在 iOS 应用程序中公开你的帐户名和帐户密钥。你可以通过查阅[共享访问签名：了解 SAS 模型](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)了解有关 SAS 的详细信息。

##异步操作
> [AZURE.NOTE] 执行对服务的请求的所有方法都是异步操作。在代码示例中，你会发现这些方法都有完成处理程序。请求完成**后**，将运行完成处理程序内的代码。正在发出请求**时**，将运行完成处理程序后的代码。

## 创建容器
Azure 存储空间中的每个 Blob 都必须驻留在一个容器中。以下示例演示如何在存储帐户中创建一个名为 newcontainer 的容器（如果它尚不存在）。在选择容器的名称时，请注意上面提到的命名规则。

     -(void)createContainer{
        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here;EndpointSuffix=core.chinacloudapi.cn"];

        // Create a blob service client object.
        AZSCloudBlobClient *blobClient = [account getBlobClient];

        // Create a local container object.
        AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"newcontainer"];

        // Create container in your Storage account if the container doesn't already exist
        [blobContainer createContainerIfNotExistsWithCompletionHandler:^(NSError *error, BOOL exists) {
            if (error){
                NSLog(@"Error in creating container.");
            }
        }];
    }

可通过查看 [Azure 管理门户](http://manage.windowsazure.cn)或任意[存储资源管理器](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/03/11/windows-azure-storage-explorers-2014.aspx)并验证 newcontainer 存在于存储帐户的容器列表中来确认此操作有效。

## 设置容器权限
默认情况下，容器的权限配置为**私有**访问权限。但是，容器提供了几个不同的容器访问权限选项：

- **私有**：仅帐户所有者可读取容器和 Blob 数据。

- **Blob**：可以通过匿名请求读取此容器中的 Blob 数据，但容器数据不可用。客户端无法通过匿名请求枚举容器中的 Blob。

- **容器**：可通过匿名请求读取容器和 Blob 数据。客户端可以通过匿名请求枚举容器中的 Blob，但无法枚举存储帐户中的容器。

以下示例演示如何创建一个具有**容器**访问权限的容器，这将允许 Internet 上的所有用户对其进行公共只读访问：

     -(void)createContainerWithPublicAccess{
        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here;EndpointSuffix=core.chinacloudapi.cn"];

        // Create a blob service client object.
        AZSCloudBlobClient *blobClient = [account getBlobClient];

        // Create a local container object.
        AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

        // Create container in your Storage account if the container doesn't already exist
        [blobContainer createContainerIfNotExistsWithAccessType:AZSContainerPublicAccessTypeContainer requestOptions:nil operationContext:nil completionHandler:^(NSError *error, BOOL exists){
            if (error){
                NSLog(@"Error in creating container.");
            }
        }];
    }

## 将 Blob 上载到容器中
如 [Blob 服务概念](#blob-service-concepts)部分中所述，Blob 存储提供了三种不同类型的 blob：块 blob、追加 blob 和页 blob。目前，Azure 存储空间 iOS 库只支持块 blob。大多数情况下，推荐使用块 Blob。

以下示例演示如何从 NSString 上载块 blob。如果此容器中已存在同名的 blob，则将覆盖该 blob 的内容。

     -(void)uploadBlobToContainer{
        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here;EndpointSuffix=core.chinacloudapi.cn"];

        // Create a blob service client object.
        AZSCloudBlobClient *blobClient = [account getBlobClient];

        // Create a local container object.
        AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

        [blobContainer createContainerIfNotExistsWithAccessType:AZSContainerPublicAccessTypeContainer requestOptions:nil operationContext:nil completionHandler:^(NSError *error, BOOL exists)
         {
             if (error){
                 NSLog(@"Error in creating container.");
             }
         	 else{
				 // Create a local blob object
             	 AZSCloudBlockBlob *blockBlob = [blobContainer blockBlobReferenceFromName:@"sampleblob"];

             	 // Upload blob to Storage
             	 [blockBlob uploadFromText:@"This text will be uploaded to Blob Storage." completionHandler:^(NSError *error) {
                 	if (error){
						NSLog(@"Error in creating blob.");
                 	}             
             	 }];
			 }
         }];
     }

你可以通过查看[Azure 管理门户](http://manage.windowsazure.cn)或任何[存储资源管理器](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/03/11/windows-azure-storage-explorers-2014.aspx)并验证容器 *containerpublic* 是否包含该 blob *sampleblob* 来确认此操作是否正常工作。在此示例中，我们使用了公共容器，因此你还可以通过转到 blob URI 来验证此操作是否正常工作：

    https://nameofyourstorageaccount.blob.core.chinacloudapi.cn/containerpublic/sampleblob

除了从 NSString 上载块 blob 外，NSData、NSInputStream 或本地文件也存在类似的方法。

## 列出容器中的 Blob
以下示例演示如何列出容器中的所有 blob。执行此操作时，应注意以下参数：

- **continuationToken** - 继续标记表示列出操作应开始的位置。如果未提供标记，它将从开头列出 blob。可以列出任意数目的 blob，从零到最大集。即使此方法返回零个结果，如果 `results.continuationToken` 不为空，则服务中也可能存在更多 blob 未列出。
- **prefix** - 你可以指定要用于 blob 列出的前缀。将仅列出以该前缀开头的 blob。
- **useFlatBlobListing** - 如[命名和引用容器和 blob](#naming-and-referencing-containers-and-blobs) 部分中所述，虽然 Blob 服务是平面存储方案，但你可以通过命名具有路径信息的 blob 来创建虚拟层次结构。但是，目前不支持非平面列表；该功能即将推出。目前，此值应为 `YES`
- **blobListingDetails** - 你可以指定在列出 blob 时要包含哪些项
	- `AZSBlobListingDetailsNone`：仅列出已提交的 blob，并且不返回 blob 元数据。
	- `AZSBlobListingDetailsSnapshots`：列出已提交的 blob 和 blob 快照。
	- `AZSBlobListingDetailsMetadata`：检索列表中返回的每个 blob 的 blob 元数据。
	- `AZSBlobListingDetailsUncommittedBlobs`：列出已提交和未提交的 blob。
	- `AZSBlobListingDetailsCopy`：在列表中包含复制属性。
	- `AZSBlobListingDetailsAll`：列出所有可用的已提交 blob、未提交 blob 和快照，并返回这些 blob 的所有元数据和复制状态。
- **maxResults** - 此操作可返回的结果的最大数目。使用 -1 以不设置限制。
- **completionHandler** - 要使用列表操作的结果执行的代码块。

在此示例中，帮助器方法用于在每次返回继续标记时递归调用列出 blob 方法。

    -(void)listBlobsInContainer{
      // Create a storage account object from a connection string.
      AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here;EndpointSuffix=core.chinacloudapi.cn"];

      // Create a blob service client object.
      AZSCloudBlobClient *blobClient = [account getBlobClient];

      // Create a local container object.
      AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

      //List all blobs in container
      [self listBlobsInContainerHelper:blobContainer continuationToken:nil prefix:nil blobListingDetails:AZSBlobListingDetailsAll maxResults:-1 completionHandler:^(NSError *error) {
         if (error != nil){
             NSLog(@"Error in creating container.");
         }
      }];
    }

    //List blobs helper method
    -(void)listBlobsInContainerHelper:(AZSCloudBlobContainer *)container continuationToken:(AZSContinuationToken *)continuationToken prefix:(NSString *)prefix blobListingDetails:(AZSBlobListingDetails)blobListingDetails maxResults:(NSUInteger)maxResults completionHandler:(void (^)(NSError *))completionHandler
    {
      [container listBlobsSegmentedWithContinuationToken:continuationToken prefix:prefix useFlatBlobListing:YES blobListingDetails:blobListingDetails maxResults:maxResults completionHandler:^(NSError *error, AZSBlobResultSegment *results) {
         if (error)
         {
             completionHandler(error);
         }
         else
         {
             for (int i = 0; i < results.blobs.count; i++) {
                 NSLog(@"%@",[(AZSCloudBlockBlob *)results.blobs[i] blobName]);
             }
             if (results.continuationToken)
             {
                 [self listBlobsInContainerHelper:container continuationToken:results.continuationToken prefix:prefix blobListingDetails:blobListingDetails maxResults:maxResults completionHandler:completionHandler];
             }
             else
             {
                 completionHandler(nil);
             }
         }
      }];
    }

## 下载 Blob

以下示例演示如何将 blob 下载到 NSString 对象中。

     -(void)downloadBlobToString{
        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here;EndpointSuffix=core.chinacloudapi.cn"];

        // Create a blob service client object.
        AZSCloudBlobClient *blobClient = [account getBlobClient];

        // Create a local container object.
        AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

        // Create a local blob object
        AZSCloudBlockBlob *blockBlob = [blobContainer blockBlobReferenceFromName:@"sampleblob"];

        // Download blob    
        [blockBlob downloadToTextWithCompletionHandler:^(NSError *error, NSString *text) {
            if (error) {
                NSLog(@"Error in downloading blob");
            }
			else{
            	NSLog(@"%@",text);
			}
        }];
    }

## 删除 Blob

以下示例说明如何删除 blob。

     -(void)deleteBlob{
        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here;EndpointSuffix=core.chinacloudapi.cn"];

        // Create a blob service client object.
        AZSCloudBlobClient *blobClient = [account getBlobClient];

        // Create a local container object.
        AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

        // Create a local blob object
        AZSCloudBlockBlob *blockBlob = [blobContainer blockBlobReferenceFromName:@"sampleblob1"];

        // Delete blob
        [blockBlob deleteWithCompletionHandler:^(NSError *error) {
          if (error) {
            NSLog(@"Error in deleting blob.");
          }
        }];
     }

## 删除 Blob 容器

以下示例说明如何删除容器。

     -(void)deleteContainer{
        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here;EndpointSuffix=core.chinacloudapi.cn"];

        // Create a blob service client object.
        AZSCloudBlobClient *blobClient = [account getBlobClient];

        // Create a local container object.
        AZSCloudBlobContainer *blobContainer = [blobClient containerReferenceFromName:@"containerpublic"];

        // Delete container
        [blobContainer deleteContainerIfExistsWithCompletionHandler:^(NSError *error, BOOL success) {
            if(error){
                NSLog(@"Error in deleting container");
            }
        }];
    }

## 后续步骤

现在，你已了解有关 Blob 存储的基础知识，可单击下面的链接来了解更复杂的存储任务。

- [Azure Storage Client Library for iOS（适用于 iOS 的 Azure 存储客户端库）](https://github.com/azure/azure-storage-ios)
- [Azure 存储空间服务 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dd179355.aspx)
- [使用 AzCopy 命令行实用程序传输数据](/documentation/articles/storage-use-azcopy/)
- [Azure 存储团队博客](http://blogs.msdn.com/b/windowsazurestorage)

如果你对此库有任何疑问，请随意将问题发布到我们的 [MSDN Azure 论坛](https://social.msdn.microsoft.com/forums/azure/zh-cn/home?forum=windowsazuredata)或[堆栈溢出](http://stackoverflow.com/questions/tagged/windows-azure-storage+or+windows-azure-storage+or+azure-storage-blobs+or+azure-storage-tables+or+azure-table-storage+or+windows-azure-queues+or+azure-storage-queues+or+azure-storage-emulator+or+azure-storage-files)。如果你有 Azure 存储空间的功能建议，请将建议发布到 [Azure 存储空间反馈](/product-feedback)。


<!---HONumber=Mooncake_0516_2016-->