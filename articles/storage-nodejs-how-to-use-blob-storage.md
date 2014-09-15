<properties linkid="dev-nodejs-how-to-blob-storage" urlDisplayName="Blob Service" pageTitle="How to use blob storage (Node.js) | Microsoft Azure" metaKeywords="Get started Azure blob, Azure unstructured data, Azure unstructured storage, Azure blob, Azure blob storage, Azure blob Node.js" description="Learn how to use the Azure blob service to upload, download, list, and delete blob content. Samples written in Node.js." metaCanonical="" services="storage" documentationCenter="Node.js" title="How to Use the Blob Service from Node.js" authors="larryfr" solutions="" manager="" editor="" />

# 如何从 Node.js 使用 Blob 服务

本指南将演示如何使用 Azure Blob 服务执行常见方案。
示例是用 Node.js API 编写的。
涉及的任务包括**上载**、**列出**、
**下载**和**删除** Blob。有关 Blob 的
详细信息，请参阅[后续步骤][]部分。

## 目录

-   [什么是 Blob 服务？][]
-   [概念][]
-   [创建 Azure 存储帐户][]
-   [创建 Node.js 应用程序][]
-   [配置应用程序以访问存储][]
-   [设置 Azure 存储连接字符串][]
-   [如何：创建容器][]
-   [如何：将 Blob 上载到容器][]
-   [如何：列出容器中的 Blob][]
-   [如何：下载 Blob][]
-   [如何：删除 Blob][]
-   [后续步骤][]

[WACOM.INCLUDE [howto-blob-storage][]]

## 创建帐户创建 Azure 存储帐户

[WACOM.INCLUDE [create-storage-account][]]

## 创建 Node.js 应用程序

创建一个空的 Node.js 应用程序。有关创建 Node.js 应用程序的说明，请参阅[创建 Node.js 应用程序并将其部署到 Azure 网站][]、[Node.js 云服务][]（使用 Windows PowerShell）或[使用 WebMatrix 构建网站][]。

## 配置应用程序以访问存储

若要使用 Azure 存储空间，你需要下载并使用 Node.js azure 包，
其中包括一组便于与存储 REST 服务
进行通信的库。

### 使用 Node 包管理器 (NPM) 可获取该程序包

1.  使用 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (Unix) 等命令行界面导航到你在其中创建了示例应用程序的文件夹。

2.  在命令窗口中键入 **npm install azure**，这应该产生
    以下输出：

        azure@0.7.5 node_modules\azure
        ├── dateformat@1.0.2-1.2.3
        ├── xmlbuilder@0.4.2
        ├── node-uuid@1.2.0
        ├── mime@1.2.9
        ├── underscore@1.4.4
        ├── validator@1.1.1
        ├── tunnel@0.0.2
        ├── wns@0.5.3
        ├── xml2js@0.2.7 (sax@0.5.2)
        └── request@2.21.0 (json-stringify-safe@4.0.0, forever-agent@0.5.0, aws-sign@0.3.0, tunnel-agent@0.3.0, oauth-sign@0.3.0, qs@0.6.5, cookie-jar@0.3.0, node-uuid@1.4.0, http-signature@0.9.11, form-data@0.0.8, hawk@0.13.1)

3.  可以手动运行 **ls** 命令来验证是否创建了
    **node\_modules** 文件夹。在该文件夹中，你将找到
    **azure** 包，其中包含你访问存储所需
    的库。

### 导入包

使用记事本或其他文本编辑器将以下内容添加到你要在其中使用存储的
应用程序的 **server.js** 文件的顶部：

    var azure = require('azure');

## 设置 Azure 存储连接

azure 模块将读取环境变量 AZURE\_STORAGE\_ACCOUNT 和 AZURE\_STORAGE\_ACCESS\_KEY 以获取连接到你的 Azure 存储帐户所需的信息。如果未设置这些环境变量，则在调用 **createBlobService** 时必须指定帐户信息。

有关在 Azure 云服务的配置文件中设置环境变量的示例，请参阅[使用存储构建 Node.js 云服务][]。

有关在管理门户中为 Azure 网站设置环境变量的示例，请参阅[使用存储构建 Node.js Web 应用程序][]。

## 如何：创建容器

使用 **BlobService** 对象可以对容器和 Blob 进行操作。以下代码
将创建一个 **BlobService** 对象。将以下内容添加
到 **server.js** 顶部附近：

    var blobService = azure.createBlobService();

所有 Blob 都驻留在一个容器中。在
**BlobService** 对象上调用 **createContainerIfNotExists** 将返回
指定的容器（如果它存在），或创建具有指定名称的新容器
（如果它尚不存在）。默认情况下，新容器是专用容器，
需要使用访问密钥才能从该容器下载 Blob。

    blobService.createContainerIfNotExists(containerName, function(error){
    if(!error){
    // 容器存在并且是专用的
        }
    });

如果你要让容器中的文件成为公共文件（这样用户不需要具有访问密钥便可访问这些文件），则可以将容器的访问级别设置为
**“Blob”**或**“容器”**。将访问级别设置为“Blob” 后，可匿名读取此容器中的 Blob 内容和元数据，但无法匿名读取容器元数据（如列出容器中的所有 Blob）。将访问级别设置为“容器” 后，可匿名读取 Blob 内容和元数据，以及容器元数据。下面的示例演示了如何将访问级别设置为**“Blob”**：

    blobService.createContainerIfNotExists(containerName
    , {publicAccessLevel :'blob'}
    , function(error){
    if(!error){
    // 容器存在并且是公共的
            }
        });

