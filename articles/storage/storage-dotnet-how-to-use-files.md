<properties
    pageTitle="在 Windows 上开始使用 Azure 文件存储 | Azure"
    description="使用 Azure 文件存储在云中存储文件数据和从 Azure 虚拟机 (VM) 或从运行 Windows 的本地应用程序装载你的云文件共享。"
    services="storage"
    documentationcenter=".net"
    author="RenaShahMSFT"
    manager="aungoo"
    editor="tysonn"
    translationtype="Human Translation" />
<tags
    ms.assetid="6a889ee1-1e60-46ec-a592-ae854f9fb8b6"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="hero-article"
    ms.author="renash"
    ms.date="3/8/2017"
    wacn.date="04/24/2017"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="d763dcb062c28cf589fc0d7fa6cbcf1e73364736"
    ms.lasthandoff="04/14/2017" />

# <a name="get-started-with-azure-file-storage-on-windows"></a>在 Windows 上开始使用 Azure 文件存储
[AZURE.INCLUDE [storage-selector-file-include](../../includes/storage-selector-file-include.md)]

[AZURE.INCLUDE [storage-check-out-samples-dotnet](../../includes/storage-check-out-samples-dotnet.md)]

[AZURE.INCLUDE [storage-file-overview-include](../../includes/storage-file-overview-include.md)]

有关通过 Linux 使用文件存储的信息，请参阅 [如何通过 Linux 使用 Azure 文件存储](/documentation/articles/storage-how-to-use-files-linux/)。

