<properties
   pageTitle="流量管理器的 Azure 资源管理器支持预览版 | Microsoft Azure"
   description="使用包含 Azure 资源管理器 (ARM) 预览版的流量管理器 PowerShell"
   services="traffic-manager"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
   editor="tysonn" />
<tags
	ms.service="traffic-manager"
	ms.date="11/12/2015"
	wacn.date=""/>



# Azure 流量管理器预览版对 Azure 资源管理器的支持
Azure 资源管理器 (ARM) 是针对 Azure 中的服务的新管理框架。现在，你可以使用基于 Azure 资源管理器的 API 和工具来管理 Azure 流量管理器配置文件。若要了解有关 Azure 资源组的详细信息，请参阅[使用资源组管理 Azure 资源](/documentation/articles/azure-preview-portal-using-resource-groups)。

>[AZURE.NOTE]流量管理器的 ARM 支持目前为预览版，包括 REST API、Azure PowerShell、Azure CLI 和 .NET SDK。



## 资源模型

Azure 流量管理器是使用名为流量管理器配置文件的一系列设置进行配置的。该配置文件包含 DNS 设置、流量路由设置、终结点监视设置，以及流量要路由到的服务终结点列表。

在 ARM 中，每个流量管理器配置文件由类型为“TrafficManagerProfiles”、受“Microsoft.Network”资源提供程序管理的 ARM 资源表示。在 REST API 级别，每个配置文件的 URI 如下：

	https://manage.windowsazure.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/trafficManagerProfiles/{profile-name}?api-version={api-version}

## 与 Azure 流量管理器服务管理 API 的比较

使用 ARM 来配置流量管理器配置文件可以访问与（非 ARM）服务管理 API 相同的流量管理器功能集，不过存在下面列出的预览版限制。

尽管功能保持相同，但一些术语已有所变化：

- “负载平衡方法”（确定流量管理器在响应特定的 DNS 请求时如何选择要将流量路由到的终结点）已重命名为“流量路由方法”。

- “循环”流量路由方法已重命名为“加权”。

- “故障转移”流量路由方法已重命名为“优先级”。

## 预览版限制
由于流量管理器对 Azure 资源管理器的支持是一项预览版服务，目前存在少量的限制：

- 使用现有（非 ARM）服务管理 API、工具和门户创建的流量管理器配置文件无法通过 ARM 使用，反之亦然。当前不支持将配置文件从非 ARM API 迁移到 ARM API。

- 	REST API 不支持修补流量管理器配置文件。若要更新配置文件属性，必须对该配置文件执行 GET，然后对修改后的配置文件执行 PUT。
- 	仅支持“外部”终结点。仍可通过这些终结点将流量管理器用于基于 Azure 的服务，采用这种做法时，将按内部终结点费率来计收这些终结点的费用。（使用外部终结点的唯一负面影响在于，当 Azure 服务被禁用或删除时，系统不会自动禁用或删除这些终结点，而你必须手动禁用或删除这些终结点）。
-	Azure 流量管理器目前无法在 Azure 管理门户中使用，而只能在管理门户上使用。

## 设置 Azure PowerShell

这些说明使用了需要通过以下步骤进行配置的 Microsoft Azure PowerShell。

对于非 PowerShell 用户，也可以通过其他界面执行相同的操作。

### 步骤 1
安装最新的 Azure PowerShell（可从 Azure 下载页获取）。
### 步骤 2
将 PowerShell 模式切换为使用 ARM cmdlet。“将 Windows PowerShell 与资源管理器一起使用”中提供了详细信息。

	PS C:\> Switch-AzureMode -Name AzureResourceManager
### 步骤 3
登录到你的 Azure 帐户。

	PS C:\> Add-AzureAccount

系统将提示你使用凭据进行身份验证。

### 步骤 4
选择要使用的 Azure 订阅。

	PS C:\> Select-AzureSubscription -SubscriptionName "MySubscription"

若要查看可用订阅的列表，请使用“Get-AzureSubscription”cmdlet。

### 步骤 5

 流量管理器服务由 Microsoft.Network 资源提供程序管理。需要将你的 Azure 订阅注册为使用此资源提供程序，然后你才可以通过 ARM 使用流量管理器。只需为每个订阅执行此操作一次。

	PS C:\> Register-AzureProvider –ProviderNamespace Microsoft.Network

