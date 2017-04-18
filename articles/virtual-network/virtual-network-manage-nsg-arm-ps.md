<properties
    pageTitle="使用 PowerShell 管理 NSG | Azure"
    description="了解如何使用 PowerShell 管理现有 NSG。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="carmonm"
    editor=""
    tags="azure-resource-manager" />  

<tags
    ms.assetid="3706ce6c-d9ae-46cb-a048-f0a4e84dc5cc"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/14/2016"
    wacn.date="12/26/2016"
    ms.author="jdial" />  


# 使用 PowerShell 管理 NSG

[AZURE.INCLUDE [virtual-network-manage-arm-selectors-include.md](../../includes/virtual-network-manage-nsg-arm-selectors-include.md)]

[AZURE.INCLUDE [virtual-network-manage-nsg-intro-include.md](../../includes/virtual-network-manage-nsg-intro-include.md)]

> [AZURE.NOTE]
Azure 具有两种不同的部署模型，用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是经典部署模型。
>

[AZURE.INCLUDE [virtual-network-manage-nsg-arm-scenario-include.md](../../includes/virtual-network-manage-nsg-arm-scenario-include.md)]

[AZURE.INCLUDE [azure-ps-prerequisites-include.md](../../includes/azure-ps-prerequisites-include.md)]

## 检索信息
可以查看你的现有 NSG、检索现有 NSG 的规则和查找与 NSG 关联的资源。

### 查看现有 NSG
若要查看订阅中的全部现有 NSG，请运行 `Get-AzureRmNetworkSecurityGroup` cmdlet。

预期结果：

    Name                 : NSG-BackEnd
    ResourceGroupName    : RG-NSG
    Location             : chinanorth
    Id                   : /subscriptions/[Subscription Id]/resourceGroups/RG-NSG/providers/
                           Microsoft.Network/networkSecurityGroups/NSG-BackEnd
    Etag                 : W/"[Id]"
    ResourceGuid         : [Id]
    ProvisioningState    : Succeeded
    Tags                 :                            
    SecurityRules        : [...]
    DefaultSecurityRules : [...]
    NetworkInterfaces    : [...]
    Subnets              : [...]

    Name                 : NSG-FrontEnd
    ResourceGroupName    : RG-NSG
    Location             : chinaeast
    Id                   : /subscriptions/[Subscription Id]/resourceGroups/NRP-RG/providers/
                           Microsoft.Network/networkSecurityGroups/NSG-FrontEnd
    Etag                 : W/"[Id]"
    ResourceGuid         : [Id]
    ProvisioningState    : Succeeded
    Tags                 : 
    SecurityRules        : [...]
    DefaultSecurityRules : [...]
    NetworkInterfaces    : [...]
    Subnets              : [...]

    Name                 : WEB1
    ResourceGroupName    : RG101
    Location             : chinaeast
    Id                   : /subscriptions/[Subscription Id]/resourceGroups/RG101/providers/M
                           icrosoft.Network/networkSecurityGroups/WEB1
    Etag                 : W/"[Id]"
    ResourceGuid         : [Id]
    ProvisioningState    : Succeeded
    Tags                 : 
    SecurityRules        : [...]
    DefaultSecurityRules : [...]
    NetworkInterfaces    : [...]
    Subnets              : [...]


若要查看特定资源组中 NSG 的列表，请运行 `Get-AzureRmNetworkSecurityGroup` cmdlet。

预期输出：

    Name                 : NSG-BackEnd
    ResourceGroupName    : RG-NSG
    Location             : chinanorth
    Id                   : /subscriptions/[Subscription Id]/resourceGroups/RG-NSG/providers/
                           Microsoft.Network/networkSecurityGroups/NSG-BackEnd
    Etag                 : W/"[Id]"
    ResourceGuid         : [Id]
    ProvisioningState    : Succeeded
    Tags                 :                            
    SecurityRules        : [...]
    DefaultSecurityRules : [...]
    NetworkInterfaces    : [...]
    Subnets              : [...]

    Name                 : NSG-FrontEnd
    ResourceGroupName    : RG-NSG
    Location             : chinaeast
    Id                   : /subscriptions/[Subscription Id]/resourceGroups/NRP-RG/providers/
                           Microsoft.Network/networkSecurityGroups/NSG-FrontEnd
    Etag                 : W/"[Id]"
    ResourceGuid         : [Id]
    ProvisioningState    : Succeeded
    Tags                 : 
    SecurityRules        : [...]
    DefaultSecurityRules : [...]
    NetworkInterfaces    : [...]
    Subnets              : [...]

