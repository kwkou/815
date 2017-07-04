<properties
    pageTitle="导航和选择 Windows VM 映像 | Azure"
    description="了解在使用 Resource Manager 部署模型创建 Windows 虚拟机时如何确定映像的发布者、产品和 SKU。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="squillace"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="188b8974-fabd-4cd3-b7dc-559cbb86b98a"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="infrastructure"
    ms.date="08/23/2016"
    wacn.date="04/24/2017"
    ms.author="rasquill"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="95f0372f5a9249cc9693ddc0a8fdd5401857fa76"
    ms.lasthandoff="04/14/2017" />

# <a name="navigate-and-select-windows-virtual-machine-images-in-azure-with-powershell"></a>使用 PowerShell 在 Azure 中导航和选择 Windows 虚拟机映像
本主题介绍如何查找每个可能部署到目标位置的 VM 映像发布者、产品、SKU 和版本。 举例来说，某些常用 Windows VM 映像包括：

## <a name="table-of-commonly-used-windows-images"></a>常用 Windows 映像表
| PublisherName | 产品 | SKU |
|:--- |:--- |:--- |:--- |
| MicrosoftSQLServer |SQL2016-WS2012R2 |Enterprise |
| MicrosoftSQLServer |SQL2016-WS2012R2 |standard |
| MicrosoftWindowsServer |WindowsServer |2012-R2-Datacenter |
| MicrosoftWindowsServer |WindowsServer |2012-Datacenter |
| MicrosoftWindowsServer |WindowsServer |2008-R2-SP1 |
| MicrosoftWindowsServer |WindowsServer |Windows-Server-Technical-Preview |
| MicrosoftWindowsServerHPCPack |WindowsServerHPCPack |2012R2-ENU |

## <a name="find-azure-images-with-powershell"></a>使用 PowerShell 查找 Azure 映像
> [AZURE.NOTE]
> 下载并配置[最新的 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs)。 如果使用低于 1.0 版本的 Azure PowerShell 模块，则仍使用以下命令，但必须先执行 `Switch-AzureMode AzureResourceManager`。 
> 
> 

使用 Azure 资源管理器创建新的虚拟机时，在某些情况下，你需要使用以下映像属性组合来指定映像：

* 发布者
* 产品
* SKU

例如， `Set-AzureRMVMSourceImage` PowerShell cmdlet 或资源组模板文件需要这些值，必须在此命令或文件中指定要创建的虚拟机类型。

如果你需要确定这些值，可以浏览映像以确定这些值：

1. 列出映像发布者。
2. 对于给定的发布者，列出其产品。
3. 对于给定的产品，列出其 SKU。

首先，使用以下命令列出发布者：

    $locName="<Azure location, such as China North>"
    Get-AzureRMVMImagePublisher -Location $locName | Select PublisherName

填写选择的发布者名称，然后运行以下命令：

    $pubName="<publisher>"
    Get-AzureRMVMImageOffer -Location $locName -Publisher $pubName | Select Offer

填写选择的产品名称，然后运行以下命令：

    $offerName="<offer>"
    Get-AzureRMVMImageSku -Location $locName -Publisher $pubName -Offer $offerName | Select Skus

从 `Get-AzureRMVMImageSku` 命令的输出来看，你已获得了为新虚拟机指定映像所需的所有信息。

下面是一个完整示例：

    PS C:\> $locName="China North"
    PS C:\> Get-AzureRMVMImagePublisher -Location $locName | Select PublisherName

    PublisherName
    -------------
    AsiaInfo.DeepSecurity
    AzureChinaMarketplace
    Canonical
    ...

对于“MicrosoftWindowsServer”发布者：

    PS C:\> $pubName="MicrosoftWindowsServer"
    PS C:\> Get-AzureRMVMImageOffer -Location $locName -Publisher $pubName | Select Offer

    Offer
    -----
    WindowsServer

对于“WindowsServer”产品：

    PS C:\> $offerName="WindowsServer"
    PS C:\> Get-AzureRMVMImageSku -Location $locName -Publisher $pubName -Offer $offerName | Select Skus

    Skus
    ----
    2008-R2-SP1
    2008-R2-SP1-zhcn
    2012-Datacenter
    2012-Datacenter-zhcn
    2012-R2-Datacenter
    2012-R2-Datacenter-zhcn
    2016-Datacenter
    2016-Datacenter-Server-Core
    2016-Datacenter-with-Containers
    2016-Datacenter-zhcn
    2016-Nano-Server
    2016-Nano-Server-Technical-Preview
    2016-Technical-Preview-with-Containers
    Windows-Server-Technical-Preview

从上面的列表中复制选择的 SKU 名称，你已获得 `Set-AzureRMVMSourceImage` PowerShell cmdlet 或资源组模板的所有信息。

## <a name="next-steps"></a>后续步骤
现在，你可以确切地选择想要使用的映像。 若要使用刚刚找到的映像信息快速创建虚拟机，或要使用包含该映像信息的模板，请参阅[使用 Resource Manager 和 PowerShell 创建 Windows VM](/documentation/articles/virtual-machines-windows-quick-create-powershell/)。

<!--Update_Description: move contents out from includes and remove linux vm realted contents-->