### 步骤 6
创建资源组（如果要使用现有的资源组，请跳过此步骤）

	PS C:\> New-AzureResourceGroup -Name MyAzureResourceGroup -location "China North"

Azure 资源管理器要求所有资源组指定一个位置。此位置将用作该资源组中的资源的默认位置。但是，由于所有流量管理器配置文件所有资源都是全局性而不是区域性的，因此，所选的资源组位置不会影响 Azure 流量管理器。

## 创建流量管理器配置文件

若要创建流量管理器配置文件，请使用 New-AzureTrafficManagerProfile cmdlet：

	PS C:\> $profile = New-AzureTrafficManagerProfile –Name MyProfile -ResourceGroupName MyAzureResourceGroup -TrafficRoutingMethod Performance -RelativeDnsName contoso -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"

参数如下：

- Name：流量管理器配置文件资源的 ARM 资源名称。同一资源组中的配置文件必须具有唯一的名称。此名称不同于用于 DNS 查询的 DNS 名称。

-	ResourceGroupName：包含配置文件资源的 ARM 资源组的名称。

-	TrafficRoutingMethod：指定流量路由方法，用于确定在响应传入 DNS 查询时要返回的终结点。可能的值为“Performance”、“Weighted”或“Priority”。

-	RelativeDnsName：指定此流量管理器配置文件提供的相对 DNS 名称。将此值与 Azure 流量管理器使用的 DNS 域名相结合，可以构成配置文件的完全限定域名 (FQDN)。例如，值“contoso”将为流量管理器配置文件提供完全限定的名称“contoso.trafficmanager.cn”。

-	TTL：指定 DNS 生存时间 (TTL)，以秒为单位。此值用于通知本地 DNS 解析器和 DNS 客户端，要将此流量管理器配置文件提供的 DNS 响应缓存多久。

-	MonitorProtocol：指定用于监视终结点运行状况的协议。可能的值为“HTTP”和“HTTPS”。

-	MonitorPort：指定用于监视终结点运行状况的 TCP 端口。

-	MonitorPath：指定用于探测终结点运行状况的终结点域名的相对路径。

