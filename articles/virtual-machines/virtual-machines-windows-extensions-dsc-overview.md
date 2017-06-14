<properties
    pageTitle="适用于 Azure 的 Desired State Configuration 概述 | Azure"
    description="有关使用 PowerShell Desired State Configuration 的 Azure 扩展的概述。内容涉及先决条件、体系结构和 cmdlet。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="zjalexander"
    manager="timlt"
    editor=""
    tags="azure-service-management,azure-resource-manager"
    keywords="" />
<tags
    ms.assetid="bbacbc93-1e7b-4611-a3ec-e3320641f9ba"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="na"
    ms.date="01/09/2017"
    wacn.date="02/24/2017"
    ms.author="zachal" />  


# Azure Desired State Configuration 扩展处理程序简介
[AZURE.INCLUDE [了解部署模型](../../includes/learn-about-deployment-models-both-include.md)]

Azure VM 代理和关联的扩展是 Azure 基础结构服务的一部分。VM 扩展是软件组件，可以扩展 VM 功能并简化各种 VM 管理操作。例如，VMAccess 扩展可用于重置管理员的密码，自定义脚本扩展可用于在 VM 上执行脚本。

本文将介绍 Azure PowerShell SDK 中包含的适用于 Azure VM 的 PowerShell Desired State Configuration \(DSC\) 扩展。使用新 cmdlet 可将 PowerShell DSC 配置上载到 PowerShell DSC 扩展启用的 Azure VM 并应用该配置。PowerShell DSC 扩展可调用 PowerShell DSC，在 VM 上启用收到的 DSC 配置。也可以通过 Azure 门户使用此功能。

## 先决条件
**本地计算机** 
需要使用 Azure 门户或 Azure PowerShell SDK 才能与 Azure VM 扩展交互。

