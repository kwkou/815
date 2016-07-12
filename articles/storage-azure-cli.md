<properties
    pageTitle="将 Azure CLI 用于 Azure 存储服务 | Azure"
    description="了解如何将 Azure 命令行界面 (Azure CLI) 用于 Azure 存储服务，以便创建和管理存储帐户并处理 Azure blob 和文件。Azure CLI 是一个跨平台工具"
    services="storage"
    documentationCenter="na"
    authors="tamram"
    manager="jdial"/>

<tags
    ms.service="storage"
   
    ms.date="05/02/2016"
    wacn.date="06/06/2016"/>

# 将 Azure CLI 用于 Azure 存储服务

## 概述

Azure CLI 提供了一组开源且跨平台的命令，这些命令可以用于 Azure 平台。它提供了[管理门户](https://manage.windowsazure.cn)所提供的很多相同功能，以及各种数据访问功能。

在本指南中，我们将探讨如何使用 [Azure 命令行界面 (Azure CLI)](/documentation/articles/xplat-cli-install/)，以便通过 Azure 存储空间执行各种开发和管理任务。在使用本指南之前，我们建议你下载和安装或者升级到最新版 Azure CLI。

本指南假定你了解 Azure 存储服务的基本概念。本指南提供了大量的脚本，用于演示 Azure CLI 与 Azure 存储服务的用法。在运行每个脚本之前，请确保根据配置更新脚本变量。

> [AZURE.NOTE] 本指南提供在 Azure 服务管理 (ASM) 模式下运行的 Azure CLI 命令和脚本的示例。若要了解如何使用 Azure CLI 命令在 Azure 资源管理 (ARM) 模式下进行存储，请参阅[将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure 资源管理配合使用](/documentation/articles/azure-cli-arm-commands/#azure-storage-commands-to-manage-your-storage-objects)。

## 在 5 分钟内开始使用 Azure 存储服务和 Azure CLI

本指南使用 Ubuntu 作为示例，但其他 OS 平台的操作应与此类似。

**Azure 新用户：**获取一个 Azure 订阅以及与该订阅关联的 Microsoft 帐户。有关 Azure 购买选项的信息，请参阅[试用](/pricing/1rmb-trial/)、[购买选项](/pricing/purchase-options/)<!--、和[成员优惠](http://azure.microsoft.com/pricing/member-offers/)（适用于 MSDN、Microsoft 合作伙伴网络和 BizSpark 以及其他 Microsoft 计划的成员）-->。

请参阅[在 Azure Active Directory (Azure AD) 中分配管理员角色](https://msdn.microsoft.com/zh-cn/library/azure/hh531793.aspx)，以了解有关 Azure 订阅的更多信息。

**创建 Azure 订阅和帐户之后：**

1. 按照[安装 Azure CLI](/documentation/articles/xplat-cli-install/) 中概述的说明，下载和安装 Azure CLI。
2. 安装了 Azure CLI 之后，你将可以从命令行界面（Bash、终端、命令提示符）使用 azure 命令访问 Azure CLI 命令。输入 `azure` 命令，你应该会看到以下输出：

    ![Azure 命令输出][Image1]

3. 在命令行界面中，输入 `azure storage` 即可列出所有 Azure 存储服务命令，并初步了解 Azure CLI 提供的功能。你可以输入带 **-h** 参数的命令名称（例如，`azure storage share create -h`），了解命令语法的详细信息。
4. 现在，我们将提供一个简单的脚本，演示用于访问 Azure 存储服务的基本 Azure CLI 命令。该脚本会首先要求你针对存储帐户和密钥设置两个变量。然后，该脚本将在此新存储帐户中创建新容器，并将现有图像文件 (Blob) 上载到该容器。脚本在列出该容器中的所有 Blob 后，就会将图像文件下载到本地计算机上的目标目录。

		#!/bin/bash
		# A simple Azure storage example

		export AZURE_STORAGE_ACCOUNT=<storage_account_name>
		export AZURE_STORAGE_ACCESS_KEY=<storage_account_key>

		export container_name=<container_name>
		export blob_name=<blob_name>
		export image_to_upload=<image_to_upload>
		export destination_folder=<destination_folder>

		echo "Creating the container..."
		azure storage container create $container_name

		echo "Uploading the image..."
		azure storage blob upload $image_to_upload $container_name $blob_name

		echo "Listing the blobs..."
		azure storage blob list $container_name

		echo "Downloading the image..."
		azure storage blob download $container_name $blob_name $destination_folder

		echo "Done"

5. 在本地计算机中，打开首选的文本编辑器（例如 vim）。在文本编辑器中输入上述脚本。

6. 现在，你需要基于你的环境配置中的设置更新脚本变量。

    - **<storage_account_name>**使用脚本中给定的名称，或输入存储帐户的新名称。**重要提示：**在 Azure 中，存储帐户的名称必须是唯一的。而且必须为小写！

    - **<storage_account_key>**你的存储帐户的访问密钥。

    - **<container_name>**使用脚本中给定的名称，或输入容器的新名称。

    - **<image_to_upload>**输入本地计算机上图片的路径，例如：“~/images/HelloWorld.png”。

    - **<destination_folder>**输入用于存储从 Azure 存储服务下载的文件的本地目录路径，例如：“~/downloadImages”。

7. 在 vim 中更新完必需的变量以后，按组合键“Esc, : , wq!”保存脚本。

8. 若要运行此脚本，在 bash 控制台中输入脚本文件名即可。运行此脚本后，应会创建包含已下载图像文件的本地目标文件夹。以下屏幕截图显示了示例输出：

运行脚本后，应会创建包含已下载图像文件的本地目标文件夹。

## 通过 Azure CLI 管理存储帐户

### 连接到你的 Azure 订阅

大多数存储命令没有 Azure 订阅也可以使用，不过我们仍建议你通过 Azure CLI 连接到你的订阅。若要配置 Azure CLI 以使用你的订阅，请执行[从 Azure CLI 连接到 Azure 订阅](/documentation/articles/xplat-cli-connect/)中的步骤。

### 新建存储帐户

若要使用 Azure 存储服务，你需要一个存储帐户。可以在将计算机配置为连接到订阅之后，创建新的 Azure 存储帐户。

        azure storage account create <account_name>

存储帐户名称必须为 3 到 24 个字符，并且只能使用数字和小写字母。

### 在环境变量中设置默认的 Azure 存储帐户

可以在订阅中设置多个存储帐户。你可以选择其中的一个存储帐户，并在环境变量中将其设置为同一会话中所有存储命令的默认存储帐户。这样，你便可以在不显式指定存储帐户和密钥的情况下运行 Azure CLI 存储命令。

        export AZURE_STORAGE_ACCOUNT=<account_name>
        export AZURE_STORAGE_ACCESS_KEY=<key>

若要设置默认存储帐户，另一种方法是使用连接字符串。首先，通过以下命令获取连接字符串：

        azure storage account connectionstring show <account_name>

然后复制输出连接字符串，并将其设置为环境变量：

        export AZURE_STORAGE_CONNECTION_STRING=<connection_string>

## 创建并管理 blob

Azure Blob 存储是用于存储大量非结构化数据（例如文本或二进制数据）的服务，这些数据可通过 HTTP 或 HTTPS 从世界各地进行访问。本部分假设你已熟悉 Azure Blob 存储的概念。有关详细信息，请参阅[通过 .NET 开始使用 Azure Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/)和[Blob 服务概念](http://msdn.microsoft.com/zh-cn/library/azure/dd179376.aspx)。

### 创建容器

Azure 存储服务中的每个 Blob 都必须在容器中。你可以使用 `azure storage container create` 命令创建专用容器：

        azure storage container create mycontainer

> [AZURE.NOTE] 有三种级别的匿名读取访问权限：**Off**、**Blob** 和 **Container**。若要防止对 Blob 进行匿名访问，请将 Permission 参数设置为 **Off**。默认情况下，新容器是专用容器，只能由帐户所有者访问。若要允许对 Blob 资源进行匿名公共读取访问，但不允许访问容器元数据或容器中的 Blob 列表，请将 Permission 参数设置为 **Blob**。若要允许对 Blob 资源、容器元数据和容器中的 Blob 列表进行完全公开读取访问，请将 Permission 参数设置为 **Container**。有关详细信息，请参阅[管理对容器和 Blob 的匿名读取访问](/documentation/articles/storage-manage-access-to-resources/)。

### 将 Blob 上载到容器中

Azure Blob 存储支持块 Blob 和页 Blob。有关详细信息，请参阅[了解块 Blob、追加 Blob 和页 Blob](http://msdn.microsoft.com/zh-cn/library/azure/ee691964.aspx)。

若要将 blob 上载到容器，可使用 `azure storage blob upload`。默认情况下，此命令会将本地文件上载到块 Blob。若要指定 Blob 的类型，可以使用 `--blobtype` 参数。

        azure storage blob upload '~/images/HelloWorld.png' mycontainer myBlockBlob

### 从容器下载 Blob

以下示例演示如何从容器下载 Blob。

        azure storage blob download mycontainer myBlockBlob '~/downloadImages/downloaded.png'

### 复制 blob

你可以在存储帐户和区域内或跨存储帐户和区域异步复制 Blob。

以下示例演示如何将一个存储帐户中的 Blob 复制到另一个存储帐户。在此示例中，我们创建了一个容器，其中的 blob 可以进行公开、匿名的访问。

    azure storage container create mycontainer2 -a <accountName2> -k <accountKey2> -p Blob

    azure storage blob upload '~/Images/HelloWorld.png' mycontainer2 myBlockBlob2 -a <accountName2> -k <accountKey2>

    azure storage blob copy start 'https://<accountname2>.blob.core.chinacloudapi.cn/mycontainer2/myBlockBlob2' mycontainer

此示例将执行异步复制。你可以通过运行 `azure storage blob copy show` 操作来监视每个复制操作的状态。

请注意，所提供的用于复制操作的源 URL 必须可以公开访问，或者必须包括一个 SAS（共享访问签名）令牌。

### 删除 Blob

若要删除 blob，请使用以下命令：

        azure storage blob delete mycontainer myBlockBlob2

##<a id="create-and-manage-file-shares"></a> 创建和管理文件共享

Azure 文件存储使用标准 SMB 协议为应用程序提供共享存储。Azure 虚拟机和云服务以及本地应用程序可以通过装载的共享来共享文件数据。你可以通过 Azure CLI 管理文件共享和文件数据。有关 Azure 文件存储的详细信息，请参阅[在 Windows 上开始使用 Azure 文件存储](/documentation/articles/storage-dotnet-how-to-use-files/)或[如何通过 Linux 使用 Azure 文件存储](/documentation/articles/storage-how-to-use-files-linux/)。

### 创建文件共享

Azure 文件共享是 Azure 中的 SMB 文件共享。所有目录和文件都必须在文件共享中创建。一个帐户可以包含无限数量的共享，一个共享可以存储无限数量的文件，直到达到存储帐户的容量限制为止。下面的示例创建名为 **myshare** 的文件共享。

        azure storage share create myshare

### 创建目录

目录提供了进行 Azure 文件共享时可以选择的层次结构。以下示例在文件共享中创建名为 **myDir** 的目录。

        azure storage directory create myshare myDir

请注意，目录路径可以包括多个级别，例如 **a/b**。但是，你必须确保所有父目录都存在。例如，对于路径 **a/b**，你必须先创建目录 **a**，然后创建目录 **b**。

### 将本地文件上载到目录

以下示例将文件从 **~/temp/samplefile.txt** 上载到 **myDir** 目录。请编辑文件路径，使其指向你本地计算机上的有效文件：

        azure storage file upload '~/temp/samplefile.txt' myshare myDir

请注意，共享中的文件最大可以为 1 TB。

### 列出共享根目录或目录中的文件

可以使用以下命令列出共享根目录或目录中的文件和子目录：

        azure storage file list myshare myDir

请注意，进行列表操作时，目录名是可选的。如果省略目录名，该命令会列出共享中根目录的内容。

### 复制文件

从 Azure CLI 的 0.9.8 版开始，可以将一个文件复制到另一个文件，将一个文件复制到一个 Blob，或将一个 Blob 复制到一个文件。下面，我们演示如何使用 CLI 命令执行这些复制操作。若要将文件复制到新目录中：

	azure storage file copy start --source-share srcshare --source-path srcdir/hello.txt --dest-share destshare --dest-path destdir/hellocopy.txt --connection-string $srcConnectionString --dest-connection-string $destConnectionString

若要将 blob 复制到一个文件目录中：

	azure storage file copy start --source-container srcctn --source-blob hello2.txt --dest-share hello --dest-path hellodir/hello2copy.txt --connection-string $srcConnectionString --dest-connection-string $destConnectionString

## 后续步骤

下面是一些相关的文章和资源，可以让你更多地了解 Azure 存储服务。

- [Azure 存储空间文档](/documentation/services/storage)
- [Azure 存储 REST API 引用](https://msdn.microsoft.com/zh-cn/library/azure/dd179355.aspx)


[Image1]: ./media/storage-azure-cli/azure_command.png

<!---HONumber=Mooncake_0530_2016-->