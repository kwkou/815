<properties 
	pageTitle="如何通过 Ruby 使用 Blob 存储 | Windows Azure" 
	description="了解如何使用 Azure Blob 服务上载、下载、列出和删除 Blob 内容。用 Ruby 编写的相关示例。" 
	services="storage" 
	documentationCenter="ruby" 
	authors="tfitzmac" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="storage" 
	ms.date="05/11/2015" 
	wacn.date="09/18/2015"/>


# 如何通过 Ruby 使用 Blob 存储

[AZURE.INCLUDE [storage-selector-blob-include](../includes/storage-selector-blob-include.md)]

## 概述

本指南将演示如何使用 Azure Blob 服务执行常见方案。相关示例是使用 Ruby API 编写的。涉及的任务包括“上载”、“列出”、“下载”和“删除”Blob。

[AZURE.INCLUDE [storage-blob-concepts-include](../includes/storage-blob-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

## 创建 Ruby 应用程序

创建 Ruby 应用程序。有关说明，请参阅 [在 Azure 上创建 Ruby 应用程序](/develop/ruby/tutorials/web-app-with-linux-vm)。

## 配置应用程序以访问存储

要使用 Azure 存储空间，你需要下载和使用 Ruby azure 包，其中包括一组便于与存储 REST 服务进行通信的库。

### 使用 RubyGems 获取该程序包

1. 使用命令行接口，例如 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (Unix)。

2. 在命令窗口中键入“gem install azure”以安装 gem 和依赖项。

### 导入包

使用常用的文本编辑器将以下内容添加到你要在其中使用存储的 Ruby 文件的顶部：

	require "azure"

## 设置 Azure 存储连接

Azure 模块将读取环境变量 **AZURE\_STORAGE\_ACCOUNT** 和 **AZURE\_STORAGE\_ACCESS\_KEY**，以便获取连接到 Azure 存储帐户所需的信息。如果未设置这些环境变量，则在使用 **Azure::BlobService** 之前必须通过以下代码指定帐户信息：

	Azure.config.storage_account_name = "<your azure storage account>"
	Azure.config.storage_access_key = "<your azure storage access key>"


获取这些值：

1. 登录到 [Azure 管理门户](https://manage.windowsazure.cn).
2. 导航到要使用的存储帐户
3. 单击导航窗格底部的“管理密钥”。
4. 在弹出对话框中，将会看到存储帐户名称、主访问密钥和辅助访问密钥。对于访问密钥，您可以使用主访问密钥，也可以使用辅助访问密钥。

## 如何：创建容器

[AZURE.INCLUDE [storage-container-naming-rules-include](../includes/storage-container-naming-rules-include.md)]

使用 **Azure::BlobService** 对象可以对容器和 Blob 进行操作。若要创建容器，请使用 **create\_container()** 方法。

以下示例创建一个容器或输出存在的错误。

	azure_blob_service = Azure::BlobService.new
	begin
	  container = azure_blob_service.create_container("test-container")
	rescue
	  puts $!
	end

若要将容器中的文件设为公用，你可以设置容器的权限。 

只需修改 <strong>create\_container()</strong> 调用即可传递 **:public\_access\_level** 选项：

	container = azure_blob_service.create_container("test-container", 
	  :public_access_level => "<public access level>")


**:public\_access\_level** 选项的有效值为：

* **Blob：**指定对容器和 Blob 数据的完整公共读取权限。客户端可以通过匿名请求枚举容器中的 Blob，但无法枚举存储帐户中的容器。

* **容器：**指定对 Blob 的公共读取权限。可以通过匿名请求读取此容器中的 Blob 数据，但容器数据不可用。客户端无法通过匿名请求枚举容器中的 Blob。

另外，还可以通过使用 **set\_container\_acl()** 方法指定公共访问级别来修改容器的公共访问级别。
 
下面的示例将公共访问级别更改为“容器”：

	azure_blob_service.set_container_acl('test-container', "container")

## 如何：将 Blob 上载到容器

若要将内容上载到 Blob，请使用 **create\_block\_blob()** 方法创建 Blob，将文件或字符串用作 Blob 的内容。 

以下代码会将文件 **test.png** 作为名为“image-blob”的新 Blob 上载到容器中。

	content = File.open("test.png", "rb") { |file| file.read }
	blob = azure_blob_service.create_block_blob(container.name,
	  "image-blob", content)
	puts blob.name

## 如何：列出容器中的 Blob

若要列出容器，请使用 **list\_containers()** 方法。若要列出容器中的 Blob，请使用 **list\_blobs()** 方法。

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

现在，您已了解有关 Blob 存储的基础知识，可单击下面的链接来了解更复杂的存储任务。

- 请参阅 MSDN 参考：[Azure 存储](http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx)
-   访问 [Azure 存储空间团队博客](http://blogs.msdn.com/b/windowsazurestorage)
- 访问 GitHub 上的 [Azure SDK for Ruby](https://github.com/WindowsAzure/azure-sdk-for-ruby) 存储库
<!---HONumber=70-->