另外，你可以通过使用 **setContainerAcl** 指定访问级别来修改容器的访问级别。下面的示例将访问级别更改为“容器”：

    blobService.setContainerAcl(containerName
    , 'container'
    , function(error){
    if(!error){
    // 容器访问级别设置为“容器”
            }
        });

### 筛选器

可以向使用 **BlobService** 执行的操作应用可选的筛选操作。筛选操作可以包括日志记录、自动重试等。筛选器是实现了具有签名的方法的对象：

        function handle (requestOptions, next)

在对请求选项执行预处理后，该方法需要调用“next”并且传递具有以下签名的回调：

        function (returnObject, finalCallback, next)

在此回调中并且在处理 returnObject（来自对服务器请求的响应）后，回调需要调用 next（如果它存在以便继续处理其他筛选器）或只调用 finalCallback 以便结束服务调用。

Azure SDK for Node.js 中附带了两个实现了重试逻辑的筛选器，分别是 **ExponentialRetryPolicyFilter** 和 **LinearRetryPolicyFilter**。下面的代码将创建一个 **BlobService** 对象，该对象使用 **ExponentialRetryPolicyFilter**：

    var retryOperations = new azure.ExponentialRetryPolicyFilter();
    var blobService = azure.createBlobService().withFilter(retryOperations);

## 如何：将 Blob 上载到容器

若要将数据上载到 Blob，请使用 **createBlockBlobFromFile**、**createBlockBlobFromStream** 或 **createBlockBlobFromText** 方法。**createBlockBlobFromFile** 上载文件的内容，而 **createBlockBlobFromStream** 上载流的内容。**createBlockBlobFromText** 上载指定文本值。

下面的示例将 **test1.txt** 文件的内容上载到“test1”Blob。

    blobService.createBlockBlobFromFile(containerName
    , 'test1'
    , 'test1.txt'
    , function(error){
    if(!error){
    // 文件已上载
            }
        });

## 如何：列出容器中的 Blob

若要列出容器中的 Blob，可使用带 **for** 循环的
**listBlobs** 方法来显示容器中每个 Blob 的名称。以下
代码将容器中每个 Blob 的**名称**输出到
控制台。

    blobService.listBlobs(containerName, function(error, blobs){
    if(!error){
    for(var index in blobs){
    console.log(blobs[index].name);
            }
        }
    });

## 如何：下载 Blob

若要从 Blob 下载数据，可使用 **getBlobToFile**、**getBlobToStream** 或 **getBlobToText**。以下示例演示了如何使用 **getBlobToStream** 下载 **test1** Blob 的内容，并使用一个流将其存储到 **output.txt** 文件：

    var fs=require('fs');
    blobService.getBlobToStream(containerName
    , 'test1'
    , fs.createWriteStream('output.txt')
    , function(error){
    if(!error){
    // 已将 Blob 写入到流
            }
        });

## 如何：删除 Blob

最后，若要删除 Blob，请调用 **deleteBlob**。下面的示例将删除名为“blob1”的 Blob。

    blobService.deleteBlob(containerName
    , 'blob1'
    , function(error){
    if(!error){
    // Blob 已删除
            }
        });

## 后续步骤

现在，你已了解有关 Blob 存储的基础知识，可单击下面的链接来了解
如何执行更复杂的存储任务。

-   查看 MSDN 参考：[在 Azure 中存储和访问数据][]。
-   访问 [Azure 存储空间团队博客][]。
-   访问 GitHub 上的 [Azure SDK for Node][] 存储库。

  [后续步骤]: #next-steps
  [什么是 Blob 服务？]: #what-is
  [概念]: #concepts
  [创建 Azure 存储帐户]: #create-account
  [创建 Node.js 应用程序]: #create-app
  [配置应用程序以访问存储]: #configure-access
  [设置 Azure 存储连接字符串]: #setup-connection-string
  [如何：创建容器]: #create-container
  [如何：将 Blob 上载到容器]: #upload-blob
  [如何：列出容器中的 Blob]: #list-blob
  [如何：下载 Blob]: #download-blobs
  [如何：删除 Blob]: #delete-blobs
  [howto-blob-storage]: ../includes/howto-blob-storage.md
  [create-storage-account]: ../includes/create-storage-account.md
  [创建 Node.js 应用程序并将其部署到 Azure 网站]: /en-us/develop/nodejs/tutorials/create-a-website-(mac)/
  [Node.js 云服务]: /en-us/documentation/articles/cloud-services-nodejs-develop-deploy-app/
  [使用 WebMatrix 构建网站]: /en-us/documentation/articles/web-sites-nodejs-use-webmatrix/
  [使用存储构建 Node.js 云服务]: /en-us/documentation/articles/storage-nodejs-use-table-storage-cloud-service-app/
  [使用存储构建 Node.js Web 应用程序]: /en-us/documentation/articles/storage-nodejs-use-table-storage-web-site/
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [Azure 存储空间团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
  [Azure SDK for Node]: https://github.com/WindowsAzure/azure-sdk-for-node
