<properties urlDisplayName="File Service" pageTitle="如何使用 Azure 文件存储 | Microsoft Azure" metaKeywords="Get started Azure file  Azure file share  Azure file shares  Azure file   Azure file storage   Azure file .NET   Azure file C#   Azure file PowerShell" description="Learn how to use Microsoft Azure File storage to create file shares and manage file content. Samples are written in PowerShell and C#." metaCanonical="" disqusComments="1" umbracoNaviHide="1" services="storage" documentationCenter=".NET" title="How to use Microsoft Azure File storage in .NET" authors="tamram" manager="adinah" />

<tags ms.service="storage" ms.workload="storage" ms.tgt_pltfrm="na" ms.devlang="dotnet" ms.topic="article" ms.date="11/10/2014" ms.author="tamram" />

# 如何使用 Azure 文件存储

在此入门指南中，我们将演示使用 Microsoft Azure 文件存储的基础知识。首先，我们将使用 PowerShell 来演示如何创建新的 Azure 文件共享、如何添加目录、如何将本地文件上载到该共享，以及如何列出该目录中的文件。然后，我们将演示如何从 Azure 虚拟机装载文件共享，就像装载任何 SMB 共享一样。

对于可能想要从本地应用程序以及 Azure 虚拟机或云服务访问共享中的文件的用户，我们将演示如何使用 Azure .NET 存储客户端库通过桌面应用程序处理文件共享。

