<properties
   pageTitle="流量管理器的 Azure 资源管理器支持预览版 | Windows Azure"
   description="使用包含 Azure 资源管理器 (ARM) 预览版的流量管理器 PowerShell"
   services="traffic-manager"
   documentationCenter="na"
   authors="joaoma"
   manager="carmonm"
   editor="tysonn" />
<tags
	ms.service="traffic-manager"
	ms.date="11/19/2015"
	wacn.date="01/29/2016"/>



# Azure 流量管理器预览版对 Azure 资源管理器的支持
Azure 资源管理器 (ARM) 是针对 Azure 中的服务的新管理框架。现在，你可以使用基于 Azure 资源管理器的 API 和工具来管理 Azure 流量管理器配置文件。若要了解有关 Azure 资源组的详细信息，请参阅[使用资源组管理 Azure 资源](/documentation/articles/azure-preview-portal-using-resource-groups)。

>[AZURE.NOTE]流量管理器的 ARM 支持目前为预览版，包括 REST API、Azure PowerShell、Azure CLI 和 .NET SDK。



## 资源模型

Azure 流量管理器是使用名为流量管理器配置文件的一系列设置进行配置的。该配置文件包含 DNS 设置、流量路由设置、终结点监视设置，以及流量要路由到的服务终结点列表。

在 ARM 中，每个流量管理器配置文件由类型为“TrafficManagerProfiles”、受“Microsoft.Network”资源提供程序管理的 ARM 资源表示。在 REST API 级别，每个配置文件的 URI 如下：

	https://manage.windowsazure.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/trafficManagerProfiles/{profile-name}?api-version={api-version}

## 与 Azure 流量管理器服务管理 API 的比较

使用 ARM 来配置流量管理器配置文件可以访问与 Azure 服务管理 (ASM) API 相同的流量管理器功能集，不过存在下面列出的预览版限制。

尽管功能保持相同，但一些术语已有所变化：

- “负载平衡方法”（确定流量管理器在响应特定的 DNS 请求时如何选择要将流量路由到的终结点）已重命名为“流量路由方法”。

- “循环”流量路由方法已重命名为“加权”。

- “故障转移”流量路由方法已重命名为“优先级”。

## 预览版限制
由于流量管理器对 Azure 资源管理器的支持是一项预览版服务，目前存在少量的限制：

- 使用现有（非 ARM）Azure 服务管理 (ASM) API、工具和“经典”门户创建的流量管理器配置文件无法通过 ARM 使用，反之亦然。目前不支持将配置文件从 ASM 迁移到 ARM API，除非是通过删除该配置文件然后又重新创建的方式。

- ARM API 目前不支持“嵌套式”流量管理器终结点。

- Azure 流量管理器目前无法在 Azure“预览”门户中使用，而只能在“经典”门户中使用。

## 设置 Azure PowerShell

这些说明使用了需要通过以下步骤进行配置的 Windows Azure PowerShell。

非 PowerShell 用户或非 Windows 用户可以通过 Azure CLI 执行类似操作。

### 步骤 1
安装最新的 Azure PowerShell（可从 Azure 下载页获取）。

### 步骤 2
登录到你的 Azure 帐户。

	PS C:\> Login-AzureRmAccopunt

系统将提示你使用凭据进行身份验证。

### 步骤 3
选择要使用的 Azure 订阅。

	PS C:\> Select-AzureRmContext -SubscriptionName "MySubscription"

若要查看可用订阅的列表，请使用“Get-AzureRmSubscription”cmdlet。

### 步骤 4

流量管理器服务由 Microsoft.Network 资源提供程序管理。需要将你的 Azure 订阅注册为使用此资源提供程序，然后你才可以通过 ARM 使用流量管理器。只需为每个订阅执行此操作一次。

	PS C:\> Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Network

### 步骤 5
创建资源组（如果要使用现有的资源组，请跳过此步骤）

	PS C:\> New-AzureRmResourceGroup -Name MyAzureResourceGroup -Location "China North"

Azure 资源管理器要求所有资源组指定一个位置。此位置将用作该资源组中的资源的默认位置。但是，由于所有流量管理器配置文件所有资源都是全局性而不是区域性的，因此，所选的资源组位置不会影响 Azure 流量管理器。

