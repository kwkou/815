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
    ms.devlang="azurecli"
    ms.topic="article"
    ms.date="03/30/2017"
    wacn.date="05/02/2017"
    ms.author="adegeo"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="dc829203d0b4d093bcdc004ec7444499f4692134"
    ms.lasthandoff="04/22/2017" />

# <a name="create-and-deploy-a-virtual-machine-scale-set"></a>创建和部署虚拟机规模集
使用虚拟机规模集可以轻松地将相同的虚拟机作为集来进行部署和管理。 规模集为超大规模应用程序提供高度可缩放且可自定义的计算层，并且它们支持 Windows 平台映像、Linux 平台映像、自定义映像和扩展。 有关规模集的详细信息，请参阅[虚拟机规模集](/documentation/articles/virtual-machine-scale-sets-overview/)。

本教程介绍如何**不**使用 Azure 门户就创建虚拟机规模集。 有关如何使用 Azure 门户的信息，请参阅[如何使用 Azure 门户创建虚拟机规模集](/documentation/articles/virtual-machine-scale-sets-portal-create/)。

>[AZURE.NOTE]
>有关 Azure Resource Manager 资源的详细信息，请参阅 [Azure Resource Manager 与经典部署](/documentation/articles/resource-manager-deployment-model/)。

## <a name="sign-in-to-azure"></a>登录 Azure

若要使用 Azure CLI 2.0 或 Azure PowerShell 创建规模集，首先需要登录到订阅。

