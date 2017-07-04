<properties
    pageTitle="Azure 虚拟网络对等互连 - 模板 | Azure"
    description="了解如何使用 Azure Resource Manager 模板创建虚拟网络对等互连。"
    services="virtual-network"
    documentationcenter=""
    author="narayanannamalai"
    manager="jefco"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="75f8d10e-23e8-44bd-9972-aab74048cf38"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="09/14/2016"
    wacn.date="03/24/2017"
    ms.author="narayan;annahar" />  


# 使用 Azure Resource Manager 模板创建虚拟网络对等互连
[AZURE.INCLUDE [virtual-networks-create-vnet-selectors-arm-include](../../includes/virtual-networks-create-vnetpeering-selectors-arm-include.md)]

[AZURE.INCLUDE [virtual-networks-create-vnet-intro](../../includes/virtual-networks-create-vnetpeering-intro-include.md)]

[AZURE.INCLUDE [virtual-networks-create-vnet-scenario-basic-include](../../includes/virtual-networks-create-vnetpeering-scenario-basic-include.md)]

若要使用 Resource Manager 模板创建 VNet 对等互连，请完成以下步骤：

1. 如果你从未使用过 Azure PowerShell，请参阅 [How to Install and Configure Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)（如何安装和配置 Azure PowerShell），并始终按照说明进行操作，以登录到 Azure 并选择你的订阅。

    > [AZURE.NOTE]
    用于管理 VNet 对等互连的 PowerShell cmdlet 随附于 [Azure PowerShell 1.6](http://www.powershellgallery.com/packages/Azure/1.6.0)。
    > 
    > 
2. 以下文本显示基于上述方案的从 VNet1 到 VNet2 的 VNet 对等互连链接定义。复制以下内容并将其保存到名为 VNetPeeringVNet1.json 的文件中。

            {
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
            },
            "variables": {
            },
            "resources": [
                {
                "apiVersion": "2016-06-01",
                "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
                "name": "VNet1/LinkToVNet2",
                "location": "[resourceGroup().location]",
                "properties": {
                "allowVirtualNetworkAccess": true,
                "allowForwardedTraffic": false,
                "allowGatewayTransit": false,
                "useRemoteGateways": false,
                    "remoteVirtualNetwork": {
                    "id": "[resourceId('Microsoft.Network/virtualNetworks', 'vnet2')]"       
            }
                }
                }
            ]
            }

3. 以下部分显示基于上述方案的从 VNet2 到 VNet1 的 VNet 对等互连链接定义。复制以下内容并将其保存到名为 VNetPeeringVNet2.json 的文件中。

            {
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
            },
            "variables": {
            },
            "resources": [
                {
                "apiVersion": "2016-06-01",
                "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
                "name": "VNet2/LinkToVNet1",
                "location": "[resourceGroup().location]",
                "properties": {
                "allowVirtualNetworkAccess": true,
                "allowForwardedTraffic": false,
                "allowGatewayTransit": false,
                "useRemoteGateways": false,
                    "remoteVirtualNetwork": {
                    "id": "[resourceId('Microsoft.Network/virtualNetworks', 'vnet1')]"       
                    }
                }
                }
            ]
            }

    如以上模板所示，VNet 对等互连有几个可配置属性：
   
    | 选项 | 说明 | 默认 |
    |:--- |:--- |:--- |
    | 允许虚拟网络访问 |是否将对等 VNet 的地址空间包括为 Virtual\_network 标记的一部分。 |是 |
    | 允许转发的流量 |是否接受或丢弃不是源自对等 VNet 的流量。 |否 |
    | 允许网关传输 |允许对等 VNet 使用你的 VNet 网关。 |否 |
    | 使用远程网关 |使用对等方的 VNet 网关。对等 VNet 必须已配置网关并且已选择 AllowGatewayTransit。如果你已配置了网关，则无法使用此选项。 |否 |
   
    VNet 对等互连中的每个链接都有上述属性集。例如，可针对 VNet1 到 VNet2 的 VNet 对等互连链接将“允许虚拟网络访问”设置为“Ture”，而针对另一方向的 VNet 对等互连链接设置为“False”。
