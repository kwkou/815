<!-- Ibiza portal: tested -->

## 将计算、网络和存储集成到 Azure 资源管理器部署模型下的优点

借助 Azure 资源管理器部署模型，你可以轻松利用预建的应用程序模板或构建一个应用程序模板，在 Azure 上部署和管理计算、网络和存储资源。在本部分内容中，我们将了解通过 Azure 资源管理器部署模型部署资源的优点。

-	化繁为简 — 从可共享的模板文件构建、集成可包括整个 Azure 资源（例如网站、SQL 数据库、虚拟机或虚拟网络）的复杂应用程序，以及在这些应用程序上进行协作
-	当使用同一模板文件时，开发、devOps 和系统管理员具有可重复部署的灵活性
-	虚拟机扩展（自定义脚本、DSC、Chef、Puppet 等）和 Azure 资源管理器深度集成到一个模板文件，允许轻松地安排虚拟机内的设置配置
-	为计算、网络和存储资源定义标记和这些标记的计费传播
-	使用 Azure 基于角色的访问控制 (RBAC) 进行简单且精确的组织资源访问管理
-	通过修改原始模板然后重新部署来简化升级/更新流程


## Azure Resource Manager 下计算、网络和存储 API 的改进

除了上面提到的优点外，在发布的 API 中还有一些重大的性能改进：

