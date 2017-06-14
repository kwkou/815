<properties
    pageTitle="创建 Azure 虚拟网络 | Azure"
    description="了解如何创建包含多个子网的虚拟网络。"
    services="virtual-network"
    documentationcenter=""
    author="jimdial"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="4ad679a4-a959-4e48-a317-d9f5655a442b"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="05/12/2017"
    wacn.date="06/05/2017"
    ms.author="v-dazen"
    ms.custom=""
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="a84ef9a118224a7f6be391c20992ab3a4fbfe9ff"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="create-a-virtual-network-with-multiple-subnets"></a>创建包含多个子网的虚拟网络

本教程介绍如何创建包含独立公共和专用子网的基本 Azure 虚拟网络 (VNet)。 可将虚拟机 (VM)、应用服务环境、虚拟机规模集、HDInsight 和云服务等 Azure 资源连接到子网。 连接到 Vnet 的资源可通过专用 Azure 网络相互通信。

以下部分包含使用 Azure [门户](#portal)、Azure [命令行界面](#cli) (CLI)、Azure [PowerShell](#powershell) 和 Azure Resource Manager [模板](#template)部署 VNet 的步骤。 不管选择使用哪种工具来部署 VNet，结果都是一样的。 单击任一工具的链接会直接转到本文的相关部分。 若要详细了解所有的 VNet 和子网设置，请阅读[管理 VNet](/documentation/articles/virtual-network-manage-network/) 和[管理子网](/documentation/articles/virtual-network-manage-subnet/)这两篇文章。

## <a name="portal"></a>Azure 门户

1. 在 Internet 浏览器中打开 Azure [门户](https://portal.azure.cn)并使用 Azure [帐户](/documentation/articles/azure-glossary-cloud-terminology/#account)登录。 如果还没有帐户，可以注册[试用版](/pricing/1rmb-trial)。
2. 在门户中，单击“+ 新建” > “网络” > “虚拟网络”。
3. 在显示的“虚拟网络”边栏选项卡中，让“选择部署模型”下的“Resource Manager”保持选中状态，然后单击“创建”。
4. 在显示的“创建虚拟网络”边栏选项卡中输入以下值，然后单击“创建”按钮：

    |设置|值|
    |---|---|
    |名称|MyVnet|
    |地址空间|*10.0.0.0/16*|
    |子网名称|公共|
    |子网地址范围|*10.0.0.0/24*|
    |资源组|让“新建”保持选中状态，并输入 MyResourceGroup|
    |订阅和位置|选择订阅和位置。

    如果不熟悉 Azure，请详细了解[资源组](/documentation/articles/azure-glossary-cloud-terminology/#resource-group)、[订阅](/documentation/articles/azure-glossary-cloud-terminology/#subscription)和位置（也称为区域）。
6. 该门户仅允许用户在创建 VNet 时创建一个子网。 对于本教程，将在创建 VNet 后创建第二个子网。 以后，可以将能够通过 Internet 访问的资源连接到公共子网，将无法通过 Internet 访问的资源连接到专用子网。 若要创建第二个子网，请在门户顶部的“搜索资源”框中输入 MyVnet。 当“MyVnet”出现在搜索结果中时，请单击它。 注意：如果订阅中有多个同名的 VNet，则会看到每个同名的 VNet 下面列出资源组名称。 请确保单击其下面包含 MyResourceGroup 的 MyVnet 结果。
7. 在显示的“MyVnet”边栏选项卡中，单击“设置”下面的“子网”。
8. 在“MyVnet - 子网”边栏选项卡中，单击“+子网”。
9. 在“添加子网”边栏选项卡中，输入“专用”作为“名称”，输入“10.0.1.0/24”作为“地址范围”，然后单击“确定”。
10. 在“MyVnet - 子网”边栏选项卡中查看子网。 将会看到所创建的“公共”和“专用”子网。
11. 可选：若要删除在本教程中创建的资源，请完成本文[删除资源](#delete-portal)部分中的步骤。

## <a name="cli"></a>CLI
无论是在 Windows、Linux 还是 macOS 上，执行的 CLI 命令都是相同的，但不同操作系统 shell 中的脚本会有差异。 以下说明适用于执行包含 CLI 命令的 Bash 脚本：

    #!/bin/bash

    # Create a resource group.
    az group create \
      --name MyResourceGroup \
      --location chinaeast

    # Create a virtual network with one subnet.
    az network vnet create \
      --name MyVnet \
      --resource-group MyResourceGroup \
      --subnet-name Public

    # Create an additional subnet within the VNet.
    az network vnet subnet create \
      --name Private \
      --address-prefix 10.0.1.0/24 \
      --vnet-name MyVnet \
      --resource-group MyResourceGroup

## <a name="powershell"></a>PowerShell
1. 安装最新版本的 Azure PowerShell [AzureRm](https://www.powershellgallery.com/packages/AzureRM/) 模块。 如果不熟悉 Azure PowerShell，请阅读 [Azure PowerShell 概述](https://docs.microsoft.com/zh-cn/powershell/azure/overview)一文。
2. 单击 Windows“开始”按钮，键入 PowerShell，然后在搜索结果中单击“PowerShell”，启动 PowerShell 会话。
3. 在 PowerShell 窗口中输入 `login-azurermaccount` 命令，使用 Azure [帐户](/documentation/articles/azure-glossary-cloud-terminology/#account)登录。 如果还没有帐户，可以注册[试用版](/pricing/1rmb-trial)。
4. 在浏览器中复制以下脚本：

        # Create a resource group
        New-AzureRmResourceGroup `
          -Name MyResourceGroup `
          -Location chinaeast

        # Create two subnets
        $Subnet1 = New-AzureRmVirtualNetworkSubnetConfig `
          -Name Public `
          -AddressPrefix 10.0.0.0/24
        $Subnet2 = New-AzureRmVirtualNetworkSubnetConfig `
          -Name Private `
          -AddressPrefix 10.0.1.0/24

        # Create a virtual network
        $Vnet=New-AzureRmVirtualNetwork `
          -ResourceGroupName MyResourceGroup `
          -Location chinaeast `
          -Name MyVnet `
          -AddressPrefix 10.0.0.0/16 `
          -Subnet $Subnet1,$Subnet2
        #

5. 若要执行该脚本，请在 PowerShell 窗口中单击右键。
6. 在 PowerShell 窗口中复制并粘贴以下命令，查看 VNet 的子网：

        $Vnet = $Vnet.subnets | Format-Table Name, AddressPrefix

7. 可选：若要删除在本教程中创建的资源，请完成本文[删除资源](#delete-powershell)部分中的步骤。

## <a name="template"></a>模板

可以使用 Azure Resource Manager 模板部署 VNet。 若要详细了解模板，请阅读[什么是 Resource Manager](/documentation/articles/resource-group-overview/#template-deployment) 一文。 若要访问模板并了解其参数，请查看 [Create a VNet with two subnets template](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vnet-two-subnets/)（使用两个子网模板创建 VNet）网页。 可以使用[门户](#template-portal)、[CLI](#template-cli) 或 [PowerShell](#template-powershell) 部署模板。

可选：若要删除在本教程中创建的资源，请完成本文[删除资源](#delete)部分所有小节中的步骤。

### <a name="template-portal"></a>门户

1. 在浏览器中打开模板[网页](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vnet-two-subnets)。
2. 单击“部署到 Azure”按钮，打开 Azure 门户登录页。
3. 使用 Azure [帐户](/documentation/articles/azure-glossary-cloud-terminology/#account)登录到门户。 如果还没有帐户，可以注册[试用版](/pricing/1rmb-trial)。
4. 输入以下参数值：

    |参数|值|
    |---|---|
    |订阅|选择你的订阅。|
    |资源组|MyResourceGroup|
    |位置|选择一个位置。|
    |VNet 名称|MyVnet|
    |VNet 地址前缀|10.0.0.0/16|
    |Subnet1Prefix|10.0.0.0/24|
    |Subnet1Name|公共|
    |Subnet2Prefix|10.0.1.0/24|
    |Subnet2Name|专用|

5. 同意条款和条件，然后单击“购买”部署 VNet。

### <a name="template-cli"></a>CLI

1. 在 Internet 浏览器中打开 Azure [门户](https://portal.azure.cn)并使用 Azure [帐户](/documentation/articles/azure-glossary-cloud-terminology/#account)登录。 如果还没有帐户，可以注册[试用版](/pricing/1rmb-trial)。
2. 在门户顶部“搜索资源”栏的右侧，单击“>_”图标启动 Bash Azure Cloud Shell（预览版）。 Cloud Shell 窗格显示在门户底部，几秒钟后将显示 **username@Azure:~$** 提示符。 Cloud Shell 使用在门户中进行身份验证时所用的凭据将你自动登录到 Azure。
3. 输入以下命令，为 VNet 创建资源组： `az group create --name MyResourceGroup --location chinaeast`
4. 可以使用以下值部署模板：
    - 默认参数值：输入以下命令：  `az group deployment create --resource-group MyResourceGroup --name VnetTutorial --template-uri https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/101-vnet-two-subnets/azuredeploy.json`
    - 自定义参数值：部署模板之前下载并修改模板、使用命令行中的参数部署模板，或使用单独的参数文件部署模板。 可以在 [Create a VNet with two subnets template](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vnet-two-subnets/)（使用两个子网模板创建 VNet）网页上单击“在 GitHub 上浏览”按钮，下载模板和参数文件。 在 GitHub 中单击 azuredeploy.parameters.json 或 azuredeploy.json 文件，然后单击该文件对应的“Raw”（原始文件）按钮。 在浏览器中，复制内容并将其保存到计算机上的某个文件。 修改模板中的参数值，或使用单独的参数文件部署模板。  

    若要详细了解如何使用这些方法部署模板，请键入 `az group deployment create --help`。

### <a name="template-powershell"></a>PowerShell

1. 安装最新版本的 Azure PowerShell [AzureRm](https://www.powershellgallery.com/packages/AzureRM/) 模块。 如果不熟悉 Azure PowerShell，请阅读 [Azure PowerShell 概述](https://docs.microsoft.com/zh-cn/powershell/azure/overview)一文。
2. 单击 Windows“开始”按钮，键入 PowerShell，然后在搜索结果中单击“PowerShell”，启动 PowerShell 会话。
3. 在 PowerShell 窗口中输入 `login-azurermaccount` 命令，使用 Azure [帐户](/documentation/articles/azure-glossary-cloud-terminology/#account)登录。 如果还没有帐户，可以注册[试用版](/pricing/1rmb-trial)。
4. 输入以下命令，为 VNet 创建资源组： `New-AzureRmResourceGroup -Name MyResourceGroup -Location chinaeast`
5. 可以使用以下值部署模板：
    - 默认参数值：为此，请输入以下命令：  `New-AzureRmResourceGroupDeployment -Name VnetTutorial -ResourceGroupName MyResourceGroup -TemplateUri https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/101-vnet-two-subnets/azuredeploy.json`        
    - 自定义参数值：为此，可以在部署模板之前下载并修改模板、使用命令行中的参数部署模板，或使用单独的参数文件部署模板。 可以在 [Create a VNet with two subnets template](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vnet-two-subnets/)（使用两个子网模板创建 VNet）网页上单击“在 GitHub 上浏览”按钮，下载模板和参数文件。 在 GitHub 中单击 azuredeploy.parameters.json 或 azuredeploy.json 文件，然后单击该文件对应的“Raw”（原始文件）按钮。 在浏览器中，复制内容并将其保存到计算机上的某个文件。 修改模板中的参数值，或使用单独的参数文件部署模板。  

    若要详细了解如何使用这些方法部署模板，请键入 `Get-Help New-AzureRmResourceGroupDeployment`。 

## <a name="delete"></a>删除资源
完成本教程后，可以删除资源，以免产生使用费。 删除资源组也会删除其中的所有资源。

### <a name="delete-portal"></a>门户

1. 在门户顶部的“搜索资源”框中开始键入 MyResourceGroup。 当“MyResourceGroup”出现在搜索结果中时，请单击它。
2. 在显示的“MyResourceGroup”边栏选项卡中，单击顶部附近的“删除”图标。
3. 若要确认删除，请在“键入资源组名称:”框中输入 MyResourceGroup，然后单击“删除”。

### <a name="delete-cli"></a>CLI

在 Cloud Shell 提示符处输入以下命令：`az group delete --name MyResourceGroup --yes`

### <a name="delete-powershell"></a>PowerShell

在 PowerShell 提示符处输入以下命令：`Remove-AzureRmResourceGroup -Name MyResourceGroup`

## <a name="next-steps"></a>后续步骤

- 若要了解所有的 VNet 和子网设置，请阅读[管理 VNet](/documentation/articles/virtual-network-manage-network/#view-vnet) 和[管理子网](/documentation/articles/virtual-network-manage-subnet/#create-subnet)这两篇文章。 可以根据不同的要求，使用多种选项创建生产 VNet 和子网。
- 通过创建[网络安全组](/documentation/articles/virtual-networks-nsg/) (NSG) 并将其应用到子网来筛选入站和出站子网流量。
- 创建 [Windows](/documentation/articles/virtual-machines-windows-hero-tutorial/) 或 [Linux](/documentation/articles/virtual-machines-linux-quick-create-portal/) VM 并将其连接到 VNet。
- 使用 [VNet 对等互连](/documentation/articles/virtual-network-peering-overview/)将 VNet 连接到同一位置中的其他 VNet。
- 使用[站点到站点虚拟专用网络](/documentation/articles/vpn-gateway-howto-multi-site-to-site-resource-manager-portal/) (VPN) 或 [ExpressRoute](/documentation/articles/expressroute-howto-linkvnet-portal-resource-manager/) 线路将 VNet 连接到本地网络。

<!--Update_Description: wording update-->