4. 若要部署模板文件，可以运行 `New-AzureRmResourceGroupDeployment` 来创建或更新部署。有关使用 Resource Manager 模板的详细信息，请参阅此[文章](/documentation/articles/resource-group-template-deploy/)。

        New-AzureRmResourceGroupDeployment -ResourceGroupName <resource group name> -TemplateFile <template file path> -DeploymentDebugLogLevel all

    > [AZURE.NOTE]
    适当替换资源组名称和模板文件。
    > 
    > 
   
    以下是基于前一方案的示例：

        New-AzureRmResourceGroupDeployment -ResourceGroupName VNet101 -TemplateFile .\VNetPeeringVNet1.json -DeploymentDebugLogLevel all

    返回的输出：
   
        DeploymentName        : VNetPeeringVNet1
        ResourceGroupName    : VNet101
        ProvisioningState        : Succeeded
        Timestamp            : MM/DD/YEAR 9:05:03 AM
        Mode            : Incremental
        TemplateLink        :
        Parameters            :
        Outputs            :
        DeploymentDebugLogLevel : RequestContent, ResponseContent

        New-AzureRmResourceGroupDeployment -ResourceGroupName VNet101 -TemplateFile .\VNetPeeringVNet2.json -DeploymentDebugLogLevel all

    返回的输出：
   
        DeploymentName        : VNetPeeringVNet2
        ResourceGroupName    : VNet101
        ProvisioningState        : Succeeded
        Timestamp            : MM/DD/YEAR 9:07:22 AM
        Mode            : Incremental
        TemplateLink        :
        Parameters            :
        Outputs            :
        DeploymentDebugLogLevel : RequestContent, ResponseContent
5. 部署完成后，请运行以下 cmdlet 查看对等互连状态：

        Get-AzureRmVirtualNetworkPeering -VirtualNetworkName VNet1 -ResourceGroupName VNet101 -Name linktoVNet2

    返回的输出：
   
        Name            : LinkToVNet2
        Id                : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/VNet101/providers/Microsoft.Network/virtualNetworks/VNet1/virtualNetworkPeerings/LinkToVNet2
        Etag            : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        ResourceGroupName    : VNet101
        VirtualNetworkName    : VNet1
        ProvisioningState        : Succeeded
        RemoteVirtualNetwork    : {
                                            "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/VNet101/providers/Microsoft.Network/virtualNetworks/VNet2"
                                        }
        AllowVirtualNetworkAccess    : True
        AllowForwardedTraffic            : False
        AllowGatewayTransit              : False
        UseRemoteGateways                : False
        RemoteGateways                   : null
        RemoteVirtualNetworkAddressSpace : null
   
    在此方案中建立对等互连后，应可跨这两个 VNet 启动任意 VM 之间的连接。默认情况下，`AllowVirtualNetworkAccess` 为 *True*，VNet 对等互连将预配适当的 ACL 来允许 VNet 之间的通信。仍可应用网络安全组 (NSG) 规则来阻止特定子网或虚拟机之间的连接，从而实现两个虚拟网络之间访问权限的细粒度控制。阅读[网络安全组](/documentation/articles/virtual-networks-create-nsg-arm-ps/)一文，了解有关 NSG 的详细信息。

[AZURE.INCLUDE [virtual-networks-create-vnet-scenario-crosssub-include](../../includes/virtual-networks-create-vnetpeering-scenario-crosssub-include.md)]

若要跨订阅创建 VNet 对等互连，请完成以下步骤：

1. 使用订阅 A 的特权用户 A 帐户登录到 Azure，并运行以下 cmdlet：

        New-AzureRmRoleAssignment -SignInName <UserB ID> -RoleDefinitionName "Network Contributor" -Scope /subscriptions/<Subscription-A-ID>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Network/VirtualNetwork/VNet5

    不一定非要这样做。只要请求匹配，即使用户分别提出针对各个 VNet 的对等互连请求，也可建立对等互连。添加另一个 VNet 的特权用户作为本地 VNet 用户可以更轻松地执行安装程序。