-	使虚拟机的大规模并行部署成为可能
-	在可用性集中支持 3 个容错域
-	改进了自定义脚本扩展，支持来自任何可公开访问的自定义 URL 的脚本规范
- 集成虚拟机和 Azure 密钥保管库，从 [FIPS-验证](http://wikipedia.org/wiki/FIPS_140-2)[硬件安全模块](http://wikipedia.org/wiki/Hardware_security_module)实现高度安全的存储和专有部署
-	通过 API 提供基本的网络构建块，使客户能够构建包含网络接口、负载平衡器和虚拟网络的复杂应用程序
-	网络接口作为新对象出现，允许持续保持和重复使用虚拟机的复杂网络配置
-	作为第一类资源的负载平衡器使 IP 地址分配成为可能
-	粒状虚拟网络 API 让你能够简化对单个虚拟网络的管理

## 新的 API 带来的概念差异

在本部分内容中，我们将了解目前可用的基于 XML 的 API 与基于 JSON 的 API（通过 Azure 资源管理器用于计算、网络和存储）之间的某些最重要的概念差异。

| 项目 | Azure 服务管理（基于 XML） | 计算、网络和存储提供程序（基于 JSON）|
|---|---|---|
| 面向虚拟机的云服务 |	云服务是一个容器，用于容纳要求平台可用性和负载平衡的虚拟机。 | 使用新模型，云服务不再是创建虚拟机所必需的对象。 |
| 可用性集 | 通过在虚拟机上配置相同的“AvailabilitySetName”来指出平台的可用性。容错域的最大数量为 2。 | 可用性集是 Microsoft.Compute 提供程序提供的一个资源。要求高可用性的虚拟机必须包含在可用性集中。现在，容错域的最大数量为 3。 |
| 地缘组 |	创建虚拟网络需要地缘组。但是，随着区域虚拟网络的引入，不再需要地缘组了。 |为了简单起见，地缘组概念不再存在于通过 Azure Resource Manager 提供的 API 中。 |
| 负载平衡 | 云服务的创建为部署的虚拟机提供了一个隐式负载平衡器。 | 负载平衡器是 Microsoft.Network 提供程序提供的一个资源。需要负载平衡的虚拟机的主网络接口应该引用负载平衡器。负载平衡器既可以是内部的，也可以是外部的。 |
|虚拟 IP 地址 | 虚拟机添加到云服务后，云服务将获得默认 VIP（虚拟 IP 地址）。虚拟 IP 地址是与隐式负载平衡器相关联的地址。 | 公共 IP 地址是 Microsoft.Network 提供程序提供的一个资源。公共 IP 地址既可以是静态（保留）的，也可以是动态的。可以将动态公共 IP 分配到一个负载平衡器。可以使用安全组保护公共 IP。 |
|保留 IP 地址|	可以在 Azure 中保留一个 IP 地址并将其与一个云服务关联在一起，以确保该 IP 地址具有粘性。 | 可以在“静态”模式下创建公共 IP 地址，并且该地址提供与“保留 IP 地址”相同的功能。目前，静态公共 IP 只能分配到一个负载平衡器。 |
|每个虚拟机一个公共 IP 地址 (PIP) | 公共 IP 地址也可以直接关联到虚拟机。 | 公共 IP 地址是 Microsoft.Network 提供程序提供的一个资源。公共 IP 地址既可以是静态（保留）的，也可以是动态的。但是，目前只有动态公共 IP 可分配到网络接口，每个虚拟机获得一个公共 IP。 |
|终结点| 需要在虚拟机上配置输入终结点，用于打开某些端口的连接。这是通过设置输入终结点来连接到虚拟机的一个常见模式。 | 可以在负载平衡器上配置入站 NAT 规则，实现在具体端口上启用终结点以连接到虚拟机的相同功能。 |
|DNS 名称| 云服务会得到一个隐式的全局唯一 DNS 名称。例如：`mycoffeeshop.chinacloudapp.cn`。 | DNS 名称是可在一个公共 IP 地址资源中指定的可选参数。FQDN 将采用以下格式 - `<domainlabel>.<region>.chinacloudapp.cn`。 |
|网络接口 | 作为虚拟机的网络配置定义主网络接口和辅助网络接口及其属性。 | 网络接口是 Microsoft.Network 提供程序提供的一个资源。网络接口的生命周期与虚拟机无关。 |

## 开始使用 Azure 虚拟机模板

你可以通过充分利用我们为开发和部署到平台而提供的各种工具来开始使用 Azure 模板。

### Azure 门户预览

Azure 门户预览将继续提供相关选项，使用户既能使用经典部署模型来部署虚拟机，同时又能使用 Resource Manager 部署模型来部署虚拟机。Azure 门户预览还将允许自定义模板部署。

### Azure PowerShell

Azure PowerShell 将拥有两种部署模式 - **AzureServiceManagement** 模式和 **AzureResourceManager** 模式。AzureResourceManager 模式现在同时包含用于管理虚拟机、虚拟网络和存储帐户的 cmdlet。你可以在[此处](/documentation/articles/powershell-azure-resource-manager/)阅读详细内容。

### Azure CLI

Azure 命令行接口 (Azure CLI) 将拥有两种部署模式 - **AzureServiceManagement** 模式和 **AzureResourceManager** 模式。AzureResourceManager 模式现在同时包含用于管理虚拟机、虚拟网络和存储帐户的命令。

### Visual Studio

使用面向 Visual Studio 的最新版 Azure SDK，你可以直接从 Visual Studio 编写和部署虚拟机以及复杂的应用程序。Visual Studio 提供从预建的模板清单或从空白模板开始部署的能力。

### REST API

你可以在[此处](https://msdn.microsoft.com/zh-cn/library/azure/dn790568.aspx)找到有关计算、网络和存储资源提供程序的 REST API 的详细说明文档。

## 常见问题

**我能使用新的 Azure 资源管理器创建一个虚拟机以在使用 Azure 服务管理 API 创建的虚拟网络或存储帐户中部署吗？**

目前此操作不受支持。不能使用新的 Azure 资源管理器 API 将虚拟机部署到使用服务管理 API 创建的虚拟网络中。

**我能使用新的 Azure 资源管理器 API 从使用 Azure 服务管理 API 创建的用户映像创建虚拟机吗？**

目前此操作不受支持。但是，你可以从使用服务管理 API 创建的存储帐户将 VHD 文件复制到使用新的 Azure 资源管理器 API 创建的新帐户中。

**对我的订阅的配额有何影响？**

通过新的 Azure 资源管理器 API 创建的虚拟机、虚拟网络和存储帐户的配额是你目前拥有的配额是分开的。每个订阅获取新的配额，以使用新的 API 创建资源。你可以在[此处](/documentation/articles/azure-subscription-service-limits/)了解有关额外配额的详细信息。

**我能通过新的 Azure 资源管理器 API 继续使用我的自动化脚本来预配虚拟机、虚拟网络、存储帐户等资源吗？**

你已经构建的所有自动化和脚本将继续适用于在 Azure 服务管理模式下创建的现有虚拟机和虚拟网络。然而，必须更新这些脚本以使用新的架构，通过新的 Azure 资源管理器模式来创建相同的资源。

**使用新的 Azure 资源管理器 API 创建的虚拟网络能够连接到我的 Express Route 线路吗？**

目前此操作不受支持。不能将使用新的 Azure 资源管理器 API 创建的虚拟网络连接到 Express Route 线路。以后将支持此功能。

**在哪里可以找到 Azure Resource Manager 模板的示例？**

可以在 [Azure Resource Manager QuickStart Templates](https://github.com/Azure/azure-quickstart-templates/)（Azure Resource Manager 快速启动模板）中找到一系列广泛的初学者模板。