> [WACOM.NOTE] 运行本指南中的 .NET 代码示例需要 Azure .NET 存储客户端库 4.x 或更高版本。存储客户端库可通过以下方式获得：[NuGet](https://www.nuget.org/packages/WindowsAzure.Storage/).


##目录

-   [什么是文件存储？][]
-   [文件存储概念][]
-   [创建 Azure 存储帐户][]
-   [使用 PowerShell 创建文件共享][]
-	[从 Azure 虚拟机装载共享][]
-   [创建本地应用程序以访问文件存储][]
-   [后续步骤][]


##<a name="what-is-file-storage"></a>什么是 Azure 文件存储？

文件存储使用标准 SMB 2.1 协议为应用程序提供共享存储。Microsoft Azure 虚拟机和云服务可通过装载的共享在应用程序组件之间共享文件数据，本地应用程序可通过文件存储 API 来访问共享中的文件数据。

在 Azure 虚拟机或云服务中运行的应用程序可以装载文件存储共享以访问文件数据，就像桌面应用程序可以装载典型 SMB 共享一样。任意数量的 Azure 虚拟机或角色可以同时装载并访问文件存储共享。

由于文件存储共享是标准的 SMB 2.1 文件共享，在 Azure 中运行的应用程序可通过文件 I/O API 来访问共享中的数据。因此，开发人员可以利用其现有代码和技术迁移现有应用程序。IT 专业人员在管理 Azure 应用程序的过程中，可以使用 PowerShell cmdlet 来创建、装载和管理文件存储共享。本指南将演示这两方面的示例。

文件存储的常见用途包括：

- 迁移依赖文件共享在 Azure 虚拟机或云服务中运行的本地应用程序，而无需进行昂贵的重写操作
- 存储共享的应用程序设置，例如在配置文件中进行存储
- 在共享位置存储诊断数据，如日志、指标和故障转储 
- 存储开发或管理 Azure 虚拟机或云服务所需的工具和实用工具

##<a name="file-storage-concepts"></a>文件存储概念

文件存储包含以下组件：

![files-concepts][files-concepts]


-   **存储帐户：** 对 Azure 存储空间进行的所有访问
都要通过存储帐户完成。有关存储帐户容量的详细信息，请参阅 [Azure 存储空间可伸缩性和性能目标](http://msdn.microsoft.com/zh-cn/library/dn249410.aspx)。

-   **共享：**文件存储共享是 Azure 中的 SMB 2.1 文件共享。 
所有目录和文件都必须在父共享中创建。一个帐户可以包含无限数量的共享，一个共享可以存储无限数量的文件，直到达到存储帐户的容量限制为止。

-   **Directory:**可选的目录层次结构。 

-	**文件：**共享中的文件。文件大小最大可以为 1 TB。

-   **URL 格式：**使用以下 URL 格式可访问文件：https://`<storage
    account>`.file.core.chinacloudapi.cn/`<share>`/`<directory/directories>`/`<file>`  
    
    可使用以下示例 URL 寻址上图中的
某个文件：  
    `http://acmecorp.file.core.chinacloudapi.cn/cloudfiles/diagnostics/log.txt`



有关如何命名共享、目录和文件的详细信息，请参阅[命名和引用共享、目录、文件和元数据](http://msdn.microsoft.com/zh-cn/library/azure/dn167011.aspx)。

##<a name="use-cmdlets"></a>使用 PowerShell 创建文件共享

###为 Azure 存储空间安装 PowerShell cmdlet

若要准备使用 PowerShell，请下载并安装 Azure PowerShell cmdlet。有关安装点和安装说明，请参阅[如何安装和配置 Azure PowerShell](/zh-cn/documentation/articles/install-configure-powershell/)。

> [WACOM.NOTE] 文件服务的 PowerShell cmdlet 只在最新的 0.8.5 版及更高版本的 Azure PowerShell 模块中提供。建议你下载并安装最新的 Azure PowerShell 模块或升级到最新模块。

通过单击**"开始"**并键入**"Microsoft Azure PowerShell"**打开 Azure PowerShell 窗口。Azure PowerShell 窗口将为你加载 Azure PowerShell 模块。

###为存储帐户和密钥创建上下文

现在，将创建存储帐户上下文。该上下文封装了帐户名称和帐户密钥。请将下面示例中的"account-name"和"account-key"替换为你的帐户名称和密钥：

    # create a context for account and key
    $ctx=New-AzureStorageContext account-name account-key
    
###创建新的文件共享

接下来，在此示例中创建名为"sampleshare"的新共享：

    # create a new share
    $s = New-AzureStorageShare sampleshare -Context $ctx

现在，你在文件存储中已有一个文件共享。接下来，我们将添加目录和文件。

###在文件共享中创建目录

接下来，将在共享中创建目录。在下面的示例中，目录名为"sampledir"：

    # create a directory in the share
    New-AzureStorageDirectory -Share $s -Path sampledir

###将本地文件上载到目录

现在，将本地文件上载到该目录。以下示例从"C:\temp\samplefile.txt"上载文件。请编辑文件路径，使其指向你本地计算机上的有效文件： 
    
    # upload a local file to the new directory
    Set-AzureStorageFileContent -Share $s -Source C:\temp\samplefile.txt -Path sampledir

###列出目录中的文件

可以列出目录的文件，以便查看其中的文件。此命令也将列出子目录，但在此示例中没有子目录，因此只列出文件。  

    # list files in the new directory
    Get-AzureStorageFile -Share $s -Path sampledir

##<a name="mount-share"></a>从 Azure 虚拟机装载共享

为了演示如何装载 Azure 文件共享，现在我们将创建一个 Azure 虚拟机，并远程登录到它内部以装载共享。 

1. 首先，按照 [创建运行 Windows Server 的虚拟机](/zh-cn/documentation/articles/virtual-machines-windows-tutorial/) 中的说明创建一个新的 Azure 虚拟机。
2. 然后，按照 [如何登录到运行 Windows Server 的虚拟机](/zh-cn/documentation/articles/virtual-machines-log-on-windows-server/) 中的说明远程登录到该虚拟机内部。
3. 在该虚拟机上打开 PowerShell 窗口。 

###保存虚拟机的存储帐户凭据

装载到文件共享之前，先在虚拟机上保存存储帐户凭据。当虚拟机重新启动时，此步骤允许 Windows 自动重新连接到文件共享。若要保存帐户凭据，请在虚拟机上的 PowerShell 窗口中执行"cmdkey"命令。请将"<storage-account>"替换为你的存储帐户名称，将"<account-key>"替换为你的存储帐户密钥：

	cmdkey /add:<storage-account>.file.core.chinacloudapi.cn /user:<storage-account> /pass:<account-key>

现在，当虚拟机重新启动时，Windows 将重新连接到你的文件共享。可以通过在 PowerShell 窗口中执行"net use"命令来验证是否已重新连接共享。

###使用保存的凭据装载文件共享

建立与虚拟机的远程连接后，便可以使用以下语法执行"net use"命令来装载文件共享了。请将"<storage-account>"替换为你的存储帐户名称，将"<share-name>"替换为你的文件存储共享名称。

	net use z: \\<storage-account>.file.core.chinacloudapi.cn\<share-name>

> [WACOM.NOTE] 由于你已在上一步中保存了存储帐户凭据，因此你无需为"net use"命令提供这些凭据。如果你尚未保存凭据，请提供凭据作为传递给"net use"命令的参数。请将"<storage-account>"替换为你的存储帐户名称，将"<share-name>"替换为你的文件存储共享名称，将"<account-key>"替换为你的存储帐户密钥：
	   
	net use z: \\<storage-account>.file.core.chinacloudapi.cn\<share-name> /u:<storage-account> <account-key>

现在，你可以使用虚拟机中的文件存储共享，就像使用任何其他驱动器一样。你可以从命令提示符发出标准文件命令，也可以从文件资源管理器查看已装载的共享及其内容。你也可以在虚拟机中运行代码，以便访问使用标准 Windows 文件 I/O API（例如 .NET Framework 中由 [System.IO 命名空间](http://msdn.microsoft.com/zh-cn/library/gg145019(v=vs.110).aspx) 提供的 API）的文件共享。 

你还可以通过远程登录到角色中从 Azure 云服务中运行的角色装载文件共享。

##<a name="create-console-app"></a>创建本地应用程序以使用文件存储

你可以从 Azure 中运行的虚拟机或云服务装载文件存储共享，如上所示。但是，你不能从本地应用程序装载文件存储共享。若要从本地应用程序访问共享数据，必须使用文件存储 API。此示例演示如何通过 [Azure .NET 存储客户端库](http://msdn.microsoft.com/zh-cn/library/wa_storage_30_reference_home.aspx) 使用文件共享。 

为了演示如何通过本地应用程序使用 API，我们将创建一个在桌面上运行的简单控制台应用程序。

###创建控制台应用程序，并获取程序集

若要在 Visual Studio 中创建新的控制台应用程序并安装 Azure 存储 NuGet 包，请执行以下操作：

1. 在 Visual Studio 中，选择**"文件"**->**"新建项目"**，然后从 Visual C# 模板列表中选择**"Windows"**->**"控制台应用程序"**。
2. 提供控制台应用程序的名称，并单击**"确定"**。
3. 创建项目后，在解决方案资源管理器中，右键单击该项目并选择**"管理 NuGet 包"**。在线搜索"MicrosoftAzure.Storage"，然后单击**"安装"**以安装 Azure 存储包和依赖项。

###将存储帐户凭据保存到 app.config 文件

接下来，将你的凭据保存到项目的 app.config 文件中。编辑 app.config 文件，使其看起来类似于下面的示例，将"myaccount"替换为你的存储帐户名称，将"mykey"替换为你的存储帐户密钥：

    <?xml version="1.0" encoding="utf-8" ?>
	<configuration>
		<startup> 
			<supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5" />
		</startup>
		<appSettings>
			<add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=mykey" />
		</appSettings>
	</configuration>

> [WACOM.NOTE] 最新版本的 Azure 存储模拟器不支持文件存储。连接字符串必须针对云中可以访问文件预览的 Azure 存储帐户。


###添加命名空间声明
从解决方案资源管理器打开 program.cs 文件，并在该文件顶部添加以下命名空间声明：

    using Microsoft.WindowsAzure;
	using Microsoft.WindowsAzure.Storage;
	using Microsoft.WindowsAzure.Storage.File;

###以编程方式检索连接字符串
可以使用"Microsoft.WindowsAzure.CloudConfigurationManager"类或"System.Configuration.ConfigurationManager"类从 app.config 文件中检索保存的凭据。此处的示例显示如何使用"CloudConfigurationManager"类检索凭据，并使用"CloudStorageAccount"类封装这些凭据。将以下代码添加到 Program.cs 的 Main() 方法中：

    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

###以编程方式访问文件存储共享

接下来，将以下代码添加到上面显示的 Main() 方法中用于检索连接字符串的代码的后面。此代码将获取我们先前创建的文件的引用，并将其内容输出到控制台窗口中。

	//Create a CloudFileClient object for credentialed access to File storage.
    CloudFileClient fileClient = storageAccount.CreateCloudFileClient();

	//Get a reference to the file share we created previously.
    CloudFileShare share = fileClient.GetShareReference("sampleshare");

	//Ensure that the share exists.
    if (share.Exists())
    {
		//Get a reference to the root directory for the share.
        CloudFileDirectory rootDir = share.GetRootDirectoryReference();

		//Get a reference to the sampledir directory we created previously.
        CloudFileDirectory sampleDir = rootDir.GetDirectoryReference("sampledir");

		//Ensure that the directory exists.
        if (sampleDir.Exists())
        {
			//Get a reference to the file we created previously.
            CloudFile file = sampleDir.GetFileReference("samplefile.txt");

			//Ensure that the file exists.
            if (file.Exists())
            {
				//Write the contents of the file to the console window.
                Console.WriteLine(file.DownloadTextAsync().Result);
            }
        }
    }

运行控制台应用程序以查看输出。

## <a name="next-steps"></a>后续步骤

现在，你已了解文件存储的基础知识，单击下面的链接
可获取更详细的信息。
<ul>
<li>查看文件服务参考文档，了解有关可用 API 的完整详细信息：
  <ul>
    <li><a href="http://msdn.microsoft.com/zh-cn/library/wa_storage_30_reference_home.aspx">.NET 存储客户端库参考</a>
    </li>
    <li><a href="http://msdn.microsoft.com/zh-cn/library/azure/dn167006.aspx">文件服务 REST API 参考</a></li>
  </ul>
</li>
<li>查看与文件服务有关的 Azure 存储团队的博客文章：
  <ul>
    <li><a href="http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/12/introducing-microsoft-azure-file-service.aspx">Microsoft Azure 文件服务简介</a>
    </li>
    <li><a href="http://blogs.msdn.com/b/windowsazurestorage/archive/2014/05/27/persisting-connections-to-microsoft-azure-files.aspx">将连接保存到 Microsoft Azure 文件中</a></li>
  </ul>
</li><li>查看更多功能指南，以了解在 Azure 中存储数据的其他方式。
  <ul>
    <li>使用 <a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-blobs/">Blob 存储</a>存储非结构化数据。</li>
    <li>使用<a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-tables/">表存储</a>存储结构化数据。</li>
    <li>使用<a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-queues/">队列存储</a>可靠地存储消息。</li>
    <li>使用 <a href="/zh-cn/documentation/articles/sql-database-dotnet-how-to-use/">SQL Database</a> 存储关系数据。</li>
  </ul>
</li>
</ul>

[后续步骤]: #next-steps
[什么是文件存储？]: #what-is-file-storage 
[文件存储概念]: #file-storage-concepts
[创建 Azure 存储帐户]: #create-account
[使用 PowerShell 创建文件共享]: #use-cmdlets
[从 Azure 虚拟机装载共享]: #mount-share
[创建本地应用程序以访问文件存储]: #create-console-app

[files-concepts]: ./media/storage-dotnet-how-to-use-files/files-concepts.png

