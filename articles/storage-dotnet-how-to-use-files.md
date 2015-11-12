<properties
			pageTitle="如何通过 PowerShell 和 .NET 使用 Azure 文件存储 | Windows Azure"
            description="了解如何使用 Azure 文件存储创建云文件共享和管理文件内容。企业可以通过文件存储将依赖 SMB 文件共享的应用程序移到 Azure。保存虚拟机的存储帐户凭据，从而在重新启动时重新连接到文件共享。"
            services="storage"
            documentationCenter=".net"
            authors="tamram"
            manager="adinah"
            editor="" />

<tags ms.service="storage"
      ms.date="08/04/2015"
      wacn.date="11/12/2015" />

# 如何通过 PowerShell 和 .NET 使用 Azure 文件存储

## 概述

Azure文件存储服务基于标准的SMB协议在云端提供文件共享存储服务。Azure文件存储正式商用版已经上线，目前同时支持SMB2.1和SMB3.0。现在，在 Azure 中运行的应用程序可以使用标准和熟悉的文件系统 API（例如 ReadFile 和 WriteFile）在 VM 之间共享文件。此外，REST 接口可打开各种混合方案，还可以通过该接口同时访问这些文件。最后，Azure 文件基于与 Blob、表和队列服务相同的技术构建，这意味着 Azure 文件能够充分利用 Azure 存储平台内置的现有可用性、持续性、可伸缩性和地域冗余性。

## 关于本教程

此入门教程演示使用 Windows Azure 文件存储的基础知识。在本教程中，我们将：

- 使用 Azure PowerShell 来演示如何创建新的 Azure 文件共享、如何添加目录、如何将本地文件上载到该共享，以及如何列出该目录中的文件。
- 从 Azure 虚拟机装载文件共享，就像装载任何 SMB 共享一样。
- 使用用于.NET 的 Azure 存储空间客户端库从本地应用程序访问文件共享。创建一个控制台应用程序并通过文件共享执行以下操作：
	- 将共享中一个文件的内容写入控制台窗口
	- 设置文件共享的配额（最大大小）
	- 若一个文件使用在共享中定义的共享访问策略，则为该文件创建一个共享访问签名
	- 将文件复制到同一存储帐户中的另一个文件
	- 将文件复制到同一存储帐户中的一个 Blob

[AZURE.INCLUDE [storage-dotnet-client-library-version-include](../includes/storage-dotnet-client-library-version-include.md)]

[AZURE.INCLUDE [storage-file-concepts-include](../includes/storage-file-concepts-include.md)]


## 创建 Azure 存储帐户

Azure 文件存储目前发布了预览版。若要请求访问预览版，请导航到 [Windows Azure 预览页](/services/preview/)，然后请求访问“Azure 文件”。你的请求得到批准后，系统就会通知你，然后你就可以访问文件存储预览版。然后，你就可以创建用于访问文件存储的存储帐户。

> [AZURE.NOTE] 文件存储目前仅提供给新的存储帐户使用。你的订阅获得访问文件存储的授权后，可创建一个新的用于本指南的存储帐户。
>
> Azure 文件存储当前不支持共享访问签名。

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

## 使用 PowerShell 创建文件共享

接下来，我们将使用 Azure PowerShell 创建文件共享。创建文件共享之后，即可以从任何支持 SMB 2.1 的文件系统装载。

### 为 Azure 存储空间安装 PowerShell cmdlet

若要准备使用 PowerShell，请下载并安装 Azure PowerShell cmdlet。有关安装点和安装说明，请参阅[如何安装和配置 Azure PowerShell](/zh-cn/documentation/articles/install-configure-powershell)。

> [WACOM.NOTE] 文件服务的 PowerShell cmdlet 只在最新的 0.8.5 版及更高版本的 Azure PowerShell 模块中提供。建议你下载并安装最新的 Azure PowerShell 模块或升级到最新模块。

通过单击"开始"并键入"Windows Azure PowerShell"打开 Azure PowerShell 窗口。Azure PowerShell 窗口将为你加载 Azure PowerShell 模块。

###为存储帐户和密钥创建上下文

