<properties
    pageTitle="创建 Azure 虚拟机规模集 | Azure"
    description="使用 Azure CLI、PowerShell、模板或 Visual Studio 创建和部署 Linux 或 Windows Azure 虚拟机规模集。"
    services="virtual-machine-scale-sets"
    documentationcenter=""
    author="Thraka"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="virtual-machine-scale-sets"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/20/2017"
    wacn.date="04/17/2017"
    ms.author="adegeo"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="a7dd93bc5ff8c917a2aef8ec51e902abea3f5f03"
    ms.lasthandoff="04/06/2017" />

# <a name="create-and-deploy-a-virtual-machine-scale-set"></a>创建和部署虚拟机规模集
使用虚拟机规模集可以轻松地将相同的虚拟机作为集来进行部署和管理。 规模集为超大规模应用程序提供高度可缩放且可自定义的计算层，并且它们支持 Windows 平台映像、Linux 平台映像、自定义映像和扩展。 有关规模集的详细信息，请参阅[虚拟机规模集](/documentation/articles/virtual-machine-scale-sets-overview/)。

本教程介绍如何**不**使用 Azure 门户预览就创建虚拟机规模集。 有关如何使用 Azure 门户预览的信息，请参阅[如何使用 Azure 门户预览创建虚拟机规模集](/documentation/articles/virtual-machine-scale-sets-portal-create/)。

>[AZURE.NOTE]
>有关 Azure Resource Manager 资源的详细信息，请参阅 [Azure Resource Manager 与经典部署](/documentation/articles/resource-manager-deployment-model/)。

## <a name="log-in-to-azure"></a>登录 Azure

如果要使用 Azure CLI 2.0 或 PowerShell 创建规模集，首先需要登录到订阅。