**来宾代理** 
由 DSC 配置进行配置的 Azure VM 必须运行支持 Windows Management Framework \(WMF\) 4.0 或 5.0 的 OS。有关支持的 OS 版本的完整列表，请参阅 [DSC Extension Version History](https://blogs.msdn.microsoft.com/powershell/2014/11/20/release-history-for-the-azure-dsc-extension/)（DSC 扩展版本历史记录）。

## 术语和概念
本指南假设你熟悉以下概念：

配置 - DSC 配置文档。

节点 - DSC 配置的目标。本文档中，“节点”一律指 Azure VM。

配置数据 - 包含配置环境数据的 .psd1 文件

## 体系结构概述
Azure DSC 扩展使用 Azure VM 代理框架来传送、启用和报告 Azure VM 上运行的 DSC 配置。DSC 扩展需要一个 .zip 文件，其中至少包含一个配置文档，以及通过 Azure PowerShell SDK 或 Azure 门户提供的一组参数。

首次调用该扩展时，它将运行安装过程。此过程将使用以下逻辑安装 Windows Management Framework \(WMF\) 版本：

1. 如果 Azure VM OS 是 Windows Server 2016，则不执行任何操作。Windows Server 2016 上已安装最新版本的 PowerShell。
2. 如果已指定 `wmfVersion` 属性，除非该 WMF 版本与 VM 的 OS 不兼容，否则将安装该版本。
3. 如果未指定 `wmfVersion` 属性，则安装 WMF 的最新适用版本。

安装 WMF 需要重新启动。重新启动后，扩展将下载 `modulesUrl` 属性中指定的 .zip 文件。如果此位置是在 Azure Blob 存储中，则可以在 `sasToken` 属性中指定 SAS 令牌来访问该文件。下载并解压缩 .zip 之后，将运行 `configurationFunction` 中定义的配置函数以生成 .MOF 文件。然后，扩展将针对生成的 MOF 文件运行 `Start-DscConfiguration -Force`。扩展将捕获输出并将其写回到 Azure 状态通道。然后，DSC LCM 将正常处理监视和更正操作。

## PowerShell cmdlet
可将 PowerShell cmdlet 与 Azure Resource Manager 或经典部署模型配合使用，以便打包、发布和监视 DSC 扩展部署。下面列出的 cmdlet 是经典部署模块，但是，可以将“Azure”替换为“AzureRm”以使用 Azure Resource Manager 模型。例如，`Publish-AzureVMDscConfiguration` 使用经典部署模型，其中 `Publish-AzureRmVMDscConfiguration` 使用 Azure Resource Manager。

`Publish-AzureVMDscConfiguration` 检索配置文件，扫描其中是否有依赖的 DSC 资源，然后创建一个 .zip 文件，其中包含启用配置所需的配置和 DSC 资源。它还可以使用 `-ConfigurationArchivePath` 参数在本地创建包。否则，它会将 .zip 文件发布到 Azure Blob 存储，然后使用 SAS 令牌保护该文件。

在此 cmdlet 创建的 .zip 文件中，存档文件夹根目录处提供了 .ps1 配置脚本。资源会将模块文件夹放置在存档文件夹中。

`Set-AzureVMDscExtension` 将 PowerShell DSC 扩展所需的设置注入 VM 配置对象。在经典部署模型中，必须使用 `Update-AzureVM` 将 VM 更改应用到 Azure VM。

`Get-AzureVMDscExtension` 可检索特定 VM 的 DSC 扩展状态。

`Get-AzureVMDscExtensionStatus` 可检索由 DSC 扩展处理程序启用的 DSC 配置的状态。可以在一个或一组 VM 上执行此操作。

`Remove-AzureVMDscExtension` 可从给定的虚拟机中删除扩展处理程序。此 cmdlet **不会**删除配置、卸载 WMF 或更改虚拟机上已应用的设置。而只删除扩展处理程序。

**ASM 和 Azure Resource Manager cmdlet 的主要差异**

* Azure Resource Manager cmdlet 是同步的。ASM cmdlet 是异步的。
* ResourceGroupName、VMName、ArchiveStorageAccountName、Version 和 Location 都是 Azure Resource Manager 中的必需参数。
* ArchiveResourceGroupName 是 Azure Resource Manager 的新可选参数。如果用户的存储帐户所属的资源组与创建虚拟机的资源组不同，用户可以指定此参数。
* ConfigurationArchive 在 Azure Resource Manager 中名为 ArchiveBlobName
* ContainerName 在 Azure Resource Manager 中名为 ArchiveContainerName
* StorageEndpointSuffix 在 Azure Resource Manager 中名为 ArchiveStorageEndpointSuffix
* 已将 AutoUpdate 开关参数添加到 Azure Resource Manager，使扩展处理程序能够在有最新版本时自动更新。请注意，当发布了新版本的 WMF 时，此参数可能会导致 VM 重新启动。

## Azure 门户功能
浏览到 VM。在“设置”-\>“常规”下面，单击“扩展”。 此时将创建一个新窗格。单击“添加”，然后选择“PowerShell DSC”。

门户需要输入。
**配置模块或脚本**：此字段必填。需要一个包含配置脚本的 .ps1 文件，或者需要一个 .zip 文件，其中的 .ps1 配置脚本位于根目录，所有依赖资源位于模块文件夹。可以使用 Azure PowerShell SDK 随附的 `Publish-AzureVMDscConfiguration -ConfigurationArchivePath` cmdlet 来创建该文件。系统会将 .zip 文件上载到受 SAS 令牌保护的用户 Blob 存储。

**配置数据 PSD1 文件**：此字段选填。如果配置要求 .psd1 中有配置数据文件，请使用此字段来进行选择，然后将它上载到受 SAS 令牌保护的用户 Blob 存储。

**配置的模块限定名称**：.ps1 文件可以包含多个配置函数。请输入配置 .ps1 脚本的名称，后面再加上 '' 和配置函数的名称。例如，如果 .ps1 脚本的名称为“configuration.ps1”，而配置为“IisInstall”，则可输入：`configuration.ps1\IisInstall`

**配置参数**：如果配置函数采用参数，请使用 `argumentName1=value1,argumentName2=value2` 格式在此处输入。请注意，此格式与通过 PowerShell cmdlet 或 Resource Manager 模板接受配置参数的方式不同。

## 入门
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
###经典模型

    #Azure PowerShell cmdlets are required
    Import-Module Azure

    #Use an existing Azure Virtual Machine, 'DscDemo1'
    $demoVM = Get-AzureVM DscDemo1

    #Publish the configuration script into user storage.
    Publish-AzureVMDscConfiguration -ConfigurationPath ".\IisInstall.ps1" -StorageContext $storageContext -Verbose -Force

    #Set the VM to run the DSC configuration
    Set-AzureVMDscExtension -VM $demoVM -ConfigurationArchive "IisInstall.ps1.zip" -StorageContext $storageContext -ConfigurationName "IisInstall" -Verbose

    #Update the configuration of an Azure Virtual Machine
    $demoVM | Update-AzureVM -Verbose

    #check on status
    Get-AzureVMDscExtensionStatus -VM $demovm -Verbose

### Azure Resource Manager 模型

    $resourceGroup = "dscVmDemo"
    $location = "chinanorth"
    $vmName = "myVM"
    $storageName = "demostorage"
    #Publish the configuration script into user storage
    Publish-AzureRmVMDscConfiguration -ConfigurationPath .\iisInstall.ps1 -ResourceGroupName $resourceGroup -StorageAccountName $storageName -force
    #Set the VM to run the DSC configuration
    Set-AzureRmVmDscExtension -Version 2.21 -ResourceGroupName $resourceGroup -VMName $vmName -ArchiveStorageAccountName $storageName -ArchiveBlobName iisInstall.ps1.zip -AutoUpdate:$true -ConfigurationName "IISInstall"

## 日志记录
日志位于：

C:\\WindowsAzure\\Logs\\Plugins\\Microsoft.Powershell.DSC\[Version Number\]

## 后续步骤
有关 PowerShell DSC 的详细信息，请[访问 PowerShell 文档中心](https://msdn.microsoft.com/powershell/dsc/overview)。

检查 [Azure Resource Manager template for the DSC extension](/documentation/articles/virtual-machines-windows-extensions-dsc-template/)（适用于 DSC 扩展的 Azure Resource Manager 模板）。

若要查找可以使用 PowerShell DSC 管理的其他功能，请[浏览 PowerShell 库](https://www.powershellgallery.com/packages?q=DscResource&x=0&y=0)以获取更多 DSC 资源。

有关将敏感参数传入配置的详细信息，请参阅 [Manage credentials securely with the DSC extension handler](/documentation/articles/virtual-machines-windows-extensions-dsc-credentials/)（使用 DSC 扩展处理程序安全管理凭据）。

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: Adding ARM model-->