2. 使用订阅 B 的特权用户 B 帐户登录到 Azure，并运行以下 cmdlet：

        New-AzureRmRoleAssignment -SignInName <UserA ID> -RoleDefinitionName "Network Contributor" -Scope /subscriptions/<Subscription-B-ID>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Network/VirtualNetwork/VNet3

    > [AZURE.IMPORTANT]
    如果要在通过 Azure Resource Manager 部署模型创建的两个 VNet 之间创建对等互连，请继续执行本部分中的余下步骤。如果两个 VNet 是通过不同部署模型创建的，请跳过本部分的余下步骤并完成本文的[将通过不同部署模型创建的虚拟网络对等互连](#x-model)部分中所述的步骤。

3. 在用户 A 的登录会话中，运行以下 cmdlet：

        New-AzureRmResourceGroupDeployment -ResourceGroupName VNet101 -TemplateFile .\VNetPeeringVNet3.json -DeploymentDebugLogLevel all

    JSON 如下：

            {
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
            },
            "variables": {
            },
            "resources": [
                {
                "apiVersion": "2016-06-01",
                "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
                "name": "VNet3/LinkToVNet5",
                "location": "[resourceGroup().location]",
                "properties": {
                "allowVirtualNetworkAccess": true,
                "allowForwardedTraffic": false,
                "allowGatewayTransit": false,
                "useRemoteGateways": false,
                    "remoteVirtualNetwork": {
                    "id": "/subscriptions/<Subscription-B-ID>/resourceGroups/<resource group name>/providers/Microsoft.Network/virtualNetworks/VNet5"
                    }
                }
                }
            ]
            }

4. 在用户 B 的登录会话中，运行以下 cmdlet：

        New-AzureRmResourceGroupDeployment -ResourceGroupName VNet101 -TemplateFile .\VNetPeeringVNet5.json -DeploymentDebugLogLevel all

    JSON 如下：

            {
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
            },
            "variables": {
            },
            "resources": [
                {
                "apiVersion": "2016-06-01",
                "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
                "name": "VNet5/LinkToVNet3",
                "location": "[resourceGroup().location]",
                "properties": {
                "allowVirtualNetworkAccess": true,
                "allowForwardedTraffic": false,
                "allowGatewayTransit": false,
                "useRemoteGateways": false,
                    "remoteVirtualNetwork": {
                    "id": "/subscriptions/Subscription-A-ID /resourceGroups/<resource group name>/providers/Microsoft.Network/virtualNetworks/VNet3"
                    }
                }
                }
            ]
            }

    此方案中的对等互连建立好后，应该能够跨订阅启动两个 Vnet 中的任何虚拟机之间的连接。

[AZURE.INCLUDE [virtual-networks-create-vnet-scenario-transit-include](../../includes/virtual-networks-create-vnetpeering-scenario-transit-include.md)]

1. 在此方案中，可以部署以下示例模板以建立 VNet 对等互连。需要将 `AllowForwardedTraffic` 属性设置为 *Ture*，以允许对等 VNet 中的网络虚拟设备发送和接收流量。

    以下 JSON 是用于创建从 HubVNet 到 VNet1 的 VNet 对等互连的模板内容。请注意将 AllowForwardedTraffic 设置为 false。

            {
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
            },
            "variables": {
            },
            "resources": [
                {
                "apiVersion": "2016-06-01",
                "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
                "name": "HubVNet/LinkToVNet1",
                "location": "[resourceGroup().location]",
                "properties": {
                "allowVirtualNetworkAccess": true,
                "allowForwardedTraffic": false,
                "allowGatewayTransit": false,
                "useRemoteGateways": false,
                    "remoteVirtualNetwork": {
                    "id": "[resourceId('Microsoft.Network/virtualNetworks', 'vnet1')]"       
                    }
                }
                }
            ]
            }

2. 以下 JSON 是用于创建从 VNet1 到 HubVnet 的 VNet 对等互连的模板内容。请注意将 AllowForwardedTraffic 设置为 true。

            {
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
            },
            "variables": {
            },
            "resources": [
                {
                "apiVersion": "2016-06-01",
                "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
                "name": "VNet1/LinkToHubVNet",
                "location": "[resourceGroup().location]",
                "properties": {
                "allowVirtualNetworkAccess": true,
                "allowForwardedTraffic": true,
                "allowGatewayTransit": false,
                "useRemoteGateways": false,
                    "remoteVirtualNetwork": {
                    "id": "[resourceId('Microsoft.Network/virtualNetworks', 'HubVnet')]"       
                    }
                }
                }
            ]
            }