有关如何使用 Azure CLI 2.0 或 PowerShell 安装、设置和登录到 Azure 的详细信息，请参阅 [Azure CLI 2.0 入门](https://docs.microsoft.com/zh-cn/cli/azure/get-started-with-azure-cli)或 [Azure PowerShell cmdlet 入门](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/)。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

    az login

<br/>

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud

## <a name="prep-create-a-resource-group"></a>准备：创建资源组

首先需要创建虚拟机规模集所关联的资源组。

    az group create --location chinanorth --name vmss-test-1

<br/>

    New-AzureRmResourceGroup -Location chinanorth -Name vmss-test-1

## <a name="create-from-azure-cli"></a>从 Azure CLI 创建

使用 Azure CLI，只需最少的工作量就可创建虚拟机规模集。 如果你省略它们，将为你提供默认值。 例如，如果未指定任何虚拟网络信息，将为你创建一个虚拟网络。 如果省略，将为你创建以下部件：一个负载均衡器、一个 VNET 和一个公共 IP 地址。

选择要用于虚拟机规模集的虚拟机映像时，有几个选项：

1. URN  
资源的标识符：  
**Win2012R2Datacenter**。

2. URN 别名  
URN 的友好名称：  
**MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest**。

3. 自定义资源 ID  
Azure 资源的路径：  
**/subscriptions/subscription-guid/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/images/MyImage**。

4. Web 资源  
HTTP URI 的路径：  
**http://contoso.blob.core.chinacloudapi.cn/vhds/osdiskimage.vhd**。

>[AZURE.TIP]
>使用 `az vm image list` 可获取可用映像的列表。

若要创建虚拟机规模集，必须指定_资源组_、_名称_、_操作系统映像_和_身份验证信息_。 以下示例创建基本虚拟机规模集（此步骤可能需要几分钟）。

    az vmss create --resource-group vmss-test-1 --name MyScaleSet --image UbuntuLTS --authentication-type password --admin-username azureuser --admin-password P@ssw0rd! --use-unmanaged-disk

## <a name="create-from-powershell"></a>从 PowerShell 创建

PowerShell 的使用比 Azure CLI 更复杂。 Azure CLI 为网络相关资源（负载均衡器、IP 地址、虚拟网络）提供默认值，而 PowerShell 则不提供。 引用映像也稍微更复杂一些。 可以使用以下 cmdlet 获取映像：

1. Get-AzureRMVMImagePublisher
2. Get-AzureRMVMImageOffer
3. Get-AzureRmVMImageSku

cmdlet 的运行结果可以按顺序通过管道传送。 下面是一个示例，介绍如何获取其发布服务器包含名称 **microsoft** 的“中国北部 2”区域的所有映像。

    Get-AzureRMVMImagePublisher -Location chinanorth | Where-Object PublisherName -Like *microsoft* | Get-AzureRMVMImageOffer | Get-AzureRmVMImageSku | Select-Object PublisherName, Offer, Skus

<br/>

    PublisherName              Offer                    Skus
    -------------              -----                    ----
    microsoft-ads              linux-data-science-vm    linuxdsvm
    microsoft-ads              standard-data-science-vm standard-data-science-vm
    MicrosoftAzureSiteRecovery Process-Server           Windows-2012-R2-Datacenter
    MicrosoftBizTalkServer     BizTalk-Server           2013-R2-Enterprise
    MicrosoftBizTalkServer     BizTalk-Server           2013-R2-Standard
    MicrosoftBizTalkServer     BizTalk-Server           2016-Developer
    MicrosoftBizTalkServer     BizTalk-Server           2016-Enterprise
    ...

用于创建虚拟机规模集的工作流是：

1. 创建包含规模集相关信息的配置对象。
2. 引用基本 OS 映像。
3. 配置操作系统设置：身份验证、VM 名称前缀、用户/密码。
4. 配置网络。
5. 创建规模集。

此示例创建安装了 Windows Server 2016 的基本 2 实例规模集。

    # Create a config object
    $vmssConfig = New-AzureRmVmssConfig -Location chinanorth -SkuCapacity 2 -SkuName Standard_A0  -UpgradePolicyMode Automatic

    # Reference a virtual machine image from the gallery
    Set-AzureRmVmssStorageProfile $vmssConfig -ImageReferencePublisher MicrosoftWindowsServer -ImageReferenceOffer WindowsServer -ImageReferenceSku 2016-Datacenter -ImageReferenceVersion latest

    # Setup information about how to authenticate with the virtual machine
    Set-AzureRmVmssOsProfile $vmssConfig -AdminUsername azureuser -AdminPassword P@ssw0rd! -ComputerNamePrefix myvmssvm

    # Create the virtual network resources
    $subnet = New-AzureRmVirtualNetworkSubnetConfig -Name "my-subnet" -AddressPrefix 10.0.0.0/24
    $vnet = New-AzureRmVirtualNetwork -Name "my-network" -ResourceGroupName "vmss-test-1" -Location "chinanorth" -AddressPrefix 10.0.0.0/16 -Subnet $subnet
    $ipConfig = New-AzureRmVmssIpConfig -Name "my-ip-address" -LoadBalancerBackendAddressPoolsId $null -SubnetId $vnet.Subnets[0].Id

    # Attach the virtual network to the config object
    Add-AzureRmVmssNetworkInterfaceConfiguration -VirtualMachineScaleSet $vmssConfig -Name "network-config" -Primary $true -IPConfiguration $ipConfig

    # Create the scale set with the config object (this step may take a few minutes)
    New-AzureRmVmss -ResourceGroupName vmss-test-1 -Name my-scale-set -VirtualMachineScaleSet $vmssConfig

## <a name="create-from-a-template"></a>从模板创建

可使用 Azure Resource Manager 模板部署虚拟机规模集。 可以创建你自己的模板，也可以使用[模板存储库](https://azure.microsoft.com/resources/templates/?term=vmss)中的模板。 可以直接将这些模板部署到 Azure 订阅。

>[AZURE.NOTE]
>若要创建自己的模板，请创建 _.json_ 文本文件。 有关如何创建和自定义模板的常规信息，请参阅 [Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates/)。

[GitHub 上](https://github.com/gatneil/mvss/tree/minimum-viable-scale-set)提供了一个示例模板。 有关如何创建和使用该示例的详细信息，请参阅[最小可行规模集](/documentation/articles/virtual-machine-scale-sets-mvss-start/)。

## <a name="create-from-visual-studio"></a>从 Visual Studio 创建

使用 Visual Studio，可以创建 Azure 资源组项目，并向其添加虚拟机规模集模板。 可以从 GitHub 或 Azure 库等位置选择要导入的模板。 还会为你生成部署 PowerShell 脚本。 有关详细信息，请参阅[如何使用 Visual Studio 创建虚拟机规模集](/documentation/articles/virtual-machine-scale-sets-vs-create/)。

## <a name="create-from-the-azure-portal-preview"></a>在 Azure 门户预览中创建

Azure 门户预览提供了快速创建规模集的简便方式。 有关详细信息，请参阅[如何使用 Azure 门户预览创建虚拟机规模集](/documentation/articles/virtual-machine-scale-sets-portal-create/)。