### 列出 NSG 的所有规则
若要查看名为 **NSG-FrontEnd** 的 NSG 的规则，请输入以下命令：

    Get-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG -Name NSG-FrontEnd | Select SecurityRules -ExpandProperty SecurityRules

预期输出：

    Name                     : rdp-rule
    Id                       : /subscriptions/[Subscription Id]/resourceGroups/RG-NSG/providers/                           Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/rdp-rule
    Etag                     : W/"[Id]"
    ProvisioningState        : Succeeded
    Description              : Allow RDP
    Protocol                 : Tcp
    SourcePortRange          : *
    DestinationPortRange     : 3389
    SourceAddressPrefix      : Internet
    DestinationAddressPrefix : *
    Access                   : Allow
    Priority                 : 100
    Direction                : Inbound

    Name                     : web-rule
    Id                       : /subscriptions/[Subscription Id]/resourceGroups/RG-NSG/providers/                           Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/web-rule
    Etag                     : W/"[Id]"
    ProvisioningState        : Succeeded
    Description              : Allow HTTP
    Protocol                 : Tcp
    SourcePortRange          : *
    DestinationPortRange     : 80
    SourceAddressPrefix      : Internet
    DestinationAddressPrefix : *
    Access                   : Allow
    Priority                 : 101
    Direction                : Inbound

> [AZURE.NOTE]
还可以使用 `Get-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG -Name "NSG-FrontEnd" | Select DefaultSecurityRules -ExpandProperty DefaultSecurityRules` 列出 **NSG-FrontEnd** NSG 的默认规则。
> 

### <a name="View-NSGs-associations"></a>查看 NSG 关联项
若要查看与 **NSG-FrontEnd** NSG 关联的资源，请运行以下命令：

    Get-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG -Name NSG-FrontEnd

查找 **NetworkInterfaces** 和 **Subnets** 属性，如下所示：

    NetworkInterfaces    : []
    Subnets              : [
                             {
                               "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/RG-NSG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
                               "IpConfigurations": []
                             }
                           ]

在前面的示例中，NSG 不与任何网络接口 (NIC) 关联，它与名为 **FrontEnd** 的子网关联。

## 管理规则
可向现有 NSG 添加规则、编辑现有规则和删除规则。

### 添加规则
若要向 **NSG-FrontEnd** NSG 添加允许来自任何计算机的**入站**流量流入端口 **443** 的规则，请完成以下步骤：

1. 运行以下命令，检索现有 NSG 并将其存储在变量中：

        $nsg = Get-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG -Name NSG-FrontEnd

2. 运行以下命令，将规则添加到 NSG 中：

        Add-AzureRmNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg `
        -Name https-rule `
        -Description "Allow HTTPS" `
        -Access Allow `
        -Protocol Tcp `
        -Direction Inbound `
        -Priority 102 `
        -SourceAddressPrefix * `
        -SourcePortRange * `
        -DestinationAddressPrefix * `
        -DestinationPortRange 443

3. 若要保存对 NSG 所做的更改，请运行以下命令：

        Set-AzureRmNetworkSecurityGroup -NetworkSecurityGroup $nsg

    预期输出仅显示安全规则：
   
        Name                 : NSG-FrontEnd
        ...
        SecurityRules        : [
                                 {
                                   "Name": "rdp-rule",
                                   ...
                                 },
                                 {
                                   "Name": "web-rule",
                                   ...
                                 },
                                 {
                                   "Name": "https-rule",
                                   "Etag": "W/\"[Id]\"",
                                   "Id": "/subscriptions/[Subscription Id]/resourceGroups/RG-NSG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/https-rule",
                                   "Description": "Allow HTTPS",
                                   "Protocol": "Tcp",
                                   "SourcePortRange": "*",
                                   "DestinationPortRange": "443",
                                   "SourceAddressPrefix": "*",
                                   "DestinationAddressPrefix": "*",
                                   "Access": "Allow",
                                   "Priority": 102,
                                   "Direction": "Inbound",
                                   "ProvisioningState": "Succeeded"
                                 }
                               ]

### 更改规则
若要将上面创建的规则更改为仅允许来自 **Internet** 的入站流量，请按照以下步骤进行操作。

1. 运行以下命令，检索现有 NSG 并将其存储在变量中：

        $nsg = Get-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG -Name NSG-FrontEnd

