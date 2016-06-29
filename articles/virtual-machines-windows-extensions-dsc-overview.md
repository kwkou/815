<properties
   pageTitle="适用于 Azure 的 Desired State Configuration 概述 | Azure"
   description="有关使用 PowerShell Desired State Configuration 的 Azure 扩展的概述。内容涉及先决条件、体系结构和 cmdlet。"
   services="virtual-machines-windows"
   documentationCenter=""
   authors="zjalexander"
   manager="timlt"
   editor=""
   tags="azure-service-management,azure-resource-manager"
   keywords=""/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="04/18/2016"
	wacn.date="06/29/2016"/>

# Azure Desired State Configuration 扩展处理程序简介 #

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]

Azure VM 代理和关联的扩展是 Azure 基础结构服务的一部分。VM 扩展是软件组件，可以扩展 VM 功能和简化各种 VM 管理操作；例如，VMAccess 扩展可用于重置管理员的密码，自定义脚本扩展可用于在 VM 上执行脚本。

本文将介绍 Azure PowerShell SDK 中包含的适用于 Azure VM 的 PowerShell Desired State Configuration (DSC) 扩展。使用新 cmdlet 可将 PowerShell DSC 配置上载到 PowerShell DSC 扩展启用的 Azure VM 并应用该配置。PowerShell DSC 扩展将调用 PowerShell DSC 在 VM 上启用收到的 DSC 配置。

## 先决条件 ##
**本地计算机**
若要与 Azure VM 扩展交互，需要使用 Azure PowerShell SDK。

**来宾代理**
要使用 DSC 配置进行配置的 Azure VM 需是支持 Windows Management Framework (WMF) 4.0 或 5.0 的 OS。有关支持的 OS 版本的完整列表，请参阅“DSC 扩展版本历史记录”。

## 术语和概念 ##
本指南假设你熟悉以下概念：

配置 - DSC 配置文档。

节点 - DSC 配置的目标。本文档中，“节点”一律是指 Azure VM。

配置数据 - 包含配置环境数据的 .psd1 文件

## 体系结构概述 ##

Azure DSC 扩展使用 Azure VM 代理框架来传送、启用和报告 Azure VM 上运行的 DSC 配置。DSC 扩展需要一个 .zip 文件，其中至少包含一个配置文档，以及通过 Azure PowerShell SDK 提供的一组参数。

首次调用该扩展时，它将运行安装过程。此过程将安装下面定义的 Windows Management Framework (WMF) 版本：

1. 如果 Azure VM OS 是 Windows Server 2016，则不执行任何操作。WS 2016 上已安装最新版本的 PowerShell。
2. 如果已指定 `wmfVersion` 属性，除非该 WMF 版本与 VM 的 OS 不兼容，否则将安装该版本。
3. 如果未指定 `wmfVersion` 属性，则安装 WMF 的最新适用版本。

安装 WMF 需要重新启动。重新启动后，扩展将下载 `modulesUrl` 属性中指定的 .zip 文件。如果此位置是在 Azure Blob 存储中，则可以在 `sasToken` 属性中指定 SAS 令牌来访问该文件。下载并解压缩 .zip 之后，将运行 `configurationFunction` 中定义的配置函数以生成 .MOF 文件。然后，扩展将针对生成的 MOF 文件运行 `Start-DscConfiguration -force`。扩展将捕获输出并将其写回到 Azure 状态通道。然后，DSC LCM 将正常处理监视和更正操作。

## PowerShell cmdlet ##

可将 Powershell cmdlet 与 ASM 配合使用，以便打包、发布和监视 DSC 扩展部署。下面列出的 cmdlet 是 ASM 模块。

`Publish-AzureVMDscConfiguration` 将检索配置文件，扫描其中是否有依赖的 DSC 资源，然后创建一个 .zip 文件，其中包含启用配置所需的配置和 DSC 资源。它还可以使用 `-ConfigurationArchivePath` 参数在本地创建包。否则，它会将 .zip 文件发布到 Azure Blob 存储，然后使用 SAS 令牌保护该文件。

在此 cmdlet 创建的 .zip 文件中，存档文件夹根目录处提供了 .ps1 配置脚本。资源会将模块文件夹放置在存档文件夹中。

`Set-AzureVMDscExtension` 将 PowerShell DSC 扩展所需的设置注入 VM 配置对象，然后，可以使用 `Update-AzureVM` 在 Azure VM 中应用此对象。

`Get-AzureVMDscExtension` 可检索特定 VM 的 DSC 扩展状态。

`Get-AzureVMDscExtensionStatus` 可检索一个或一组 VM 上由 DSC 扩展处理程序启用的 DSC 配置的状态。

`Remove-AzureVMDscExtension` 可从给定的虚拟机中删除扩展处理程序。它**不会**删除配置、卸载 WMF 或更改虚拟机上已应用的设置。而只删除扩展处理程序。

## 入门 ##

Azure DSC 扩展将检索并在 Azure VM 上启用 DSC 配置文档。下面是一个简单的配置示例。以“IisInstall.ps1”的名称将它保存在本地：

	configuration IISInstall 
	{ 
	    node "localhost" 
	    { 
	        WindowsFeature IIS 
	        { 
	            Ensure = "Present" 
	            Name = "Web-Server"                       
	        } 
	    } 
	}

以下步骤将 IisInstall.ps1 脚本放在指定的 VM 上，执行配置，然后报告状态。
 
	#Azure PowerShell cmdlets are required
	Import-Module Azure
	
	#Use an existing Azure Virtual Machine, 'DscDemo1'
	$demoVM = Get-AzureVM DscDemo1
	
	#Publish the configuration script into user storage.
	Publish-AzureVMDscConfiguration -ConfigurationPath ".\IisInstall.ps1" -StorageContext $storageContext -Verbose -Force
	
	#Set the VM to run the DSC configuration
	Set-AzureVMDscExtension -VM $demoVM -ConfigurationArchive "demo.ps1.zip" -StorageContext $storageContext -ConfigurationName "runScript" -Verbose
	
	#Update the configuration of an Azure Virtual Machine
	$demoVM | Update-AzureVM -Verbose
	
	#check on status
	Get-AzureVMDscExtensionStatus -VM $demovm -Verbose

## 日志记录 ##

日志位于：

C:\\WindowsAzure\\Logs\\Plugins\\Microsoft.Powershell.DSC[Version Number]

## 后续步骤 ##

有关有关 PowerShell DSC 的详细信息，请[访问 PowerShell 文档中心](https://msdn.microsoft.com/zh-cn/powershell/dsc/overview)。

若要查找可以使用 PowerShell DSC 管理的其他功能，请[浏览 PowerShell 库](https://www.powershellgallery.com/packages?q=DscResource&x=0&y=0)以获取更多 DSC 资源。

有关将敏感参数传入配置的详细信息，请参阅 [Manage credentials securely with the DSC extension handler（使用 DSC 扩展处理程序安全管理凭据）](/documentation/articles/virtual-machines-windows-extensions-dsc-credentials)。
<!---HONumber=Mooncake_0503_2016-->