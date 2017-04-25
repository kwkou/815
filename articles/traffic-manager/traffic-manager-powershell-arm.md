<properties
    pageTitle="使用 PowerShell 管理 Azure 中的流量管理器 | Azure"
    description="将适用于流量管理器的 PowerShell 与 Azure Resource Manager 配合使用"
    services="traffic-manager"
    documentationcenter="na"
    author="kumudd"
    manager="timlt"
    translationtype="Human Translation" />
<tags
    ms.assetid="bc247448-1d2e-4104-ac03-42b59ebde065"
    ms.service="traffic-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/16/2017"
    wacn.date="04/17/2017"
    ms.author="kumud"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="909fad5bfcc30bef4dffb59d4c5f1cdfd40d2a24"
    ms.lasthandoff="04/06/2017" />

# <a name="using-powershell-to-manage-traffic-manager"></a>使用 PowerShell 管理流量管理器

Azure Resource Manager 是 Azure 中的首选服务管理接口。 可以使用基于 Azure Resource Manager 的 API 和工具来管理 Azure 流量管理器配置文件。

## <a name="resource-model"></a>资源模型

Azure 流量管理器是使用名为流量管理器配置文件的一系列设置进行配置的。 此配置文件包含 DNS 设置、流量路由设置、终结点监视设置，以及流量要路由到的服务终结点列表。

每个流量管理器配置文件以一个“TrafficManagerProfiles”类型的资源表示。 在 REST API 级别，每个配置文件的 URI 如下：

    https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/trafficManagerProfiles/{profile-name}?api-version={api-version}

## <a name="setting-up-azure-powershell"></a>设置 Azure PowerShell

本部分中的说明使用 Azure PowerShell。 以下文章介绍了如何安装和配置 Azure PowerShell。

* [如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs)

本文中的示例假设已有一个资源组。 可以使用以下命令创建资源组：

    New-AzureRmResourceGroup -Name MyRG -Location "China North"

> [AZURE.NOTE]
> Azure Resource Manager 要求所有资源组都有一个位置。 此位置将用作该资源组中创建的资源的默认位置。 但是，由于流量管理器配置文件资源是全局性而不是区域性的，因此，所选的资源组位置不会影响 Azure 流量管理器。

## <a name="create-a-traffic-manager-profile"></a>创建流量管理器配置文件

若要创建流量管理器配置文件，请使用 `New-AzureRmTrafficManagerProfile` cmdlet：

    $profile = New-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName contoso -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"

下表描述了参数：

| 参数 | 说明 |
| --- | --- |
| 名称 |流量管理器配置文件资源的资源名称。 同一资源组中的配置文件必须具有唯一的名称。 此名称不同于用于 DNS 查询的 DNS 名称。 |
| ResourceGroupName |包含配置文件资源的资源组的名称 |
| TrafficRoutingMethod |指定流量路由方法，用于确定在响应 DNS 查询时要返回的终结点。 可能的值为“Performance”、“Weighted”或“Priority”。 |
| RelativeDnsName |指定此流量管理器配置文件提供的 DNS 名称的主机名部分。 将此值与 Azure 流量管理器使用的 DNS 域名相结合，可以构成配置文件的完全限定域名 (FQDN)。 例如，设置“contoso”值可以构成“contoso.trafficmanager.cn”。 |
| TTL |指定 DNS 生存时间 (TTL)，以秒为单位。 此 TTL 通知本地 DNS 解析器和 DNS 客户端缓存此流量管理器配置文件的 DNS 响应的时长。 |
| MonitorProtocol |指定用于监视终结点运行状况的协议。 可能的值为“HTTP”和“HTTPS”。 |
| MonitorPort |指定用于监视终结点运行状况的 TCP 端口。 |
| MonitorPath |指定用于探测终结点运行状况的终结点域名的相对路径。 |

