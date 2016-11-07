<properties
    pageTitle="流量管理器终结点类型 | Azure"
    description="本文介绍可以通过 Azure 流量管理器来使用的不同类型的终结点"
    services="traffic-manager"
    documentationCenter=""
    authors="sdwheeler"
    manager="carmonm"
    editor=""
/>  

<tags
    ms.service="traffic-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="10/11/2016"
    wacn.date="11/07/2016"
    ms.author="sewhee"
/>

# 流量管理器终结点

使用 Azure 流量管理器可以控制如何将网络流量分布到在不同数据中心运行的应用程序部署。需要在流量管理器中将每个应用程序部署配置为一个“终结点”。当流量管理器收到 DNS 请求时，将选择要在在 DNS 响应中返回的可用终结点。流量管理器根据当前终结点状态和流量路由方法做出这种选择。相关详细信息，请参阅[流量管理器工作原理](/documentation/articles/traffic-manager-how-traffic-manager-works/)。

流量管理器支持三种类型的终结点：

- **Azure 终结点**用于在 Azure 中托管的服务。
- **外部终结点**用于在 Azure 外部托管的服务，不管是在本地托管还是通过其他托管提供商进行托管。
- **嵌套终结点**用于组合流量管理器配置文件，以便创建更灵活的流量路由方案，从而满足更大、更复杂部署的需求。

你可以不受限制地在单个流量管理器配置文件中通过各种方式组合不同类型的终结点。每个配置文件都可以包含任何组合形式的终结点类型。

以下各节更进一步地描述了每个终结点类型。

## Azure 终结点

Azure 终结点用于流量管理器中基于 Azure 的服务。支持以下 Azure 资源类型：

- “经典”IaaS VM 和 PaaS 云服务。
- Web 应用
- PublicIPAddress 资源（可直接或通过 Azure Load Balancer 连接到 VM）。必须为 publicIpAddress 分配一个 DNS 名称，然后才能在流量管理器配置文件中使用它。

PublicIPAddress 资源属于 Azure Resource Manager 资源。经典部署模型中没有这些资源。因此，这些资源仅在流量管理器的 Azure Resource Manager 体验中受支持。其他终结点类型通过 Resource Manager 和经典部署模型受到支持。