2. 使用新规则设置运行以下命令：

        Set-AzureRmNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg `
        -Name https-rule `
        -Description "Allow HTTPS" `
        -Access Allow `
        -Protocol Tcp `
        -Direction Inbound `
        -Priority 102 `
        -SourceAddressPrefix * `
        -SourcePortRange Internet `
        -DestinationAddressPrefix * `
        -DestinationPortRange 443

3. 若要保存对 NSG 所做的更改，请运行以下命令：

        Set-AzureRmNetworkSecurityGroup -NetworkSecurityGroup $nsg

    预期输出仅显示安全规则：
   
        Name                 : NSG-FrontEnd
        ...
        SecurityRules        : [
                                 {
                                   "Name": "rdp-rule",
                                   ...
                                 },
                                 {
                                   "Name": "web-rule",
                                   ...
                                 },
                                 {
                                   "Name": "https-rule",
                                   "Etag": "W/\"[Id]\"",
                                   "Id": "/subscriptions/[Subscription Id]/resourceGroups/RG-NSG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd/securityRules/https-rule",
                                   "Description": "Allow HTTPS",
                                   "Protocol": "Tcp",
                                   "SourcePortRange": "*",
                                   "DestinationPortRange": "443",
                                   "SourceAddressPrefix": "Internet",
                                   "DestinationAddressPrefix": "*",
                                   "Access": "Allow",
                                   "Priority": 102,
                                   "Direction": "Inbound",
                                   "ProvisioningState": "Succeeded"
                                 }
                               ]

### 删除规则
1. 运行以下命令，检索现有 NSG 并将其存储在变量中：

        $nsg = Get-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG -Name NSG-FrontEnd

2. 运行以下命令，从 NSG 中删除规则：

        Remove-AzureRmNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg -Name https-rule

3. 运行以下命令，保存对 NSG 所做的更改：

        Set-AzureRmNetworkSecurityGroup -NetworkSecurityGroup $nsg

    预期输出仅显示安全规则，请注意，不再列出 **https-rule**：
   
        Name                 : NSG-FrontEnd
        ...
        SecurityRules        : [
                                 {
                                   "Name": "rdp-rule",
                                   ...
                                 },
                                 {
                                   "Name": "web-rule",
                                   ...
                                 }
                               ]

## 管理关联项
可将 NSG 关联到子网和 NIC。还可取消 NSG 与任何相关联的资源之间的关联。

### 将 NSG 关联到 NIC
若要将 **NSG-FrontEnd** NSG 关联到 **TestNICWeb1** NIC，请完成以下步骤：

1. 运行以下命令，检索现有 NSG 并将其存储在变量中：

        $nsg = Get-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG -Name NSG-FrontEnd

2. 运行以下命令，检索现有 NIC 并将其存储在变量中：

        $nic = Get-AzureRmNetworkInterface -ResourceGroupName RG-NSG -Name TestNICWeb1

3. 输入以下命令，将 **NIC** 变量的 **NetworkSecurityGroup** 属性设置为 **NSG** 变量的值：

        $nic.NetworkSecurityGroup = $nsg

4. 若要保存对 NIC 所做的更改，请运行以下命令：

        Set-AzureRmNetworkInterface -NetworkInterface $nic

    预期输出仅显示 **NetworkSecurityGroup** 属性：
   
        NetworkSecurityGroup : {
                                 "SecurityRules": [],
                                 "DefaultSecurityRules": [],
                                 "NetworkInterfaces": [],
                                 "Subnets": [],
                                 "Id": "/subscriptions/[Subscription Id]/resourceGroups/RG-NSG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd"
                               }

### <a name="Dissociate-an-NSG-from-a-NIC"></a>取消 NSG 与 NIC 之间的关联
若要取消 **NSG-FrontEnd** NSG 与 **TestNICWeb1** NIC 之间的关联，请完成以下步骤：

1. 运行以下命令，检索现有 NSG 并将其存储在变量中：

        $nsg = Get-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG -Name NSG-FrontEnd

2. 运行以下命令，检索现有 NIC 并将其存储在变量中：

        $nic = Get-AzureRmNetworkInterface -ResourceGroupName RG-NSG -Name TestNICWeb1

3. 运行以下命令，将 **NIC** 变量的 **NetworkSecurityGroup** 属性设置为 **$null**：

        $nic.NetworkSecurityGroup = $null