有关文件存储的可伸缩性和性能目标的信息，请参阅 [Azure 存储可伸缩性和性能目标](/documentation/articles/storage-scalability-targets/#scalability-targets-for-blobs-queues-tables-and-files)。

[AZURE.INCLUDE [storage-dotnet-client-library-version-include](../../includes/storage-dotnet-client-library-version-include.md)]

[AZURE.INCLUDE [storage-file-concepts-include](../../includes/storage-file-concepts-include.md)]

## <a name="about-this-tutorial"></a>关于本教程
此入门教程演示使用 Azure 文件存储的基础知识。 在本教程中，我们将：

* 使用 Azure 门户预览或 PowerShell 创建新的 Azure 文件共享、添加目录、将本地文件上载到该共享，以及列出该目录中的文件。
* 装载文件共享，就像装载任何 SMB 共享一样。
* 使用用于.NET 的 Azure 存储空间客户端库从本地应用程序访问文件共享。 创建一个控制台应用程序并通过文件共享执行以下操作：
  * 将共享中一个文件的内容写入控制台窗口。
  * 设置文件共享的配额（最大大小）。
  * 若一个文件使用在共享中定义的共享访问策略，则为该文件创建一个共享访问签名。
  * 将文件复制到同一存储帐户中的另一个文件。
  * 将文件复制到同一存储帐户中的一个 Blob。
* 使用 Azure 存储空间度量值进行故障排除

现在所有存储帐户均支持文件存储，因此你可以使用现有存储帐户，也可以创建新的存储帐户。请参阅[如何创建存储帐户](/documentation/articles/storage-create-storage-account/#create-a-storage-account)，了解有关创建新存储帐户的信息。

## <a name="use-the-azure-portal-preview-to-manage-a-file-share"></a>使用 Azure 门户预览管理文件共享
[Azure 门户预览](https://portal.azure.cn)提供了方便客户管理文件共享的用户界面。在门户中，你可以：

* 创建文件共享
* 将文件上载到文件共享以及从文件共享下载文件。
* 监视每个文件共享的实际使用情况
* 调整共享大小配额
* 获取用于从 Windows 客户端装载文件共享的 `net use` 命令

### <a name="create-file-share"></a>创建文件共享
1. 登录 Azure 门户预览。

2. 在导航菜单中，单击“存储帐户”或“存储帐户(经典)”。

	![演示如何在门户中创建文件共享的屏幕快照](./media/storage-dotnet-how-to-use-files/files-create-share-0.png)  


3. 选择存储帐户。

	![演示如何在门户中创建文件共享的屏幕快照](./media/storage-dotnet-how-to-use-files/files-create-share-1.png)

4. 选择“文件”服务。

	![演示如何在门户中创建文件共享的屏幕快照](./media/storage-dotnet-how-to-use-files/files-create-share-2.png)

5. 单击“文件共享”，然后单击相应链接创建第一个文件共享。

	![演示如何在门户中创建文件共享的屏幕快照](./media/storage-dotnet-how-to-use-files/files-create-share-3.png)

6. 填写文件共享名称以及文件共享的大小（最高 5120 GB），以便创建你的第一个文件共享。创建文件共享之后，即可从任何支持 SMB 2.1 或 SMB 3.0 的文件系统装载它。

	![演示如何在门户中创建文件共享的屏幕快照](./media/storage-dotnet-how-to-use-files/files-create-share-4.png)  

### <a name="upload-and-download-files"></a>上传和下载文件
1. 选择一个你已创建的文件共享。

	![演示如何从门户上载和下载文件的屏幕快照](./media/storage-dotnet-how-to-use-files/files-upload-download-1.png)

2. 单击“上载”打开用于文件上载的用户界面。

	![演示如何从门户上载文件的屏幕快照](./media/storage-dotnet-how-to-use-files/files-upload-download-2.png)

3. 右键单击一个文件，然后选择“下载”将其下载到本地。

	![演示如何从门户下载文件的屏幕快照](./media/storage-dotnet-how-to-use-files/files-upload-download-3.png)

### <a name="manage-file-share"></a>管理文件共享
1. 单击“配额”更改文件共享大小（高达 5120 GB）。

	![演示如何配置文件共享配额的屏幕快照](./media/storage-dotnet-how-to-use-files/files-manage-1.png)  


2. 单击“连接”获取从 Windows 装载文件共享的命令行。

	![演示如何装载文件共享的屏幕快照](./media/storage-dotnet-how-to-use-files/files-manage-2.png)  


	![演示如何装载文件共享的屏幕快照](./media/storage-dotnet-how-to-use-files/files-manage-3.png)

	>[AZURE.TIP] 若要查找用于装载的存储帐户访问密钥，请先单击存储帐户的“设置”，然后单击“访问密钥”。

	![演示如何查找存储帐户访问密钥的屏幕快照](./media/storage-dotnet-how-to-use-files/files-manage-4.png)

	![演示如何查找存储帐户访问密钥的屏幕快照](./media/storage-dotnet-how-to-use-files/files-manage-5.png)

## <a name="use-powershell-to-manage-a-file-share"></a>使用 PowerShell 管理文件共享
也可以使用 Azure PowerShell 来创建和管理文件共享。

### <a name="install-the-powershell-cmdlets-for-azure-storage"></a>为 Azure 存储空间安装 PowerShell cmdlet
若要准备使用 PowerShell，请下载并安装 Azure PowerShell cmdlet。 有关安装点和安装说明，请参阅 [如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs) 。

> [AZURE.NOTE] 建议你下载并安装最新的 Azure PowerShell 模块或升级到最新模块。

通过单击“开始”并键入 **Windows PowerShell** 打开 Azure PowerShell 窗口。 PowerShell 窗口将为你加载 Azure PowerShell 模块。

### <a name="create-a-context-for-your-storage-account-and-key"></a>为存储帐户和密钥创建上下文
现在，将创建存储帐户上下文。该上下文封装了存储帐户名称和帐户密钥。有关从[Azure 门户预览](https://portal.azure.cn)复制帐户密钥的说明，请参阅[查看和复制存储访问密钥](/documentation/articles/storage-create-storage-account/#view-and-copy-storage-access-keys)。

请将下面示例中的 `storage-account-name` 和 `storage-account-key` 替换为你的存储帐户名称和密钥。

	# create a context for account and key
	$ctx=New-AzureStorageContext -Environment AzureChinaCloud storage-account-name storage-account-key

### <a name="create-a-new-file-share"></a>创建新的文件共享
接下来，创建名为 `logs`的新共享。

	# create a new share
	$s = New-AzureStorageShare logs -Context $ctx

现在，你在文件存储中已有一个文件共享。接下来，我们将添加目录和文件。

> [AZURE.IMPORTANT] 文件共享的名称必须是全部小写。有关命名文件共享和文件的完整详细信息，请参阅[命名和引用共享、目录、文件和元数据](https://msdn.microsoft.com/zh-cn/library/azure/dn167011.aspx)。


### <a name="create-a-directory-in-the-file-share"></a>在文件共享中创建目录
接下来，将在共享中创建目录。 在下面的示例中，目录名为 `CustomLogs`。

    # create a directory in the share
    New-AzureStorageDirectory -Share $s -Path CustomLogs

### <a name="upload-a-local-file-to-the-directory"></a>将本地文件上传到目录
现在，将本地文件上载到该目录。 以下示例从 `C:\temp\Log1.txt`上载文件。 请编辑文件路径，使其指向你本地计算机上的有效文件。

    # upload a local file to the new directory
    Set-AzureStorageFileContent -Share $s -Source C:\temp\Log1.txt -Path CustomLogs

### <a name="list-the-files-in-the-directory"></a>列出目录中的文件
若要查看目录中的文件，你可以列出目录的所有文件。 此命令将返回 CustomLogs 目录中的文件和子目录（如果有的话）。

    # list files in the new directory
    Get-AzureStorageFile -Share $s -Path CustomLogs | Get-AzureStorageFile

Get-AzureStorageFile 将返回任何传入的目录对象的文件和目录列表。“Get-AzureStorageFile -Share $s”将返回根目录中的文件和目录列表。若要获取子目录中的文件列表，必须将子目录传递给 Get AzureStorageFile。这就是此功能的作用 -- 到达管道的命令的第一部分将返回子目录 CustomLogs 的目录实例。然后，该实例将传递到 Get-AzureStorageFile，从而返回 CustomLogs 中的文件和目录。

### <a name="copy-files"></a>复制文件
从 Azure PowerShell 的 0.9.7 版开始，可以将一个文件复制到另一个文件，将一个文件复制到一个 Blob，或将一个 Blob 复制到一个文件。 下面，我们演示如何使用 PowerShell cmdlet 执行这些复制操作。

    # copy a file to the new directory
    Start-AzureStorageFileCopy -SrcShareName srcshare -SrcFilePath srcdir/hello.txt -DestShareName destshare -DestFilePath destdir/hellocopy.txt -Context $srcCtx -DestContext $destCtx

    # copy a blob to a file directory
    Start-AzureStorageFileCopy -SrcContainerName srcctn -SrcBlobName hello2.txt -DestShareName hello -DestFilePath hellodir/hello2copy.txt -DestContext $ctx -Context $ctx

## <a name="mount-the-file-share"></a>装载文件共享
由于支持 SMB 3.0，文件存储现在支持加密和持久保留 SMB 3.0 客户端中的句柄。 支持加密意味着 SMB 3.0 客户端可以从任意位置装载文件共享，这些位置包括：

* 同一区域中的 Azure 虚拟机（SMB 2.1 也支持）
* 不同区域中的 Azure 虚拟机（仅适用于 SMB 3.0）
* 本地客户端应用程序（仅适用于 SMB 3.0）

当客户端访问文件存储时，所使用的 SMB 版本取决于操作系统支持的 SMB 版本。 下表提供了有关 Windows 客户端支持的摘要。 有关 [SMB 版本](http://blogs.technet.com/b/josebda/archive/2013/10/02/windows-server-2012-r2-which-version-of-the-smb-protocol-smb-1-0-smb-2-0-smb-2-1-smb-3-0-or-smb-3-02-you-are-using.aspx)的更多详细信息，请参阅此博客。

| Windows 客户端 | 支持的 SMB 版本 |
|:--- |:--- |
| Windows 7 |SMB 2.1 |
| Windows Server 2008 R2 |SMB 2.1 |
| Windows 8 |SMB 3.0 |
| Windows Server 2012 |SMB 3.0 |
| Windows Server 2012 R2 |SMB 3.0 |
| Windows 10 |SMB 3.0 |

### <a name="mount-the-file-share-from-an-azure-virtual-machine-running-windows"></a>从运行 Windows 的 Azure 虚拟机装载文件共享
为了演示如何装载 Azure 文件共享，现在我们将创建一个运行 Windows 的 Azure 虚拟机，并远程登录到它内部以装载共享。

1. 首先，按照[在 Azure 门户预览中创建 Windows 虚拟机](/documentation/articles/virtual-machines-windows-hero-tutorial/)中的说明创建新的 Azure 虚拟机。
2. 接下来，按照[使用 Azure 门户预览登录到 Windows 虚拟机](/documentation/articles/virtual-machines-windows-connect-logon/)中的说明远程登录到虚拟机。
3. 在该虚拟机上打开 PowerShell 窗口。

### <a name="persist-your-storage-account-credentials-for-the-virtual-machine"></a>保存虚拟机的存储帐户凭据
装载到文件共享之前，先在虚拟机上保存存储帐户凭据。 当虚拟机重新启动时，此步骤允许 Windows 自动重新连接到文件共享。 若要持久保存帐户凭据，请在虚拟机上的 PowerShell 窗口中运行 `cmdkey` 命令。 请将 `<storage-account-name>` 替换为你的存储帐户名称，将 `<storage-account-key>` 替换为你的存储帐户密钥。 必须显式指定域“AZURE”，如以下示例所示。 

    cmdkey /add:<storage-account-name>.file.core.chinacloudapi.cn /user:AZURE\<storage-account-name> /pass:<storage-account-key>

现在，当虚拟机重新启动时，Windows 将重新连接到你的文件共享。 可以通过在 PowerShell 窗口中运行 `net use` 命令来验证是否已重新连接共享。

请注意，凭据仅在运行 `cmdkey` 的上下文中持久保存。 如果你正在开发作为服务运行的应用程序，你也需要在该上下文中持久保存凭据。

### <a name="mount-the-file-share-using-the-persisted-credentials"></a>使用保存的凭据装载文件共享
建立与虚拟机的远程连接后，便可以使用以下语法运行 `net use` 命令来装载文件共享了。 请将 `<storage-account-name>` 替换为你的存储帐户名称，将 `<share-name>` 替换为你的文件存储共享名称。

    net use <drive-letter>: \\<storage-account-name>.file.core.chinacloudapi.cn\<share-name>

    example :
    net use z: \\samples.file.core.chinacloudapi.cn\logs

由于你已在上一步中保存了存储帐户凭据，因此你无需随 `net use` 命令提供这些凭据。 如果你尚未保存凭据，请作为传递给 `net use` 命令的参数提供凭据，如以下示例所示。

    net use <drive-letter>: \\<storage-account-name>.file.core.chinacloudapi.cn\<share-name> /u:AZURE\<storage-account-name> <storage-account-key>

    example :
    net use z: \\samples.file.core.chinacloudapi.cn\logs /u:AZURE\samples <storage-account-key>

现在，你可以使用虚拟机中的文件存储共享，就像使用任何其他驱动器一样。 你可以从命令提示符发出标准文件命令，也可以从文件资源管理器查看已装载的共享及其内容。 你也可以在虚拟机中运行代码，以便访问使用标准 Windows 文件 I/O API 的文件共享，例如 .NET Framework 中由 [System.IO 命名空间](http://msdn.microsoft.com/zh-cn/library/gg145019.aspx) 提供的文件共享。

你还可以通过远程登录到角色中从 Azure 云服务中运行的角色装载文件共享。

### <a name="mount-the-file-share-from-an-on-premises-client-running-windows"></a>从运行 Windows 的本地客户端装载文件共享
若要从本地客户端装载文件共享，必须首先执行以下步骤：

* 安装支持 SMB 3.0 的 Windows 版本。 Windows 将利用 SMB 3.0 加密来在本地客户端和云中的 Azure 文件共享之间安全地传输数据。
* 根据 SMB 协议的需要，在本地网络中打开端口 445（TCP 出站）的 Internet 访问。

> [AZURE.NOTE]
> 某些 Internet 服务提供商可能会阻止端口 445，因此你可能需要与你的服务提供商核实。
> 
> 

## <a name="develop-with-file-storage"></a>使用文件存储进行开发
若要编写调用文件存储的代码，可以使用适用于 .NET 和 Java 的存储客户端库或 Azure 存储 REST API。 本部分中的示例演示如何通过在桌面上运行的简单控制台应用程序使用 [用于 .NET 的 Azure 存储客户端库](https://msdn.microsoft.com/zh-cn/library/mt347887.aspx) 处理文件共享。

### <a name="create-the-console-application-and-obtain-the-assembly"></a>创建控制台应用程序，并获取程序集
在 Visual Studio 中创建新的控制台应用程序并安装包含 Azure 存储空间客户端库的 NuGet 包：

1. 在 Visual Studio 中，选择“文件”>“新建项目”，然后从 Visual C# 模板列表中选择“Windows”>“控制台应用程序”。
2. 提供控制台应用程序的名称，然后单击“确定”。
3. 创建项目后，在解决方案资源管理器中右键单击该项目并选择“管理 NuGet 包”。 在线搜索“WindowsAzure.Storage”，然后单击“安装”以安装适用于 .NET 包和依赖项的 Azure 存储客户端库。

此文章中的代码示例还使用 [Azure Configuration Manager 库](https://msdn.microsoft.com/zh-cn/library/azure/mt634646.aspx)从控制台应用程序中的 app.config 文件中检索存储连接字符串。 使用 Azure Configuration Manager，你可以在运行时检索连接字符串，无论你的应用程序是在 Azure 中运行还是从桌面、移动、或 Web 应用程序中运行。 

若要安装 Azure Configuration Manager 包，请在“解决方案资源管理器”中右键单击项目，然后选择“管理 NuGet 包”。 在线搜索“ConfigurationManager”，然后单击“安装”以安装包。

不一定要使用 Azure Configuration Manager。 还可以使用 API，例如 .NET Framework 的 [ConfigurationManager 类](https://msdn.microsoft.com/zh-cn/library/system.configuration.configurationmanager.aspx)。

### <a name="save-your-storage-account-credentials-to-the-appconfig-file"></a>将存储帐户凭据保存到 app.config 文件
接下来，将你的凭据保存到项目的 app.config 文件中。 编辑 app.config 文件，使其看起来类似于下面的示例，将 `myaccount` 替换为你的存储帐户名称，并将 `mykey` 替换为你的存储帐户密钥。

    <?xml version="1.0" encoding="utf-8" ?>
    <configuration>
        <startup>
            <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5" />
        </startup>
        <appSettings>
            <add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=StorageAccountKeyEndingIn==;EndpointSuffix=core.chinacloudapi.cn" />
        </appSettings>
    </configuration>

> [AZURE.NOTE]
> 最新版本的 Azure 存储模拟器不支持文件存储。 连接字符串必须针对云中要使用文件存储的 Azure 存储帐户。
> 
> 

### <a name="add-namespace-declarations"></a>添加命名空间声明
从解决方案资源管理器打开 `program.cs` 文件，并在该文件顶部添加以下命名空间声明。

    using Microsoft.Azure; // Namespace for Azure Configuration Manager
    using Microsoft.WindowsAzure.Storage; // Namespace for Storage Client Library
    using Microsoft.WindowsAzure.Storage.Blob; // Namespace for Blob storage
    using Microsoft.WindowsAzure.Storage.File; // Namespace for File storage

[AZURE.INCLUDE [storage-cloud-configuration-manager-include](../../includes/storage-cloud-configuration-manager-include.md)]

### <a name="access-the-file-share-programmatically"></a>以编程方式访问文件共享
接下来，将以下代码添加到 `Main()` 方法（在上面显示的代码后面）以检索连接字符串。 此代码将获取我们先前创建的文件的引用，并将其内容输出到控制台窗口中。

	// Create a CloudFileClient object for credentialed access to File storage.
	CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

	// Get a reference to the file share we created previously.
	CloudFileShare share = fileClient.GetShareReference("logs");

	// Ensure that the share exists.
	if (share.Exists())
	{
	    // Get a reference to the root directory for the share.
	    CloudFileDirectory rootDir = share.GetRootDirectoryReference();

	    // Get a reference to the directory we created previously.
	    CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("CustomLogs");

	    // Ensure that the directory exists.
	    if (sampleDir.Exists())
	    {
	        // Get a reference to the file we created previously.
	        CloudFile file = sampleDir.GetFileReference("Log1.txt");

	        // Ensure that the file exists.
	        if (file.Exists())
	        {
	            // Write the contents of the file to the console window.
	            Console.WriteLine(file.DownloadTextAsync().Result);
	        }
	    }
	}

运行控制台应用程序以查看输出。

### <a name="set-the-maximum-size-for-a-file-share"></a>设置文件共享的最大大小
从 Azure 存储空间客户端库的 5.x 版开始，可以设置文件共享的配额（或最大大小），单位为千兆字节。 你还可以查看共享当前存储了多少数据。

通过设置一个共享的配额，可以限制在该共享上存储的文件的总大小。如果共享上文件的总大小超过在共享上设定的配额，则客户端将不能增加现有文件的大小或创建新文件，除非这些文件是空的。

下面的示例演示如何检查共享的当前使用情况，以及如何设置共享的配额。

    // Parse the connection string for the storage account.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create a CloudFileClient object for credentialed access to File storage.
    CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

    // Get a reference to the file share we created previously.
    CloudFileShare share = fileClient.GetShareReference("logs");

    // Ensure that the share exists.
    if (share.Exists())
    {
        // Check current usage stats for the share.
        // Note that the ShareStats object is part of the protocol layer for the File service.
        Microsoft.WindowsAzure.Storage.File.Protocol.ShareStats stats = share.GetStats();
        Console.WriteLine("Current share usage: {0} GB", stats.Usage.ToString());

        // Specify the maximum size of the share, in GB.
        // This line sets the quota to be 10 GB greater than the current usage of the share.
        share.Properties.Quota = 10 + stats.Usage;
        share.SetProperties();

        // Now check the quota for the share. Call FetchAttributes() to populate the share's properties.
        share.FetchAttributes();
        Console.WriteLine("Current share quota: {0} GB", share.Properties.Quota);
    }

### <a name="generate-a-shared-access-signature-for-a-file-or-file-share"></a>为文件或文件共享生成共享访问签名
从 Azure 存储空间客户端库的 5.x 版开始，可以为文件共享或单个文件生成共享访问签名 (SAS)。 还可以在文件共享上创建一个共享访问策略以管理共享访问签名。 建议创建共享访问策略，因为它提供了一种在受到威胁时撤消 SAS 的方式。

以下示例在一个共享上创建共享访问策略，然后使用该策略为共享中的一个文件提供 SAS 约束。

    // Parse the connection string for the storage account.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create a CloudFileClient object for credentialed access to File storage.
    CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

    // Get a reference to the file share we created previously.
    CloudFileShare share = fileClient.GetShareReference("logs");

    // Ensure that the share exists.
    if (share.Exists())
    {
        string policyName = "sampleSharePolicy" + DateTime.UtcNow.Ticks;

        // Create a new shared access policy and define its constraints.
        SharedAccessFilePolicy sharedPolicy = new SharedAccessFilePolicy()
            {
                SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
                Permissions = SharedAccessFilePermissions.Read | SharedAccessFilePermissions.Write
            };

        // Get existing permissions for the share.
        FileSharePermissions permissions = share.GetPermissions();

        // Add the shared access policy to the share's policies. Note that each policy must have a unique name.
        permissions.SharedAccessPolicies.Add(policyName, sharedPolicy);
        share.SetPermissions(permissions);

        // Generate a SAS for a file in the share and associate this access policy with it.
        CloudFileDirectory rootDir = share.GetRootDirectoryReference();
        CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("CustomLogs");
        CloudFile file = sampleDir.GetFileReference("Log1.txt");
        string sasToken = file.GetSharedAccessSignature(null, policyName);
        Uri fileSasUri = new Uri(file.StorageUri.PrimaryUri.ToString() + sasToken);

        // Create a new CloudFile object from the SAS, and write some text to the file.
        CloudFile fileSas = new CloudFile(fileSasUri);
        fileSas.UploadText("This write operation is authenticated via SAS.");
        Console.WriteLine(fileSas.DownloadText());
    }

有关创建和使用共享访问签名的更多信息，请参阅[使用共享访问签名 (SAS)](/documentation/articles/storage-dotnet-shared-access-signature-part-1/) 和[创建 SAS 并将其与 Blob 存储结合使用](/documentation/articles/storage-dotnet-shared-access-signature-part-2/)。

### <a name="copy-files"></a>复制文件
从 Azure 存储空间客户端库的 5.x 版开始，可以将一个文件复制到另一个文件，将一个文件复制到一个 Blob，或将一个 Blob 复制到一个文件。 在后续部分中，我们将演示如何以编程方式执行这些复制操作。

还可以使用 AzCopy 将一个文件复制到另一个文件或将一个 Blob 复制到一个文件，反之亦然。 请参阅 [使用 AzCopy 命令行实用程序传输数据](/documentation/articles/storage-use-azcopy/)。

> [AZURE.NOTE]
> 如果将一个 Blob 复制到一个文件，或将一个文件复制到一个 Blob，必须使用共享访问签名 (SAS) 对源对象进行身份验证，即使你在同一存储帐户内进行复制。
> 
> 

**将一个文件复制到另一个文件**

以下示例将一个文件复制到同一共享中的另一个文件。 因为此操作在同一存储帐户中的文件之间进行复制，可以使用共享密钥身份验证来进行复制。

    // Parse the connection string for the storage account.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create a CloudFileClient object for credentialed access to File storage.
    CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

    // Get a reference to the file share we created previously.
    CloudFileShare share = fileClient.GetShareReference("logs");

    // Ensure that the share exists.
    if (share.Exists())
    {
        // Get a reference to the root directory for the share.
        CloudFileDirectory rootDir = share.GetRootDirectoryReference();

        // Get a reference to the directory we created previously.
        CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("CustomLogs");

        // Ensure that the directory exists.
        if (sampleDir.Exists())
        {
            // Get a reference to the file we created previously.
            CloudFile sourceFile = sampleDir.GetFileReference("Log1.txt");

            // Ensure that the source file exists.
            if (sourceFile.Exists())
            {
                // Get a reference to the destination file.
                CloudFile destFile = sampleDir.GetFileReference("Log1Copy.txt");

                // Start the copy operation.
                destFile.StartCopy(sourceFile);

                // Write the contents of the destination file to the console window.
                Console.WriteLine(destFile.DownloadText());
            }
        }
    }

**将一个文件复制到一个 Blob**

以下示例创建一个文件并将其复制到同一存储帐户中的某个 blob。 该示例为源文件创建一个 SAS，服务在复制操作期间使用该 SAS 验证对源文件的访问。

    // Parse the connection string for the storage account.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create a CloudFileClient object for credentialed access to File storage.
    CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

    // Create a new file share, if it does not already exist.
    CloudFileShare share = fileClient.GetShareReference("sample-share");
    share.CreateIfNotExists();

    // Create a new file in the root directory.
    CloudFile sourceFile = share.GetRootDirectoryReference().GetFileReference("sample-file.txt");
    sourceFile.UploadText("A sample file in the root directory.");

    // Get a reference to the blob to which the file will be copied.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
    CloudBlobContainer container = blobClient.GetContainerReference("sample-container");
    container.CreateIfNotExists();
    CloudBlockBlob destBlob = container.GetBlockBlobReference("sample-blob.txt");

    // Create a SAS for the file that's valid for 24 hours.
    // Note that when you are copying a file to a blob, or a blob to a file, you must use a SAS
    // to authenticate access to the source object, even if you are copying within the same
    // storage account.
    string fileSas = sourceFile.GetSharedAccessSignature(new SharedAccessFilePolicy()
    {
        // Only read permissions are required for the source file.
        Permissions = SharedAccessFilePermissions.Read,
        SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24)
    });

    // Construct the URI to the source file, including the SAS token.
    Uri fileSasUri = new Uri(sourceFile.StorageUri.PrimaryUri.ToString() + fileSas);

    // Copy the file to the blob.
    destBlob.StartCopy(fileSasUri);

    // Write the contents of the file to the console window.
    Console.WriteLine("Source file contents: {0}", sourceFile.DownloadText());
    Console.WriteLine("Destination blob contents: {0}", destBlob.DownloadText());

可以用相同的方式将一个 Blob 复制到一个文件。 如果源对象是一个 Blob，则创建一个 SAS 以复制操作期间验证对该 Blob 的访问。

## <a name="troubleshooting-file-storage-using-metrics"></a>使用指标对文件存储进行故障排除
Azure 存储空间分析现在支持用于文件存储的指标。 使用指标数据，可以跟踪请求和诊断问题。

可以从 [Azure 门户预览](https://portal.azure.cn)为文件存储启用指标。你还可以通过 REST API 或存储客户端库中的类似物之一调用“设置文件服务属性”操作，以编程方式启用指标。

下面的代码示例演示如何使用适用于 .NET 的存储客户端库启用文件存储的指标。

首先，在添加以上语句后，将以下 `using` 语句添加到你的 program.cs 文件中：

    using Microsoft.WindowsAzure.Storage.File.Protocol;
    using Microsoft.WindowsAzure.Storage.Shared.Protocol;

请注意，Blob、表和队列存储使用 `Microsoft.WindowsAzure.Storage.Shared.Protocol` 命名空间中的共享 `ServiceProperties` 类型，而文件存储使用其自己的类型，即 `Microsoft.WindowsAzure.Storage.File.Protocol` 命名空间中的 `FileServiceProperties` 类型。 但是，你的代码中必须同时引用这两个命名空间，才能编译后续代码。

    // Parse your storage connection string from your application's configuration file.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
            Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));
    // Create the File service client.
    CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

    // Set metrics properties for File service.
    // Note that the File service currently uses its own service properties type,
    // available in the Microsoft.WindowsAzure.Storage.File.Protocol namespace.
    fileClient.SetServiceProperties(new FileServiceProperties()
    {
        // Set hour metrics
        HourMetrics = new MetricsProperties()
        {
            MetricsLevel = MetricsLevel.ServiceAndApi,
            RetentionDays = 14,
            Version = "1.0"
        },
        // Set minute metrics
        MinuteMetrics = new MetricsProperties()
        {
            MetricsLevel = MetricsLevel.ServiceAndApi,
            RetentionDays = 7,
            Version = "1.0"
        }
    });

    // Read the metrics properties we just set.
    FileServiceProperties serviceProperties = fileClient.GetServiceProperties();
    Console.WriteLine("Hour metrics:");
    Console.WriteLine(serviceProperties.HourMetrics.MetricsLevel);
    Console.WriteLine(serviceProperties.HourMetrics.RetentionDays);
    Console.WriteLine(serviceProperties.HourMetrics.Version);
    Console.WriteLine();
    Console.WriteLine("Minute metrics:");
    Console.WriteLine(serviceProperties.MinuteMetrics.MetricsLevel);
    Console.WriteLine(serviceProperties.MinuteMetrics.RetentionDays);
    Console.WriteLine(serviceProperties.MinuteMetrics.Version);

还可以参考 [Azure 文件故障排除文章](/documentation/articles/storage-troubleshoot-file-connection-problems/) 了解有关端到端故障排除指南。 

## <a name="file-storage-faq"></a>文件存储常见问题
1. **文件存储是否支持基于 Active Directory 的身份验证？**
   
    我们目前不支持基于 AD 的身份验证或 ACL，但会将其列入我们的功能请求列表中。 目前，Azure 存储帐户密钥用于为文件共享提供身份验证。 我们确实提供通过 REST API 或客户端库使用共享访问签名 (SAS) 的解决方法。 使用 SAS，可以生成具有在指定的时间间隔内有效的特定权限的令牌。 例如，你可以生成对给定文件具有只读访问权限的令牌。 在此令牌的有效期内拥有此令牌的任何人对该文件具有只读访问权限。
   
    仅通过 REST API 或客户端库支持 SAS。 通过 SMB 协议装载文件共享时，不能使用 SAS 委派对其内容的访问权限。 

2. **如何通过 Web 浏览器提供对特定文件的访问权限？**

   使用 SAS，可以生成具有在指定的时间间隔内有效的特定权限的令牌。 例如，可以生成一个令牌，在特定的时段对特定的文件进行只读访问。 只要该 URL 有效，任何人都可以使用它直接从任何 Web 浏览器进行下载。 可以轻松地通过 UI（例如存储资源管理器）生成 SAS 密钥。

3.   **访问 Azure 文件存储中的文件可以通过哪些不同方式？**

    可以使用 SMB 3.0 协议将文件共享装载在本地计算机上，也可以使用 [存储资源管理器](http://storageexplorer.com/) 或 Cloudberry 之类的工具访问文件共享中的文件。 可以通过应用程序使用客户端库、REST API 或 Powershell 访问 Azure 文件共享中的文件。
    
4.   **如何在本地计算机上装载 Azure 文件共享？** 

    可以通过 SMB 协议装载文件共享，只要端口 445（TCP 出站）处于打开状态且客户端支持 SMB 3.0 协议（*例如*，Windows 8 或 Windows Server 2012）。 请通过本地 ISP 提供商来取消阻止端口。 在过渡期间，可以使用存储资源管理器或任何其他的第三方（例如 Cloudberry）查看文件。

5. **Azure 虚拟机与文件共享之间的网络流量是否算作对订阅计费的外部带宽？**
   
    如果文件共享和虚拟机位于不同的区域，则它们之间的流量将作为外部带宽收费。
6. **如果是虚拟机和同一区域中的文件共享之间的网络流量，是免费吗？**
   
    是的。 如果流量在同一区域，是免费的。
7. **从本地虚拟机连接到 Azure 文件存储是否依赖于 Azure ExpressRoute？**
   
    否。 如果你没有 ExpressRoute，你仍可以从本地访问文件共享，只要你将端口 445（TCP 出站）打开供 Internet 访问。 但是，如果你愿意，你可以将 ExpressRoute 用于文件存储。
8. **故障转移群集的“文件共享见证”是 Azure 文件存储的使用案例之一吗？**
   
    目前，不支持此功能。
9. **当前仅通过 LRS 或 GRS 复制文件存储，对吗？**  
   
    我们计划支持 RA-GRS，但尚没有共享时间表。
10. **何时能够将现有存储帐户用于 Azure 文件存储？**
   
    现已为所有存储帐户启用 Azure 文件存储。
11. **是否会将重命名操作也添加到 REST API？**
   
    在我们的 REST API 中尚不支持重命名。
12. **能否使用嵌套共享，换而言之就是共享下的共享？**
    
    否。 文件共享是你可以装载的虚拟驱动程序，因此不支持嵌套共享。
13. **是否可以对共享中的文件夹指定只读或只写权限？**
    
    如果通过 SMB 装载文件共享，你不具有此级别的权限控制。 但是，你可以通过 REST API 或客户端库创建共享访问签名 (SAS) 来实现此控制。  
14. **尝试将文件解压缩到文件存储中时我的性能速度太慢。我该怎样做？**
    
    若要将大量文件传输到文件存储，建议使用 AzCopy、Azure Powershell (Windows) 或 Azure CLI (Linux/Unix)，因为这些工具已针对网络传输进行优化。
15. **发布了修复 Azure 文件慢速性能问题的修补程序**
    
    Windows 团队最近发布了一个修补程序，旨在修复客户从 Windows 8.1 计算机或 Windows Server 2012 R2 服务器访问 Azure 文件存储时遇到的慢速性能问题。有关详细信息，请查看相关的知识库文章：[从 Windows 8.1 或 Server 2012 R2 访问 Azure 文件存储时性能降低](https://support.microsoft.com/zh-cn/kb/3114025)。
16. **通过 IBM MQ 使用 Azure 文件存储**
    
    IBM 已发布相关文档来指导 IBM MQ 客户通过其服务配置 Azure 文件存储。有关详细信息，请查阅 [如何通过 Azure 文件服务设置 IBM MQ 多实例队列管理器](https://github.com/ibm-messaging/mq-azure/wiki/How-to-setup-IBM-MQ-Multi-instance-queue-manager-with-Microsoft-Azure-File-Service)。
17. **如何排除 Azure 文件存储错误？**
    
    可以参考 [Azure 文件故障排除文章](/documentation/articles/storage-troubleshoot-file-connection-problems/)了解有关端到端故障排除指南。

18. **如何为 Azure 文件启用服务器端加密？**

    [服务器端加密](/documentation/articles/storage-service-encryption/)目前提供预览版。 在预览期间，只能为新创建的 Azure Resource Manager (ARM) 存储帐户启用此功能。
    可以通过 Azure 门户预览在 Azure Resource Manager 存储帐户上启用该功能。 我们计划二月底在 [Azure Powershell](https://msdn.microsoft.com/zh-cn/library/azure/mt607151.aspx)、[Azure CLI](/documentation/articles/storage-azure-cli-nodejs/) 或 [Azure 存储资源提供程序 API](https://docs.microsoft.com/zh-cn/rest/api/storagerp/storageaccounts) 上启用针对文件存储的加密。 启用该功能没有额外收费。 为 Azure 文件存储启用存储服务加密时，系统会为用户自动加密数据。 

## <a name="next-steps"></a>后续步骤
请参阅以下链接以获取有关 Azure 文件存储的更多信息。

### <a name="conceptual-articles-and-videos"></a>概念性文章和视频
* [Azure 文件存储：适用于 Windows 和 Linux 的顺畅的云 SMB 文件系统](https://azure.microsoft.com/documentation/videos/azurecon-2015-azure-files-storage-a-frictionless-cloud-smb-file-system-for-windows-and-linux/)
* [如何通过 Linux 使用 Azure 文件存储](/documentation/articles/storage-how-to-use-files-linux/)

### <a name="tooling-support-for-file-storage"></a>文件存储的工具支持
* [对 Azure 存储空间使用 Azure PowerShell](/documentation/articles/storage-powershell-guide-full/)
* [如何对 Azure 存储使用 AzCopy](/documentation/articles/storage-use-azcopy/)
* [将 Azure CLI 用于 Azure 存储空间](/documentation/articles/storage-azure-cli/#create-and-manage-file-shares)
* [排查 Azure 文件存储问题](/documentation/articles/storage-troubleshoot-file-connection-problems/)

### <a name="reference"></a>引用
* [.NET 存储客户端库参考](https://msdn.microsoft.com/zh-cn/library/azure/dn261237.aspx)
* [文件服务 REST API 参考](http://msdn.microsoft.com/zh-cn/library/azure/dn167006.aspx)

### <a name="blog-posts"></a>博客文章
* [Azure 文件存储现已正式发布](https://azure.microsoft.com/zh-cn/blog/azure-file-storage-now-generally-available/)
* [Azure 文件存储内部](https://azure.microsoft.com/zh-cn/blog/inside-azure-file-storage/)
* [Azure 文件服务简介](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/12/introducing-microsoft-azure-file-service.aspx)
* [持久连接到 Azure 文件](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/27/persisting-connections-to-microsoft-azure-files.aspx)
<!--Update_Description: wording update; add anchors to H2 titles-->