该 cmdlet 将在 Azure 流量管理器中创建流量管理器配置文件，并返回相应的配置文件对象。此时，该配置文件不包含任何终结点 - 有关如何将终结点添加到流量管理器配置文件的详细信息，请参阅[更新流量管理器配置文件](#update-a-traffic-manager-profile)。

## 获取流量管理器配置文件

若要检索现有的流量管理器配置文件对象，请使用 Get-AzureTrafficManagerProfle cmdlet：

	PS C:\> $profile = Get-AzureTrafficManagerProfile –Name MyProfile -ResourceGroupName MyAzureResourceGroup

此 cmdlet 将返回流量管理器配置文件对象。

## 更新流量管理器配置文件 [](#update-traffic-manager-profile)

例如，修改流量管理器配置文件以添加或删除终结点或修改配置文件设置需要遵循三个步骤：

1.	使用 Get-AzureTrafficManagerProfile 检索配置文件（或者使用 New-AzureTrafficManagerProfile 返回的配置文件）。

2.	通过添加终结点、删除终结点、更改终结点参数或更改配置文件参数来修改该配置文件。这些更改是脱机进行的 - 只会更改代表该配置文件的本地对象。

3.	使用 Set-AzureTrafficManagerProfile cmdlet 提交所做的更改。这会将 Azure 流量管理器中的现有配置文件替换为提供的配置文件。

可以使用以下示例对此做进一步的解释：

### 将终结点添加到配置文件

可以使用“Add-AzureTrafficManagerEndpointConfig”cmdlet 将终结点添加到流量管理器配置文件：

	PS C:\> $profile = Get-AzureTrafficManagerProfile –Name MyProfile -ResourceGroupName MyAzureResourceGroup
	PS C:\> Add-AzureTrafficManagerEndpointConfig –EndpointName site1 –TrafficManagerProfile $profile –Type ExternalEndpoints –Target site1.contoso.com –EndpointStatus Enabled –Weight 10 –Priority 1 –EndpointLocation “China North”
	PS C:\> Set-AzureTrafficManagerProfile –TrafficManagerProfile $profile

Add-AzureTrafficManagerEndpointConfig 的参数如下：

- EndpointName：终结点的名称。同一配置文件中的终结点必须具有不同的名称。此名称用于在执行服务管理操作期间引用该终结点，它不是该终结点的 DNS 名称。

-	TrafficManagerProfile：要将终结点添加到的流量管理器配置文件对象。

-	Type：流量管理器终结点的类型。目前，使用 ARM API 时仅支持“ExternalEndpoint”类型（请参阅[预览版限制](#preview-limitations)）。

-	Target：终结点的完全限定 DNS 名称。流量管理器会在 DNS 响应中返回此值，以将流量定向到此终结点。

-	EndpointStatus：指定终结点的状态。如果终结点已启用，则会探测该终结点的运行状况，并将其包含在流量路由方法中。可能的值为“Enabled”或“Disabled”。

-	Weight：指定分配给终结点的权重。仅当流量管理器配置文件配置为使用“加权”流量路由方法时，才使用此选项。可能的值为 1 到 1000。

-	Priority：指定使用“优先级”流量路由方法时此终结点的优先级。优先级必须在 1...1000 的范围内。值越小，表示优先级越高。

-	EndpointLocation：指定用于“性能”流量路由方法的外部终结点的位置。有关可能位置的列表，请参阅 Get-AzureLocation。

终结点状态、权重和优先级是可选参数。如果省略，则 PowerShell 不会传递这些参数，并应用服务器端默认值。

### 从配置文件中删除终结点

若要从配置文件中删除终结点，请使用“Remove-AzureTrafficmanagerEndpointConfig”，并指定要删除的终结点的名称：

	PS C:\> $profile = Get-AzureTrafficManagerProfile –Name MyProfile -ResourceGroupName MyAzureResourceGroup
	PS C:\> Remove-AzureTrafficManagerEndpointConfig –EndpointName site1 –TrafficManagerProfile $profile
	PS C:\> Set-AzureTrafficManagerProfile –TrafficManagerProfile $profile

用于添加或删除终结点的操作序列也可以是“piped”，这会通过管道而不是作为参数传递配置文件对象。例如：

	PS C:\> Get-AzureTrafficManagerProfile –Name MyProfile -ResourceGroupName MyAzureResourceGroup | Remove-AzureTrafficManagerEndpointConfig –EndpointName site1 | Set-AzureTrafficManagerProfile

### 更改配置文件或终结点设置

可以脱机更改配置文件和终结点参数，并使用 Set-AzureTrafficManagerProfile 提交所做的更改。唯一的例外情况是，在创建配置文件后，无法更改配置文件 RelativeDnsName（若要更改此值，请删除再重新创建该配置文件）。例如，若要更改配置文件 TTL 和第一个终结点的状态，请执行以下操作：

	PS C:\> $profile = Get-AzureTrafficManagerProfile –Name MyProfile -ResourceGroupName MyAzureResourceGroup
	PS C:\> $profile.Ttl = 300
	PS C:\> $profile.Endpoints[0].EndpointStatus = "Disabled"
	PS C:\> Set-AzureTrafficManagerProfile –TrafficManagerProfile $profile

### 删除流量管理器配置文件
若要删除流量管理器配置文件，请使用 Remove-AzureTrafficManagerProfile cmdlet，并指定配置文件名称和资源组名称：

	PS C:\> Remove-AzureTrafficManagerProfile –Name MyProfile -ResourceGroupName MyAzureResourceGroup [-Force]

此 cmdlet 将提示你确认。可以使用可选的“-Force”开关来隐藏此提示。也可以使用配置文件对象指定要删除的配置文件：

	PS C:\> $profile = Get-AzureTrafficManagerProfile –Name MyProfile -ResourceGroupName MyAzureResourceGroup
	PS C:\> Remove-AzureTrafficManagerProfile –TrafficManagerProfile $profile [-Force]

还可以通过管道执行此序列：

	PS C:\> Get-AzureTrafficManagerProfile –Name MyProfile -ResourceGroupName MyAzureResourceGroup | Remove-AzureTrafficManagerProfile [-Force]


## 后续步骤

[流量管理器监视](/documentation/articles/traffic-manager-monitoring)

[流量管理器性能注意事项](/documentation/articles/traffic-manager-performance-considerations)
 

<!---HONumber=Mooncake_1221_2015-->