## 创建流量管理器配置文件

若要创建流量管理器配置文件，请使用 New-AzureRmTrafficManagerProfile cmdlet：

	PS C:\> $profile = New-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyAzureResourceGroup -TrafficRoutingMethod Performance -RelativeDnsName contoso -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"

参数如下：

- Name：流量管理器配置文件资源的 ARM 资源名称。同一资源组中的配置文件必须具有唯一的名称。此名称不同于用于 DNS 查询的 DNS 名称。

- ResourceGroupName：包含配置文件资源的 ARM 资源组的名称。

- TrafficRoutingMethod：指定流量路由方法，用于确定在响应传入 DNS 查询时要返回的终结点。可能的值为“Performance”、“Weighted”或“Priority”。

- RelativeDnsName：指定此流量管理器配置文件提供的相对 DNS 名称。将此值与 Azure 流量管理器使用的 DNS 域名相结合，可以构成配置文件的完全限定域名 (FQDN)。例如，值“contoso”将为流量管理器配置文件提供完全限定的名称“contoso.trafficmanager.cn”。

- TTL：指定 DNS 生存时间 (TTL)，以秒为单位。此值用于通知本地 DNS 解析器和 DNS 客户端，要将此流量管理器配置文件提供的 DNS 响应缓存多久。

- MonitorProtocol：指定用于监视终结点运行状况的协议。可能的值为“HTTP”和“HTTPS”。

- MonitorPort：指定用于监视终结点运行状况的 TCP 端口。

- MonitorPath：指定用于探测终结点运行状况的终结点域名的相对路径。

