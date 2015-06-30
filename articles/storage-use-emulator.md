<properties 
	pageTitle="使用 Azure 存储模拟器进行开发和测试" 
	description="了解如何使用 Azure 存储模拟器进行开发和测试。" 
	services="storage" 
	documentationCenter="" 
	authors="tamram" 
	manager="adinah" 
	editor=""/>
<tags ms.service="storage"
    ms.date="02/20/2015"
    wacn.date="04/15/2015"
    />

# 使用 Azure 存储模拟器进行开发和测试

## 概述
Microsoft Azure 存储模拟器提供了一个模拟 Azure Blob、队列和表服务以进行开发的本地环境。使用存储模拟器，你可以在本地针对存储服务测试应用程序，而不会产生任何费用。

> [AZURE.NOTE] 存储模拟器作为 Microsoft Azure SDK 的一部分提供。此外，还可以作为独立程序包来安装存储模拟器。
若要配置存储模拟器，必须在计算机上具有管理权限。 
> 请注意，在一个版本的存储模拟器中创建的数据不保证在使用不同版本时可以访问。如果需要长期保存数据，建议将该数据存储在 Azure 存储帐户中，而不是存储在存储模拟器中。
 
存储模拟器与 Azure 存储服务之间存在一些差异。有关这些差异的详细信息，请参阅[存储模拟器与 Azure 存储服务之间的差异](https://msdn.microsoft.com/zh-cn/library/azure/gg433135.aspx)。

存储模拟器使用 Microsoft(r) SQL Server(tm) 实例和本地文件系统来模拟 Azure 存储服务。默认情况下，为 Microsoft(r) SQL Server(tm) 2012 Express LocalDB 中的数据库配置存储模拟器。你可以安装 SQL Server Management Studio Express 来管理 LocalDB 安装。存储模拟器使用 Windows 身份验证连接到 SQL Server 或 LocalDB。你可以选择使用存储模拟器命令行工具参考将存储模拟器配置为访问 SQL Server 的本地实例而不是 LocalDB。

## 对请求进行身份验证

存储模拟器只支持单一固定的帐户和众所周知的身份验证密钥。此帐户和密钥是允许用于存储模拟器的唯一凭据。它们具有以下特点：

    Account name: devstoreaccount1
    Account key: Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtrKBHBeksoGMGw==
    
> [AZURE.NOTE] 存储模拟器支持的身份验证密钥仅用于测试客户端身份验证代码的功能。它没有任何安全用途。不能在存储模拟器中使用生产存储帐户和密钥。另请注意：不应将开发帐户用于生产数据。
 

## 启动和初始化存储模拟器
若要启动 Azure 存储模拟器，请选择"开始"按钮或按 Windows 键。开始键入"Azure 存储模拟器"，然后从应用程序列表中选择"Azure 存储模拟器"。 

或者，如果 Azure 计算模拟器已在运行，则可以通过右键单击系统任务栏图标，然后选择"启动存储模拟器"来启动存储模拟器。有关运行计算模拟器的详细信息，请参阅[在计算模拟器中运行 Azure 应用程序](https://msdn.microsoft.com/zh-cn/library/azure/hh403990.aspx)。

存储模拟器在启动时，将显示命令行。可以使用此命令行来启动和停止存储模拟器以及清除数据、获取当前状态和初始化模拟器。有关详细信息，请参阅[存储模拟器命令行工具参考](https://msdn.microsoft.com/zh-cn/library/azure/gg433005.aspx)。

关闭命令行窗口后，存储模拟器将继续运行。若要重新显示命令行，请执行上述步骤，就像启动存储模拟器一样。


首次运行存储模拟器时，会为你初始化本地存储环境。可以使用存储模拟器命令行工具指向不同的数据库实例或重新初始化现有数据库。初始化过程在 LocalDB 中创建一个数据库，并为每个本地存储服务保留 HTTP 端口。此步骤需要管理权限。有关详细信息，请参阅[存储模拟器命令行工具参考](https://msdn.microsoft.com/zh-cn/library/azure/gg433005.aspx)。

## 关于存储服务 URI

如何对 Azure 存储服务的资源进行寻址因资源位于 Azure 中还是位于存储模拟器服务中而异。一个 URI 方案用于对 Azure 中的存储资源进行寻址，另一个 URI 方案用于对存储模拟器中的存储资源进行寻址。之所以有差异是因为，本地计算机不执行域名解析。这两个 URI 方案都始终包括所请求资源的帐户名称和地址。

## 对云中的 Azure 存储资源进行寻址

在用于对 Azure 中的存储资源进行寻址的 URI 方案中，帐户名称是 URI 主机名的一部分，要寻址的资源是 URI 路径的一部分。以下基本寻址方案用于访问存储资源：

    <http|https>://<account-name>.<service-name>.core.chinacloudapi.cn/<resource-path>


 `<account-name>`  是存储帐户的名称。 `<service-name>`  是要访问的服务的名称， `<resource-path>`  是要请求的资源的路径。下面的列表显示了每个存储服务的 URI 方案：

	Blob Service: <http|https>://<account-name>.blob.core.chinacloudapi.cn/<resource-path>
	Queue Service: <http|https>://<account-name>.queue.core.chinacloudapi.cn/<resource-path>
	Table Service: <http|https>://<account-name>.table.core.chinacloudapi.cn/<resource-path>

例如，以下地址可用于访问云中的 blob：
    
    http://myaccount.blob.core.chinacloudapi.cn/mycontainer/myblob.txt

> [AZURE.NOTE] 你还可以将自定义域名与云中的 Blob 存储终结点相关联，并使用该自定义域名来定义 blob 和容器的地址。请参阅 
 
## 对存储模拟器中的本地 Azure 存储资源进行寻址

在存储模拟器中，因为本地计算机不执行域名解析，帐户名称将是 URI 路径的一部分。存储模拟器中运行的资源的 URI 方案遵循以下格式：

    http://<local-machine-address>:<port>/<account-name>/<resource-path>

以下格式用于对存储模拟器中运行的资源进行寻址：

	Blob Service: http://127.0.0.1:10000/<account-name>/<resource-path>
	Queue Service: http://127.0.0.1:10001/<account-name>/<resource-path>
	Table Service: http://127.0.0.1:10002/<account-name>/<resource-path>

例如，以下地址可用于访问存储模拟器中的 blob：

    http://127.0.0.1:10000/myaccount/mycontainer/myblob.txt

> [AZURE.NOTE] HTTPS 不是一种允许对本地存储资源进行寻址的协议。
 
## 使用 RA-GRS 对帐户辅助副本进行寻址
存储模拟器从 3.1 版开始，支持读取访问地域冗余复制 (RA-GRS)。对于同时位于云中和本地模拟器中的存储资源，可以通过在帐户名称后面追加 -secondary 来访问辅助位置。例如，以下地址可用于访问使用存储模拟器中的只读辅助副本的 blob：

    http://127.0.0.1:10000/myaccount-secondary/mycontainer/myblob.txt 

> [AZURE.NOTE] 若要使用存储模拟器以编程方式访问辅助副本，请使用 Storage Client Library for .NET 3.2 版或更高版本。有关详细信息，请参阅"存储客户端库参考"。
 
## 使用命令行工具初始化存储模拟器
可以使用存储模拟器命令行工具初始化存储模拟器，使其指向默认数据库实例以外的其他数据库实例。如果要使用默认 LocalDB 数据库实例，则没有必要运行单独的初始化步骤。

默认情况下，此工具将安装到 C:\Program Files(x86)\Microsoft SDKs\Azure\Storage Emulator\ 目录。首次启动模拟器时，将自动运行初始化程序。

> [AZURE.NOTE] 你必须具有管理特权才能运行下面所述的`init`命令。

1. 单击**"开始"**按钮或按 **Windows** 键。开始键入 `Azure Storage Emulator` 并在它出现时选择它。此时将弹出命令提示符窗口。


2. 在命令提示符窗口中，键入以下命令，其中 `<SQLServerInstance>` 是 SQL Server 实例的名称。若要使用 LocalDb，请指定 `(localdb)\v11.0` 作为 SQL Server 实例。

    WAStorageEmulator init /sqlInstance <SQLServerInstance> 
    
也可以使用以下命令，该命令指示模拟器使用默认 SQL Server 实例：

    WAStorageEmulator init /server .&#92; 

或者，可以使用以下命令来重新初始化数据库：

    WAStorageEmulator init /forceCreate 

## 存储模拟器命令行工具参考

从 3.0 版开始，当你启动存储模拟器时，将看到命令提示符弹出窗口。可使用命令提示符启动和停止模拟器以及查询状态和执行其他操作。

> [AZURE.NOTE] 如果已安装计算模拟器，则在启动存储模拟器时，将显示一个系统任务栏图标。右键单击该图标可显示一个菜单，其中提供了启动和停止存储模拟器的图形化方式。

### 命令行语法

	WAStorageEmulator [/start] [/stop] [/status] [/clear] [/init] [/help]

### 选项

若要查看选项列表，请在命令提示符处键入 `/help` 。

## 后续步骤
- [存储模拟器命令行工具参考](https://msdn.microsoft.com/zh-cn/library/azure/gg433005.aspx)
- [存储模拟器与 Azure 存储服务之间的差异](https://msdn.microsoft.com/zh-cn/library/azure/gg433135.aspx)
- [存储模拟器发行说明](https://msdn.microsoft.com/zh-cn/library/azure/dn683879.aspx)

<!--HONumber=50-->