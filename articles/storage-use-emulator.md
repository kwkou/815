<properties 
	pageTitle="使用 Azure 存储模拟器进行开发和测试 | Azure" 
	description="Azure 存储模拟器为针对 Azure 存储的开发和测试提供了免费的本地开发环境。了解有关存储模拟器的信息，包括如何对请求进行身份验证、如何从您的应用程序连接到模拟器和如何使用命令行工具。" 
	services="storage" 
	documentationCenter="" 
	authors="tamram" 
	manager="adinah" 
	editor=""/>
<tags 
	ms.service="storage" 
	ms.date="03/28/2016" 
	wacn.date="05/23/2016"/>

# 使用 Azure 存储模拟器进行开发和测试

## 概述

Azure 存储模拟器提供了一个模拟 Azure Blob、队列和表服务以进行开发的本地环境。使用存储模拟器，您可以在本地针对存储服务测试应用程序，而无需创建 Azure 订阅且不会产生任何费用。如果您对应用程序在模拟器中的工作情况感到满意，则可以切换到在云中使用 Azure 存储帐户。

> [AZURE.NOTE] 存储模拟器作为 [Azure SDK](/downloads/) 的一部分提供。此外，还可以使用[独立安装程序](https://go.microsoft.com/fwlink/?linkid=717179&clcid=0x409)来安装存储模拟器。若要配置存储模拟器，必须在计算机上具有管理权限。
>  
> 请注意，在一个版本的存储模拟器中创建的数据不保证在使用不同版本时可以访问。如果需要长期保存数据，建议将该数据存储在 Azure 存储帐户中，而不是存储在存储模拟器中。

## 存储模拟器的工作原理
 
存储模拟器使用本地 Microsoft SQL Server 实例和本地文件系统来模拟 Azure 存储服务。默认情况下，存储模拟器使用 Microsoft SQL Server 2012 Express LocalDB 中的数据库。您可以选择将存储模拟器配置为访问 SQL Server 的本地实例而不是 LocalDB 实例。请参阅下面的[启动并初始化存储模拟器](#start-and-initialize-the-storage-emulator)以获取详细信息。

你可以安装 SQL Server Management Studio Express 来管理 LocalDB 安装。存储模拟器使用 Windows 身份验证连接到 SQL Server 或 LocalDB。

存储模拟器与 Azure 存储服务之间存在一些功能差异。有关这些差异的详细信息，请参阅[存储模拟器与 Azure 存储之间的差异](#differences-between-the-storage-emulator-and-azure-storage)。

## 针对存储模拟器的请求进行身份验证

与在云中的 Azure 存储相同、针对存储模拟器进行的每个请求都必须进行身份验证，除非它是匿名请求。可以使用共享密钥身份验证或使用共享的访问签名 (SAS) 针对存储模拟器的请求进行身份验证。

### 使用共享密钥凭据进行身份验证

[AZURE.INCLUDE [storage-emulator-connection-string-include](../includes/storage-emulator-connection-string-include.md)]

有关连接字符串的详细信息，请参阅[配置到 Azure 存储的连接字符串](/documentation/articles/storage-configure-connection-string/)。

### 使用共享访问签名进行身份验证 

某些 Azure 存储客户端库（诸如 Xamarin 库）仅支持使用共享的访问签名 (SAS) 令牌进行身份验证。您将需要使用支持共享密钥身份验证的工具或应用程序创建此 SAS 令牌。生成该 SAS 令牌的简单方法是通过 Azure PowerShell：

1. 安装 Azure PowerShell（如果尚未安装）。建议使用 Azure PowerShell cmdlet 最新版本。请查看[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/#Install) 以获取安装说明。

2. 请打开 Azure PowerShell 并运行以下命令。请记住要使用您自己的凭据替换 *ACCOUNT_NAME* 和 *ACCOUNT_KEY = =*。将 *CONTAINER_NAME* 替换为您选择的名称。

		$context = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName "ACCOUNT_NAME" -StorageAccountKey "ACCOUNT_KEY=="
		
		New-AzureStorageContainer CONTAINER_NAME -Permission Off -Context $context
		
		$now = Get-Date 
		
		New-AzureStorageContainerSASToken -Name CONTAINER_NAME -Permission rwdl -ExpiryTime $now.AddDays(1.0) -Context $context -FullUri

得到的新容器的共享访问签名 URI 应如下所示：

	https://storageaccount.blob.core.chinacloudapi.cn/sascontainer?sv=2012-02-12&se=2015-07-08T00%3A12%3A08Z&sr=c&sp=wl&sig=t%2BbzU9%2B7ry4okULN9S0wst%2F8MCUhTjrHyV9rDNLSe8g%3Dsss

此示例中创建的共享访问签名的有效期为一天。该签名将授予对容器内 Blob 的完整权限（例如，读取、写入、删除和列出）。

有关共享访问签名的详细信息，请参阅[共享访问签名：了解 SAS 模型](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)。


##<a id="start-and-initialize-the-storage-emulator"></a> 启动和初始化存储模拟器

若要启动 Azure 存储模拟器，请选择“开始”按钮或按 Windows 键。开始键入“Azure 存储模拟器”，然后从应用程序列表中选择该模拟器。

当运行模拟器时，您将在 Windows 任务栏通知区域中看到一个图标。

存储模拟器在启动时，将显示命令行窗口。可以使用此命令行窗口来启动和停止存储模拟器以及清除数据、获取当前状态和初始化模拟器。有关详细信息，请参阅[存储模拟器命令行工具参考](#storage-emulator-command-line-tool-reference)。

关闭命令行窗口后，存储模拟器将继续运行。若要重新显示命令行，请执行上述步骤，就像启动存储模拟器一样。

首次运行存储模拟器时，会为你初始化本地存储环境。初始化过程在 LocalDB 中创建一个数据库，并为每个本地存储服务保留 HTTP 端口。

存储模拟器默认情况下安装到 C:\\Program Files (x86) \\Microsoft SDKs\\Azure\\Storage 模拟器目录。

### 初始化存储模拟器以使用其他的 SQL 数据库

可以使用存储模拟器命令行工具初始化存储模拟器，使其指向默认 LocalDB 实例以外的其他 SQL 数据库实例。请注意，您必须使用管理权限运行命令行工具来初始化存储模拟器的后端数据库：

1. 单击“开始”按钮或按 **Windows** 键。开始键入 `Azure Storage Emulator` 并在其启动存储模拟器命令行工具时将其选中。
2. 在命令提示符窗口中，键入以下命令，其中 `<SQLServerInstance>` 是 SQL Server 实例的名称。若要使用 LocalDb，请指定 `(localdb)\v11.0` 作为 SQL Server 实例。

		AzureStorageEmulator init /server <SQLServerInstance> 
    
	也可以使用以下命令，该命令指示模拟器使用默认 SQL Server 实例：

    	AzureStorageEmulator init /server .\\ 

	或者，可以使用以下命令将数据库重新初始化为默认的 LocalDB 实例：

    	AzureStorageEmulator init /forceCreate 

有关这些命令的详细信息，请参阅[存储模拟器命令行工具参考](#storage-emulator-command-line-tool-reference)。

##<a id="addressing-resources-in-the-storage-emulator"></a> 对存储模拟器中的资源进行寻址

存储模拟器的服务终结点不同于 Azure 存储帐户。不同之处是因为本地计算机不执行域名解析，因此存储模拟器终结点需要本地地址而不是域名。

在对 Azure 存储帐户中的资源进行寻址时，可以使用以下方案，其中帐户名称是 URI 主机名的一部分，要寻址的资源是 URI 路径的一部分：

    <http|https>://<account-name>.<service-name>.core.chinacloudapi.cn/<resource-path>

例如，下面的 URI 是 Azure 存储帐户的 Blob 中的有效地址：

	https://myaccount.blob.core.chinacloudapi.cn/mycontainer/myblob.txt

在存储模拟器中，因为本地计算机不执行域名解析，帐户名称将是 URI 路径（而非主机名）的一部分。可以对在存储模拟器中运行的资源使用以下方案：

    http://<local-machine-address>:<port>/<account-name>/<resource-path>

例如，以下地址可用于访问存储模拟器中的 blob：

    http://127.0.0.1:10000/myaccount/mycontainer/myblob.txt

存储模拟器的服务终结点是：

	Blob Service: http://127.0.0.1:10000/<account-name>/<resource-path>
	Queue Service: http://127.0.0.1:10001/<account-name>/<resource-path>
	Table Service: http://127.0.0.1:10002/<account-name>/<resource-path>

### 使用 RA-GRS 对帐户辅助副本进行寻址

存储模拟器从 3.1 版开始，支持读取访问地域冗余复制 (RA-GRS)。对于同时位于云中和本地模拟器中的存储资源，可以通过在帐户名称后面追加 -secondary 来访问辅助位置。例如，以下地址可用于访问使用存储模拟器中的只读辅助副本的 blob：

    http://127.0.0.1:10000/myaccount-secondary/mycontainer/myblob.txt 

> [AZURE.NOTE] 若要使用存储模拟器以编程方式访问辅助副本，请使用 Storage Client Library for .NET 3.2 版或更高版本。有关详细信息，请参阅[用于 .NET 的 Microsoft Azure 存储客户端库](https://msdn.microsoft.com/zh-cn/library/azure/dn261237.aspx)。

##<a id="storage-emulator-command-line-tool-reference"></a> 存储模拟器命令行工具参考

从 3.0 版开始，当您启动存储模拟器时，将看到命令提示符弹出窗口。可使用命令提示符窗口启动和停止模拟器以及查询状态和执行其他操作。

> [AZURE.NOTE] 如果已安装 Azure 计算模拟器，则在启动存储模拟器时，将显示一个系统任务栏图标。右键单击该图标可显示一个菜单，其中提供了启动和停止存储模拟器的图形化方式。

### 命令行语法

	AzureStorageEmulator [start] [stop] [status] [clear] [init] [help]

### 选项

若要查看选项列表，请在命令提示符下键入 `/help`。

| 选项 | 说明 | 命令 | 参数 |
|--------|----------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| **启动** | 启动存储模拟器。 | `AzureStorageEmulator start [-inprocess]` | -inprocess：在当前进程中启动模拟器而不是创建一个新的进程。 |
| **停止** | 停止存储模拟器。 | `AzureStorageEmulator stop` | |
| **状态** | 打印存储模拟器的状态。 | `AzureStorageEmulator status` | |
| **清除** | 清除命令行上指定的所有服务中的数据。 | `AzureStorageEmulator clear [blob] [table] [queue] [all]                                                    `| blob：清除 Blob 数据。<br/>queue：清除队列数据。<br/>table：清除表数据。<br/>all：清除所有服务中的所有数据。 |
| **Init** | 执行一次性初始化以设置模拟器。 | `AzureStorageEmulator.exe init [-server serverName] [-sqlinstance instanceName] [-forcecreate] [-inprocess]` | -server serverName\\instanceName：指定托管 SQL 实例的服务器。<br/>-sqlinstance instanceName：指定要用于默认服务器实例中的 SQL 实例的名称。<br/>-forcecreate：强制创建 SQL 数据库，即使该数据库已经存在。<br/>-inprocess：在当前进程而不是生成新的进程中执行初始化。您必须使用提升的权限启动当前进程以执行初始化。 |
                                                                                                                  
##<a id="differences-between-the-storage-emulator-and-azure-storage"></a> 存储模拟器与 Azure 存储之间的差异

由于存储模拟器是在本地的 SQL 实例中运行的模拟环境，模拟器与云中的 Azure 存储帐户之间存在一些功能差异：

- 存储模拟器只支持单一固定的帐户和众所周知的身份验证密钥。

- 存储模拟器不是可扩展的存储服务，并且不支持大量并发客户端。

- 如[对存储模拟器中的资源进行寻址](#addressing-resources-in-the-storage-emulator)中所述，存储模拟器与 Azure 存储帐户中的资源以不同方式寻址。存在这种差异是因为在云中可进行域名解析，但在本地计算机上不提供域名解析。

- 存储模拟器从 3.1 版开始，支持读取访问地域冗余复制 (RA-GRS)。在模拟器中，所有帐户都已启用 RA-GRS，在主要和辅助副本之间不会有任何延迟。获取 Blob 服务统计信息、获取队列服务统计信息和获取表服务统计信息操作在帐户辅助副本上受支持，并且将始终根据基础 SQL 数据库返回 `LastSyncTime` 响应元素的值作为当前时间。

	若要使用存储模拟器以编程方式访问辅助副本，请使用 Storage Client Library for .NET 3.2 版或更高版本。有关详细信息，请参阅[用于 .NET 的 Microsoft Azure 存储客户端库](https://msdn.microsoft.com/zh-cn/library/azure/dn261237.aspx)。

- 文件服务和 SMB 协议服务终结点当前在存储模拟器中不受支持。

- 如果您所使用的模拟器版本不支持您所使用的存储服务的版本，存储模拟器将返回 VersionNotSupportedByEmulator 错误（HTTP 状态代码 400 - 错误请求）。

### Blob 存储的差异 

以下差异适用于模拟器中的 Blob 存储：

- 存储模拟器仅支持最大为 2 GB 的 Blob。

- 对存在于存储模拟器中并具有活动租约的 Blob 执行的放置 Blob 操作可能会成功，即使在请求中未指定租约 ID。

- 追加 Blob 操作不受模拟器支持。尝试对追加 Blob 执行的操作将返回 FeatureNotSupportedByEmulator 错误（HTTP 状态代码 400 - 错误请求）。

### 表存储的差异 

以下差异适用于模拟器中的表存储：

- 存储模拟器中的表服务中的日期属性仅支持 SQL Server 2005 支持的范围（即，要求它们晚于 1753 年 1 月 1 日）。1753 年 1 月 1 日之前的所有日期都会更改为此值。日期的精度仅限于 SQL Server 2005 的精度，这意味着日期将精确到 1/300 秒。

- 存储模拟器支持小于 512 个字节的分区键和行键属性值（每个）。此外，帐户名称、表名和键属性名称合在一起的总大小不能超过 900 个字节。

- 存储模拟器中的表中的某行的总大小被限制为小于 1 MB。

- 在存储模拟器中，数据类型 `Edm.Guid` 或 `Edm.Binary` 的属性仅支持查询筛选器字符串中的 `Equal (eq)` 和 `NotEqual (ne)` 比较运算符。

### 队列存储的差异

模拟器中的队列存储没有任何差异。

## 存储模拟器发行说明

### 版本 4.3

- 存储模拟器现在支持 Blob、队列和表服务终结点上的 2015-07-08 版本的存储服务。

### 版本 4.2

- 存储模拟器现在支持 Blob、队列和表服务终结点上的 2015-04-05 版本的存储服务。

### 4.1 版

- 存储模拟器现在支持 Blob、队列和表服务终结点上的 2015-02-21 版本的存储服务，新的“追加 Blob”功能除外。 

- 如果模拟器尚不支持您使用的存储服务的版本，存储模拟器现在会返回有意义的错误消息。我们建议使用最新版本的模拟器。如果您遇到 VersionNotSupportedByEmulator 错误（HTTP 状态代码 400 - 错误请求），请下载最新版本的存储模拟器。

- 修复了在并发合并操作期间发生争用情况下导致表实体数据不正确的错误。

### 4.0 版

- 存储模拟器可执行文件已重命名为 *AzureStorageEmulator.exe*。

### 3.2 版
- 存储模拟器现在支持 Blob、队列和表服务终结点上的 2014-02-14 版本的存储服务。请注意，文件服务终结点目前在存储模拟器中不受支持。请参阅 [Azure 存储服务的版本控制](https://msdn.microsoft.com/zh-cn/library/azure/dd894041.aspx)以了解有关 2014-02-14 版本的详细信息。

### 3.1 版
- 在存储模拟器中现在支持读取访问异地冗余存储 (RA-GRS)。获取 Blob 服务统计信息、获取队列服务统计信息和获取表服务统计信息 API 在帐户辅助副本上受支持，并且将始终根据基础 SQL 数据库返回 LastSyncTime 响应元素的值作为当前时间。若要使用存储模拟器以编程方式访问辅助副本，请使用 Storage Client Library for .NET 3.2 版或更高版本。有关详细信息，请参阅 .NET 的 Azure 存储空间客户端库。

### 版本 3.0
- Azure 存储模拟器中不再与计算模拟器在同一个包中提供。

- 为支持可编脚本的命令行界面，已弃用存储模拟器图形用户界面。有关命令行界面的详细信息，请参阅“存储模拟器命令行工具参考”。图形界面将继续存在于 3.0 版中，但仅在安装了计算模拟器的情况下通过右键单击系统托盘图标并选择“显示存储模拟器用户界面”来访问。

- 现在完全支持版本 2013年-08-15 的 Azure 存储服务。（以前仅存储模拟器 2.2.1 预览版本支持此版本。）


<!---HONumber=Mooncake_0516_2016-->