现在，将创建存储帐户上下文。该上下文封装了存储帐户名称和帐户密钥。有关从 Azure 门户复制你的帐户密钥的说明，请参阅[查看、复制和重新生成存储访问密钥](/documentation/articles/storage-create-storage-account#view-copy-and-regenerate-storage-access-keys)。

请将下面示例中的 `storage-account-name` 和 `storage-account-key` 替换为你的帐户名称和密钥：

	# create a context for account and key
	$ctx=New-AzureStorageContext -Environment AzureChinaCloud storage-account-name storage-account-key

### 创建新的文件共享

接下来，在此示例中创建名为 `logs` 的新共享：

	# create a new share
	$s = New-AzureStorageShare logs -Context $ctx

现在，你在文件存储中已有一个文件共享。接下来，我们将添加目录和文件。

> [AZURE.IMPORTANT] 文件共享的名称必须是全部小写。有关命名文件共享和文件的详细信息，请参阅[命名和引用共享、目录、文件和元数据](https://msdn.microsoft.com/zh-cn/library/azure/dn167011.aspx)。

### 在文件共享中创建目录

接下来，将在共享中创建目录。在下面的示例中，目录名为 `CustomLogs`：

    # create a directory in the share
    New-AzureStorageDirectory -Share $s -Path CustomLogs

###将本地文件上载到目录

现在，将本地文件上载到该目录。以下示例从 `C:\temp\Log1.txt` 上载文件。请编辑文件路径，使其指向你本地计算机上的有效文件：

    # upload a local file to the new directory
    Set-AzureStorageFileContent -Share $s -Source C:\temp\Log1.txt -Path CustomLogs

###列出目录中的文件

可以列出目录的文件，以便查看其中的文件。此命令也将列出子目录，但在此示例中没有子目录，因此只列出文件。  

	# list files in the new directory
	Get-AzureStorageFile -Share $s -Path CustomLogs

## 从运行 Windows 的 Azure 虚拟机装载共享

为了演示如何装载 Azure 文件共享，现在我们将创建一个运行 Windows 的 Azure 虚拟机，并远程登录到它内部以装载共享。

1. 首先，按照[创建运行 Windows Server 的虚拟机](/documentation/articles/virtual-machines-windows-tutorial)中的说明创建一个新的 Azure 虚拟机。
2. 然后，按照[如何登录到运行 Windows Server 的虚拟机](/documentation/articles/virtual-machines-log-on-windows-server)中的说明远程登录到该虚拟机内部。
3. 在该虚拟机上打开 PowerShell 窗口。

###保存虚拟机的存储帐户凭据

装载到文件共享之前，先在虚拟机上保存存储帐户凭据。当虚拟机重新启动时，此步骤允许 Windows 自动重新连接到文件共享。若要保存帐户凭据，请在虚拟机上的 PowerShell 窗口中执行 `cmdkey` 命令。请将 `<storage-account-name>` 替换为你的存储帐户名称，将 `<storage-account-key>` 替换为你的存储帐户密钥：

	cmdkey /add:<storage-account-name>.file.core.chinacloudapi.cn /user:<storage-account-name> /pass:<storage-account-key>

现在，当虚拟机重新启动时，Windows 将重新连接到你的文件共享。可以通过在 PowerShell 窗口中执行  `net use` 命令来验证是否已重新连接共享。

###使用保存的凭据装载文件共享

建立与虚拟机的远程连接后，便可以使用以下语法执行 `net use` 命令来装载文件共享了。请将 `<storage-account-name>` 替换为你的存储帐户名称，将 `<share-name>` 替换为你的文件存储共享名称。

    net use <drive-letter>: \<storage-account-name>.file.core.chinacloudapi.cn<share-name>

	example :
	net use z: \\samples.file.core.chinacloudapi.cn\logs

由于你已在上一步中保存了存储帐户凭据，因此你无需随 `net use` 命令提供这些凭据。如果你尚未保存凭据，请作为传递给 `net use` 命令的参数提供凭据，如以下示例所示：

    net use <drive-letter>: \<storage-account-name>.file.core.chinacloudapi.cn<share-name> /u:<storage-account-name> <storage-account-key>

	example :
	net use z: \\samples.file.core.chinacloudapi.cn\logs /u:samples <storage-account-key>

现在，你可以使用虚拟机中的文件存储共享，就像使用任何其他驱动器一样。你可以从命令提示符发出标准文件命令，也可以从文件资源管理器查看已装载的共享及其内容。你也可以在虚拟机中运行代码，以便访问使用标准 Windows 文件 I/O API 的文件共享，例如 .NET Framework 中由 [System.IO 命名空间](http://msdn.microsoft.com/zh-cn/library/gg145019.aspx)提供的文件共享。

你还可以通过远程登录到角色中从 Azure 云服务中运行的角色装载文件共享。

## 创建本地应用程序以使用文件存储

你可以从在 Azure 中运行的虚拟机或云服务装载文件存储共享，如上所示。然而，你不能从本地应用程序装载文件共享。若要从本地应用程序访问共享数据，必须使用文件存储 API。此示例说明如何通过 [Azure .NET 存储空间客户端库](https://msdn.microsoft.com/en-us/library/wa_storage_30_reference_home.aspx)使用文件共享。

为了演示如何从本地应用程序使用 API，我们将创建一个在桌面上运行的简单控制台应用程序。

###创建控制台应用程序，并获取程序集

若要在 Visual Studio 中创建新的控制台应用程序并安装 Azure 存储 NuGet 包，请执行以下操作：

1. 在 Visual Studio 中，选择“文件”->“新建项目”，然后从 Visual C# 模板的列表中选择“Windows”->“控制台应用程序”。
2. 为控制台应用程序提供一个名称，然后单击“确定”。
3. 创建项目后，在解决方案资源管理器中右键单击该项目并选择“管理 NuGet 包”。在线搜索“WindowsAzure.Storage”，然后单击“安装”以安装 Azure 存储包和依赖项。

###将存储帐户凭据保存到 app.config 文件

接下来，将你的凭据保存到项目的 app.config 文件中。编辑 app.config 文件，使其看起来类似于下面的示例，将  `myaccount` 替换为你的存储帐户名称，将  `mykey` 替换为你的存储帐户密钥：

    <?xml version="1.0" encoding="utf-8" ?>
	<configuration>
		<startup> 
			<supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5" />
		</startup>
		<appSettings>
	        <add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=StorageAccountKeyEndingIn==;EndpointSuffix=core.Chinacloudapi.cn" />
		</appSettings>
	</configuration>

> [WACOM.NOTE] 最新版本的 Azure 存储模拟器不支持文件存储。连接字符串必须针对云中可以访问文件预览的 Azure 存储帐户。


###添加命名空间声明
从解决方案资源管理器打开 program.cs 文件，并在该文件顶部添加以下命名空间声明：

    using Microsoft.WindowsAzure;
	using Microsoft.WindowsAzure.Storage;
	using Microsoft.WindowsAzure.Storage.Blob;
	using Microsoft.WindowsAzure.Storage.File;

### 以编程方式检索连接字符串

可以使用 `Microsoft.WindowsAzure.CloudConfigurationManager` 类或 `System.Configuration.ConfigurationManager ` 类从 app.config 文件中检索保存的凭据。Windows Azure 配置管理器包，其中包括 `Microsoft.WindowsAzure.CloudConfigurationManager` 类，可从 [Nuget](https://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager) 获得。

此处的示例显示如何使用 `CloudConfigurationManager` 类检索凭据，并使用 `CloudStorageAccount` 类封装这些凭据。将以下代码添加到 Program.cs 的 `Main()` 方法中：

    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

### 以编程方式访问文件共享

接下来，将以下代码添加到上面显示的  `Main()` 方法中用于检索连接字符串的代码的后面。此代码将获取我们先前创建的文件的引用，并将其内容输出到控制台窗口中。

	//Create a CloudFileClient object for credentialed access to File storage.
    CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

	//Get a reference to the file share we created previously.
	CloudFileShare share = fileClient.GetShareReference("logs");

	//Ensure that the share exists.
    if (share.Exists())
    {
		//Get a reference to the root directory for the share.
        CloudFileDirectory rootDir = share.GetRootDirectoryReference();

	    //Get a reference to the directory we created previously.
	    CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("CustomLogs");

	    //Ensure that the directory exists.
	    if (sampleDir.Exists())
	    {
	        //Get a reference to the file we created previously.
	        CloudFile file = sampleDir.GetFileReference("Log1.txt");

			//Ensure that the file exists.
            if (file.Exists())
            {
				//Write the contents of the file to the console window.
                Console.WriteLine(file.DownloadTextAsync().Result);
            }
        }
    }

运行控制台应用程序以查看输出。

## 设置文件共享的最大大小

从 Azure 存储空间客户端库的 5.x 版开始，可以设置文件共享的配额（或最大大小），单位为千兆字节。你还可以查看共享当前存储了多少数据。

通过设置一个共享的配额，可以限制在该共享上存储的文件的总大小。如果共享上文件的总大小超过在共享上设定的配额，则客户端将不能增加现有文件的大小或创建新文件，除非这些文件是空的。

下面的示例演示如何检查共享的当前使用情况，以及如何设置共享的配额。

    //Parse the connection string for the storage account.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

    //Create a CloudFileClient object for credentialed access to File storage.
    CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

    //Get a reference to the file share we created previously.
    CloudFileShare share = fileClient.GetShareReference("logs");

    //Ensure that the share exists.
    if (share.Exists())
    {
        //Check current usage stats for the share.
		//Note that the ShareStats object is part of the protocol layer for the File service.
        Microsoft.WindowsAzure.Storage.File.Protocol.ShareStats stats = share.GetStats();
        Console.WriteLine("Current share usage: {0} GB", stats.Usage.ToString());

        //Specify the maximum size of the share, in GB.
        //This line sets the quota to be 10 GB greater than the current usage of the share.
        share.Properties.Quota = 10 + stats.Usage;
        share.SetProperties();

        //Now check the quota for the share. Call FetchAttributes() to populate the share's properties. 
        share.FetchAttributes();
        Console.WriteLine("Current share quota: {0} GB", share.Properties.Quota);
    }

## 为文件或文件共享生成共享访问签名

从 Azure 存储空间客户端库的 5.x 版开始，可以为文件共享或单个文件生成共享访问签名 (SAS)。还可以在文件共享上创建一个共享访问策略以管理共享访问签名。建议创建共享访问策略，因为它提供了一种在受到威胁时撤消 SAS 的方式。

下面的示例在一个共享上创建共享访问策略，然后使用该策略为共享中的一个文件提供 SAS 约束。

    //Parse the connection string for the storage account.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

    //Create a CloudFileClient object for credentialed access to File storage.
    CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

    //Get a reference to the file share we created previously.
    CloudFileShare share = fileClient.GetShareReference("logs");

    //Ensure that the share exists.
    if (share.Exists())
    {
        string policyName = "sampleSharePolicy" + DateTime.UtcNow.Ticks;

        //Create a new shared access policy and define its constraints.
        SharedAccessFilePolicy sharedPolicy = new SharedAccessFilePolicy()
            {
                SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
                Permissions = SharedAccessFilePermissions.Read | SharedAccessFilePermissions.Write
            };

        //Get existing permissions for the share.
        FileSharePermissions permissions = share.GetPermissions();

        //Add the shared access policy to the share's policies. Note that each policy must have a unique name.
        permissions.SharedAccessPolicies.Add(policyName, sharedPolicy);
        share.SetPermissions(permissions);

        //Generate a SAS for a file in the share and associate this access policy with it.
        CloudFileDirectory rootDir = share.GetRootDirectoryReference();
        CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("CustomLogs");
        CloudFile file = sampleDir.GetFileReference("Log1.txt");
        string sasToken = file.GetSharedAccessSignature(null, policyName);
        Uri fileSasUri = new Uri(file.StorageUri.PrimaryUri.ToString() + sasToken);

        //Create a new CloudFile object from the SAS, and write some text to the file. 
        CloudFile fileSas = new CloudFile(fileSasUri);
        fileSas.UploadText("This write operation is authenticated via SAS.");
        Console.WriteLine(fileSas.DownloadText());
    }

有关创建和使用共享访问签名的更多信息，请参阅[共享访问签名：了解 SAS 模型](/documentation/articles/storage-dotnet-shared-access-signature-part-1)和[创建 SAS 并将 SAS 用于 Blob 服务](/documentation/articles/storage-dotnet-shared-access-signature-part-2)。

## 复制文件

从 Azure 存储空间客户端库的 5.x 版开始，可以将一个文件复制到另一个文件，将一个文件复制到一个 Blob，或将一个 Blob 复制到一个文件。下面，我们演示如何以编程方式执行这些复制操作。

还可以使用 AzCopy 将一个文件复制到另一个文件或将一个 Blob 复制到一个文件，反之亦然。请参阅[如何将 AzCopy 与 Windows Azure 存储空间一起使用](/documentation/articles/storage-use-azcopy#copy-files-in-azure-file-storage-with-azcopy-preview-version-only)以了解有关用 AzCopy 复制文件的详细信息。

> [AZURE.NOTE] 如果将一个 Blob 复制到一个文件，或将一个文件复制到一个 Blob，必须使用共享访问签名 (SAS) 对源对象进行身份验证，即使你在同一存储帐户内进行复制。

### 将一个文件复制到另一个文件

下面的示例将一个文件复制到同一共享中的另一个文件。因为此操作在同一存储帐户中的文件之间进行复制，可以使用共享密钥身份验证来进行复制。

    //Parse the connection string for the storage account.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

    //Create a CloudFileClient object for credentialed access to File storage.
    CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

    //Get a reference to the file share we created previously.
    CloudFileShare share = fileClient.GetShareReference("logs");

    //Ensure that the share exists.
    if (share.Exists())
    {
        //Get a reference to the root directory for the share.
        CloudFileDirectory rootDir = share.GetRootDirectoryReference();

        //Get a reference to the directory we created previously.
        CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("CustomLogs");

        //Ensure that the directory exists.
        if (sampleDir.Exists())
        {
            //Get a reference to the file we created previously.
            CloudFile sourceFile = sampleDir.GetFileReference("Log1.txt");

            //Ensure that the source file exists.
            if (sourceFile.Exists())
            {
                //Get a reference to the destination file.
                CloudFile destFile = sampleDir.GetFileReference("Log1Copy.txt");

                //Start the copy operation.
                destFile.StartCopy(sourceFile);

                //Write the contents of the destination file to the console window.
                Console.WriteLine(destFile.DownloadText());
            }
        }
    }


### 将一个文件复制到一个 Blob

下面的示例创建一个文件并将其复制到同一存储帐户内的一个 Blob。该示例为源文件创建一个 SAS，服务在复制操作期间使用该 SAS 验证对源文件的访问。

    //Parse the connection string for the storage account.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

    //Create a CloudFileClient object for credentialed access to File storage.
    CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

    //Create a new file share, if it does not already exist.
    CloudFileShare share = fileClient.GetShareReference("sample-share");
    share.CreateIfNotExists();

    //Create a new file in the root directory.
    CloudFile sourceFile = share.GetRootDirectoryReference().GetFileReference("sample-file.txt");
    sourceFile.UploadText("A sample file in the root directory.");

    //Get a reference to the blob to which the file will be copied.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
    CloudBlobContainer container = blobClient.GetContainerReference("sample-container");
    container.CreateIfNotExists();
    CloudBlockBlob destBlob = container.GetBlockBlobReference("sample-blob.txt");

    //Create a SAS for the file that's valid for 24 hours.
    //Note that when you are copying a file to a blob, or a blob to a file, you must use a SAS
    //to authenticate access to the source object, even if you are copying within the same 
    //storage account.
    string fileSas = sourceFile.GetSharedAccessSignature(new SharedAccessFilePolicy()
    {
        //Only read permissions are required for the source file.
        Permissions = SharedAccessFilePermissions.Read,
        SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24)
    });

    //Construct the URI to the source file, including the SAS token. 
    Uri fileSasUri = new Uri(sourceFile.StorageUri.PrimaryUri.ToString() + fileSas);

    //Copy the file to the blob.
    destBlob.StartCopy(fileSasUri);

    //Write the contents of the file to the console window.
    Console.WriteLine("Source file contents: {0}", sourceFile.DownloadText());
    Console.WriteLine("Destination blob contents: {0}", destBlob.DownloadText());

可以用相同的方式将一个 Blob 复制到一个文件。如果源对象是一个 Blob，则创建一个 SAS 以复制操作期间验证对该 Blob 的访问。

## 在 Linux 上使用文件存储

若要从 Linux 创建和管理文件共享，请使用 Azure CLI。请参阅[将 Azure CLI 用于 Azure 存储空间](/documentation/articles/storage-azure-cli#create-and-manage-file-shares)以了解有关将 Azure CLI 用于文件存储的信息。

可以从运行 Linux 的虚拟机装载 Azure 文件共享。在创建 Azure 虚拟机时，可以从 Azure 映像库中指定支持 SMB 2.1 的 Linux 映像，例如最新版本的 Ubuntu。然而，任何支持 SMB 2.1 的 Linux 分发版都可以装载 Azure 文件共享。

若要了解有关如何在 Linux 上装载 Azure 文件共享的更多信息，请参阅第 9 频道上的[通过 Azure 文件在 Linux 上共享存储预览 - 第 1 部分](http://channel9.msdn.com/Blogs/Open/Shared-storage-on-Linux-via-Azure-Files-Preview-Part-1)。

## 后续步骤

请参阅以下链接以获取有关 Azure 文件存储的更多信息。

### 教程和参考

- [.NET 存储客户端库参考](https://msdn.microsoft.com/zh-cn/library/azure/dn261237.aspx)
- [文件服务 REST API 参考](http://msdn.microsoft.com/zh-cn/library/azure/dn167006.aspx)
- [如何将 AzCopy 与 Windows Azure 存储空间一起使用](/documentation/articles/storage-use-azcopy)
- [对 Azure 存储空间使用 Azure PowerShell](/documentation/articles/storage-powershell-guide-full)
- [将 Azure CLI 用于 Azure 存储空间](/documentation/articles/storage-azure-cli)

### 博客文章

- [Windows Azure 文件服务简介](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/12/introducing-microsoft-azure-file-service.aspx)
- [将连接保存到 Windows Azure 文件中](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/27/persisting-connections-to-microsoft-azure-files.aspx)
 

<!---HONumber=70-->