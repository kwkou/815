<properties linkid="dev-ruby-how-to-blob-storage" urlDisplayName="Blob Service" pageTitle="How to use blob storage (Ruby) | Microsoft Azure" metaKeywords="Get started Azure blob, Azure unstructured data, Azure unstructured storage, Azure blob, Azure blob storage, Azure blob Ruby" description="Learn how to use the Azure blob service to upload, download, list, and delete blob content. Samples written in Ruby." metaCanonical="" services="storage" documentationCenter="Ruby" title="How to Use the Blob Service from Ruby" authors="guayan" solutions="" manager="" editor="" />

# 如何通过 Ruby 使用 Blob 服务

本指南将演示如何使用 Azure Blob 服务执行常见方案。
示例是用 Ruby API 编写的。涉及的方案包括
**上载、列出、下载**和**删除** Blob。
有关 Blob 的详细信息，请参阅[后续步骤][]部分。

## 目录

-   [什么是 Blob 服务？][]
-   [概念][]
-   [创建 Azure 存储帐户][]
-   [创建 Ruby 应用程序][]
-   [配置应用程序以访问存储][]
-   [设置 Azure 存储连接][]
-   [如何：创建容器][]
-   [如何：将 Blob 上载到容器][]
-   [如何：列出容器中的 Blob][]
-   [如何：下载 Blob][]
-   [如何：删除 Blob][]
-   [后续步骤][1]

[WACOM.INCLUDE [howto-blob-storage][]]

## 创建 Azure 存储帐户

[WACOM.INCLUDE [create-storage-account][]]

## 创建 Ruby 应用程序

创建 Ruby 应用程序。有关说明，请参阅
[在 Azure 上创建 Ruby 应用程序][]。

## 配置应用程序以访问存储

若要使用 Azure 存储空间，你需要下载并使用 Ruby azure 包，其中包括一组便于与存储 REST 服务进行通信的库。

### 使用 RubyGems 获取该程序包

1.  使用命令行接口，例如 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (Unix)。

2.  在命令窗口中键入“gem install azure”以安装 gem 和依赖项。

### 导入包

使用常用的文本编辑器将以下内容添加到你要在其中使用存储的 Ruby 文件的顶部：

    require "azure"

## 设置 Azure 存储连接

azure 模块将读取环境变量 **AZURE\_STORAGE\_ACCOUNT** 和 **AZURE\_STORAGE\_ACCESS\_KEY** 以获取
连接到你的 Azure 存储帐户所需的信息。如果未设置这些环境变量，则在使用 **Azure::BlobService** 之前必须通过以下代码指定帐户信息：

    Azure.config.storage_account_name = "<your azure storage account>"
    Azure.config.storage_access_key = "<your azure storage access key>"

获取这些值：

1.  登录到 [Azure 管理门户][]。
2.  导航到要使用的存储帐户
3.  单击导航窗格底部的**“管理密钥”**。
4.  在弹出对话框中，你将会看到存储帐户名称、主访问密钥和辅助访问密钥。对于访问密钥，你可以使用主访问密钥，也可以使用辅助访问密钥。

## 如何：创建容器

使用 **Azure::BlobService** 对象可以对容器和 Blob 进行操作。若要创建容器，请使用 **create\_container()** 方法。

以下示例创建一个容器或输出存在的错误。

    azure_blob_service = Azure::BlobService.new
    begin
    container = azure_blob_service.create_container("test-container")
    rescue
    puts $!
    end

若要将容器中的文件设为公用，你可以设置容器的权限。

只需修改 **create\_container()** 调用来传递 **:public\_access\_level** 选项即可：

    container = azure_blob_service.create_container("test-container", 
    :public_access_level => "<public access level>")

**:public\_access\_level** 选项的有效值为：

-   **blob：**指定对容器和 Blob 数据的完整公共读取权限。客户端可以通过匿名请求枚举容器中的 Blob，但无法枚举存储帐户中的容器。

-   **container：**指定对 Blob 的公共读取权限。可以通过匿名请求读取此容器中的 Blob 数据，但无法使用容器数据。客户端无法通过匿名请求枚举容器中的 Blob。

另外，还可以通过使用 **set\_container\_acl()** 方法指定公共访问级别来修改容器的公共访问级别。

下面的示例将公共访问级别更改为“容器” ：

    azure_blob_service.set_container_acl('test-container', "container")

## 如何：将 Blob 上载到容器

若要将内容上载到 Blob，请使用 **create\_block\_blob()** 方法创建 Blob，将文件或字符串用作 Blob 的内容。

以下代码会将文件 **test.png** 作为名为“image-blob”的新 Blob 上载到容器中。

    content = File.open("test.png", "rb") { |file| file.read }
    blob = azure_blob_service.create_block_blob(container.name,
    "image-blob", content)
    puts blob.name

## 如何：列出容器中的 Blob

若要列出容器，请使用 **list\_containers()** 方法。
若要列出容器中的 Blob，请使用 **list\_blobs()** 方法。

这将输出帐户的所有容器中的所有 Blog 的 URL。

    containers = azure_blob_service.list_containers()
    containers.each do |container|
    blobs = azure_blob_service.list_blobs(container.name)
    blobs.each do |blob|
    puts blob.name
    end
    end

## 如何：下载 Blob

若要下载 Blob，请使用 **get\_blob()** 方法来检索内容。

以下示例演示了如何使用 **get\_blob()** 下载“image-blob”的内容并将其写入本地文件中。

    blob, content = azure_blob_service.get_blob(container.name,"image-blob")
    File.open("download.png","wb") {|f| f.write(content)}

## 如何：删除 Blob

最后，若要删除 Blob，请使用 **delete\_blob()** 方法。下面的示例演示了如何删除 Blob。

    azure_blob_service.delete_blob(container.name, "image-blob")

## 后续步骤

现在，你已了解有关 Blob 存储的基础知识，可单击下面的链接来了解如何执行更复杂的存储任务。

-   查看 MSDN 参考：[在 Azure 中存储和访问数据][]
-   访问 [Azure 存储空间团队博客][]
-   访问 GitHub 上的 [Azure SDK for Ruby][] 存储库

  [后续步骤]: #next-steps
  [什么是 Blob 服务？]: #what-is
  [概念]: #concepts
  [创建 Azure 存储帐户]: #CreateAccount
  [创建 Ruby 应用程序]: #CreateRubyApp
  [配置应用程序以访问存储]: #ConfigAccessStorage
  [设置 Azure 存储连接]: #SetupStorageConnection
  [如何：创建容器]: #CreateContainer
  [如何：将 Blob 上载到容器]: #UploadBlob
  [如何：列出容器中的 Blob]: #ListBlobs
  [如何：下载 Blob]: #DownloadBlobs
  [如何：删除 Blob]: #DeleteBlob
  [1]: #NextSteps
  [howto-blob-storage]: ../includes/howto-blob-storage.md
  [create-storage-account]: ../includes/create-storage-account.md
  [在 Azure 上创建 Ruby 应用程序]: /en-us/develop/ruby/tutorials/web-app-with-linux-vm/
  [Azure 管理门户]: https://manage.windowsazure.cn/
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [Azure 存储空间团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
  [Azure SDK for Ruby]: https://github.com/WindowsAzure/azure-sdk-for-ruby
