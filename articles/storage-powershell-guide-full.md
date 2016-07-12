<properties
	pageTitle="对 Azure 存储空间使用 Azure PowerShell | Azure"
	description="了解如何使用 Azure 存储的 Azure PowerShell cmdlet 来创建和管理存储帐户；使用 Blob、表、队列和文件；配置和查询存储分析并创建共享访问签名。"
	services="storage"
	documentationCenter="na"
	authors="robinsh" 
	manager="carmonm"/>

<tags
	ms.service="storage"
	ms.date="02/19/2016"
	wacn.date="04/18/2016"/>

# 对 Azure 存储空间使用 Azure PowerShell

## 概述

Azure PowerShell 是一个模块，提供用于通过 Windows PowerShell 管理 Azure 的 cmdlet。它是一种基于任务的命令行 shell 和脚本语言，专门用于系统管理。使用 PowerShell，你可以轻松控制和自动化 Azure 服务与应用程序的管理。例如，这些 cmdlet 可让你执行在 [Azure 门户](https://manage.windowsazure.cn)中可以执行的任务。

在本指南中，我们将探讨如何使用[Azure 存储服务 Cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/mt269418.aspx)，以便通过 Azure 存储空间执行各种开发和管理任务。

本指南假设你有使用 [Azure 存储服务](/documentation/services/storage)和 [Windows PowerShell](http://technet.microsoft.com/zh-cn/library/bb978526.aspx) 的经验。本指南提供了大量的脚本，用于演示 PowerShell 与 Azure 存储空间的用法。在运行每个脚本之前，你应该根据配置更新脚本变量。

本指南的第一部分提供 Azure 存储空间和 PowerShell 的概览。有关详细信息和说明，请先了解[对 Azure 存储空间使用 Azure PowerShell 的先决条件](#prerequisites-for-using-azure-powershell-with-azure-storage)。


## 在 5 分钟内开始使用 Azure 存储空间和 PowerShell

本部分说明如何在 5 分钟内通过 PowerShell 访问 Azure 存储空间。

Azure 新用户：获取一个 Azure 订阅以及与该订阅关联的 Microsoft 帐户。有关 Azure 购买选项的信息，请参阅[免费试用](/pricing/1rmb-trial/)、[购买选项](/pricing/purchase-options/)。

请参阅[在 Azure Active Directory (Azure AD) 中分配管理员角色](https://msdn.microsoft.com/zh-cn/library/azure/hh531793.aspx)，以了解有关 Azure 订阅的详细信息。

**创建 Azure 订阅和帐户之后：**

1.	下载和安装 [Azure PowerShell](http://go.microsoft.com/?linkid=9811175&clcid=0x409)。
2.	启动 Windows PowerShell 集成脚本环境 (ISE)：在本地计算机上，请转到“开始”菜单。键入“管理工具”，并单击以运行它。在“管理工具”窗口中，右键单击“Windows PowerShell ISE”，然后单击“以管理员身份运行”。
3.	在“Windows PowerShell ISE”中，单击“文件”>“新建”以创建新的脚本文件。
4.	现在，我们将提供一个简单的脚本，演示用于访问 Azure 存储空间的基本 PowerShell 命令。该脚本首先会请求提供你的 Azure 帐户凭据，以将你的 Azure 帐户添加到本地 PowerShell 环境。然后，该脚本将设置默认 Azure 订阅，并在 Azure 中创建新的存储帐户。接下来，该脚本将在此新存储帐户中创建新容器，并将现有图像文件 (Blob) 上载到该容器。在脚本列出该容器中的所有 Blob 后，它将在本地计算机中创建新的目标目录，并下载图像文件。
5.	在以下代码部分中，选择注释 **#begin** 和 **#end** 之间的脚本。按 CTRL+C 将其复制到剪贴板。

    	#begin
    	# Update with the name of your subscription.
    	$SubscriptionName="YourSubscriptionName"

    	# Give a name to your new storage account. It must be lowercase!
    	$StorageAccountName="yourstorageaccountname"

    	# Choose "China North" as an example.
    	$Location = "China North"

    	# Give a name to your new container.
    	$ContainerName = "imagecontainer"

    	# Have an image file and a source directory in your local computer.
    	$ImageToUpload = "C:\Images\HelloWorld.png"

    	# A destination directory in your local computer.
    	$DestinationFolder = "C:\DownloadImages"

    	# Add your Azure account to the local PowerShell environment.
    	Add-AzureAccount -Environment AzureChinaCloud

    	# Set a default Azure subscription.
    	Select-AzureSubscription -SubscriptionName $SubscriptionName –Default

    	# Create a new storage account.
    	New-AzureStorageAccount –StorageAccountName $StorageAccountName -Location $Location

    	# Set a default storage account.
    	Set-AzureSubscription -CurrentStorageAccountName $StorageAccountName -SubscriptionName $SubscriptionName

    	# Create a new container.
    	New-AzureStorageContainer -Name $ContainerName -Permission Off

    	# Upload a blob into a container.
    	Set-AzureStorageBlobContent -Container $ContainerName -File $ImageToUpload

    	# List all blobs in a container.
    	Get-AzureStorageBlob -Container $ContainerName

    	# Download blobs from the container:
    	# Get a reference to a list of all blobs in a container.
    	$blobs = Get-AzureStorageBlob -Container $ContainerName

    	# Create the destination directory.
    	New-Item -Path $DestinationFolder -ItemType Directory -Force  

    	# Download blobs into the local destination directory.
    	$blobs | Get-AzureStorageBlobContent –Destination $DestinationFolder
    	#end

5.	在 **Windows PowerShell ISE** 中，按 CTRL + V 复制该脚本。单击“文件”>“保存”。在“另存为”对话窗口中，键入脚本文件的名称，例如“mystoragescript”。单击“保存”。

6.	现在，你需要基于配置设置更新脚本变量。你必须根据自己的订阅更新 **$SubscriptionName** 变量。可以保留脚本中指定的其他变量，或根据需要更新。

	- **$SubscriptionName：**你必须根据自己的订阅更新此变量。请遵循找到订阅名称的以下三种方法之一：

		a.在“Windows PowerShell ISE”中，单击“文件”>“新建”以创建新的脚本文件。将以下脚本复制到新脚本文件，然后单击“调试”>“运行”。以下脚本会先请求你提供 Azure 帐户凭据以将你的 Azure 帐户添加到本地 PowerShell 环境，然后显示已连接到本地 PowerShell 会话的所有订阅。记下你在学习本教程时要使用的订阅名称：
		
    		Add-AzureAccount -Environment AzureChinaCloud
       		Get-AzureSubscription | Format-Table SubscriptionName, IsDefault, IsCurrent, CurrentStorageAccountName


		b.Azure 当前在中国支持的一个门户：[Azure 管理门户](https://manage.windowsazure.cn/)。如果登录到了当前的 [Azure 管理门户](https://manage.windowsazure.cn/)，请向下滚动，然后单击门户左侧的“设置”。单击“订阅”。复制你在运行本指南中指定的脚本时要使用的订阅名称。有关示例，请参阅下面的屏幕截图。

		![Azure 管理门户][Image1]




	- **$StorageAccountName：**使用脚本中给定的名称，或输入存储帐户的新名称。**重要提示：**在 Azure 中，存储帐户的名称必须是唯一的。它还必须为小写！

	- **$Location：**使用脚本中给定的“China North”，或者选择其他 Azure 位置，例如 China East 等等。

	- **$ContainerName：**使用脚本中给定的名称，或输入容器的新名称。

	- **$ImageToUpload：**输入本地计算机上图片的路径，例如："C:\Images\HelloWorld.png"。

	- **$DestinationFolder：**输入用于存储从 Azure 存储空间下载的文件的本地目录路径，例如："C:\DownloadImages"。

7.	在更新“mystoragescript.ps1”文件中的脚本变量后，请单击“文件”>“保存”。然后，单击“调试”>“运行”，或按 **F5** 运行该脚本。

运行脚本后，应会创建包含已下载图像文件的本地目标文件夹。以下屏幕截图显示了示例输出：

![下载 Blob][Image3]


> [AZURE.NOTE] “在 5 分钟内开始使用 Azure 存储空间和 PowerShell”部分提供了有关如何对 Azure 存储空间使用 Azure PowerShell 的简介。有关详细信息和说明，建议你阅读以下部分。

##<a id="prerequisites-for-using-azure-powershell-with-azure-storage"></a> 对 Azure 存储空间使用 Azure PowerShell 的先决条件
如上所述，你需要一个 Azure 订阅和帐户来运行本指南中指定的 PowerShell cmdlet。

Azure PowerShell 是一个模块，提供用于通过 Windows PowerShell 管理 Azure 的 cmdlet。有关安装和设置 Azure PowerShell 的信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。建议你在使用本指南之前下载并安装或者升级到最新的 Azure PowerShell 模块。

可以在 Azure PowerShell 控制台、标准 Windows PowerShell 控制台或 Windows PowerShell 集成脚本环境 (ISE) 中运行 cmdlet。例如，若要打开“Azure PowerShell 控制台”，请转到“开始”菜单，键入 Azure PowerShell，单击右键，然后单击“以管理员身份运行”。若要打开 **Windows PowerShell ISE**，请转到“开始”菜单，键入“管理工具”，然后单击以运行它。在“管理工具”窗口中，右键单击“Windows PowerShell ISE”，然后单击“以管理员身份运行”。

## 如何管理 Azure 中的存储帐户

### 如何设置默认的 Azure 订阅
若要使用 Azure PowerShell 管理 Azure 存储空间，需要通过 Azure Active Directory 身份验证或基于证书的身份验证在 Azure 中对你的客户端环境进行身份验证。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)教程。本指南使用 Azure Active Directory 身份验证。

1.	在 Azure PowerShell 控制台或 Windows PowerShell ISE 中键入以下命令，将你的 Azure 帐户添加到本地 PowerShell 环境：

    `Add-AzureAccount -Environment AzureChinaCloud`

2.	在“登录 Azure”窗口中，键入与你的帐户关联的电子邮件地址和密码。Azure 将对凭据信息进行身份验证和保存，然后关闭该窗口。

3.	接下来，运行以下命令以查看本地 PowerShell 环境中的 Azure 帐户，并检查是否列出了你的帐户：

	`Get-AzureAccount`

4.	然后，运行以下 cmdlet 以查看已连接到本地 PowerShell 会话的所有订阅，并检查是否列出了你的订阅：

	`Get-AzureSubscription | Format-Table SubscriptionName, IsDefault, IsCurrent, CurrentStorageAccountName`

5.	若要设置默认 Azure 订阅，请运行 Select-AzureSubscription cmdlet：

	    $SubscriptionName = 'Your subscription Name'
    	Select-AzureSubscription -SubscriptionName $SubscriptionName –Default

6.	通过运行 Get-AzureSubscription cmdlet 检查默认订阅的名称：

	`Get-AzureSubscription -Default`

7.	若要查看 Azure 存储空间的所有可用 PowerShell cmdlet，请运行：

	`Get-Command -Module Azure -Noun *Storage*`

### 如何创建新的 Azure 存储帐户
若要使用 Azure 存储空间，你需要一个存储帐户。可以在将计算机配置为连接到订阅之后，创建新的 Azure 存储帐户。

1.	运行 Get-AzureLocation cmdlet 查找所有可用的数据中心位置：

    `Get-AzureLocation | format-Table -Property Name, AvailableServices, StorageAccountTypes`

2.	接下来，运行 New-AzureStorageAccount cmdlet 创建新的存储帐户。以下示例将在“中国北部”数据中心创建新的存储帐户。

    	$location = "China North"
	    $StorageAccountName = "yourstorageaccount"
	    New-AzureStorageAccount –StorageAccountName $StorageAccountName -Location $location

> [AZURE.IMPORTANT] 存储帐户的名称在 Azure 中是唯一的，并且必须采用小写。有关命名约定和限制，请参阅[关于 Azure 存储帐户](/documentation/articles/storage-create-storage-account/)、[命名和引用容器、Blob 和元数据](http://msdn.microsoft.com/zh-cn/library/azure/dd135715.aspx)。

### 如何设置默认的 Azure 存储帐户
可以在订阅中设置多个存储帐户。你可以选择其中的一个存储帐户，并将其设置为同一个 PowerShell 会话中所有存储命令的默认存储帐户。这样，你便可以在不显式指定存储上下文的情况下运行 Azure PowerShell 存储命令。

1.	若要设置订阅的默认存储帐户，可以运行 Set-AzureSubscription cmdlet。

		$SubscriptionName = "Your subscription name"
     	$StorageAccountName = "yourstorageaccount"  
    	Set-AzureSubscription -Environment AzureChinaCloud -CurrentStorageAccountName $StorageAccountName -SubscriptionName $SubscriptionName 

2.	接下来，运行 Get-AzureSubscription cmdlet，以检查该存储帐户是否与你的默认订阅帐户关联。此命令将返回当前订阅中的订阅属性，包括其当前存储帐户。

	    Get-AzureSubscription –Current

### 如何列出订阅中的所有 Azure 存储帐户
每个 Azure 订阅最多可以有 100 个存储帐户。有关限制的最新信息，请参阅 [Azure 订阅和服务限制、配额与约束](/documentation/articles/azure-subscription-service-limits/)。

运行以下 cmdlet，以找出当前订阅中存储帐户的名称和状态：

    Get-AzureStorageAccount | Format-Table -Property StorageAccountName, Location, AccountType, StorageAccountStatus

### 如何创建 Azure 存储上下文
Azure 存储上下文是 PowerShell 中用于封装存储凭据的对象。运行任何后续 cmdlet 时使用存储上下文可以对请求进行身份验证，而无需显式指定存储帐户及其访问密钥。可以通过多种方式创建存储上下文，例如，使用存储帐户名称和访问密钥、共享访问签名 (SAS) 令牌、连接字符串或匿名。有关详细信息，请参阅 [New-AzureStorageContext](http://msdn.microsoft.com/zh-cn/library/azure/dn806380.aspx)。

使用以下三种方法之一创建存储上下文：

- 运行 [Get-AzureStorageKey](http://msdn.microsoft.com/zh-cn/library/azure/dn495235.aspx) cmdlet，找出 Azure 存储帐户的主存储访问密钥。接下来，调用 [New-AzureStorageContext](http://msdn.microsoft.com/zh-cn/library/azure/dn806380.aspx) cmdlet 以创建存储上下文：

    	$StorageAccountName = "yourstorageaccount"
    	$StorageAccountKey = Get-AzureStorageKey -StorageAccountName $StorageAccountName
    	$Ctx = New-AzureStorageContext -Environment AzureChinaCloud $StorageAccountName -StorageAccountKey $StorageAccountKey.Primary


- 生成 Azure 存储容器的共享访问签名令牌，并使用它来创建存储上下文：

    	$sasToken = New-AzureStorageContainerSASToken -Container abc -Permission rl
    	$Ctx = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $StorageAccountName -SasToken $sasToken

	有关详细信息，请参阅 [New-AzureStorageContainerSASToken](http://msdn.microsoft.com/zh-cn/library/azure/dn806416.aspx) 和[共享访问签名，第 1 部分：了解 SAS 模型](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)。

- 在某些情况下，你可能想要在创建新的存储上下文时指定服务终结点。如果你已将存储帐户的自定义域名注册到 Blob 服务，或者你想要使用共享访问签名来访问存储资源，则可能需要进行这种指定。在连接字符串中设置服务终结点，并使用它来创建新的存储上下文，如下所示：

    	$ConnectionString = "DefaultEndpointsProtocol=http;BlobEndpoint=<blobEndpoint>;QueueEndpoint=<QueueEndpoint>;TableEndpoint=<TableEndpoint>;AccountName=<AccountName>;AccountKey=<AccountKey>"
    	$Ctx = New-AzureStorageContext -Environment AzureChinaCloud -ConnectionString $ConnectionString

有关如何配置存储连接字符串的详细信息，请参阅[配置连接字符串](/documentation/articles/storage-configure-connection-string/)。

现在，你已设置计算机并已了解如何使用 Azure PowerShell 管理订阅和存储帐户。请转到下一部分，了解如何管理 Azure Blob 和 Blob 快照。

## 如何管理 Azure blob
Azure Blob 存储是用于存储大量非结构化数据（例如文本或二进制数据）的服务，这些数据可通过 HTTP 或 HTTPS 从世界各地进行访问。本部分假设你已熟悉 Azure Blob 存储服务的概念。有关详细信息，请参阅[如何通过 .NET 使用 Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/)和[Blob 服务概念](http://msdn.microsoft.com/zh-cn/library/azure/dd179376.aspx)。

### 如何创建容器
Azure 存储空间中的每个 Blob 都必须在容器中。你可以使用 New-AzureStorageContainer cmdlet 创建专用容器：

    $StorageContainerName = "yourcontainername"
    New-AzureStorageContainer -Name $StorageContainerName -Permission Off

> [AZURE.NOTE] 有三种级别的匿名读取访问权限：**Off**、**Blob** 和 **Container**。若要防止对 Blob 进行匿名访问，请将 Permission 参数设置为 **Off**。默认情况下，新容器是专用容器，只能由帐户所有者访问。若要允许对 Blob 资源进行匿名公共读取访问，但不允许访问容器元数据或容器中的 Blob 列表，请将 Permission 参数设置为 **Blob**。若要允许对 Blob 资源、容器元数据和容器中的 Blob 列表进行完全公开读取访问，请将 Permission 参数设置为 **Container**。有关详细信息，请参阅[管理对容器和 Blob 的匿名读取访问](/documentation/articles/storage-manage-access-to-resources/)。

### 如何将 Blob 上载到容器
Azure Blob 存储支持块 Blob 和页 Blob。有关详细信息，请参阅[了解块 Blob、追加 Blob 和页 Blob](http://msdn.microsoft.com/zh-cn/library/azure/ee691964.aspx)。

若要将 Blob 上载到容器，可以使用 [Set-AzureStorageBlobContent](http://msdn.microsoft.com/zh-cn/library/azure/dn806379.aspx) cmdlet。默认情况下，此命令会将本地文件上载到块 Blob。若要指定 Blob 的类型，可以使用 -BlobType 参数。

以下示例将运行 [Get-ChildItem](http://technet.microsoft.com/zh-cn/library/hh849800.aspx) cmdlet 以获取指定文件夹中的所有文件，然后，通过使用管道运算符将这些文件传递到下一个 cmdlet。[Set-AzureStorageBlobContent](http://msdn.microsoft.com/zh-cn/library/azure/dn806379.aspx) cmdlet 将本地文件上载到容器：

    Get-ChildItem –Path C:\Images* | Set-AzureStorageBlobContent -Container "yourcontainername"

### 如何从容器下载 Blob
以下示例演示如何从容器下载 Blob。该示例首先使用存储帐户上下文（包括存储帐户名称及其主访问密钥）与 Azure 存储空间建立连接。然后，该示例将使用 [Get-AzureStorageBlob](http://msdn.microsoft.com/zh-cn/library/azure/dn806392.aspx) cmdlet 检索 Blob 引用。接下来，该示例使用 [Get-AzureStorageBlobContent](http://msdn.microsoft.com/zh-cn/library/azure/dn806418.aspx) cmdlet 将 Blob 下载到本地目标文件夹。

    #Define the variables.
    $ContainerName = "yourcontainername"
    $DestinationFolder = "C:\DownloadImages"

    #Define the storage account and context.
    $StorageAccountName = "yourstorageaccount"
    $StorageAccountKey = "Storage key for yourstorageaccount ends with =="
    $Ctx = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

    #List all blobs in a container.
    $blobs = Get-AzureStorageBlob -Container $ContainerName -Context $Ctx

    #Download blobs from a container.
    New-Item -Path $DestinationFolder -ItemType Directory -Force
    $blobs | Get-AzureStorageBlobContent -Destination $DestinationFolder -Context $Ctx

### 如何将 Blob 复制到另一个存储容器
你可以跨存储帐户和区域异步复制 Blob。以下示例演示如何在两个不同的存储帐户，将一个存储容器中的 Blob 复制到另一个存储容器。该示例首先设置源和目标存储帐户的变量，然后创建每个帐户的存储上下文。接下来，该示例使用 [Start-AzureStorageBlobCopy](http://msdn.microsoft.com/zh-cn/library/azure/dn806394.aspx) cmdlet 将 Blob 从源容器复制到目标容器。该示例假设源存储帐户、目标存储帐户和容器已存在。

    #Define the source storage account and context.
    $SourceStorageAccountName = "yoursourcestorageaccount"
    $SourceStorageAccountKey = "Storage key for yoursourcestorageaccount"
    $SrcContainerName = "yoursrccontainername"
    $SourceContext = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $SourceStorageAccountName -StorageAccountKey $SourceStorageAccountKey

    #Define the destination storage account and context.
    $DestStorageAccountName = "yourdeststorageaccount"
    $DestStorageAccountKey = "Storage key for yourdeststorageaccount"
    $DestContainerName = "destcontainername"
    $DestContext = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $DestStorageAccountName -StorageAccountKey $DestStorageAccountKey

    #Get a reference to blobs in the source container.
    $blobs = Get-AzureStorageBlob -Container $SrcContainerName -Context $SourceContext

    #Copy blobs from one container to another.
    $blobs| Start-AzureStorageBlobCopy -DestContainer $DestContainerName -DestContext $DestContext

请注意，此示例将执行异步复制。可以通过运行 [Get-AzureStorageBlobCopyState](http://msdn.microsoft.com/zh-cn/library/azure/dn806406.aspx) cmdlet 来监视每个副本的状态。

### 如何从辅助位置复制 Blob
您可以从一个已启用 RA-GRS 帐户的辅助位置复制 Blob。

    #define secondary storage context using a connection string constructed from secondary endpoints. 
    $SrcContext = New-AzureStorageContext -Environment AzureChinaCloud -ConnectionString "DefaultEndpointsProtocol=https;AccountName=***;AccountKey=***;BlobEndpoint=http://***-secondary.blob.core.chinacloudapi.cn;FileEndpoint=http://***-secondary.file.core.chinacloudapi.cn;QueueEndpoint=http://***-secondary.queue.core.chinacloudapi.cn; TableEndpoint=http://***-secondary.table.core.chinacloudapi.cn;"
    Start-AzureStorageBlobCopy –Container *** -Blob *** -Context $SrcContext –DestContainer *** -DestBlob *** -DestContext $DestContext

### 如何删除 Blob
若要删除 Blob，首先需要获取 Blob 引用，然后对该引用调用 Remove-AzureStorageBlob cmdlet。以下示例将删除给定容器中的所有 Blob。该示例首先设置存储帐户的变量，然后创建存储上下文。接下来，该示例使用 [Get-AzureStorageBlob](http://msdn.microsoft.com/zh-cn/library/azure/dn806392.aspx) cmdlet 检索 Blob 引用，然后运行 [Remove-AzureStorageBlob](http://msdn.microsoft.com/zh-cn/library/azure/dn806399.aspx) cmdlet 来删除 Azure 存储空间内某个容器中的 Blob。

    #Define the storage account and context.
    $StorageAccountName = "yourstorageaccount"
    $StorageAccountKey = "Storage key for yourstorageaccount ends with =="
    $ContainerName = "containername"
    $Ctx = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

    #Get a reference to all the blobs in the container.
    $blobs = Get-AzureStorageBlob -Container $ContainerName -Context $Ctx

    #Delete blobs in a specified container.
    $blobs| Remove-AzureStorageBlob

## 如何管理 Azure Blob 快照
Azure 允许你创建 Blob 的快照。快照是在某一时间点拍摄的只读版本的 Blob。在创建快照后，可以读取、复制或删除该快照，但无法对其进行修改。利用快照，你可以在某个时间点备份 Blob。有关详细信息，请参阅[创建 Blob 的快照](http://msdn.microsoft.com/zh-cn/library/azure/hh488361.aspx)。

### 如何创建 Blob 快照
若要创建 Blob 的快照，首先需要获取 Blob 引用，然后对其调用 `ICloudBlob.CreateSnapshot` 方法。以下示例首先设置存储帐户的变量，然后创建存储上下文。接下来，该示例使用 [Get-AzureStorageBlob](http://msdn.microsoft.com/zh-cn/library/azure/dn806392.aspx) cmdlet 检索 Blob 引用，然后运行 [ICloudBlob.CreateSnapshot](http://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.blob.icloudblob.aspx) 方法来创建一个快照。

    #Define the storage account and context.
    $StorageAccountName = "yourstorageaccount"
    $StorageAccountKey = "Storage key for yourstorageaccount ends with =="
    $ContainerName = "yourcontainername"
    $BlobName = "yourblobname"
    $Ctx = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

    #Get a reference to a blob.
    $blob = Get-AzureStorageBlob -Context $Ctx -Container $ContainerName -Blob $BlobName

    #Create a snapshot of the blob.
    $snap = $blob.ICloudBlob.CreateSnapshot()

### 如何列出 Blob 的快照
你可以根据需要为 Blob 创建任意数目的快照。可以列出与 Blob 关联的快照，以跟踪当前快照。以下示例使用预定义的 Blob，并调用 [Get-AzureStorageBlob](http://msdn.microsoft.com/zh-cn/library/azure/dn806392.aspx) cmdlet 列出该 Blob 的快照。

    #Define the blob name.
    $BlobName = "yourblobname"

    #List the snapshots of a blob.
    Get-AzureStorageBlob –Context $Ctx -Prefix $BlobName -Container $ContainerName  | Where-Object  { $_.ICloudBlob.IsSnapshot -and $_.Name -eq $BlobName }

### 如何复制 Blob 的快照
你可以复制 Blob 的快照以还原快照。有关详细信息和限制，请参阅[创建 Blob 的快照](http://msdn.microsoft.com/zh-cn/library/azure/hh488361.aspx)。以下示例首先设置存储帐户的变量，然后创建存储上下文。接下来，该示例将定义容器和 Blob 的名称变量。该示例使用 [Get-AzureStorageBlob](http://msdn.microsoft.com/zh-cn/library/azure/dn806392.aspx) cmdlet 检索 Blob 引用，然后运行 [ICloudBlob.CreateSnapshot](http://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.blob.icloudblob.aspx) 方法来创建一个快照。然后，该示例运行 [Start-AzureStorageBlobCopy](http://msdn.microsoft.com/zh-cn/library/azure/dn806394.aspx) cmdlet，以使用源 Blob 的 ICloudBlob 对象复制 Blob 的快照。在运行该示例之前，请务必根据你的配置更新变量。请注意，以下示例假设源容器、目标容器和源 Blob 已存在。

    #Define the storage account and context.
    $StorageAccountName = "yourstorageaccount"
    $StorageAccountKey = "Storage key for yourstorageaccount ends with =="
    $Ctx = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

    #Define the variables.
    $SrcContainerName = "yoursourcecontainername"
    $DestContainerName = "yourdestcontainername"
    $SrcBlobName = "yourblobname"
    $DestBlobName = "CopyBlobName"

    #Get a reference to a blob.
    $blob = Get-AzureStorageBlob -Context $Ctx -Container $SrcContainerName -Blob $SrcBlobName

    #Create a snapshot of a blob.
    $snap = $blob.ICloudBlob.CreateSnapshot()

    #Copy the snapshot to another container.
    Start-AzureStorageBlobCopy –Context $Ctx -ICloudBlob $snap -DestBlob $DestBlobName -DestContainer $DestContainerName

现在，你已了解如何使用 Azure PowerShell 管理 Azure Blob 和 Blob 快照。请转到下一部分，了解如何管理表、队列和文件。

## 如何管理 Azure 表和表实体
Azure 表存储服务是一种 NoSQL 数据存储，可用于存储和查询大量的结构化非关系型数据。该服务的主要组件包括表、实体和属性。表是实体的集合。实体是一组属性。每个实体最多可以有 252 个属性（都是一些名称-值对）。本部分假设你已熟悉 Azure 表存储服务的概念。有关详细信息，请参阅[了解表服务数据模型](http://msdn.microsoft.com/zh-cn/library/azure/dd179338.aspx)和[通过 .NET 开始使用 Azure 表存储](/documentation/articles/storage-dotnet-how-to-use-tables/)。

在以下小节中，你将了解如何使用 Azure PowerShell 管理 Azure 表存储服务。涉及的情景包括“创建”、“删除”和“检索”表，以及“添加”、“查询”和“删除”表实体。

### 如何创建表
每个表必须驻留在 Azure 存储帐户中。以下示例演示如何在 Azure 存储空间中创建一个表。该示例首先使用存储帐户上下文（包括存储帐户名称及其访问密钥）与 Azure 存储空间建立连接。接下来，它将使用 [New-AzureStorageTable](http://msdn.microsoft.com/zh-cn/library/azure/dn806417.aspx) cmdlet 在 Azure 存储空间中创建表。

    #Define the storage account and context.
    $StorageAccountName = "yourstorageaccount"
    $StorageAccountKey = "Storage key for yourstorageaccount ends with =="
    $Ctx = New-AzureStorageContext -Environment AzureChinaCloud $StorageAccountName -StorageAccountKey $StorageAccountKey 

    #Create a new table.
    $tabName = "yourtablename"
    New-AzureStorageTable –Name $tabName –Context $Ctx

### 如何检索表
你可以查询和检索存储帐户中的一个或所有表。以下示例演示如何使用 [Get-AzureStorageTable](http://msdn.microsoft.com/zh-cn/library/azure/dn806411.aspx) cmdlet 检索给定的表。

    #Retrieve a table.
    $tabName = "yourtablename"
    Get-AzureStorageTable –Name $tabName –Context $Ctx

如果调用不带任何参数的 Get-AzureStorageTable cmdlet，则会获取存储帐户的所有存储表。

### 如何删除表
可以使用 [Remove-AzureStorageTable](http://msdn.microsoft.com/zh-cn/library/azure/dn806393.aspx) cmdlet 从存储帐户中删除表。

    #Delete a table.
    $tabName = "yourtablename"
    Remove-AzureStorageTable –Name $tabName –Context $Ctx

### 如何管理表实体
目前，Azure PowerShell 未直接提供用于管理表实体的 cmdlet。若要对表实体执行操作，可以使用[用于 .NET 的 Azure 存储客户端库](http://msdn.microsoft.com/zh-cn/library/azure/wa_storage_30_reference_home.aspx)中提供的类。

#### 如何添加表实体
若要将实体添加到表中，请先创建用于定义实体属性的对象。一个实体最多可以有 255 个属性，包括 3 个系统属性：**PartitionKey**、**RowKey** 和 **Timestamp**。你需要负责插入和更新 **PartitionKey** 与 **RowKey** 的值。服务器将管理 **Timestamp** 的值，该值不可修改。**PartitionKey** 和 **RowKey** 共同唯一标识表中的每个实体。

-	**PartitionKey**：确定实体存储在其中的分区。
-	**RowKey**：唯一标识分区内的实体。

最多可为一个实体定义 252 个自定义属性。有关详细信息，请参阅[了解表服务数据模型](http://msdn.microsoft.com/zh-cn/library/azure/dd179338.aspx)。

以下示例演示如何向表中添加实体。该示例演示如何检索员工表，并将多个实体添加到其中。该示例首先使用存储帐户上下文（包括存储帐户名称及其访问密钥）与 Azure 存储空间建立连接。接下来，使用 [Get-AzureStorageTable](http://msdn.microsoft.com/zh-cn/library/azure/dn806411.aspx) cmdlet 检索给定的表。如果该表不存在，则使用 [New-AzureStorageTable](http://msdn.microsoft.com/zh-cn/library/azure/dn806417.aspx) cmdlet 在 Azure 存储空间中创建一个表。接下来，该示例将定义一个自定义函数 Add-Entity，以通过指定每个实体的分区和行键，将实体添加到表中。Add-Entity 函数对 [Microsoft.WindowsAzure.Storage.Table.DynamicTableEntity](http://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.table.dynamictableentity.aspx) 类调用 [New-Object](http://technet.microsoft.com/zh-cn/library/hh849885.aspx) cmdlet，以创建实体对象。随后，该示例对此实体对象调用 [Microsoft.WindowsAzure.Storage.Table.TableOperation.Insert](http://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.table.tableoperation.insert.aspx) 方法，以将它添加到表中。

    #Function Add-Entity: Adds an employee entity to a table.
    function Add-Entity() {
        [CmdletBinding()]
        param(
           $table,
           [String]$partitionKey,
           [String]$rowKey,
           [String]$name,
           [Int]$id
        )

      $entity = New-Object -TypeName Microsoft.WindowsAzure.Storage.Table.DynamicTableEntity -ArgumentList $partitionKey, $rowKey
      $entity.Properties.Add("Name", $name)
      $entity.Properties.Add("ID", $id)

      $result = $table.CloudTable.Execute([Microsoft.WindowsAzure.Storage.Table.TableOperation]::Insert($entity))
    }

    #Define the storage account and context.
    $StorageAccountName = "yourstorageaccount"
    $StorageAccountKey = Get-AzureStorageKey -StorageAccountName $StorageAccountName
    $Ctx = New-AzureStorageContext -Environment AzureChinaCloud $StorageAccountName -StorageAccountKey $StorageAccountKey.Primary
    $TableName = "Employees"

    #Retrieve the table if it already exists.
    $table = Get-AzureStorageTable –Name $TableName -Context $Ctx -ErrorAction Ignore

    #Create a new table if it does not exist.
    if ($table -eq $null)
    {
       $table = New-AzureStorageTable –Name $TableName -Context $Ctx
    }

    #Add multiple entities to a table.
    Add-Entity -Table $table -PartitionKey Partition1 -RowKey Row1 -Name Chris -Id 1
    Add-Entity -Table $table -PartitionKey Partition1 -RowKey Row2 -Name Jessie -Id 2
    Add-Entity -Table $table -PartitionKey Partition2 -RowKey Row1 -Name Christine -Id 3
    Add-Entity -Table $table -PartitionKey Partition2 -RowKey Row2 -Name Steven -Id 4

#### 如何查询表实体
若要查询表，请使用 [Microsoft.WindowsAzure.Storage.Table.TableQuery](http://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.table.tablequery.aspx) 类。以下示例假设你已运行本指南的“如何添加实体”部分中指定的脚本。该示例首先使用存储上下文（包括存储帐户名称及其访问密钥）与 Azure 存储空间建立连接。接下来，它将尝试使用 [Get-AzureStorageTable](http://msdn.microsoft.com/zh-cn/library/azure/dn806411.aspx) cmdlet 检索前面创建的“Employees”表。对 Microsoft.WindowsAzure.Storage.Table.TableQuery 类调用 [New-Object](http://technet.microsoft.com/zh-cn/library/hh849885.aspx) cmdlet 会创建一个新的查询对象。该示例将查找字符串筛选器中指定的“ID”列值为 1 的实体。有关详细信息，请参阅[查询表和实体](http://msdn.microsoft.com/zh-cn/library/azure/dd894031.aspx)。当你运行此查询时，它将返回与筛选条件匹配的所有实体。

    #Define the storage account and context.
    $StorageAccountName = "yourstorageaccount"
    $StorageAccountKey = Get-AzureStorageKey -StorageAccountName $StorageAccountName
    $Ctx = New-AzureStorageContext -Environment AzureChinaCloud –StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey.Primary;
    $TableName = "Employees"

    #Get a reference to a table.
    $table = Get-AzureStorageTable –Name $TableName -Context $Ctx

    #Create a table query.
    $query = New-Object Microsoft.WindowsAzure.Storage.Table.TableQuery

    #Define columns to select.
    $list = New-Object System.Collections.Generic.List[string]
    $list.Add("RowKey")
    $list.Add("ID")
    $list.Add("Name")

    #Set query details.
    $query.FilterString = "ID gt 0"
    $query.SelectColumns = $list
    $query.TakeCount = 20

    #Execute the query.
    $entities = $table.CloudTable.ExecuteQuery($query)

    #Display entity properties with the table format.
    $entities  | Format-Table PartitionKey, RowKey, @{ Label = "Name"; Expression={$_.Properties["Name"].StringValue}}, @{ Label = "ID"; Expression={$_.Properties[“ID”].Int32Value}} -AutoSize

#### 如何删除表实体
可以使用实体的分区键和行键删除实体。以下示例假设你已运行本指南的“如何添加实体”部分中指定的脚本。该示例首先使用存储上下文（包括存储帐户名称及其访问密钥）与 Azure 存储空间建立连接。接下来，它将尝试使用 [Get-AzureStorageTable](http://msdn.microsoft.com/zh-cn/library/azure/dn806411.aspx) cmdlet 检索前面创建的“Employees”表。如果该表存在，该示例将调用 [Microsoft.WindowsAzure.Storage.Table.TableOperation.Retrieve](http://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.table.tableoperation.retrieve.aspx) 方法，以其于实体的分区键和行键值来检索实体。然后，将该实体传递到 [Microsoft.WindowsAzure.Storage.Table.TableOperation.Delete](http://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.table.tableoperation.delete.aspx) 方法以将其删除。

    #Define the storage account and context.
    $StorageAccountName = "yourstorageaccount"
    $StorageAccountKey = Get-AzureStorageKey -StorageAccountName $StorageAccountName
    $Ctx = New-AzureStorageContext -Environment AzureChinaCloud –StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey.Primary

    #Retrieve the table.
    $TableName = "Employees"
    $table = Get-AzureStorageTable -Name $TableName -Context $Ctx -ErrorAction Ignore

    #If the table exists, start deleting its entities.
    if ($table -ne $null) {
       #Together the PartitionKey and RowKey uniquely identify every  
       #entity within a table.
       $tableResult = $table.CloudTable.Execute([Microsoft.WindowsAzure.Storage.Table.TableOperation]::Retrieve(“Partition2”, "Row1"))
       $entity = $tableResult.Result;
    if ($entity -ne $null)
    {
       #Delete the entity.$table.CloudTable.Execute([Microsoft.WindowsAzure.Storage.Table.TableOperation]::Delete($entity))
    }
    }

## 如何管理 Azure 队列和队列消息
Azure 队列存储是一项可存储大量消息的服务，用户可以通过经验证的呼叫，使用 HTTP 或 HTTPS 从世界任何地方访问这些消息。本部分假设你已熟悉 Azure 队列存储服务的概念。有关详细信息，请参阅[通过 .NET 开始使用 Azure 队列存储](/documentation/articles/storage-dotnet-how-to-use-queues/)。

本部分将说明如何使用 Azure PowerShell 管理 Azure 队列存储服务。涉及的情景包括“插入”和“删除”队列消息，以及“创建”、“删除”和“检索队列”。

### 如何创建队列
以下示例首先使用存储帐户上下文（包括存储帐户名称及其访问密钥）与 Azure 存储空间建立连接。接下来，它将调用 [New-AzureStorageQueue](http://msdn.microsoft.com/zh-cn/library/azure/dn806382.aspx) cmdlet 以创建名为“queuename”的队列。

    #Define the storage account and context.
    $StorageAccountName = "yourstorageaccount"
    $StorageAccountKey = Get-AzureStorageKey -StorageAccountName $StorageAccountName
    $Ctx = New-AzureStorageContext -Environment AzureChinaCloud –StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey.Primary
    $QueueName = "queuename"
    $Queue = New-AzureStorageQueue –Name $QueueName -Context $Ctx

命名 Azure 队列服务命名约定的信息，请参阅[命名队列和元数据](http://msdn.microsoft.com/zh-cn/library/azure/dd179349.aspx)。

### 如何检索队列
你可以查询和检索存储帐户中的特定队列，或者所有队列的列表。以下示例演示如何使用 [Get-AzureStorageQueue](http://msdn.microsoft.com/zh-cn/library/azure/dn806377.aspx) cmdlet 检索指定的队列。

    #Retrieve a queue.
    $QueueName = "queuename"
    $Queue = Get-AzureStorageQueue –Name $QueueName –Context $Ctx

如果您调用不带任何参数的 [Get AzureStorageQueue](http://msdn.microsoft.com/zh-cn/library/azure/dn806377.aspx) cmdlet，它可获取所有队列的列表。

### 如何删除队列
若要删除队列及其中包含的所有消息，请调用 Remove-AzureStorageQueue cmdlet。以下示例演示如何使用 Remove-AzureStorageQueue cmdlet 删除指定的队列。

    #Delete a queue.
    $QueueName = "yourqueuename"
    Remove-AzureStorageQueue –Name $QueueName –Context $Ctx

#### 如何在队列中插入消息
若要在现有队列中插入一条消息，请先创建 [Microsoft.WindowsAzure.Storage.Queue.CloudQueueMessage](http://msdn.microsoft.com/zh-cn/library/azure/jj732474.aspx) 类的新实例。接下来，调用 [AddMessage](http://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.queue.cloudqueue.addmessage.aspx) 方法。可从字符串（UTF-8 格式）或字节数组创建 CloudQueueMessage。

以下示例演示如何向队列中添加消息。该示例首先使用存储帐户上下文（包括存储帐户名称及其访问密钥）与 Azure 存储空间建立连接。接下来，它将使用 [Get-AzureStorageQueue](https://msdn.microsoft.com/zh-cn/library/azure/dn806377.aspx) cmdlet 检索指定的队列。如果该队列存在，则使用 [New-Object](http://technet.microsoft.com/zh-cn/library/hh849885.aspx) cmdlet 创建 [Microsoft.WindowsAzure.Storage.Queue.CloudQueueMessage](http://msdn.microsoft.com/zh-cn/library/azure/jj732474.aspx) 类的实例。随后，该示例对此消息对象调用 [AddMessage](http://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.queue.cloudqueue.addmessage.aspx) 方法，以将它添加到队列中。以下是检索队列并插入消息“MessageInfo”的代码：

    #Define the storage account and context.
    $StorageAccountName = "yourstorageaccount"
    $StorageAccountKey = Get-AzureStorageKey -StorageAccountName $StorageAccountName
    $Ctx = New-AzureStorageContext -Environment AzureChinaCloud –StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey.Primary

    #Retrieve the queue.
    $QueueName = "queuename"
    $Queue = Get-AzureStorageQueue -Name $QueueName -Context $ctx

    #If the queue exists, add a new message.
    if ($Queue -ne $null) {
       # Create a new message using a constructor of the CloudQueueMessage class.
       $QueueMessage = New-Object -TypeName Microsoft.WindowsAzure.Storage.Queue.CloudQueueMessage -ArgumentList MessageInfo

       #Add a new message to the queue.
       $Queue.CloudQueue.AddMessage($QueueMessage)
    }


#### 如何取消下一条消息的排队
你的代码通过两个步骤来取消对队列中某条消息的排队。当你调用 [Microsoft.WindowsAzure.Storage.Queue.CloudQueue.GetMessage](http://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.queue.cloudqueue.getmessage.aspx) 方法时，将获取队列中的下一条消息。从 **GetMessage** 返回的消息变得对从此队列读取消息的任何其他代码不可见。若要完成从队列中删除消息，还必须调用 [Microsoft.WindowsAzure.Storage.Queue.CloudQueue.DeleteMessage](http://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.queue.cloudqueue.deletemessage.aspx) 方法。此删除消息的两步过程可确保，如果你的代码因硬件或软件故障而无法处理消息，则你的代码的其他实例可以获取相同消息并重试。你的代码在处理消息后会立即调用 **DeleteMessage**。

    #Define the storage account and context.
    $StorageAccountName = "yourstorageaccount"
    $StorageAccountKey = Get-AzureStorageKey -StorageAccountName $StorageAccountName
    $Ctx = New-AzureStorageContext -Environment AzureChinaCloud –StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey.Primary

    #Retrieve the queue.
    $QueueName = "queuename"
    $Queue = Get-AzureStorageQueue -Name $QueueName -Context $ctx

    $InvisibleTimeout = [System.TimeSpan]::FromSeconds(10)

    #Get the message object from the queue.
    $QueueMessage = $Queue.CloudQueue.GetMessage($InvisibleTimeout)
    #Delete the message.
    $Queue.CloudQueue.DeleteMessage($QueueMessage)

## 如何管理 Azure 文件共享和文件
Azure 文件存储使用标准 SMB 协议为应用程序提供共享存储。Azure 虚拟机和云服务可以通过装入的共享在应用程序组件之间共享文件数据，本地应用程序可以通过文件存储 API 或 Azure PowerShell 访问共享中的文件数据。

有关 Azure 文件存储的详细信息，请参阅[在 Windows 上开始使用 Azure 文件存储](/documentation/articles/storage-dotnet-how-to-use-files/)和[文件服务 REST API](http://msdn.microsoft.com/zh-cn/library/azure/dn167006.aspx)。

## 如何设置和查询存储分析
可使用 [Azure 存储分析](/documentation/articles/storage-analytics/)从 Azure 存储帐户收集度量值，并记录与发送到存储帐户的请求有关的数据。你可以使用存储度量值监视存储帐户的运行状况，并使用存储日志记录诊断和解决与存储帐户有关的问题。对于存储服务，默认情况下不启用存储度量值。你可以使用 Azure 管理门户或 Windows PowerShell 启用监视，也可以使用存储客户端库以编程方式启用监视。存储日志记录在服务器端执行，可用于在存储帐户中记录成功和失败请求的相关详细信息。使用这些日志，可以查看针对表、队列和 Blob 的读取、写入和删除操作的详细信息，以及请求失败的原因。

若要了解如何使用 PowerShell 启用和查看存储度量值数据，请参阅[如何使用 PowerShell 启用存储度量值](http://msdn.microsoft.com/zh-cn/library/azure/dn782843.aspx#HowtoenableStorageMetricsusingPowerShell)。

若要了解如何使用 PowerShell 启用和检索存储日志记录数据，请参阅[如何使用 PowerShell 启用存储日志记录](http://msdn.microsoft.com/zh-cn/library/azure/dn782840.aspx#HowtoenableStorageLoggingusingPowerShell)和[查找存储日志记录的日志数据](http://msdn.microsoft.com/zh-cn/library/azure/dn782840.aspx#FindingyourStorageLogginglogdata)。有关使用“存储度量值”和“存储日志记录”排查存储问题的详细信息，请参阅[对 Azure 存储空间进行监视、诊断和故障排除](/documentation/articles/storage-monitoring-diagnosing-troubleshooting/)。

## 如何管理共享访问签名 (SAS) 和存储访问策略
共享访问签名是对使用 Azure 存储空间的任何应用程序创建安全模型的重要环节。它们用于将存储帐户的受限权限提供给不应具有帐户密钥的客户端。默认情况下，只有存储帐户的所有者可访问该帐户中的 Blob、表和队列。如果服务或应用程序需要向其他客户端提供这些资源但不共享访问密钥，你可以使用三个选项：

- 设置容器的权限以允许匿名读取该容器及其 Blob。表或队列不允许这样做。
- 使用共享访问签名，它可以在特定时间间隔内授予对容器、Blob、队列和表的受限访问权限的。
- 使用存储访问策略，以更高程度地控制容器或其 Blob、队列或表的共享访问签名。使用存储访问策略可以更改签名的开始时间、到期时间或权限，或者在颁发签名后将其吊消。

共享访问签名可以采取以下两种形式的一种：

- **Ad hoc SAS**：在你创建一个临时 SAS 时，针对该 SAS 的开始时间、到期时间和权限全都在 SAS URI 上指定。可以在容器、Blob、表或队列上创建这种不可吊销的 SAS。
- **具有存储访问策略的 SAS**：存储访问策略是对资源容器（Blob 容器、表或队列）定义的，可用于管理针对一个或多个共享访问签名的约束。在你将某一 SAS 与一个存储访问策略相关联时，该 SAS 将继承对该存储访问策略定义的约束：开始时间、到期时间和权限。这种类型的 SAS 可吊销。

有关详细信息，请参阅[共享访问签名：了解 SAS 模型](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)和[管理对容器和 Blob 的匿名读取访问](/documentation/articles/storage-manage-access-to-resources/)。

在下一部分中，你将学习如何为 Azure 表创建共享访问签名令牌和存储访问策略。Azure PowerShell 为容器、Blob 和队列提供了类似的 cmdlet。若要运行本部分中的脚本，下载 [Azure PowerShell 版本 0.8.14](http://go.microsoft.com/?linkid=9811175&clcid=0x409) 或更高版本。

### 如何创建基于共享访问签名令牌的策略
使用 New-AzureStorageTableStoredAccessPolicy cmdlet 创建新的存储访问策略。然后，调用 [New-AzureStorageTableSASToken](http://msdn.microsoft.com/zh-cn/library/azure/dn806400.aspx) cmdlet 为 Azure 存储表创建新的基于策略的共享访问签名令牌。

    $policy = "policy1"
    New-AzureStorageTableStoredAccessPolicy -Name $tableName -Policy $policy -Permission "rd" -StartTime "2015-01-01" -ExpiryTime "2016-01-01" -Context $Ctx
    New-AzureStorageTableSASToken -Name $tableName -Policy $policy -Context $Ctx

### 如何创建临时（不可吊销的）共享访问签名令牌
使用 [New-AzureStorageTableSASToken](http://msdn.microsoft.com/zh-cn/library/azure/dn806400.aspx) cmdlet 为 Azure 存储表创建新的临时（不可吊销的）共享访问签名令牌：

    New-AzureStorageTableSASToken -Name $tableName -Permission "rqud" -StartTime "2015-01-01" -ExpiryTime "2015-02-01" -Context $Ctx

### 如何创建存储访问策略
使用 New-AzureStorageTableStoredAccessPolicy cmdlet 为 Azure 存储表创建新的存储访问策略：

    $policy = "policy1"
    New-AzureStorageTableStoredAccessPolicy -Name $tableName -Policy $policy -Permission "rd" -StartTime "2015-01-01" -ExpiryTime "2016-01-01" -Context $Ctx

### 如何更新存储访问策略
使用 Set-AzureStorageTableStoredAccessPolicy cmdlet 更新 Azure 存储表的现有存储访问策略：

    Set-AzureStorageTableStoredAccessPolicy -Policy $policy -Table $tableName -Permission "rd" -NoExpiryTime -NoStartTime -Context $Ctx

### 如何删除存储访问策略
使用 Remove-AzureStorageTableStoredAccessPolicy cmdlet 删除 Azure 存储表的存储访问策略：

    Remove-AzureStorageTableStoredAccessPolicy -Policy $policy -Table $tableName -Context $Ctx


## 如何在 Azure 中国区使用 Azure 存储空间
Azure 环境的部署独立于 Azure，例如[中国 21Vianet 运营的 AzureChinaCloud for Azure](http://www.windowsazure.cn/)。你可以为 Azure 中国区部署新的 Azure 环境。

若要将 Azure 存储空间用于 AzureChinaCloud，需要创建与 AzureChinaCloud 关联的存储上下文。请按照以下步骤开始：

1.	运行 [Get-AzureEnvironment](https://msdn.microsoft.com/zh-cn/library/azure/dn790368.aspx) cmdlet 以查看可用的 Azure 环境：

    `Get-AzureEnvironment`

2.	将 Azure 中国区帐户添加到 Windows PowerShell：

    `Add-AzureAccount –Environment AzureChinaCloud`

3.	为 AzureChinaCloud 帐户创建存储上下文：

    	$Ctx = New-AzureStorageContext -StorageAccountName $AccountName -StorageAccountKey $AccountKey> -Environment AzureChinaCloud


- [面向全球 Azure 的 AzureCloud 与中国 21Vianet 运营的 AzureChinaCloud 之间的差异](https://msdn.microsoft.com/zh-cn/library/azure/dn578439.aspx)

## 后续步骤
在本指南中，你已了解如何使用 Azure PowerShell 管理 Azure 存储空间。下面是一些相关的文章和了解有关这些更多的资源。

- [Azure 存储文档](/documentation/services/storage)
- [Azure 存储空间 PowerShell Cmdlet](http://msdn.microsoft.com/zh-cn/library/azure/dn806401.aspx)
- [Windows PowerShell 参考](https://msdn.microsoft.com/zh-cn/library/ms714469.aspx)

[Image1]: ./media/storage-powershell-guide-full/Subscription_currentportal.png
[Image2]: ./media/storage-powershell-guide-full/Subscription_Previewportal.png
[Image3]: ./media/storage-powershell-guide-full/Blobdownload.png


[在 5 分钟内开始使用 Azure 存储空间和 PowerShell]: #getstart
[对 Azure 存储空间使用 Azure PowerShell 的先决条件]: #pre
[如何管理 Azure 中的存储帐户]: #manageaccount
[如何设置默认的 Azure 订阅]: #setdefsub
[如何创建新的 Azure 存储帐户]: #createaccount
[如何设置默认的 Azure 存储帐户]: #defaultaccount
[如何列出订阅中的所有 Azure 存储帐户]: #listaccounts
[如何创建 Azure 存储上下文]: #createctx
[如何管理 Azure Blob 和 Blob 快照]: #manageblobs
[如何创建容器]: #container
[如何将 Blob 上载到容器]: #uploadblob
[如何从容器下载 Blob]: #downblob
[如何将 Blob 复制到另一个存储容器]: #copyblob
[如何删除 Blob]: #deleteblob
[如何管理 Azure Blob 快照]: #manageshots
[如何创建 Blob 快照]: #createshot
[如何列出 Blob 的快照]: #listshot
[如何复制 Blob 的快照]: #copyshot
[如何管理 Azure 表和表实体]: #managetables
[如何创建表]: #createtable
[如何检索表]: #gettable
[如何删除表]: #remtable
[如何管理表实体]: #mngentity
[如何添加表实体]: #addentity
[如何查询表实体]: #queryentity
[如何删除表实体]: #deleteentity
[如何管理 Azure 队列和队列消息]: #managequeues
[如何创建队列]: #createqueue
[如何检索队列]: #getqueue
[如何删除队列]: #remqueue
[如何管理队列消息]: #mngqueuemsg
[如何在队列中插入消息]: #addqueuemsg
[如何取消下一条消息的排队]: #dequeuemsg
[如何管理 Azure 文件共享和文件]: #files
[如何设置和查询存储分析]: #stganalytics
[如何管理共享访问签名 (SAS) 和存储访问策略]: #sas
[如何在美国政府部门和 Azure 中国区使用 Azure 存储空间]: #gov
[后续步骤]: #next

<!---HONumber=Mooncake_0411_2016-->