<properties
    pageTitle="Azure Service Fabric 的网络模式 | Azure"
    description="介绍 Service Fabric 的常见网络模式以及如何使用 Azure 网络功能创建群集。"
    services="service-fabric"
    documentationcenter=".net"
    author="rwike77"
    manager="timlt"
    editor="" />
<tags
    ms.assetid=""
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="02/27/2017"
    wacn.date="05/02/2017"
    ms.author="ryanwi"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="491c05352214c09800328e6765efa485702a15ee"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="04/21/2017" />

# <a name="service-fabric-networking-patterns"></a>Service Fabric 网络模式
可将 Azure Service Fabric 群集与其他 Azure 网络功能集成。 本文说明如何创建使用以下功能的群集：

- [现有虚拟网络或子网](#existingvnet)
- [静态公共 IP 地址](#staticpublicip)
- [仅限内部的负载均衡器](#internallb)
- [内部和外部负载均衡器](#internalexternallb)

Service Fabric 在标准的虚拟机规模集中运行。 可在虚拟机规模集中使用的任何功能同样可在 Service Fabric 群集中使用。 虚拟机规模集与 Service Fabric 的 Azure Resource Manager 模板的网络部分是相同的。 部署到现有虚拟网络后，可以轻松地整合其他网络功能，例如 Azure ExpressRoute、Azure VPN 网关、网络安全组和虚拟网络对等互连。

与其他网络功能相比，Service Fabric 的独特之处体现在一个方面。 [Azure 门户预览](https://portal.azure.cn)在内部使用 Service Fabric 资源提供程序连接到群集，以获取有关节点和应用程序的信息。 Service Fabric 资源提供程序需要对管理终结点上的 HTTP 网关端口（默认为 19080）具有可公开访问的入站访问权限。 [Service Fabric Explorer](/documentation/articles/service-fabric-visualizing-your-cluster/) 使用该管理终结点来管理群集。 Service Fabric 资源提供程序还使用此端口来查询有关群集的信息，以便在 Azure 门户预览中显示。 

如果无法通过 Service Fabric 资源提供程序访问端口 19080，门户中会显示一条类似于“找不到节点”的消息，并且节点和应用程序列表显示为空。 如果想要在 Azure 门户预览中查看群集，负载均衡器必须公开一个公共 IP 地址，并且网络安全组必须允许端口 19080 上的传入流量。 如果设置不满足这些要求，Azure 门户预览不会显示群集的状态。

## <a name="templates"></a>模板

所有 Service Fabric 模板在[一个下载文件](https://msdnshared.blob.core.windows.net/media/2016/10/SF_Networking_Templates.zip)中提供。 使用以下 PowerShell 命令应可按原样部署模板。 若要部署现有的 Azure 虚拟网络模板或静态公共 IP 模板，请先阅读本文的[初始设置](#initialsetup)部分。

<a id="initialsetup"></a>
## <a name="initial-setup"></a>初始设置

### <a name="existing-virtual-network"></a>现有虚拟网络

在以下示例中，我们从 **ExistingRG** 资源组中名为 ExistingRG-vnet 的现有虚拟网络着手。 子网命名为 default。 这些默认资源是在使用 Azure 门户预览创建标准虚拟机 (VM) 时创建的。 可以只创建虚拟网络和子网而不创建 VM，但是，将群集添加到现有虚拟网络的主要目的是提供与其他 VM 之间的网络连接。 创建 VM 可以很好地示范现有虚拟网络的典型用法。 如果 Service Fabric 群集仅使用不带公共 IP 地址的内部负载均衡器，则可以将 VM 及其公共 IP 用作安全的*转接盒*。

### <a name="static-public-ip-address"></a>静态公共 IP 地址

静态公共 IP 地址通常是一个专用资源，与其所分配的 VM 分开管理。 它在专用网络资源组中（而不是在 Service Fabric 群集资源组本身中）预配。 使用 Azure 门户预览或 PowerShell 在同一个 ExistingRG 资源组中创建名为 staticIP1 的静态公共 IP 地址：

    PS C:\Users\user> New-AzureRmPublicIpAddress -Name staticIP1 -ResourceGroupName ExistingRG -Location "China East" -AllocationMethod Static -DomainNameLabel sfnetworking

    Name                     : staticIP1
    ResourceGroupName        : ExistingRG
    Location                 : China East
    Id                       : /subscriptions/1237f4d2-3dce-1236-ad95-123f764e7123/resourceGroups/ExistingRG/providers/Microsoft.Network/publicIPAddresses/staticIP1
    Etag                     : W/"fc8b0c77-1f84-455d-9930-0404ebba1b64"
    ResourceGuid             : 77c26c06-c0ae-496c-9231-b1a114e08824
    ProvisioningState        : Succeeded
    Tags                     :
    PublicIpAllocationMethod : Static
    IpAddress                : 40.83.182.110
    PublicIpAddressVersion   : IPv4
    IdleTimeoutInMinutes     : 4
    IpConfiguration          : null
    DnsSettings              : {
                                 "DomainNameLabel": "sfnetworking",
                                 "Fqdn": "sfnetworking.chinaeast.cloudapp.chinacloudapi.cn"
                               }

### <a name="service-fabric-template"></a>Service Fabric 模板

本文中的示例使用 Service Fabric template.json。 在创建群集之前，可以使用标准门户向导下载该模板。 

<a id="existingvnet"></a>
## <a name="existing-virtual-network-or-subnet"></a>现有虚拟网络或子网

1. 将子网参数更改为现有子网的名称，然后添加两个新参数以引用现有的虚拟网络：

            "subnet0Name": {
                    "type": "string",
                    "defaultValue": "default"
                },
                "existingVNetRGName": {
                    "type": "string",
                    "defaultValue": "ExistingRG"
                },

                "existingVNetName": {
                    "type": "string",
                    "defaultValue": "ExistingRG-vnet"
                },
                /*
                "subnet0Name": {
                    "type": "string",
                    "defaultValue": "Subnet-0"
                },
                "subnet0Prefix": {
                    "type": "string",
                    "defaultValue": "10.0.0.0/24"
                },*/

2. 将 `vnetID` 变量更改为指向现有虚拟网络：

                /*old "vnetID": "[resourceId('Microsoft.Network/virtualNetworks',parameters('virtualNetworkName'))]",*/
                "vnetID": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', parameters('existingVNetRGName'), '/providers/Microsoft.Network/virtualNetworks/', parameters('existingVNetName'))]",

3. 从资源中删除 `Microsoft.Network/virtualNetworks`，使 Azure 不会创建新的虚拟网络：

        /*{
        "apiVersion": "[variables('vNetApiVersion')]",
        "type": "Microsoft.Network/virtualNetworks",
        "name": "[parameters('virtualNetworkName')]",
        "location": "[parameters('computeLocation')]",
        "properities": {
            "addressSpace": {
                "addressPrefixes": [
                    "[parameters('addressPrefix')]"
                ]
            },
            "subnets": [
                {
                    "name": "[parameters('subnet0Name')]",
                    "properties": {
                        "addressPrefix": "[parameters('subnet0Prefix')]"
                    }
                }
            ]
        },
        "tags": {
            "resourceType": "Service Fabric",
            "clusterName": "[parameters('clusterName')]"
        }
        },*/

4. 从 `Microsoft.Compute/virtualMachineScaleSets` 的 `dependsOn` 属性中注释掉虚拟网络，避免非得要创建新的虚拟网络：

        "apiVersion": "[variables('vmssApiVersion')]",
        "type": "Microsoft.Computer/virtualMachineScaleSets",
        "name": "[parameters('vmNodeType0Name')]",
        "location": "[parameters('computeLocation')]",
        "dependsOn": [
            /*"[concat('Microsoft.Network/virtualNetworks/', parameters('virtualNetworkName'))]",
            */
            "[Concat('Microsoft.Storage/storageAccounts/', variables('uniqueStringArray0')[0])]",


5. 部署模板：

        New-AzureRmResourceGroup -Name sfnetworkingexistingvnet -Location "China East"
        New-AzureRmResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkingexistingvnet -TemplateFile C:\SFSamples\Final\template\_existingvnet.json

    部署后，虚拟网络应包含新的规模集 VM。 虚拟机规模集节点类型应显示现有虚拟网络和子网。 还可以使用远程桌面协议 (RDP) 访问虚拟网络中已有的 VM，并对新规模集 VM 执行 ping 操作：

        C:>\Users\users>ping 10.0.0.5 -n 1
        C:>\Users\users>ping NOde1000000 -n 1

请参阅[并非特定于 Service Fabric 的另一个示例](https://github.com/gbowerman/azure-myriad/tree/master/existing-vnet)。


<a id="staticpublicip"></a>
## <a name="static-public-ip-address"></a>静态公共 IP 地址

1. 添加现有静态 IP 资源组名称、名称和完全限定的域名 (FQDN) 的参数：

        "existingStaticIPResourceGroup": {
                    "type": "string"
                },
                "existingStaticIPName": {
                    "type": "string"
                },
                "existingStaticIPDnsFQDN": {
                    "type": "string"
        }

2. 删除 `dnsName` 参数。 （静态 IP 已有名称。）

        /*
        "dnsName": {
            "type": "string"
        },
        */

3. 添加一个变量来引用现有的静态 IP 地址：

        "existingStaticIP": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', parameters('existingStaticIPResourceGroup'), '/providers/Microsoft.Network/publicIPAddresses/', parameters('existingStaticIPName'))]",

4. 从资源中删除 `Microsoft.Network/publicIPAddresses`，使 Azure 不会创建新的 IP 地址：

        /*
        {
            "apiVersion": "[variables('publicIPApiVersion')]",
            "type": "Microsoft.Network/publicIPAddresses",
            "name": "[concat(parameters('lbIPName'),)'-', '0')]",
            "location": "[parameters('computeLocation')]",
            "properties": {
                "dnsSettings": {
                    "domainNameLabel": "[parameters('dnsName')]"
                },
                "publicIPAllocationMethod": "Dynamic"        
            },
            "tags": {
                "resourceType": "Service Fabric",
                "clusterName": "[parameters('clusterName')]"
            }
        }, */

5. 从 `Microsoft.Network/loadBalancers` 的 `dependsOn` 属性中注释掉 IP 地址，避免非得要创建新的 IP 地址：

        "apiVersion": "[variables('lbIPApiVersion')]",
        "type": "Microsoft.Network/loadBalancers",
        "name": "[concat('LB', '-', parameters('clusterName'), '-', parameters('vmNodeType0Name'))]",
        "location": "[parameters('computeLocation')]",
        /*
        "dependsOn": [
            "[concat('Microsoft.Network/publicIPAddresses/', concat(parameters('lbIPName'), '-', '0'))]"
        ], */
        "properties": {

6. 在 `Microsoft.Network/loadBalancers` 资源中，将 `frontendIPConfigurations` 的 `publicIPAddress` 元素更改为引用现有的静态 IP 地址而不是新建的 IP 地址：

                    "frontendIPConfigurations": [
                            {
                                "name": "LoadBalancerIPConfig",
                                "properties": {
                                    "publicIPAddress": {
                                        /*"id": "[resourceId('Microsoft.Network/publicIPAddresses',concat(parameters('lbIPName'),'-','0'))]"*/
                                        "id": "[variables('existingStaticIP')]"
                                    }
                                }
                            }
                        ],

7. 在 `Microsoft.ServiceFabric/clusters` 资源中，将 `managementEndpoint` 更改为静态 IP 地址的 DNS FQDN。 如果使用安全群集，请确保将 *http://* 更改为 *https://*。 （请注意，此步骤仅适用于 Service Fabric 群集。 如果使用虚拟机规模集，请跳过此步骤。）

                        "fabricSettings": [],
                        /*"managementEndpoint": "[concat('http://',reference(concat(parameters('lbIPName'),'-','0')).dnsSettings.fqdn,':',parameters('nt0fabricHttpGatewayPort'))]",*/
                        "managementEndpoint": "[concat('http://',parameters('existingStaticIPDnsFQDN'),':',parameters('nt0fabricHttpGatewayPort'))]",

8. 部署模板：

        New-AzureRmResourceGroup -Name sfnetworkingstaticip -Location "China East"

        $staticip = Get-AzureRmPublicIpAddress -Name staticIP1 -ResourceGroupName ExistingRG

        $staticip

        New-AzureRmResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkingstaticip -TemplateFile C:\SFSamples\Final\template\_staticip.json -existingStaticIPResourceGroup $staticip.ResourceGroupName -existingStaticIPName $staticip.Name -existingStaticIPDnsFQDN $staticip.DnsSettings.Fqdn

部署后，可以看到负载均衡器已绑定到其他资源组中的公共静态 IP 地址。 Service Fabric 客户端连接终结点和 [Service Fabric Explorer](/documentation/articles/service-fabric-visualizing-your-cluster/) 终结点指向静态 IP 地址的 DNS FQDN。

<a id="internallb"></a>
## <a name="internal-only-load-balancer"></a>仅限内部的负载均衡器

本方案用仅限内部的负载均衡器替代默认 Service Fabric 模板中的外部负载均衡器。 有关 Azure 门户预览和 Service Fabric 资源提供程序的含义，请参阅前面的部分。

1. 删除 `dnsName` 参数。 （不需要此参数。）

        /*
        "dnsName": {
            "type": "string"
        },
        */

2. （可选）如果使用静态分配方法，可添加静态 IP 地址参数。 如果使用动态分配方法，则无需执行此步骤。

                "internalLBAddress": {
                    "type": "string",
                    "defaultValue": "10.0.0.250"
                }

3. 从资源中删除 `Microsoft.Network/publicIPAddresses`，使 Azure 不会创建新的 IP 地址：

        /*
        {
            "apiVersion": "[variables('publicIPApiVersion')]",
            "type": "Microsoft.Network/publicIPAddresses",
            "name": "[concat(parameters('lbIPName'),)'-', '0')]",
            "location": "[parameters('computeLocation')]",
            "properties": {
                "dnsSettings": {
                    "domainNameLabel": "[parameters('dnsName')]"
                },
                "publicIPAllocationMethod": "Dynamic"        
            },
            "tags": {
                "resourceType": "Service Fabric",
                "clusterName": "[parameters('clusterName')]"
            }
        }, */

4. 删除 `Microsoft.Network/loadBalancers` 的 IP 地址 `dependsOn` 属性，避免非得要创建新的 IP 地址。 添加虚拟网络 `dependsOn` 属性，因为负载均衡器现在依赖于虚拟网络中的子网：

                    "apiVersion": "[variables('lbApiVersion')]",
                    "type": "Microsoft.Network/loadBalancers",
                    "name": "[concat('LB','-', parameters('clusterName'),'-',parameters('vmNodeType0Name'))]",
                    "location": "[parameters('computeLocation')]",
                    "dependsOn": [
                        /*"[concat('Microsoft.Network/publicIPAddresses/',concat(parameters('lbIPName'),'-','0'))]"*/
                        "[concat('Microsoft.Network/virtualNetworks/',parameters('virtualNetworkName'))]"
                    ],

5. 将负载均衡器的 `frontendIPConfigurations` 设置从使用 `publicIPAddress` 更改为使用子网和 `privateIPAddress`。 `privateIPAddress` 使用预定义的静态内部 IP 地址。 若要使用动态 IP 地址，请删除 `privateIPAddress` 元素，然后将 `privateIPAllocationMethod` 更改为 **Dynamic**。

                    "frontendIPConfigurations": [
                            {
                                "name": "LoadBalancerIPConfig",
                                "properties": {
                                    /*
                                    "publicIPAddress": {
                                        "id": "[resourceId('Microsoft.Network/publicIPAddresses',concat(parameters('lbIPName'),'-','0'))]"
                                    } */
                                    "subnet" :{
                                        "id": "[variables('subnet0Ref')]"
                                    },
                                    "privateIPAddress": "[parameters('internalLBAddress')]",
                                    "privateIPAllocationMethod": "Static"
                                }
                            }
                        ],

6. 在 `Microsoft.ServiceFabric/clusters` 资源中，将 `managementEndpoint` 更改为指向内部负载均衡器地址。 如果使用安全群集，请确保将 *http://* 更改为 *https://*。 （请注意，此步骤仅适用于 Service Fabric 群集。 如果使用虚拟机规模集，请跳过此步骤。）

                        "fabricSettings": [],
                        /*"managementEndpoint": "[concat('http://',reference(concat(parameters('lbIPName'),'-','0')).dnsSettings.fqdn,':',parameters('nt0fabricHttpGatewayPort'))]",*/
                        "managementEndpoint": "[concat('http://',reference(variables('lbID0')).frontEndIPConfigurations[0].properties.privateIPAddress,':',parameters('nt0fabricHttpGatewayPort'))]",

7. 部署模板：

        New-AzureRmResourceGroup -Name sfnetworkinginternallb -Location "China East"

        New-AzureRmResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkinginternallb -TemplateFile C:\SFSamples\Final\template\_internalonlyLB.json

部署后，负载均衡器将使用专用静态 IP 地址 10.0.0.250。 如果同一虚拟网络中还有其他计算机，可以转到内部 [Service Fabric Explorer](/documentation/articles/service-fabric-visualizing-your-cluster/) 终结点。 可以看到，该终结点已连接到负载均衡器后面的某个节点。

<a id="internalexternallb"></a>
## <a name="internal-and-external-load-balancer"></a>内部和外部负载均衡器

本方案从现有的单节点类型外部负载均衡器着手，添加一个相同节点类型的内部负载均衡器。 附加到后端地址池的后端端口只能分配给单个负载均衡器。 选择哪个负载均衡器使用应用程序端口，哪个负载均衡器使用管理终结点（端口 19000 和 19080）。 如果将管理终结点放在内部负载均衡器上，请记住前文所述的 Service Fabric 资源提供程序限制。 本示例将管理终结点保留在外部负载均衡器上。 还需要添加一个端口号为 80 的应用程序端口，并将其放在内部负载均衡器上。

在双节点类型的群集中，一个节点类型位于外部负载均衡器上。 另一个节点类型位于内部负载均衡器上。 若要使用双节点类型的群集，请在门户创建的双节点类型模板（附带两个负载均衡器）中，将第二个负载均衡器切换为内部负载均衡器。 有关详细信息，请参阅[仅限内部的负载均衡器](#internallb)部分。

1. 添加静态内部负载均衡器 IP 地址参数。 （有关使用动态 IP 地址的说明，请参阅本文的前面部分。）

                "internalLBAddress": {
                    "type": "string",
                    "defaultValue": "10.0.0.250"
                }

2. 添加应用程序端口 80 参数。

3. 若要添加现有网络变量的内部版本，请复制并粘贴这些变量，并在名称中添加“-Int”：

        /* Add internal load balancer networking variables */
                "lbID0-Int": "[resourceId('Microsoft.Network/loadBalancers', concat('LB','-', parameters('clusterName'),'-',parameters('vmNodeType0Name'), '-Internal'))]",
                "lbIPConfig0-Int": "[concat(variables('lbID0-Int'),'/frontendIPConfigurations/LoadBalancerIPConfig')]",
                "lbPoolID0-Int": "[concat(variables('lbID0-Int'),'/backendAddressPools/LoadBalancerBEAddressPool')]",
                "lbProbeID0-Int": "[concat(variables('lbID0-Int'),'/probes/FabricGatewayProbe')]",
                "lbHttpProbeID0-Int": "[concat(variables('lbID0-Int'),'/probes/FabricHttpGatewayProbe')]",
                "lbNatPoolID0-Int": "[concat(variables('lbID0-Int'),'/inboundNatPools/LoadBalancerBEAddressNatPool')]",
                /* Internal load balancer networking variables end */

4. 如果从使用应用程序端口 80、由门户生成的模板开始，则默认门户模板会在外部负载均衡器上添加 AppPort1（端口 80）。 在此情况下，请将 AppPort1 从外部负载均衡器 `loadBalancingRules` 和探测中删除，以便将其添加到内部负载均衡器中：

        "loadBalancingRules": [
            {
                "name": "LBHttpRule",
                "properties":{
                    "backendAddressPool": {
                        "id": "[variables('lbPoolID0')]"
                    },
                    "backendPort": "[parameters('nt0fabricHttpGatewayPort')]",
                    "enableFloatingIP": "false",
                    "frontendIPConfiguration": {
                        "id": "[variables('lbIPConfig0')]"            
                    },
                    "frontendPort": "[parameters('nt0fabricHttpGatewayPort')]",
                    "idleTimeoutInMinutes": "5",
                    "probe": {
                        "id": "[variables('lbHttpProbeID0')]"
                    },
                    "protocol": "tcp"
                }
            } /* Remove AppPort1 from the external load balancer.
            {
                "name": "AppPortLBRule1",
                "properties": {
                    "backendAddressPool": {
                        "id": "[variables('lbPoolID0')]"
                    },
                    "backendPort": "[parameters('loadBalancedAppPort1')]",
                    "enableFloatingIP": "false",
                    "frontendIPConfiguration": {
                        "id": "[variables('lbIPConfig0')]"            
                    },
                    "frontendPort": "[parameters('loadBalancedAppPort1')]",
                    "idleTimeoutInMinutes": "5",
                    "probe": {
                        "id": "[concate(variables('lbID0'), '/probes/AppPortProbe1')]"
                    },
                    "protocol": "tcp"
                }
            }*/

        ],
        "probes": [
            {
                "name": "FabricGatewayProbe",
                "properties": {
                    "intervalInSeconds": 5,
                    "numberOfProbes": 2,
                    "port": "[parameters('nt0fabricTcpGatewayPort')]",
                    "protocol": "tcp"
                }
            },
            {
                "name": "FabricHttpGatewayProbe",
                "properties": {
                    "intervalInSeconds": 5,
                    "numberOfProbes": 2,
                    "port": "[parameters('nt0fabricHttpGatewayPort')]",
                    "protocol": "tcp"
                }
            } /* Remove AppPort1 from the external load balancer.
            {
                "name": "AppPortProbe1",
                "properties": {
                    "intervalInSeconds": 5,
                    "numberOfProbes": 2,
                    "port": "[parameters('loadBalancedAppPort1')]",
                    "protocol": "tcp"
                }
            } */

        ],
        "inboundNatPools": [

5. 添加第二个 `Microsoft.Network/loadBalancers` 资源。 该资源与[仅限内部的负载均衡器](#internallb)部分中创建的内部负载均衡器类似，不过它使用的是“-Int”负载均衡器变量，并且仅实现应用程序端口 80。 这样做还会删除 `inboundNatPools`，以便将 RDP 终结点保留在公共负载均衡器上。 如果要将 RDP 放在内部负载均衡器上，请将 `inboundNatPools` 从外部负载均衡器移到此内部负载均衡器：

                /* Add a second load balancer, configured with a static privateIPAddress and the "-Int" load balancer variables. */
                {
                    "apiVersion": "[variables('lbApiVersion')]",
                    "type": "Microsoft.Network/loadBalancers",
                    /* Add "-Internal" to the name. */
                    "name": "[concat('LB','-', parameters('clusterName'),'-',parameters('vmNodeType0Name'), '-Internal')]",
                    "location": "[parameters('computeLocation')]",
                    "dependsOn": [
                        /* Remove public IP dependsOn, add vnet dependsOn
                        "[concat('Microsoft.Network/publicIPAddresses/',concat(parameters('lbIPName'),'-','0'))]"
                        */
                        "[concat('Microsoft.Network/virtualNetworks/',parameters('virtualNetworkName'))]"
                    ],
                    "properties": {
                        "frontendIPConfigurations": [
                            {
                                "name": "LoadBalancerIPConfig",
                                "properties": {
                                    /* Switch from Public to Private IP address
                                    */
                                    "publicIPAddress": {
                                        "id": "[resourceId('Microsoft.Network/publicIPAddresses',concat(parameters('lbIPName'),'-','0'))]"
                                    }
                                    */
                                    "subnet" :{
                                        "id": "[variables('subnet0Ref')]"
                                    },
                                    "privateIPAddress": "[parameters('internalLBAddress')]",
                                    "privateIPAllocationMethod": "Static"
                                }
                            }
                        ],
                        "backendAddressPools": [
                            {
                                "name": "LoadBalancerBEAddressPool",
                                "properties": {}
                            }
                        ],
                        "loadBalancingRules": [
                            /* Add the AppPort rule. Be sure to reference the "-Int" versions of backendAddressPool, frontendIPConfiguration, and the probe variables. */
                            {
                                "name": "AppPortLBRule1",
                                "properties": {
                                    "backendAddressPool": {
                                        "id": "[variables('lbPoolID0-Int')]"
                                    },
                                    "backendPort": "[parameters('loadBalancedAppPort1')]",
                                    "enableFloatingIP": "false",
                                    "frontendIPConfiguration": {
                                        "id": "[variables('lbIPConfig0-Int')]"
                                    },
                                    "frontendPort": "[parameters('loadBalancedAppPort1')]",
                                    "idleTimeoutInMinutes": "5",
                                    "probe": {
                                        "id": "[concat(variables('lbID0-Int'),'/probes/AppPortProbe1')]"
                                    },
                                    "protocol": "tcp"
                                }
                            }
                        ],
                        "probes": [
                        /* Add the probe for the app port. */
                        {
                                "name": "AppPortProbe1",
                                "properties": {
                                    "intervalInSeconds": 5,
                                    "numberOfProbes": 2,
                                    "port": "[parameters('loadBalancedAppPort1')]",
                                    "protocol": "tcp"
                                }
                            }
                        ],
                        "inboundNatPools": [
                        ]
                    },
                    "tags": {
                        "resourceType": "Service Fabric",
                        "clusterName": "[parameters('clusterName')]"
                    }
                },

6. 在 `Microsoft.Compute/virtualMachineScaleSets` 资源的 `networkProfile` 中，添加内部后端地址池：

        "loadBalancerBackendAddressPools": [
                                                            {
                                                                "id": "[variables('lbPoolID0')]"
                                                            },
                                                            {
                                                                /* Add internal BE pool */
                                                                "id": "[variables('lbPoolID0-Int')]"
                                                            }
        ],

7. 部署模板：

        New-AzureRmResourceGroup -Name sfnetworkinginternalexternallb -Location "China East"

        New-AzureRmResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkinginternalexternallb -TemplateFile C:\SFSamples\Final\template\_internalexternalLB.json

部署后，可在资源组中看到两个负载均衡器。 如果浏览这两个负载均衡器，可以看到公共 IP 地址和分配给公共 IP 地址的管理终结点（端口 19000 和 19080）。 此外，还会看到静态内部 IP 地址和分配给内部负载均衡器的应用程序终结点（端口 80）。 这两个负载均衡器使用同一个虚拟机规模集后端池。

## <a name="next-steps"></a>后续步骤
[创建群集](/documentation/articles/service-fabric-cluster-creation-via-arm/)