使用 Azure 终结点时，流量管理器可检测“经典”IaaS VM、云服务或 Web 应用的停止和启动时间。此状态反映在终结点状态中。有关详细信息，请参阅 [Traffic Manager endpoint monitoring](/documentation/articles/traffic-manager-monitoring/#endpoint-and-profile-status)（流量管理器终结点监视）。当基础服务停止时，流量管理器不会执行终结点运行状况检查，或者将流量定向到终结点。已停止的实例不会发生流量管理器计费事件。重新启动服务后，计费将会恢复，终结点可以接收流量。此项检测不适用于 PublicIpAddress 终结点。

## 外部终结点

外部终结点用于 Azure 外部的服务。例如，本地托管的服务或者具不同提供程序的服务。外部终结点可以单独使用，也可以在同一流量管理器配置文件中与 Azure 终结点结合使用。可以将 Azure 终结点与外部终结点结合用于多种方案：

- 在主动-主动或主动-被动故障转移模型中，可以使用 Azure 为现有的本地应用程序提供增强的冗余。
- 若要为全球各地的用户降低应用程序延迟，可以将现有的本地应用程序扩展到 Azure 中的其他地理位置。有关详细信息，请参阅 [Traffic Manager 'Performance' traffic routing](/documentation/articles/traffic-manager-routing-methods/#performance-traffic-routing-method)（流量管理器“性能”流量路由）。
- 使用 Azure 为现有的本地应用程序提供额外容量既可以持续满足高峰需求，也可以通过“云爆发”解决方案满足此类需求。

在某些情况下，可以使用外部终结点来引用 Azure 服务（有关示例，请参阅[常见问题](#faq)）。在本示例中，针对运行状况检查的计费是按照 Azure 终结点费率而非外部终结点费率进行的。但与 Azure 终结点不同，如果停止或删除基础服务，运行状况检查将持续计费，直到在流量管理器中禁用或删除该终结点为止。

## 嵌套式终结点

嵌套式终结点可以组合多个流量管理器配置文件，创建灵活的流量路由方案，帮助满足更大、更复杂的部署需求。使用嵌套式终结点时，会将“子”配置文件作为终结点添加到“父”配置文件。子配置文件和父配置文件可包含任何类型的其他终结点，包括其他嵌套式配置文件。有关详细信息，请参阅[嵌套式流量管理器配置文件](/documentation/articles/traffic-manager-nested-profiles/)。

## Web Apps 作为终结点

在流量管理器中将 Web Apps 配置为终结点时，还需考虑其他因素：

1. 仅“标准”SKU 或更高版 SKU 的 Web Apps 可以用于流量管理器。尝试添加 SKU 较低的 Web 应用将会失败。降低现有 Web 应用的 SKU 会导致流量管理器不再将流量发送到该 Web 应用。

2. 当某个终结点收到 HTTP 请求时，将使用请求中的“host”标头来确定应通过哪个 Web 应用来处理请求。主机头包含用于启动请求的 DNS 名称，例如“contosoapp.chinacloudsites.cn”。若要对 Web 应用使用其他 DNS 名称，必须将该 DNS 名称注册为该应用的自定义域名。将 Web 应用终结点添加为 Azure 终结点时，系统会自动为该应用注册流量管理器配置文件 DNS 名称。删除终结点时，将自动删除该注册。

3. 每个流量管理器配置文件最多允许一个 Azure 区域有一个 Web 应用终结点。若要克服这种约束，可为外部终结点配置一个 Web 应用。有关详细信息，请参阅[常见问题](#faq)。

## 启用和禁用终结点

在流量管理器中禁用终结点可能有助于从处于维护模式或正在重新部署的终结点中临时删除流量。在终结点再次运行后，可以重新启用该终结点。

可以通过流量管理器门户、PowerShell、CLI 或 REST API 来启用和禁用终结点，Resource Manager 和经典部署模型都支持这些操作。

>[AZURE.NOTE] 禁用某个 Azure 终结点对其在 Azure 中的部署状态没有任何影响。Azure 服务（例如 VM 或 Web 应用）将保持运行并可接收流量，即使已在流量管理器中禁用。流量可直接定向到该服务实例，而无需通过流量管理器配置文件 DNS 名称。相关详细信息，请参阅[流量管理器工作原理](/documentation/articles/traffic-manager-how-traffic-manager-works/)。

目前，每个终结点接收流量的资格取决于以下因素：

- 配置文件状态（已启用/已禁用）
- 终结点状态（已启用/已禁用）
- 该终结点的运行状况检查结果

相关详细信息，请参阅[流量管理器终结点监视](/documentation/articles/traffic-manager-monitoring/#endpoint-and-profile-status)。

>[AZURE.NOTE] 由于流量管理器是在 DNS 级别工作的，因此不可能影响任何终结点的现有连接。当某个终结点不可用时，流量管理器会将新连接定向到另一个可用的终结点。不过，已禁用或不正常的终结点后面的主机仍可通过现有连接继续接收流量，直到相关会话被终止。应用程序应该限制会话持续时间，以便耗尽现有连接的流量。

如果禁用配置文件中的所有终结点，或者禁用配置文件本身，则流量管理器会向新的 DNS 查询发送“NXDOMAIN”响应。

## <a name="faq"></a>常见问题

### 能否将流量管理器用于多个订阅的终结点？

不能对 Azure Web 应用使用多个订阅中的终结点。Web 应用要求其所用的任何自定义域名只能在单个订阅中使用。无法对多个订阅中的 Web 应用使用同一个域名。

对于其他终结点类型，可在多个订阅中结合使用流量管理器和终结点。流量管理器的配置方式取决于使用的是经典部署模型还是 Resource Manager 体验。

- 在 Resource Manager 中，只要配置流量管理器配置文件的人员具有终结点的读取访问权限，任何订阅的终结点就都可添加到流量管理器中。可使用 [Azure Resource Manager 基于角色的访问控制 (RBAC)](/documentation/articles/role-based-access-control-configure/) 授予这些权限。
- 在经典部署模型接口中，流量管理器要求配置为 Azure 终结点的云服务或 Web 应用必须与流量管理器配置文件位于同一个订阅中。可以将其他订阅中的云服务终结点作为“外部”终结点添加到流量管理器。这些外部终结点按 Azure 终结点而不是外部费率计费。

### 能否将流量管理器用于云服务的“过渡”槽？

是的。可以在流量管理器中将云服务的“过渡”槽配置为“外部”终结点。运行状况检查仍按 Azure 终结点费率计费。由于使用的是“外部”终结点类型，系统不会自动拾取对基础服务所做的更改。使用外部终结点时，流量管理器无法检测停止或删除云服务的时间。因此，在禁用或删除终结点之前，流量管理器会一直对运行状况检查计费。

### 流量管理器是否支持 IPv6 终结点？

流量管理器当前不提供 IPv6 可寻址的名称服务器。但是，IPv6 客户端仍可使用流量管理器连接到 IPv6 终结点。客户端不会直接向流量管理器发出 DNS 请求，而是使用递归 DNS 服务。仅使用 IPv6 的客户端通过 IPv6 向递归 DNS 服务发送请求。然后，该递归服务应该能够使用 IPv4 联系流量管理器名称服务器。

流量管理器使用终结点的 DNS 名称做出响应。若要支持 IPv6 终结点，必须有一条可将终结点 DNS 名称指向 IPv6 地址的 DNS AAAA 记录。流量管理器运行状况检查仅支持 IPv4 地址。服务需要在相同的 DNS 名称中公开 IPv4 终结点。

### 流量管理器是否可用于同一区域中的多个 Web 应用？

通常情况下，流量管理器用于将流量引导到部署在不同区域中的应用程序。不过，流量管理器也可用于一个应用程序在同一区域中存在多个部署的情形。流量管理器 Azure 终结点不允许将同一 Azure 区域中的多个 Web 应用终结点添加到同一流量管理器配置文件。

执行以下步骤即可解除该约束：

1. 检查终结点是否在不同的 Web 应用“缩放单位”中。域名必须映射到给定缩放单位中的单个站点。因此，同一缩放单位中的两个 Web 应用不能共享一个流量管理器配置文件。
2. 将虚构域名作为自定义主机名添加到每个 Web 应用。每个 Web 应用必须位于不同的缩放单位中。所有 Web 应用必须属于同一个订阅。
3. 将一个（仅限一个）Web 应用终结点作为 Azure 终结点添加到流量管理器配置文件。
4. 将每个附加的 Web 应用终结点作为“外部”终结点添加到流量管理器配置文件。只能使用 Resource Manager 部署模型添加外部终结点。
5. 在虚构域中创建一条指向流量管理器配置文件 DNS 名称 (<…>.trafficmanager.cn) 的 DNS CNAME 记录。
6. 通过虚构域名而不是流量管理器配置文件 DNS 名称来访问你的站点。

## 后续步骤

- 了解[流量管理器工作原理](/documentation/articles/traffic-manager-how-traffic-manager-works/)。
- 了解流量管理器[终结点监视和自动故障转移](/documentation/articles/traffic-manager-monitoring/)。
- 了解流量管理器[流量路由方法](/documentation/articles/traffic-manager-routing-methods/)。

<!---HONumber=Mooncake_1031_2016-->