有关如何使用 Azure CLI 或 PowerShell 安装、设置和登录到 Azure 的详细信息，请参阅 [Azure CLI 2.0 入门](https://docs.microsoft.com/zh-cn/cli/azure/get-started-with-azure-cli)或 [Azure PowerShell cmdlet 入门](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/)。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

    az login

<br/>

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud

## <a name="create-a-resource-group"></a>创建资源组

首先需要创建虚拟机规模集所关联的资源组。

    az group create --location chinanorth --name vmss-test-1

<br/>

    New-AzureRmResourceGroup -Location chinanorth -Name vmss-test-1

## <a name="create-from-azure-cli"></a>从 Azure CLI 创建

使用 Azure CLI，只需最少的工作量就可创建虚拟机规模集。 如果你省略默认值，系统会自动提供这些值。 例如，如果你未指定任何虚拟网络信息，系统将自动创建一个虚拟网络。 如果你省略以下组成部分，系统会自动予以创建： 
- 负载均衡器
- 虚拟网络
- 公共 IP 地址

选择要用于虚拟机规模集的虚拟机映像时，可以使用几个选项：

- URN  
资源的标识符：  
**Win2012R2Datacenter**

- URN 别名  
URN 的友好名称：  
**MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest**

- 自定义资源 ID  
Azure 资源的路径：  
**/subscriptions/subscription-guid/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/images/MyImage**

- Web 资源  
HTTP URI 的路径：  
**http://contoso.blob.core.chinacloudapi.cn/vhds/osdiskimage.vhd**

>[AZURE.TIP]
>使用 `az vm image list` 可获取可用映像的列表。

若要创建虚拟机规模集，必须指定以下各项：

- 资源组 
- 名称
- 操作系统映像
- 身份验证信息 

以下示例创建基本虚拟机规模集（此步骤可能需要几分钟）。

    az vmss create --resource-group vmss-test-1 --name MyScaleSet --image UbuntuLTS --authentication-type password --admin-username azureuser --admin-password P@ssw0rd!

完成该命令后，即已创建虚拟机规模集。 你可能需要获取虚拟机的 IP 地址，以便能够与它建立连接。 可以使用以下命令获取有关虚拟机的各种不同信息（包括 IP 地址）。 

    az vmss list-instance-connection-info --resource-group vmss-test-1 --name MyScaleSet

## <a name="create-from-powershell"></a>从 PowerShell 创建

PowerShell 的用法比 Azure CLI 更复杂。 Azure CLI 为网络相关的资源（例如负载均衡器、IP 地址和虚拟网络）提供默认值，而 PowerShell 则不提供。 使用 PowerShell 引用映像也略微复杂一些。 可以使用以下 cmdlet 获取映像：

1. Get-AzureRMVMImagePublisher
2. Get-AzureRMVMImageOffer
3. Get-AzureRmVMImageSku

cmdlet 的运行结果可以按顺序通过管道传送。 以下示例演示如何获取其发布服务器包含名称 **microsoft** 的**中国北部**的所有映像。

    Get-AzureRMVMImagePublisher -Location ChinaNorth | Where-Object PublisherName -Like *microsoft* | Get-AzureRMVMImageOffer | Get-AzureRmVMImageSku | Select-Object PublisherName, Offer, Skus

<br/>

    PublisherName              Offer          Skus
    -------------              -----          ----
    MicrosoftAzureSiteRecovery Process-Server Windows-2012-R2-Datacenter
    MicrosoftOSTC              FreeBSD        10.3
    MicrosoftOSTC              FreeBSD        11.0
    MicrosoftRServer           RServer-CentOS Enterprise
    MicrosoftRServer           RServer-RedHat Enterprise
    MicrosoftRServer           RServer-Ubuntu Enterprise
    MicrosoftRServer           RServer-WS2016 Enterprise
    ...

用于创建虚拟机规模集的工作流如下：

1. 创建包含规模集相关信息的配置对象。
2. 引用基本 OS 映像。
3. 配置操作系统设置：身份验证、VM 名称前缀和用户/密码。
4. 配置网络。
5. 创建规模集。

本示例为装有 Windows Server 2016 的计算机创建一个基本的双实例规模集。

    # Create a config object
    $vmssConfig = New-AzureRmVmssConfig -Location ChinaNorth -SkuCapacity 2 -SkuName Standard_A0  -UpgradePolicyMode Automatic

    # Reference a virtual machine image from the gallery
    Set-AzureRmVmssStorageProfile $vmssConfig -ImageReferencePublisher MicrosoftWindowsServer -ImageReferenceOffer WindowsServer -ImageReferenceSku 2016-Datacenter -ImageReferenceVersion latest

    # Set up information for authenticating with the virtual machine
    Set-AzureRmVmssOsProfile $vmssConfig -AdminUsername azureuser -AdminPassword P@ssw0rd! -ComputerNamePrefix myvmssvm

    # Create the virtual network resources
    $subnet = New-AzureRmVirtualNetworkSubnetConfig -Name "my-subnet" -AddressPrefix 10.0.0.0/24
    $vnet = New-AzureRmVirtualNetwork -Name "my-network" -ResourceGroupName "vmss-test-1" -Location "chinanorth" -AddressPrefix 10.0.0.0/16 -Subnet $subnet
    $ipConfig = New-AzureRmVmssIpConfig -Name "my-ip-address" -LoadBalancerBackendAddressPoolsId $null -SubnetId $vnet.Subnets[0].Id

    # Attach the virtual network to the config object
    Add-AzureRmVmssNetworkInterfaceConfiguration -VirtualMachineScaleSet $vmssConfig -Name "network-config" -Primary $true -IPConfiguration $ipConfig

    # Create the scale set with the config object (this step might take a few minutes)
    New-AzureRmVmss -ResourceGroupName vmss-test-1 -Name my-scale-set -VirtualMachineScaleSet $vmssConfig

## <a name="create-from-a-template"></a>从模板创建

可以使用 Azure Resource Manager 模板部署虚拟机规模集。 可以创建你自己的模板，也可以使用[模板存储库](https://www.github.com/Azure/azure-quickstart-templates/)中的模板。 可直接将这些模板部署到 Azure 订阅。

>[AZURE.NOTE]
>若要创建自己的模板，请创建一个 JSON 文本文件。 有关如何创建和自定义模板的常规信息，请参阅 [Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates/)。

[GitHub 上](https://github.com/gatneil/mvss/tree/minimum-viable-scale-set)提供了一个示例模板。 有关如何创建和使用该示例的详细信息，请参阅[最小的可行规模集](/documentation/articles/virtual-machine-scale-sets-mvss-start/)。

## <a name="create-from-visual-studio"></a>从 Visual Studio 创建

使用 Visual Studio 可以创建 Azure 资源组项目，并在其中添加虚拟机规模集模板。 可以选择是要从 GitHub 还是 Azure Web 应用程序库导入该模板。 还会为你生成部署 PowerShell 脚本。 有关详细信息，请参阅[如何使用 Visual Studio 创建虚拟机规模集](/documentation/articles/virtual-machine-scale-sets-vs-create/)。

## <a name="create-from-the-azure-portal-preview"></a>在 Azure 门户中创建

Azure 门户提供了快速创建规模集的简便方式。 有关详细信息，请参阅[如何使用 Azure 门户创建虚拟机规模集](/documentation/articles/virtual-machine-scale-sets-portal-create/)。

## <a name="next-steps"></a>后续步骤

了解如何[管理应用](/documentation/articles/virtual-machine-scale-sets-deploy-app/)。

<!--Update_Description: wording update-->