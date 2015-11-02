<properties
   pageTitle="Azure 资源管理器下的 Azure 计算、网络和存储提供程序"
   description="计算、网络和存储资源提供程序（CRP、NRP 和 SRP）的概念性概述"
   services="virtual-machines"
   documentationCenter="dev-center-name"
   authors="mahthi"
   manager="coreysa"
   editor=""
   tags="azure-resource-manager,azure-service-management"/>

<tags 
   ms.service="virtual-machines"
   ms.date="04/29/2015" 
   wacn.date="08/29/2015"/>

# Azure 资源管理器下的 Azure 计算、网络和存储提供程序

Azure 资源管理器包含了计算、网络和存储功能，这将从根本上简化 IaaS 上运行的复杂应用程序的部署和管理。许多应用程序需要资源（包括虚拟网络、存储帐户、虚拟机和网络接口）的组合。Azure 资源管理器提供构造 JSON 模板的功能，以便将所有这些资源作为单个应用程序一起部署和管理。

## 集成 Azure 资源管理器下的计算、网络和存储提供程序的优点

Azure 资源管理器提供能够轻松利用预构建的应用程序模板或构建应用程序模板的功能，以部署和管理 Azure 上的计算、网络和存储资源。在此部分中，我们将逐步介绍通过 Azure 资源管理器部署资源的优点。

-	从复杂变得简单 — 在可以包括某个可共享模板文件中所有的 Azure 资源（例如网站、SQL 数据库、虚拟机或虚拟网络）的复杂应用程序上进行构建、集成和协作
-	当使用相同的模板文件时，开发、devOps 和系统管理员具有重复部署的灵活性
-	VM 扩展（自定义脚本、DSC、Chef、Puppet 等）与模板文件中 Azure 资源管理器的深度集成允许 VM 内安装程序配置的简易业务流程
-	用于计算、网络和存储资源的定义标记以及这些标记的计费传播
-	使用 Azure 基于角色的访问控制 (RBAC) 进行简单而精确的组织资源访问管理
-	通过修改原始模板，然后对其进行重新部署的简化的升级/更新情景


## Azure 资源管理器下计算、网络和存储 API 的改进完善

除了以上提到的优点外，发布的 API 中还有一些重大的性能改进。