4. 若要保存对 NIC 所做的更改，请运行以下命令：

        Set-AzureRmNetworkInterface -NetworkInterface $nic

    预期输出仅显示 **NetworkSecurityGroup** 属性：
   
        NetworkSecurityGroup : null

### <a name="Dissociate-an-NSG-from-a-subnet"></a>取消 NSG 与子网之间的关联
若要取消 **NSG-FrontEnd** NSG 与 **FrontEnd** 子网之间的关联，请完成以下步骤：

1. 运行以下命令，检索现有 VNet 并将其存储在变量中：

        $vnet = Get-AzureRmVirtualNetwork -ResourceGroupName RG-NSG -Name TestVNet

2. 运行以下命令，检索 **FrontEnd** 子网并将其存储在变量中：

        $subnet = Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name FrontEnd

3. 输入以下命令，将 **subnet** 变量的 **NetworkSecurityGroup** 属性设置为 **$null**：

        $subnet.NetworkSecurityGroup = $null

4. 若要保存对子网所做的更改，请运行以下命令：

        Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

    预期输出仅显示 **FrontEnd** 子网的属性。请注意，**NetworkSecurityGroup** 没有属性：
   
            ...
            Subnets           : [
                                  {
                                    "Name": "FrontEnd",
                                    "Etag": "W/\"[Id]\"",
                                    "Id": "/subscriptions/[Subscription Id]/resourceGroups/RG-NSG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
                                    "AddressPrefix": "192.168.1.0/24",
                                    "IpConfigurations": [
                                      {
                                        "Id": "/subscriptions/[Subscription Id]/resourceGroups/RG-NSG/providers/Microsoft.Network/networkInterfaces/TestNICWeb2/ipConfigurations/ipconfig1"
                                      },
                                      {
                                        "Id": "/subscriptions/[Subscription Id]/resourceGroups/RG-NSG/providers/Microsoft.Network/networkInterfaces/TestNICWeb1/ipConfigurations/ipconfig1"
                                      }
                                    ],
                                    "ProvisioningState": "Succeeded"
                                  },
                                    ...
                                ]

### 将 NSG 关联到子网
若要再次将 **NSG-FrontEnd** NSG 关联到 **FronEnd** 子网，请完成以下步骤：

1. 运行以下命令，检索现有 VNet 并将其存储在变量中：

        $vnet = Get-AzureRmVirtualNetwork -ResourceGroupName RG-NSG -Name TestVNet

2. 运行以下命令，检索 **FrontEnd** 子网并将其存储在变量中：

        $subnet = Get-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name FrontEnd

3. 运行以下命令，检索现有 NSG 并将其存储在变量中：

        $nsg = Get-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG -Name NSG-FrontEnd

4. 运行以下命令，将 **subnet** 变量的 **NetworkSecurityGroup** 属性设置为 **$null**：

        $subnet.NetworkSecurityGroup = $nsg

5. 若要保存对子网所做的更改，请运行以下命令：

        Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

    预期输出仅显示 **FrontEnd** 子网的 **NetworkSecurityGroup** 属性：
   
        ...
        "NetworkSecurityGroup": {
                                  "SecurityRules": [],
                                  "DefaultSecurityRules": [],
                                  "NetworkInterfaces": [],
                                  "Subnets": [],
                                  "Id": "/subscriptions/[Subscription Id]/resourceGroups/RG-NSG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd"
                                }
        ...

## 删除 NSG
仅当 NSG 不与任何资源关联时，才能删除 NSG。若要删除 NSG，请按照以下步骤进行操作。

1. 若要查看与 NSG 关联的资源，请运行 `azure network nsg show`，如[查看 NSG 关联项](#View-NSGs-associations)中所示。
2. 如果 NSG 关联到任意 NIC，请为每个 NIC 运行 `azure network nic set`，如[取消 NSG 与 NIC 之间的关联](#Dissociate-an-NSG-from-a-NIC)中所示。
3. 如果 NSG 关联到任意子网，请为每个子网运行 `azure network vnet subnet set`，如[取消 NSG 与子网之间的关联](#Dissociate-an-NSG-from-a-subnet)中所示。
4. 若要删除 NSG，请运行以下命令：

        Remove-AzureRmNetworkSecurityGroup -ResourceGroupName RG-NSG -Name NSG-FrontEnd -Force

   > [AZURE.NOTE]
   `-Force` 参数可确保无需确认删除。
   > 

## 后续步骤
* 为 NSG [启用日志记录](/documentation/articles/virtual-network-nsg-manage-log/)。

<!---HONumber=Mooncake_1219_2016-->