该 cmdlet 在 Azure 流量管理器中创建流量管理器配置文件，将相应的配置文件对象返回到 PowerShell。 此时，配置文件不包含任何终结点。 有关将终结点添加到流量管理器配置文件的详细信息，请参阅 [Adding Traffic Manager Endpoints](#adding-traffic-manager-endpoints)（添加流量管理器终结点）。

## <a name="get-a-traffic-manager-profile"></a>获取流量管理器配置文件

若要检索现有的流量管理器配置文件对象，请使用 `Get-AzureRmTrafficManagerProfle` cmdlet：

    $profile = Get-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG

此 cmdlet 将返回流量管理器配置文件对象。

## <a name="update-a-traffic-manager-profile"></a>更新流量管理器配置文件

修改流量管理器配置文件遵循 3 步流程：

1. 使用 `Get-AzureRmTrafficManagerProfile` 检索配置文件，或使用 `New-AzureRmTrafficManagerProfile` 返回的配置文件。
2. 修改配置文件。 可以添加和删除终结点，或者更改终结点或配置文件参数。 这些更改属于脱机操作。 你只是更改了内存中代表配置文件的本地对象。
3. 使用 `Set-AzureRmTrafficManagerProfile` cmdlet 提交更改。

所有的配置文件属性都可更改，配置文件的 RelativeDnsName 除外。 若要更改 RelativeDnsName，必须删除配置文件，然后使用新名称新建一个配置文件。

以下示例演示如何更改配置文件的 TTL：

    $profile = Get-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG
    $profile.Ttl = 300
    Set-AzureRmTrafficManagerProfile -TrafficManagerProfile $profile

有三种类型的流量管理器终结点：

1. **Azure 终结点** 是在 Azure 中托管的服务
2. **外部终结点** 是在 Azure 外部托管的服务
3. **嵌套式终结点** 用于构造流量管理器配置文件的嵌套式层次结构。 嵌套式终结点可以针对复杂的应用程序启用高级流量路由配置。

在所有三种情况下，都可以通过两种方式添加终结点：

1. 使用上面所述的三步骤过程。 此方法的优点是可以通过一次更新完成多项终结点更改。
2. 使用 New-AzureRmTrafficManagerEndpoint cmdlet。 此 cmdlet 通过一个操作将终结点添加到现有的流量管理器配置文件。

## <a name="adding-traffic-manager-endpoints"></a> 添加 Azure 终结点

Azure 终结点引用 Azure 中托管的服务。 支持 2 种类型的 Azure 终结点：

1. Azure Web 应用
2. Azure PublicIpAddress 资源（可以附加到负载均衡器或虚拟机 NIC）。 必须为 PublicIpAddress 分配一个 DNS 名称，然后才能在流量管理器中使用。

在每种情况下：

* 服务是使用 `Add-AzureRmTrafficManagerEndpointConfig` 或 `New-AzureRmTrafficManagerEndpoint` 的“targetResourceId”参数指定。
* “Target”和“EndpointLocation”由 TargetResourceId 表明。
* 指定“Weight”是可选操作。 仅当配置文件配置为使用“加权”流量路由方法时，才使用权重。 否则会忽略该参数。 如果指定权重，其值必须是介于 1 和 1000 之间的数字。 默认值为“1”。
* 指定“Priority”是可选操作。 仅当配置文件配置为使用“优先级”流量路由方法时，才使用优先级。 否则会忽略该参数。 有效值为从 1 到 1000，值越小，优先级越高。 如果为一个终结点指定了该值，则必须为所有终结点指定该值。 如果省略，则会按排列终结点的顺序从“1”开始应用默认值。

### <a name="example-1-adding-web-app-endpoints-using-add-azurermtrafficmanagerendpointconfig"></a>示例 1：使用 `Add-AzureRmTrafficManagerEndpointConfig` 添加 Web 应用终结点

在此示例中，我们创建了一个流量管理器配置文件，并使用 `Add-AzureRmTrafficManagerEndpointConfig` cmdlet 添加了两个 Web 应用终结点。

    $profile = New-AzureRmTrafficManagerProfile -Name myprofile -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName myapp -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
    $webapp1 = Get-AzureRMWebApp -Name webapp1
    Add-AzureRmTrafficManagerEndpointConfig -EndpointName webapp1ep -TrafficManagerProfile $profile -Type AzureEndpoints -TargetResourceId $webapp1.Id -EndpointStatus Enabled
    $webapp2 = Get-AzureRMWebApp -Name webapp2
    Add-AzureRmTrafficManagerEndpointConfig -EndpointName webapp2ep -TrafficManagerProfile $profile -Type AzureEndpoints -TargetResourceId $webapp2.Id -EndpointStatus Enabled
    Set-AzureRmTrafficManagerProfile -TrafficManagerProfile $profile

### <a name="example-2-adding-a-publicipaddress-endpoint-using-new-azurermtrafficmanagerendpoint"></a>示例 2：使用 `New-AzureRmTrafficManagerEndpoint` 添加 publicIpAddress 终结点

在此示例中，公共 IP 地址资源添加到了流量管理器配置文件中。 公共 IP 地址必须配置了 DNS 名称，并且可以绑定到 VM 的 NIC 或者绑定到负载均衡器。

    $ip = Get-AzureRmPublicIpAddress -Name MyPublicIP -ResourceGroupName MyRG
    New-AzureRmTrafficManagerEndpoint -Name MyIpEndpoint -ProfileName MyProfile -ResourceGroupName MyRG -Type AzureEndpoints -TargetResourceId $ip.Id -EndpointStatus Enabled

## <a name="adding-external-endpoints"></a>添加外部终结点

流量管理器使用外部终结点将流量定向到在 Azure 外部承载的服务。 与 Azure 终结点一样，可以使用 `Add-AzureRmTrafficManagerEndpointConfig`（后跟 `Set-AzureRmTrafficManagerProfile`）或 `New-AzureRMTrafficManagerEndpoint` 添加外部终结点。

指定外部终结点时：

* 必须使用“Target”参数指定终结点域名
* 如果使用“性能”流量路由方法，则需要“EndpointLocation”。 否则，该参数是可选的。 该值必须是有效的 Azure 区域名称。
* “加权”和“优先级”是可选项。

### <a name="example-1-adding-external-endpoints-using-add-azurermtrafficmanagerendpointconfig-and-set-azurermtrafficmanagerprofile"></a>示例 1：使用 `Add-AzureRmTrafficManagerEndpointConfig` 和 `Set-AzureRmTrafficManagerProfile` 添加外部终结点

在此示例中，我们创建了一个流量管理器配置文件，添加了两个外部终结点，并提交了所做的更改。

    $profile = New-AzureRmTrafficManagerProfile -Name myprofile -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName myapp -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
    Add-AzureRmTrafficManagerEndpointConfig -EndpointName eu-endpoint -TrafficManagerProfile $profile -Type ExternalEndpoints -Target app-eu.contoso.com -EndpointStatus Enabled
    Add-AzureRmTrafficManagerEndpointConfig -EndpointName us-endpoint -TrafficManagerProfile $profile -Type ExternalEndpoints -Target app-us.contoso.com -EndpointStatus Enabled
    Set-AzureRmTrafficManagerProfile -TrafficManagerProfile $profile

### <a name="example-2-adding-external-endpoints-using-new-azurermtrafficmanagerendpoint"></a>示例 2：使用 `New-AzureRmTrafficManagerEndpoint` 添加外部终结点

在此示例中，我们向已有配置文件添加了一个外部终结点。 该配置文件是使用配置文件名称和资源组名称指定的。

    New-AzureRmTrafficManagerEndpoint -Name eu-endpoint -ProfileName MyProfile -ResourceGroupName MyRG -Type ExternalEndpoints -Target app-eu.contoso.com -EndpointStatus Enabled

## <a name="adding-nested-endpoints"></a>添加“嵌套式”终结点

每个流量管理器配置文件都会指定一个流量路由方法。 但在某些情况下，所需的流量路由方法比单个流量管理器配置文件所提供的方法更复杂。 可以嵌套流量管理器配置文件，将多个流量路由方法的优势结合在一起。 使用嵌套式配置文件可以重写默认的流量管理器行为，支持更大、更复杂的应用程序部署。 有关更多详细示例，请参阅[嵌套式流量管理器配置文件](/documentation/articles/traffic-manager-nested-profiles/)。

在父配置文件中使用特定的终结点类型“NestedEndpoints”配置嵌套式终结点。 指定嵌套式终结点时：

* 必须使用“targetResourceId”参数指定终结点
* 如果使用“性能”流量路由方法，则需要“EndpointLocation”。 否则，该参数是可选的。 该值必须是有效的 Azure 区域名称。
* 与指定 Azure 终结点时一样，“Weight”和“Priority”是可选的。
* “MinChildEndpoints”参数是可选的。 默认值为“1”。 如果可用终结点数低于此阈值，则父配置文件会将此子配置文件视为“已降级”，并将流量转移到父配置文件中的其他终结点。

### <a name="example-1-adding-nested-endpoints-using-add-azurermtrafficmanagerendpointconfig-and-set-azurermtrafficmanagerprofile"></a>示例 1：使用 `Add-AzureRmTrafficManagerEndpointConfig` 和 `Set-AzureRmTrafficManagerProfile` 添加嵌套终结点

在此示例中，我们将创建新的流量管理器子配置文件和父配置文件，将子配置文件添加为父配置文件中的嵌套式终结点，并提交所做的更改。

    $child = New-AzureRmTrafficManagerProfile -Name child -ResourceGroupName MyRG -TrafficRoutingMethod Priority -RelativeDnsName child -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
    $parent = New-AzureRmTrafficManagerProfile -Name parent -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName parent -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
    Add-AzureRmTrafficManagerEndpointConfig -EndpointName child-endpoint -TrafficManagerProfile $parent -Type NestedEndpoints -TargetResourceId $child.Id -EndpointStatus Enabled -EndpointLocation "China North" -MinChildEndpoints 2
    Set-AzureRmTrafficManagerProfile -TrafficManagerProfile $profile

在本示例中为方便起见，我们未将任何其他终结点添加到子或父配置文件。

### <a name="example-2-adding-nested-endpoints-using-new-azurermtrafficmanagerendpoint"></a>示例 2：使用 `New-AzureRmTrafficManagerEndpoint` 添加嵌套终结点

在此示例中，我们将现有子配置文件作为嵌套式终结点添加到了现有父配置文件。 该配置文件是使用配置文件名称和资源组名称指定的。

    $child = Get-AzureRmTrafficManagerEndpoint -Name child -ResourceGroupName MyRG
    New-AzureRmTrafficManagerEndpoint -Name child-endpoint -ProfileName parent -ResourceGroupName MyRG -Type NestedEndpoints -TargetResourceId $child.Id -EndpointStatus Enabled -EndpointLocation "China North" -MinChildEndpoints 2

## <a name="update-a-traffic-manager-endpoint"></a>更新流量管理器终结点

有两种方法可以更新现有的流量管理器终结点：

1. 使用 `Get-AzureRmTrafficManagerProfile` 获取流量管理器配置文件，更新配置文件中的终结点属性，然后使用 `Set-AzureRmTrafficManagerProfile` 提交更改。 此方法的优势在于能够通过单个操作更新多个终结点。
2. 使用 `Get-AzureRmTrafficManagerEndpoint` 获取流量管理器终结点，更新终结点属性，然后使用 `Set-AzureRmTrafficManagerEndpoint` 提交更改。 此方法更简单，因为不需要在配置文件的终结点数组中进行索引操作。

### <a name="example-1-updating-endpoints-using-get-azurermtrafficmanagerprofile-and-set-azurermtrafficmanagerprofile"></a>示例 1：使用 `Get-AzureRmTrafficManagerProfile` 和 `Set-AzureRmTrafficManagerProfile` 更新终结点

在此示例中，我们将修改现有配置文件中两个终结点上的优先级。

    $profile = Get-AzureRmTrafficManagerProfile -Name myprofile -ResourceGroupName MyRG
    $profile.Endpoints[0].Priority = 2
    $profile.Endpoints[1].Priority = 1
    Set-AzureRmTrafficManagerProfile -TrafficManagerProfile $profile

### <a name="example-2-updating-an-endpoint-using-get-azurermtrafficmanagerendpoint-and-set-azurermtrafficmanagerendpoint"></a>示例 2：使用 `Get-AzureRmTrafficManagerEndpoint` 和 `Set-AzureRmTrafficManagerEndpoint` 更新终结点

在此示例中，我们将修改现有配置文件中单个终结点的权重。

    $endpoint = Get-AzureRmTrafficManagerEndpoint -Name myendpoint -ProfileName myprofile -ResourceGroupName MyRG -Type ExternalEndpoints
    $endpoint.Weight = 20
    Set-AzureRmTrafficManagerEndpoint -TrafficManagerEndpoint $endpoint

## <a name="enabling-and-disabling-endpoints-and-profiles"></a>启用和禁用终结点和配置文件

可以通过流量管理器启用和禁用各个终结点，以及启用和禁用整个配置文件。
这些更改可以通过获取/更新/设置终结点或配置文件资源来完成。 为了简化这些常用操作，也可通过专用 cmdlet 来完成它们。

### <a name="example-1-enabling-and-disabling-a-traffic-manager-profile"></a>示例 1：启用和禁用流量管理器配置文件

若要启用流量管理器配置文件，请使用 `Enable-AzureRmTrafficManagerProfile`。 可以使用配置文件对象指定该配置文件。 可以通过管道或使用“-TrafficManagerProfile”参数传递配置文件对象。 本示例按配置文件和资源组名称指定配置文件。

    Enable-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyResourceGroup

若要禁用流量管理器配置文件，请执行以下操作：

    Disable-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyResourceGroup

Disable-AzureRmTrafficManagerProfile cmdlet 将提示确认。 可以使用使用“-Force”参数消除此提示。

### <a name="example-2-enabling-and-disabling-a-traffic-manager-endpoint"></a>示例 2：启用和禁用流量管理器终结点

若要启用流量管理器终结点，请使用 `Enable-AzureRmTrafficManagerEndpoint`。 有两种方法可以指定终结点

1. 使用通过管道传递的 TrafficManagerEndpoint 对象，或使用“-TrafficManagerEndpoint”参数
2. 使用终结点名称、终结点类型、配置文件名称和资源组名称：

        Enable-AzureRmTrafficManagerEndpoint -Name MyEndpoint -Type AzureEndpoints -ProfileName MyProfile -ResourceGroupName MyRG

    同样，若要禁用流量管理器终结点，请执行以下操作：

        Disable-AzureRmTrafficManagerEndpoint -Name MyEndpoint -Type AzureEndpoints -ProfileName MyProfile -ResourceGroupName MyRG -Force

    与 `Disable-AzureRmTrafficManagerProfile` 一样，`Disable-AzureRmTrafficManagerEndpoint` cmdlet 也会提示你进行确认。 可以使用“-Force”参数取消该提示。

## <a name="delete-a-traffic-manager-endpoint"></a>删除流量管理器终结点

若要单独删除终结点，请使用 `Remove-AzureRmTrafficManagerEndpoint` cmdlet：

    Remove-AzureRmTrafficManagerEndpoint -Name MyEndpoint -Type AzureEndpoints -ProfileName MyProfile -ResourceGroupName MyRG

此 cmdlet 提示你进行确认。 可以使用使用“-Force”参数消除此提示。

## <a name="delete-a-traffic-manager-profile"></a>删除流量管理器配置文件

若要删除流量管理器配置文件，请使用 `Remove-AzureRmTrafficManagerProfile`，同时指定配置文件和资源组名称：

    Remove-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG [-Force]

此 cmdlet 提示你进行确认。 可以使用使用“-Force”参数消除此提示。

也可以使用配置文件对象指定要删除的配置文件：

    $profile = Get-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG
    Remove-AzureRmTrafficManagerProfile -TrafficManagerProfile $profile [-Force]

还可以通过管道执行此序列：

    Get-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG | Remove-AzureRmTrafficManagerProfile [-Force]

## <a name="next-steps"></a>后续步骤

[流量管理器监视](/documentation/articles/traffic-manager-monitoring/)

[流量管理器性能注意事项](/documentation/articles/traffic-manager-performance-considerations/)
<!--Update_Description: wording update-->