-	启用大规模、并行的虚拟机部署
-	支持 3 容错域的可用性集
-	改进的自定义脚本扩展，允许来自任何可公开访问的自定义 URL 的脚本规范
- 虚拟机与 Azure 密钥保管库的集成，以从 [FIPS 验证的](https://zh.wikipedia.org/wiki/FIPS_140-2)[硬件安全模块](https://zh.wikipedia.org/wiki/Hardware_security_module)进行高度安全的存储和专用机密的部署
-	通过 API 提供网络的基本构造块，以使客户能够构建包含网络接口、负载平衡器和虚拟网络的复杂应用程序
-	作为新对象的网络接口允许复杂的网络配置对虚拟机持续并可重复使用
-	作为第一类资源的负载平衡器启用 IP 地址分配
-	精细虚拟网络 API 允许简化单个虚拟网络的管理

## 新 API 的引入的概念差异

在本部分中，我们将通过 Azure 资源管理器的计算、网络和存储，逐步介绍如今基于 XML 的可用 API 和基于 JSON 的可用 API 之间的一些最重要的概念差异。

 项目 | Azure 服务管理（基于 XML） | 计算、网络和存储提供程序（基于 JSON）
 ---|---|---
| 虚拟机的云服务 |	云服务是用于保存需要从平台和负载平衡获得可用性的虚拟机的容器。 | 云服务不再是使用新模型创建虚拟机所需的对象。 |
| 可用性集 | 通过在虚拟机上配置相同的“AvailabilitySetName”指示平台的可用性。容错域的最大计数为 2。 | 可用性集是由 Microsoft.Compute 提供程序公开的资源。要求高可用性的虚拟机必须包括在可用性集中。容错域的最大计数目前为 3。 |
| 地缘组 |	创建虚拟网络需要地缘组。但是，引入区域虚拟网络后，不再作此要求。 |为了简化，地缘组概念不存在于通过 Azure 资源管理器公开的 API 中。 |
| 负载平衡 | 云服务的创建为部署的虚拟机提供了隐式负载平衡器。 | 负载平衡器是由 Microsoft.Network 提供程序公开的资源。需要进行负载平衡的虚拟机的主网络接口应引用负载平衡器。负载平衡器可以为内部或外部。<!--[-->了解详细信息。<!--](/documentation/articles/resource-groups-networking)--> |
|虚拟 IP 地址 | 当 VM 添加到云服务时，云服务将获得默认的 VIP（虚拟 IP 地址）。虚拟 IP 地址是与隐式负载平衡器相关联的地址。 | 公共 IP 地址是由 Microsoft.Network 提供程序公开的资源。公共 IP 地址可为静态（保留）或动态。可以将动态公共 IP 分配到负载平衡器。可以使用安全组来保护公共 IP。 |
|保留 IP 地址|	可以将 IP 地址保留在 Azure 中并将其与云服务关联，以确保 IP 地址为粘性。 | 可以在“静态”模式下创建公共 IP 地址，并且它提供了与“保留 IP 地址”相同的功能。目前，仅可以将静态公共 IP 分配到负载平衡器。 |
|每个 VM 的公共 IP 地址 (PIP) | 公共 IP 地址也可以直接关联到 VM。 | 公共 IP 地址是由 Microsoft.Network 提供程序公开的资源。公共 IP 地址可为静态（保留）或动态。但是，仅动态公共 IP 可分配到网络接口，以立即获取每个 VM 的公共 IP。 |
|终结点| 需要在要打开特定端口的连接的虚拟机上配置的输入终结点。通过设置输入终结点，完成了连接到虚拟机的常见模式之一。 | 可以在负载平衡器上配置入站 NAT 规则，以实现启用用于连接到 VM 的特定端口上的终结点的相同功能。 |
|DNS 名称| 云服务将获得一个隐式全局唯一 DNS 名称。例如：`mycoffeeshop.chinacloudapp.cn`。 | DNS 名称是可以在公共 IP 地址资源中指定的可选参数。FQDN 将为以下格式 - `<domainlabel>.<region>.cloudapp.azure.com`。 |
|网络接口 | 主要和辅助网络接口以及其属性被定义为虚拟机的网络配置。 | 网络接口是由 Microsoft.Network 提供程序公开的资源。网络接口的生命周期不依赖于虚拟机。 |

## 虚拟机的 Azure 模板入门

可以通过利用我们拥有的用于开发和部署到平台的各种工具，开始使用 Azure 模板。

### Azure 门户

Azure 门户将继续同时具有部署虚拟机和虚拟机（预览版）的选项。Azure 门户也允许自定义模板部署。

### Azure PowerShell

Azure PowerShell 将具有两种部署模式 - **AzureServiceManagement** 模式和 **AzureResourceManager** 模式。AzureResourceManager 模式现在还包含用于管理虚拟机、虚拟网络和存储帐户的 cmdlet。你可以在[此处](/documentation/articles/powershell-azure-resource-manager)阅读有关详细内容。

### Azure CLI

Azure 命令行界面 (Azure CLI) 将具有两种部署模式 - **AzureServiceManagement** 模式和 **AzureResourceManager** 模式。AzureResourceManager 模式现在还包含用于管理虚拟机、虚拟网络和存储帐户的命令。你可以在[此处](/documentation/articles/xplat-cli-azure-resource-manager)阅读有关详细内容。

### Visual Studio

使用 Visual Studio 的最新 Azure SDK 版本，可以直接从 Visual Studio 创作和部署虚拟机以及复杂的应用程序。Visual Studio 提供了从预建的模板列表部署或从空模板启动的功能。

### REST API

你可以在[此处](https://msdn.microsoft.com/zh-cn/library/azure/dn790568.aspx)找到有关计算、网络和存储资源提供程序的详细 REST API 文档。

## 常见问题

**是否可以使用新的 Azure 资源管理器来创建虚拟机，用来在虚拟网络或在使用 Azure 服务管理 API 创建的存储帐户中进行部署？**

目前不支持此操作。不能使用新的 Azure 资源管理器 API 部署，以将虚拟机部署到使用服务管理 API 创建的虚拟网络。

**是否可以使用用 Azure 服务管理 API 创建的用户映像中的新 Azure 资源管理器 API 来创建虚拟机？**

目前不支持此操作。但是，可以将 VHD 文件从使用服务管理 API 创建的存储帐户复制到使用新的 Azure 资源管理器 API 创建的一个新帐户。

**对我的订阅的配额有什么影响？**

通过新的 Azure 资源管理器 API 创建的虚拟机、虚拟网络和存储帐户的配额与你当前所具有的配额是分开的。每个订阅都会获取新的配额，以使用新的 API 创建资源。你可以在<!--[-->此处<!--](/documentation/articles/azure-subscription-service-limits)-->阅读有关其他配额的详细内容。

**是否可以继续使用我的自动脚本，用于通过新的 Azure 资源管理器 API 预配虚拟机、虚拟网络和存储帐户等？**

所有的自动化和已构建的脚本将继续适用于现有的在 Azure 服务管理模式下创建的虚拟机、虚拟网络。但是，这些脚本必须进行更新，以使用用于通过新的 Azure 资源管理器模式创建相同资源的新架构。阅读有关如何修改 <!--[-->Azure CLI 脚本<!--](/documentation/articles/xplat-cli-azure-manage-vm-asm-arm)-->的详细内容。

**使用新的 Azure 资源管理器 API 创建的虚拟网络是否可以连接到我的 Express Route 线路？**

目前不支持此操作。不能将使用新的 Azure 资源管理器 API 创建的虚拟网络与 Express Route 线路进行连接。未来将支持此操作。
 

<!---HONumber=67-->