3. 对等互连建立后，可以参考[此文](/documentation/articles/virtual-network-create-udr-arm-ps/)来定义用户定义的路由 (UDR)，以便通过虚拟设备重定向 VNet1 流量以使用其功能。在路由中指定下一个跃点地址时，可以在对等 VNet HubVNet 中将其设置为虚拟设备的 IP 地址。

[AZURE.INCLUDE [virtual-networks-create-vnet-scenario-asmtoarm-include](../../includes/virtual-networks-create-vnetpeering-scenario-asmtoarm-include.md)]

若要在不同部署模型中的虚拟网络之间创建对等互连，请完成以下步骤：

1. 如果要在*同一*订阅中通过不同部署模型部署的 VNet 之间创建对等互连，请跳到步骤 2。在**预览版**中，可在*不同*订阅中创建通过不同部署模型部署的 VNet 之间的对等互连。预览版功能不提供与正式版功能相同级别的可靠性和服务级别协议。如果要在不同订阅中通过不同部署模型部署的 VNet 之间创建对等互连，必须先完成以下任务：
    - 在 PowerShell 中输入以下命令，在 Azure 订阅中注册预览版功能：`Register-AzureRmProviderFeature -FeatureName AllowClassicCrossSubscriptionPeering -ProviderNamespace Microsoft.Network` 和 `Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Network`。
    - 完成本文的[跨订阅对等互连](#x-sub)部分中的步骤 1-2。
2. 以下文本显示本方案中 VNET1 到 VNET2 的 VNet 对等互连链接定义。只需使用一个链接即可将经典虚拟网络对等互连到 Azure Resource Manager 虚拟网络。

    请务必输入经典虚拟网络或 VNET2 所在的订阅 ID，并将 MyResouceGroup 更改为相应的资源组名称。

           {
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
            },
            "variables": {
            },
            "resources": [

                {
                "apiVersion": "2016-06-01",
                "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
                "name": "VNET1/LinkToVNET2",
                "location": "[resourceGroup().location]",
                "properties": {
                "allowVirtualNetworkAccess": true,
                "allowForwardedTraffic": false,
                "allowGatewayTransit": false,
                "useRemoteGateways": false,
                    "remoteVirtualNetwork": {
                    "id": "[resourceId('Microsoft.ClassicNetwork/virtualNetworks', 'VNET2')]"
            }

                }
                }
            ]
            }

3. 若要部署模板文件，请运行以下 cmdlet 创建或更新部署：

            New-AzureRmResourceGroupDeployment -ResourceGroupName MyResourceGroup -TemplateFile .\VnetPeering.json -DeploymentDebugLogLevel all

    返回的输出：
   
        DeploymentName          : VnetPeering
        ResourceGroupName       : MyResourceGroup
        ProvisioningState       : Succeeded
        Timestamp               : XX/XX/YYYY 5:42:33 PM
        Mode                    : Incremental
        TemplateLink            :
        Parameters              :
        Outputs                 :
        DeploymentDebugLogLevel : RequestContent, ResponseContent
4. 部署成功后，可以运行以下 cmdlet 查看对等互连状态：

        Get-AzureRmVirtualNetworkPeering -VirtualNetworkName VNET1 -ResourceGroupName MyResourceGroup -Name LinkToVNET2

    返回的输出：
   
        Name                             : LinkToVNET2
        Id                               : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/MyResource
                                   Group/providers/Microsoft.Network/virtualNetworks/VNET1/virtualNetworkPeering
                                   s/LinkToVNET2
        Etag                             : W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        ResourceGroupName                : MyResourceGroup
        VirtualNetworkName               : VNET1
        PeeringState                     : Connected
        ProvisioningState                : Succeeded
        RemoteVirtualNetwork             : {
                                     "Id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/M
                                   yResourceGroup/providers/Microsoft.ClassicNetwork/virtualNetworks/VNET2"
                                   }
        AllowVirtualNetworkAccess        : True
        AllowForwardedTraffic            : False
        AllowGatewayTransit              : False
        UseRemoteGateways                : False
        RemoteGateways                   : null
        RemoteVirtualNetworkAddressSpace : null

    在经典虚拟网络与 Resource Manager VNet 之间建立对等互连后，应该可以从 VNET1 中的任一虚拟机向 VNET2 中的任一虚拟机发起连接，反之亦然。

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description: wording update-->