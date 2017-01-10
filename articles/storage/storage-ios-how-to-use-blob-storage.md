<properties
    pageTitle="如何通过 iOS 使用 Azure Blob 存储 | Azure"
    description="使用 Azure Blob 存储（对象存储）将非结构化数据存储在云中。"
    services="storage"
    documentationcenter="ios"
    author="micurd"
    manager="jahogg"
    editor="tysonn" />
<tags
    ms.assetid="df188021-86fc-4d31-a810-1b0e7bcd814b"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="objective-c"
    ms.topic="article"
    ms.date="11/21/2016"
    wacn.date="01/06/2017"
    ms.author="micurd" />  


# 如何通过 iOS 使用 Blob 存储
[AZURE.INCLUDE [storage-selector-blob-include](../../includes/storage-selector-blob-include.md)]

## 概述
本文将演示如何使用 Azure Blob 存储执行常见任务。示例是用 Objective-C 编写的，并使用了[用于 iOS 的 Azure 存储客户端库](https://github.com/Azure/azure-storage-ios)。涉及的任务包括上传、列出、下载和删除 Blob。有关 Blob 的详细信息，请参阅[后续步骤](#next-steps)部分。也可下载[示例应用](https://github.com/Azure/azure-storage-ios/tree/master/BlobSample)，快速了解如何在 iOS 应用程序中使用 Azure 存储。

[AZURE.INCLUDE [storage-blob-concepts-include](../../includes/storage-blob-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

## 将 Azure 存储 iOS 库导入到应用程序
可使用 [Azure 存储 CocoaPod](https://cocoapods.org/pods/AZSClient) 或导入 **Framework** 文件，将 Azure 存储 iOS 库导入到应用程序。

## CocoaPod
1. 如果尚未在计算机上[安装 CocoaPods](https://guides.cocoapods.org/using/getting-started.html#toc_3)，请打开终端窗口并运行以下命令，以实现此操作
   
    sudo gem install cocoapods

2. 接下来，在项目目录（包含 .xcodeproj 文件的目录）中创建名为 _Podfile_ 的文件（无扩展名）。将以下命令添加到 _Podfile_ 并保存。
   
    pod 'AZSClient'

3. 在终端窗口中，导航到项目目录并运行以下命令
   
    pod install

4. 如果已在 Xcode 中打开 .xcodeproj，请将其关闭。在项目目录中打开新建的项目文件（扩展名为 .xcworkspace）。从现在开始将使用此文件。

## 框架
若要使用 Azure 存储 iOS 库，首先需要生成框架文件。

1. 首先，下载或克隆 [azure-storage-ios repo](https://github.com/azure/azure-storage-ios)。
2. 转到“azure-storage-ios”->“Lib”->“Azure 存储客户端库”，并在 Xcode 中打开 AZSClient.xcodeproj。。
3. 在 Xcode 的左上方，将活动方案从“Azure 存储客户端库”更改为“Framework”。
4. 生成项目 (⌘+B)。此操作在桌面上创建了一个 AZSClient.framework 文件。

可以通过执行以操作将框架文件导入到应用程序：

1. 在 Xcode 中创建一个新项目或打开现有项目。
2. 单击左侧导航栏中的项目，然后单击项目编辑器顶部的“常规”选项卡。
3. 在“链接的框架和库”部分下，单击“添加”按钮 (+)。
4. 单击“添加其他...”。导航到刚创建的 `AZSClient.framework` 文件并添加它。
5. 在“链接的框架和库”部分下，再次单击“添加”按钮 (+)。
6. 在已提供的库的列表中，搜索 `libxml2.2.dylib` 并将其添加到你的项目。
7. 单击位于项目编辑器顶部的“生成设置”选项卡。
8. 在“搜索路径”部分中，双击“框架搜索路径”，并将该路径添加到 `AZSClient.framework` 文件。

## Import 语句
需要在要调用 Azure 存储 API 的文件中添加以下导入语句。

    // Include the following import statement to use blob APIs.
    #import <AZSClient/AZSClient.h>

[AZURE.INCLUDE [存储移动身份验证指南](../../includes/storage-mobile-authentication-guidance.md)]

## 异步操作
> [AZURE.NOTE] 执行对服务的请求的所有方法都是异步操作。在代码示例中，可发现这些方法都拥有完成处理程序。请求完成**后**，将运行完成处理程序内的代码。正在发出请求**时**，将运行完成处理程序后的代码。

## 创建容器
Azure 存储中的每个 Blob 都必须驻留在一个容器中。以下示例演示如何在存储帐户中创建一个名为 *newcontainer* 的容器（如果它尚不存在）。在选择容器的名称时，请注意上面提到的命名规则。

    -(void)createContainer{
      NSError *accountCreationError;

      // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here;EndpointSuffix=core.chinacloudapi.cn" error:&accountCreationError];

      if(accountCreationError){
         NSLog(@"Error in creating account.");
      }

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

可通过查看 [Azure 存储资源管理器](http://storageexplorer.com)并验证 *newcontainer* 存在于存储帐户的容器列表中来确认此操作有效。

## 设置容器权限
默认情况下，容器的权限配置为**私有**访问权限。但是，容器提供了几个不同的容器访问权限选项：

* **私有**：仅帐户所有者可读取容器和 Blob 数据。
* **Blob**：可以通过匿名请求读取此容器中的 Blob 数据，但容器数据不可用。客户端无法通过匿名请求枚举容器中的 Blob。
* **容器**：可通过匿名请求读取容器和 Blob 数据。客户端可以通过匿名请求枚举容器中的 Blob，但无法枚举存储帐户中的容器。

以下示例演示如何创建一个具有**容器**访问权限的容器，这将允许 Internet 上的所有用户对其进行公共只读访问：

    -(void)createContainerWithPublicAccess{
        NSError *accountCreationError;

        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here;EndpointSuffix=core.chinacloudapi.cn" error:&accountCreationError];

        if(accountCreationError){
            NSLog(@"Error in creating account.");
        }

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

## 将 Blob 上传到容器
如 [Blob 服务概念](#blob-service-concepts)部分中所述，Blob 存储提供了三种不同类型的 blob：块 blob、追加 blob 和页 blob。目前，Azure 存储 iOS 库只支持块 Blob。大多数情况下，推荐使用块 Blob。

以下示例演示如何从 NSString 上传块 Blob。如果此容器中已存在同名的 Blob，则将覆盖该 Blob 的内容。

    -(void)uploadBlobToContainer{
        NSError *accountCreationError;

        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here;EndpointSuffix=core.chinacloudapi.cn" error:&accountCreationError];

        if(accountCreationError){
            NSLog(@"Error in creating account.");
        }

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

你可以通过查看 [Azure 存储资源管理器](http://storageexplorer.com)并验证容器 *containerpublic* 是否包含该 Blob *sampleblob* 来确认此操作是否正常工作。在此示例中，我们使用了公共容器，因此还可以通过转到 Blob URI 来验证此操作是否正常工作：

    https://nameofyourstorageaccount.blob.core.chinacloudapi.cn/containerpublic/sampleblob

除从 NSString 上传块 Blob 外，NSData、NSInputStream 或本地文件也存在类似的方法。

## 列出容器中的 Blob
以下示例演示如何列出容器中的所有 Blob。执行此操作时，应注意以下参数：

* **continuationToken** - 继续标记表示列出操作应开始的位置。如果未提供标记，它将从开头列出 Blob。可以列出任意数目的 Blob，从零到最大集。即使此方法返回零个结果，如果 `results.continuationToken` 不为空，则服务中也可能存在更多 blob 未列出。
* **prefix** - 你可以指定要用于 blob 列出的前缀。将仅列出以该前缀开头的 Blob。
* **useFlatBlobListing** - 如[命名和引用容器和 blob](https://docs.microsoft.com/en-us/rest/api/storageservices/fileservices/Naming-and-Referencing-Containers--Blobs--and-Metadata?redirectedfrom=MSDN) 部分中所述，虽然 Blob 服务是平面存储方案，但你可以通过命名具有路径信息的 blob 来创建虚拟层次结构。但是，目前不支持非平面列表；该功能即将推出。目前，此值应为 **YES**。
* **blobListingDetails** - 你可以指定在列出 blob 时要包含哪些项
  * _AZSBlobListingDetailsNone_：仅列出已提交的 Blob，不返回 Blob 元数据。
  * _AZSBlobListingDetailsSnapshots_：列出已提交的 blob 和 blob 快照。
  * _AZSBlobListingDetailsMetadata_：检索列表中返回的每个 Blob 的 Blob 元数据。
  * _AZSBlobListingDetailsUncommittedBlobs_：列出已提交和未提交的 blob。
  * _AZSBlobListingDetailsCopy_：在列表中包括复制属性。
  * _AZSBlobListingDetailsAll_：列出所有可用的已提交 Blob、未提交 Blob 和快照，并返回这些 Blob 的所有元数据和复制状态。
* **maxResults** - 此操作可返回的结果的最大数目。使用 -1 以不设置限制。
* **completionHandler** - 要使用列表操作的结果执行的代码块。

在此示例中，帮助器方法用于在每次返回继续标记时递归调用列出 Blob 方法。

    -(void)listBlobsInContainer{
        NSError *accountCreationError;

        // Create a storage account object from a connection string.
      AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here;EndpointSuffix=core.chinacloudapi.cn" error:&accountCreationError];

        if(accountCreationError){
            NSLog(@"Error in creating account.");
        }

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
以下示例演示如何将 Blob 下载到 NSString 对象。

    -(void)downloadBlobToString{
        NSError *accountCreationError;

        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here;EndpointSuffix=core.chinacloudapi.cn" error:&accountCreationError];

        if(accountCreationError){
            NSLog(@"Error in creating account.");
        }

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
以下示例说明如何删除 Blob。

    -(void)deleteBlob{
        NSError *accountCreationError;

        // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here;EndpointSuffix=core.chinacloudapi.cn" error:&accountCreationError];

        if(accountCreationError){
            NSLog(@"Error in creating account.");
        }

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
      NSError *accountCreationError;

      // Create a storage account object from a connection string.
        AZSCloudStorageAccount *account = [AZSCloudStorageAccount accountFromConnectionString:@"DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here;EndpointSuffix=core.chinacloudapi.cn" error:&accountCreationError];

      if(accountCreationError){
         NSLog(@"Error in creating account.");
      }

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

##<a name="next-steps"></a> 后续步骤

既已了解如何从 iOS 使用 Blob 存储，可单击以下链接详细了解 iOS 库和存储服务。

- [Azure Storage Client Library for iOS（适用于 iOS 的 Azure 存储客户端库）](https://github.com/azure/azure-storage-ios)
- [Azure 存储空间 iOS 参考文档](http://azure.github.io/azure-storage-ios/)
- [Azure 存储空间服务 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dd179355.aspx)
- [Azure 存储团队博客](http://blogs.msdn.com/b/windowsazurestorage)

如果你对此库有任何疑问，请随意将问题发布到我们的 [MSDN Azure 论坛](https://social.msdn.microsoft.com/forums/azure/zh-cn/home?forum=windowsazuredata)或[堆栈溢出](http://stackoverflow.com/questions/tagged/windows-azure-storage+or+windows-azure-storage+or+azure-storage-blobs+or+azure-storage-tables+or+azure-table-storage+or+windows-azure-queues+or+azure-storage-queues+or+azure-storage-emulator+or+azure-storage-files)。如果你有 Azure 存储空间的功能建议，请将建议发布到 [Azure 存储空间反馈](/product-feedback)。

<!---HONumber=Mooncake_0103_2017-->