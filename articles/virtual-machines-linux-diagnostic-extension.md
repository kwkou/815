<properties
		pageTitle="使用 linux 诊断扩展监视 linux VM 的性能和诊断数据"
		description="了解如何使用 linux 诊断扩展监视 linux VM 的性能和诊断数据。"
		services="virtual-machines"
		documentationCenter=""
    	authors="Ning"
		manager="timlt"
		editor=""
    	tags=""/>

<tags
		ms.service="virtual-machines"
		ms.date="06/04/2015"
		wacn.date="08/29/2015"/>


#使用 Linux 诊断扩展监视 Linux VM 的性能和诊断数据

##介绍 

Linux 诊断扩展可利用以下功能帮助用户监视在 Azure 上运行的 Linux VM：

1.	收集 Linux VM 的系统性能、诊断和 syslog 数据，并将其上载到用户的存储表。
2.	让用户能够自定义将要收集并上载的数据指标。	
3.	让用户能够将指定日志文件上载到指定的存储表。

对于 2.0 版，数据包括：

1.	所有 Linux Rsyslog 记录，包括系统、安全和应用程序日志。
2.	此[文档](https://scx.codeplex.com/wikipage?title=xplatproviders&referringTitle=Documentation "文档")中指定的所有系统数据。
3.	用户指定的日志文件。

##如何启用扩展 
通过 Azure 管理门户、Azure PowerShell 或 Azure CLI 脚本可以启用该扩展。

若要直接从 [Azure 管理门户](https://manage.windowsazure.cn)查看和配置系统和性能数据，请执行以下[步骤](http://azure.microsoft.com/blog/2014/09/02/windows-azure-virtual-machine-monitoring-with-wad-extension/ "Windows 博客 URL")。
 

本文将重点介绍通过 Azure CLI 命令启用和配置扩展。这让你能够直接从存储表读取和查看数据。


##先决条件
1. Windows Azure Linux Agent 2.0.6 版或更高版本。请注意，大部分 Azure VM Linux 库映像都包含 2.0.6 以上版本。你可以运行 WAAgent -version 以确认 VM 中安装的版本。如果 VM 正在运行的版本早于 2.0.6，则可以按照以下[说明](https://github.com/Azure/WALinuxAgent "说明")进行更新。
2.  [Azure CLI](/documentation/articles/xplat-cli "Azure CLI") 请按照[此指南](/documentation/articles/xplat-cli-install "此指南")中的说明在计算机上设置 Azure CLI 环境。安装 Azure CLI 后，你将能够从命令行界面（Bash、终端、命令提示符）使用 azure 命令访问 Azure CLI 命令；例如，运行“azure vm extension set --help”可以获取详细用法，运行“azure login”可以登录到 Azure，运行“azure vm list”可以列出你在 Azure 上拥有的所有虚拟机。
3. 用于存储数据的存储帐户。你将需要预先创建的存储帐户名称和访问密钥，以将数据上载到存储中。


##使用 Azure CLI 命令启用 Linux 诊断扩展 

###  方案 1.使用默认数据集启用扩展
对于 2.0 版及更高版本，将会收集的默认数据包括：
 
1. 所有 Rsyslog 信息（包括系统、安全和应用程序日志）  
2. 基础系统数据的核心集，请注意，完整数据集如此[文档](https://scx.codeplex.com/wikipage?title=xplatproviders&referringTitle=Documentation "文档")中所述。如果想要启用额外的数据，请继续执行方案 2 和 3 中的步骤。

步骤 1.使用以下内容创建名为 PrivateConfig.json 的文件：

	{
     	"storageAccountName":"the storage account to receive data",
     	"storageAccountKey":"the key of the account"
	}

步骤 2.运行“azure vm extension set vm\_name LinuxDiagnostic Microsoft.OSTCExtensions 2.* --private-config-path PrivateConfig.json”。


###   方案 2.自定义性能监视器指标  
如果想要自定义性能和诊断数据表，请执行以下步骤。

步骤 1.使用以下内容创建名为 PrivateConfig.json 的文件，指定你想要收集的特定数据。

有关所有受支持的提供程序和变量，请参阅此[文档](https://scx.codeplex.com/wikipage?title=xplatproviders&referringTitle=Documentation "文档")。你可以拥有多个查询，通过将更多查询追加到脚本中，你还可以将它们存储为多个表。

请注意，默认始终收集 Rsyslog 数据。

	{
     	"storageAccountName":"storage account to receive data",
     	"storageAccountKey":"key of the account",
      	"perfCfg":[
           	{"query":"SELECT PercentAvailableMemory, AvailableMemory, UsedMemory ,PercentUsedSwap FROM SCX_MemoryStatisticalInformation","table":"LinuxMemory"
           	}   
          ]
	}


步骤 2.运行“azure vm extension set vm\_name LinuxDiagnostic Microsoft.OSTCExtensions 2.* --private-config-path PrivateConfig.json”


###   方案 3.上载自己的日志文件
如果想要收集特定的日志文件并上载到存储帐户，请执行以下步骤。你需要指定日志文件的路径和用于存储日志的表名。你可以将多个文件/表条目添加到脚本，以拥有多个日志文件。

步骤 1.使用以下内容创建名为 PrivateConfig.json 的文件：

	{
     	"storageAccountName":"the storage account to receive data",
     	"storageAccountKey":"key of the account",
      	"fileCfg":[
           	{"file":"/var/log/mysql.err",
             "table":"mysqlerr"
           	} 
          ]
	}


步骤 2.运行“azure vm extension set vm\_name LinuxDiagnostic Microsoft.OSTCExtensions 2.* --private-config-path PrivateConfig.json”


###   方案 4.禁用 Linux 监视器扩展
步骤 1.使用以下内容创建名为 PrivateConfig.json 的文件：

	{
     	"storageAccountName":"the storage account to receive data",
     	"storageAccountKey":"the key of the account",
     	“perfCfg”:[],
     	“enableSyslog”:”False”
	}


步骤 2.运行“azure vm extension set vm\_name LinuxDiagnostic Microsoft.OSTCExtensions 2.* --private-config-path PrivateConfig.json”


##查看数据
性能和诊断数据存储在 Azure 存储表中，请查看[这篇文章](/documentation/articles/storage-ruby-how-to-use-table-storage "这篇文章")了解有关如何使用 Azure CLI 脚本访问存储表中的数据。

此外，可以使用以下 UI 工具来访问数据：

1.	使用 Visual Studio 服务器资源管理器。导航到你的存储帐户。VM 运行约 5 分钟后，你应该会看到默认表。“LinuxCpu”、“LinuxDisk”、“LinuxMemory”、“Linuxsyslog”。双击表名以查看数据。 
2.	使用 [Azure 存储空间资源管理器](https://azurestorageexplorer.codeplex.com/ "Azure 存储空间资源管理器")访问数据。 

![图像](./media/virtual-machines-linux-diagnostic-extension/no1.png)

如果已经启用方案 2 和 3 中指定的 fileCfg 或 perfCfg，你还将能够使用以上工具查看非默认数据。



##已知问题
1. 对于 2.0 版，只能通过脚本访问 Rsyslog 信息和客户指定的日志文件。
2. 对于 2.0 版，如果首先通过脚本启用了 Linux 诊断扩展，那么你无法从门户查看数据。相反，如果先从门户启用扩展，那么脚本仍将正常工作。

我们预计在下一版本中修复这两个问题。


 

<!---HONumber=67-->