该 cmdlet 将在 Azure 流量管理器中创建流量管理器配置文件，并返回相应的配置文件对象。此时，该配置文件不包含任何终结点 - 有关如何将终结点添加到流量管理器配置文件的详细信息，请参阅[添加流量管理器终结点](#adding-traffic-manager-endpoints)。

## 获取流量管理器配置文件

若要检索现有的流量管理器配置文件对象，请使用 Get-AzureRmTrafficManagerProfle cmdlet：

	PS C:\> $profile = Get-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyAzureResourceGroup

此 cmdlet 将返回流量管理器配置文件对象。

##<a name="update-traffic-manager-profile"></a> 更新流量管理器配置文件

例如，修改流量管理器配置文件以添加或删除终结点或修改配置文件设置需要遵循三个步骤：

1.	使用 Get-AzureRmTrafficManagerProfile 检索配置文件（或者使用 New-AzureRmTrafficManagerProfile 返回的配置文件）。

2.	通过添加终结点、删除终结点、更改终结点参数或更改配置文件参数来修改该配置文件。这些更改是脱机进行的操作，只会更改代表该配置文件的本地对象。

3.	使用 Set-AzureRmTrafficManagerProfile cmdlet 提交所做的更改。这会将 Azure 流量管理器中的现有配置文件替换为提供的配置文件。

所有配置文件属性均可更改，例外情况是，在创建配置文件后，无法更改配置文件 RelativeDnsName（若要更改此值，请删除该配置文件，然后再重新创建）。

例如，若要更改配置文件 TTL，请执行以下操作：

	PS C:\> $profile = Get-AzureTrafficManagerProfile -Name MyProfile -ResourceGroupName MyAzureResourceGroup
	PS C:\> $profile.Ttl = 300
	PS C:\> Set-AzureTrafficManagerProfile -TrafficManagerProfile $profile

##<a name="adding-traffic-manager-endpoints"></a> 添加流量管理器终结点
有三种类型的流量管理器终结点：
1.Azure 终结点：表示在 Azure 中托管的服务。
2.外部终结点：表示在 Azure 外部托管的服务。
3.嵌套式终结点：用于构造流量管理器配置文件的嵌套式层次结构，以便为更复杂的应用程序启用高级流量路由配置。目前尚无法通过 ARM API 使用这些终结点。

在所有三种情况下，可以通过两种方式来添加终结点：
1.使用类似于[更新流量管理器配置文件](#update-traffic-manager-profile)中描述的 3 步流程：使用 Get-AzureRmTrafficManagerProfile 获取配置文件对象；使用 Add-AzureRmTrafficManagerEndpointConfig 对其进行脱机更新以添加终结点；使用 Set-AzureRmTrafficManagerProfile 将更改上载到 Azure 流量管理器。此方法的优点是可以通过一次更新完成许多终结点更改。
2.使用 New-AzureRmTrafficManagerEndpoint cmdlet。这样可在一个操作中将终结点添加到现有的流量管理器配置文件中。

### 添加 Azure 终结点
Azure 终结点会引用在 Azure 中托管的其他服务。目前支持 3 类 Azure 终结点：
1.Azure 网站 
2.“经典”云服务（可能包含 PaaS 服务或 IaaS 虚拟机）
3.ARM Microsoft.Network/publicIpAddress 资源（可以附加到负载平衡器或虚拟机 NIC）。请注意，必须为 publicIpAddress 指定 DNS 名称，然后才能在流量管理器中使用它。

在每种情况下，
 - 均可使用 Add-AzureRmTrafficManagerEndpointConfig 或 New-AzureRmTrafficManagerEndpoint 的“targetResourceId”参数指定服务。
 - 不应指定“Target”和“EndpointLocation”，它们是通过上面指定的 TargetResourceId 隐式指定的。
 - 指定“Weight”为可选操作。仅当配置文件被配置为使用“加权”流量路由方法时，才使用加权，否则会忽视加权。如果指定，则其范围必须为 1...1000。默认值为“1”。
 - 指定“Priority”为可选操作。仅当配置文件被配置为使用“优先级”流量路由方法时，才使用优先级，否则会忽视优先级。有效值为 1 到 1000（值越低，优先级越高）。如果为一个终结点指定了该值，则必须为所有终结点指定该值。如果省略，则会按提供终结点的顺序应用默认值（从 1 开始，然后依次为 2、3，等等）。

#### 示例 1：使用 Add-AzureRmTrafficManagerEndpointConfig 添加网站终结点
在此示例中，我们使用 Add-AzureRmTrafficManagerEndpointConfig cmdlet 创建了一个新的流量管理器配置文件并添加了两个网站终结点，然后使用 Set-AzureRmTrafficManagerProfile 将更新的配置文件提交到了 Azure 流量管理器。

	PS C:\> $profile = New-AzureRmTrafficManagerProfile -Name myprofile -ResourceGroupName myrg -TrafficRoutingMethod Performance -RelativeDnsName myapp -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
	PS C:\> $webapp1 = Get-AzureRMWebApp -Name webapp1
	PS C:\> Add-AzureRmTrafficManagerEndpointConfig -EndpointName webapp1ep -TrafficManagerProfile $profile -Type AzureEndpoints -TargetResourceId $webapp1.Id -EndpointStatus Enabled
	PS C:\> $webapp2 = Get-AzureRMWebApp -Name webapp2
	PS C:\> Add-AzureRmTrafficManagerEndpointConfig -EndpointName webapp2ep -TrafficManagerProfile $profile -Type AzureEndpoints -TargetResourceId $webapp2.Id -EndpointStatus Enabled
	PS C:\> Set-AzureRmTrafficManagerProfile -TrafficManagerProfile $profile  

#### 示例 2：使用 New-AzureRmTrafficManagerEndpoint 添加“经典”云服务终结点
在此示例中，“经典”云服务终结点添加到了流量管理器配置文件中。请注意，在这种情况下，我们选择使用配置文件名称和资源组名称来指定配置文件，而不是通过传递配置文件对象的方式来指定（两种方法均受支持）。

	PS C:\> $cloudService = Get-AzureRmResource -ResourceName MyCloudService -ResourceType "Microsoft.ClassicCompute/domainNames" -ResourceGroupName MyCloudService
	PS C:\> New-AzureRmTrafficManagerEndpoint -Name MyCloudServiceEndpoint -ProfileName MyProfile -ResourceGroupName MyRG -Type AzureEndpoints -TargetResourceId $cloudService.Id -EndpointStatus Enabled

#### 示例 3：使用 New-AzureRmTrafficManagerEndpoint 添加 publicIpAddress 终结点
在此示例中，ARM 公共 IP 地址资源添加到了流量管理器配置文件中。公共 IP 地址必须配置了 DNS 名称，并且可以绑定到 VM 的 NIC 或者绑定到负载平衡器。

	PS C:\> $ip = Get-AzureRmPublicIpAddress -Name MyPublicIP -ResourceGroupName MyResourceGroup
	PS C:\> New-AzureRmTrafficManagerEndpoint -Name MyIpEndpoint -ProfileName MyProfile -ResourceGroupName MyRG -Type AzureEndpoints -TargetResourceId $ip.Id -EndpointStatus Enabled

### 添加外部终结点
流量管理器使用外部终结点将流量定向到在 Azure 外部承载的服务。对于 Azure 终结点，外部终结点可以通过先使用 Add-AzureRmTrafficManagerEndpointConfig 再使用 Set-AzureRmTrafficManagerProfile 的方式来添加，也可以通过 New-AzureRMTrafficManagerEndpoint 来添加。

指定外部终结点时：
 - 终结点域名必须使用“Target”参数指定
 - 如果使用了“性能”流量路由方法，则“EndpointLocation”为必填项，否则为可选项。该值必须是有效的 Azure 区域名称。
 - 对于 Azure 终结点来说，“Weight”和“Priority”为可选项。

#### 示例 1：使用 Add-AzureRmTrafficManagerEndpointConfig 和 Set-AzureRmTrafficManagerProfile 添加外部终结点
在此示例中，我们创建了一个新的流量管理器配置文件，添加了两个外部终结点，并提交了所做的更改。

	PS C:\> $profile = New-AzureRmTrafficManagerProfile -Name myprofile -ResourceGroupName myrg -TrafficRoutingMethod Performance -RelativeDnsName myapp -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
	PS C:\> Add-AzureRmTrafficManagerEndpointConfig -EndpointName eu-endpoint -TrafficManagerProfile $profile -Type ExternalEndpoints -Target app-eu.contoso.com -EndpointStatus Enabled
	PS C:\> Add-AzureRmTrafficManagerEndpointConfig -EndpointName us-endpoint -TrafficManagerProfile $profile -Type ExternalEndpoints -Target app-us.contoso.com -EndpointStatus Enabled
	PS C:\> Set-AzureRmTrafficManagerProfile -TrafficManagerProfile $profile  

#### 示例 2：使用 New-AzureRmTrafficManagerEndpoint 添加外部终结点
在此示例中，我们将外部终结点添加到了现有的配置文件，该配置文件是使用配置文件名称和资源组名称指定的。

	PS C:\> New-AzureRmTrafficManagerEndpoint -Name eu-endpoint -ProfileName MyProfile -ResourceGroupName MyRG -Type ExternalEndpoints -Target app-eu.contoso.com -EndpointStatus Enabled

## 更新流量管理器终结点
有两种方法来更新现有的流量管理器终结点：
1.使用 Get-AzureRmTrafficManagerProfile 获取流量管理器配置文件，更新该配置文件中的终结点属性，然后使用 Set-AzureRmTrafficManagerProfile 提交所做的更改。此方法的优势在于能够通过单个操作更新多个终结点。
2.使用 Get-AzureRmTrafficManagerEndpoint 获取流量管理器终结点，更新终结点属性，然后使用 Set-AzureRmTrafficManagerEndpoint 提交所做的更改。此方法更简单，因为不需要在配置文件的终结点数组中进行索引操作。

#### 示例 1：使用 Get-AzureRmTrafficManagerProfile 和 Set-AzureRmTrafficManagerProfile 更新终结点
在此示例中，我们将修改现有配置文件中两个终结点上的优先级。

	PS C:\> $profile = Get-AzureRmTrafficManagerProfile -Name myprofile -ResourceGroupName myrg
	PS C:\> $profile.Endpoints[0].Priority = 2
	PS C:\> $profile.Endpoints[1].Priority = 1
	PS C:\> Set-AzureRmTrafficManagerProfile -TrafficManagerProfile $profile

#### 示例 2：使用 Get-AzureRmTrafficManagerEndpoint 和 Set-AzureRmTrafficManagerEndpoint 更新终结点
在此示例中，我们将修改现有配置文件中单个终结点的权重。

	PS C:\> $endpoint = Get-AzureRmTrafficManagerEndpoint -Name myendpoint -ProfileName myprofile -ResourceGroupName myrg -Type ExternalEndpoints
	PS C:\> $endpoint.Weight = 20
	PS C:\> Set-AzureRmTrafficManagerEndpoint -TrafficManagerEndpoint $endpoint

## 启用和禁用终结点和配置文件
可以通过流量管理器启用和禁用各个终结点，以及启用和禁用整个配置文件。
这些更改可以通过获取/更新/设置终结点或配置文件资源来完成。为了简化这些常用操作，也可通过专用 cmdlet 来完成它们。

#### 示例 1：启用和禁用流量管理器配置文件
若要启用流量管理器配置文件，请使用 Enable-AzureRmTrafficManagerProfile。可以通过配置文件对象（通过管道或“-TrafficManagerProfile”参数来传递）来指定配置文件，也可以通过配置文件名称和资源组名称来指定，如本示例所示。

	PS C:\> Enable-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyResourceGroup

同样，若要禁用流量管理器配置文件，请执行以下操作：

	PS C:\> Disable-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyResourceGroup

Disable-AzureRmTrafficManagerProfile cmdlet 会提示你进行确认，该提示可以使用“-Force”参数来取消。

#### 示例 2：启用和禁用流量管理器终结点
若要启用流量管理器终结点，请使用 Enable-AzureRmTrafficManagerEndpoint。可以使用 TrafficManagerEndpoint 对象（通过管道或“-TrafficManagerEndpoint”参数传递）来指定终结点，也可以通过终结点名称、终结点类型、配置文件名称和资源组名称来指定：

	PS C:\> Enable-AzureRmTrafficManagerEndpoint -Name MyEndpoint -Type AzureEndpoints -ProfileName MyProfile -ResourceGroupName MyResourceGroup

同样，若要禁用流量管理器配置文件，请执行以下操作：

 	PS C:\> Disable-AzureRmTrafficManagerEndpoint -Name MyEndpoint -Type AzureEndpoints -ProfileName MyProfile -ResourceGroupName MyResourceGroup -Force

与 Disable-AzureRmTrafficManagerProfile 一样，Disable-AzureRmTrafficManagerEndpoint cmdlet 包括一个确认提示，该提示可以使用“-Force”参数取消。

## 删除流量管理器终结点
若要删除流量管理器终结点，一种方法是先检索配置文件对象（使用 Get-AzureRmTrafficManagerProfile），在本地配置文件对象中更新终结点列表，然后提交所做的更改（使用 Set-AzureRmTrafficManagerProfile）。此方法允许多个终结点更改一起提交。

另一种删除单个终结点的方法是使用 Remove-AzureRmTrafficManagerEndpoint cmdlet：

	PS C:\> Remove-AzureRmTrafficManagerEndpoint -Name MyEndpoint -Type AzureEndpoints -ProfileName MyProfile -ResourceGroupName MyResourceGroup
	
此 cmdlet 将提示你进行确认，除非已使用“-Force”参数取消了该提示。

## 删除流量管理器配置文件
若要删除流量管理器配置文件，请使用 Remove-AzureRmTrafficManagerProfile cmdlet，并指定配置文件名称和资源组名称：

	PS C:\> Remove-AzureRmTrafficManagerProfile -Name MyProfile -ResourceGroupName MyAzureResourceGroup [-Force]

此 cmdlet 将提示你确认。可以使用可选的“-Force”开关来取消此提示。也可以使用配置文件对象指定要删除的配置文件：

	PS C:\> $profile = Get-AzureTrafficManagerProfile -Name MyProfile -ResourceGroupName MyAzureResourceGroup
	PS C:\> Remove-AzureTrafficManagerProfile -TrafficManagerProfile $profile [-Force]

还可以通过管道执行此序列：

	PS C:\> Get-AzureTrafficManagerProfile -Name MyProfile -ResourceGroupName MyAzureResourceGroup | Remove-AzureTrafficManagerProfile [-Force]

## 后续步骤

[流量管理器监视](/documentation/articles/traffic-manager-monitoring)

[流量管理器性能注意事项](/documentation/articles/traffic-manager-performance-considerations)
 

<!---HONumber=Mooncake_0118_2016-->