<properties
    pageTitle="Azure 密钥保管库开发人员指南 | Azure"
    description="开发人员可以使用 Azure 密钥保管库来管理 Azure 环境中的加密密钥。"
    services="key-vault"
    documentationcenter=""
    author="BrucePerlerMS"
    manager="mbaldwin" />
<tags
    ms.service="key-vault"
    ms.topic="article"
    ms.workload="identity"
    ms.date="05/10/2017"
    wacn.date="06/12/2017"
    ms.author="bruceper" 
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="9b528a470412e98174d74ab429b8d42c611a77b4"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="azure-key-vault-developers-guide"></a>Azure 密钥保管库开发人员指南

使用密钥保管库可以从应用程序中安全地访问敏感信息：

- 无需自己编写代码即可保护密钥和机密信息，并且能够轻松地在应用程序中使用它们。
- 能够让客户拥有和管理其自己的密钥，因此可以专注于提供核心软件功能。 这样，你的应用程序便不会对客户的租户密钥和机密承担职责或潜在责任。
- 应用程序可以使用密钥进行签名和加密，不过使密钥管理与应用程序分开，可以使解决方案适用于地理分散的应用。
- 自 2016 年 9 月版本的密钥保管库发布起，应用程序现在可以使用密钥保管库[证书](https://docs.microsoft.com/zh-cn/rest/api/keyvault/certificate-operations)。 有关详细信息，请参阅 [About keys, secrets, and certificates](https://docs.microsoft.com/zh-cn/rest/api/keyvault/about-keys--secrets-and-certificates)（关于密钥、机密和证书）。

有关 Azure 密钥保管库的更多常规信息，请参阅[什么是密钥保管库](/documentation/articles/key-vault-whatis/)。

## <a name="public-preview---may-10-2017"></a>公共预览版 - 2017 年 5 月 10 日

>[AZURE.NOTE]
>在此预览版 Azure 密钥保管库中，只有“软删除”功能处于预览版阶段。 作为一个整体，Azure 密钥保管库是一项完整的生产服务。

此预览版包括全新软删除功能、密钥保管库和密钥保管库对象的可恢复删除，以及更新的开发人员界面；[.NET/C#](https://docs.microsoft.com/dotnet/api/microsoft.azure.keyvault/)、[REST](https://docs.microsoft.com/zh-cn/rest/api/keyvault/) 和 [PowerShell](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.keyvault/)。 

有关全新软删除功能的详细信息，请参阅 [Azure 密钥保管库软删除概述](/documentation/articles/key-vault-ovw-soft-delete/)。

## <a name="creating-and-managing-key-vaults"></a>创建和管理密钥保管库

在代码中使用 Azure 密钥保管库之前，可以通过 REST、 资源管理器模板、PowerShell 或 CLI 创建和管理保管库，如以下文章中所述：

- [使用 REST 创建和管理密钥保管库](https://docs.microsoft.com/zh-cn/rest/api/keyvault/)
- [使用 PowerShell 创建和管理密钥保管库](/documentation/articles/key-vault-get-started/)
- [使用 CLI 创建和管理密钥保管库](/documentation/articles/key-vault-manage-with-cli2/)
- [通过 Azure 资源管理器模板创建密钥保管库并添加机密](https://docs.microsoft.com/en-us/azure/templates/microsoft.keyvault/vaults)

> [AZURE.NOTE]
> 针对密钥保管库执行的操作通过 AAD 进行身份验证并通过密钥保管库自己的访问策略（按保管库定义）进行授权。

## <a name="coding-with-key-vault"></a>使用密钥保管库进行编码

程序员的密钥保管库管理系统包含多个接口，并以 REST 作为基础。 通过 REST 接口，可以访问所有密钥保管库资源：密钥、机密和证书。 [Key Vault REST API Reference](https://docs.microsoft.com/zh-cn/rest/api/keyvault/)（密钥保管库 REST API 参考）。 

### <a name="supported-programming-languages"></a>支持的编程语言

#### <a name="net"></a>.NET

- [.NET API refence for Key Vault](https://docs.microsoft.com/dotnet/api/microsoft.azure.keyvault)（适用于密钥保管库的 .NET API 参考） 

有关 .NET SDK 2.x 版的详细信息，请参阅[发行说明](/documentation/articles/key-vault-dotnet2api-release-notes/)。

#### <a name="java"></a>Java

- [Java SDK for Key Vault](https://docs.microsoft.com/java/api/com.microsoft.azure.keyvault)（适用于密钥保管库的 Java SDK）

#### <a name="nodejs"></a>Node.js

- [Node.js API reference for Key Vault Managment](http://azure.github.io/azure-sdk-for-node/azure-arm-keyvault/latest/)（适用于密钥保管库管理的 Node.js API 参考）
- [Node.js API reference for Key Vault Operations](http://azure.github.io/azure-sdk-for-node/azure-keyvault/latest/)（适用于密钥保管库操作的 Node.js API 参考） 

### <a name="quick-start"></a>快速启动

- [Create Key Vault](https://github.com/Azure/azure-quickstart-templates/tree/master/101-key-vault-create)（创建密钥保管库）
- [Getting started with Key Vault in Node.js](https://azure.microsoft.com/en-us/resources/samples/key-vault-node-getting-started/)（Node.js 中的密钥保管库入门）

### <a name="code-examples"></a>代码示例

有关在应用程序中使用密钥保管库的完整示例，请参阅：

- [Azure 密钥保管库代码示例](http://www.microsoft.com/download/details.aspx?id=45343) - .NET 示例应用程序 HelloKeyVault 和 Azure Web 服务示例。 
- [从 Web 应用程序使用 Azure 密钥保管库](/documentation/articles/key-vault-use-from-web-application/) - 此教程介绍如何从 Azure 中的 Web 应用程序使用 Azure 密钥保管库。 

## <a name="how-tos"></a>操作方法

以下文章和方案提供了特定于任务的指导，方便用户使用 Azure 密钥保管库：

- [订阅移动后更改密钥保管库租户 ID](/documentation/articles/key-vault-subscription-move-fix/) - 将 Azure 订阅从租户 A 移到租户 B 时，租户 B.中的主体（用户和应用程序）无法访问现有的密钥保管库。使用本指南解决此问题。
- [访问防火墙后面的密钥保管库](/documentation/articles/key-vault-access-behind-firewall/) - 若要访问密钥保管库，密钥保管库客户端应用程序需要能够访问多个终结点才能使用各种功能。
- [如何在部署期间传递安全值（如密码）](/documentation/articles/resource-manager-keyvault-parameter/) - 需要在部署期间以参数形式传递安全值（例如密码）时，可以将该值存储为 Azure 密钥保管库中的机密，并在其他资源管理模板中引用该值。
- [如何使用密钥保管库，以便通过 SQL Server 进行可扩展的密钥管理](https://msdn.microsoft.com/zh-cn/library/dn198405.aspx) - 适用于 Azure 密钥保管库的 SQL Server 连接器允许 SQL Server 和 VM 中的 SQL 将 Azure 密钥保管库服务用作可扩展密钥管理 (EKM) 提供程序，以便保护其针对应用程序链接的加密密钥；透明数据加密、备份加密和列级加密。
- [如何将密钥保管库中的证书部署到 VM](https://blogs.technet.microsoft.com/kv/2015/07/14/deploy-certificates-to-vms-from-customer-managed-key-vault/) - 在 Azure 上的 VM 中运行的云应用程序需要一个证书。 现在，如何将此证书部署到此 VM 中？
- [如何使用端到端密钥轮换和审核设置密钥保管库](/documentation/articles/key-vault-key-rotation-log-monitoring/) - 逐步介绍如何设置 Azure 密钥保管库的密钥轮换和审核。
- [通过密钥保管库部署 Azure Web 应用证书]( https://blogs.msdn.microsoft.com/appserviceteam/2016/05/24/deploying-azure-web-app-certificate-through-key-vault/)提供有关部署作为[应用服务证书](https://azure.microsoft.com/zh-cn/blog/internals-of-app-service-certificate/)产品的一部分存储在密钥保管库中的证书的分步说明。

如需更多将密钥保管库与 Azure 集成和结合使用的特定于任务的指导，请参阅 [Ryan Jones Azure资源管理器template examples for Key Vault](https://github.com/rjmax/ArmExamples/tree/master/keyvaultexamples)（针对密钥保管库的 Ryan Jones Azure 资源管理器模板示例）。

## <a name="key-vault-overviews-and-concepts"></a>密钥保管库概述和概念

- [密钥保管库安全体系](/documentation/articles/key-vault-ovw-security-worlds/)
- [密钥保管库软删除](/documentation/articles/key-vault-ovw-soft-delete/)

## <a name="social"></a>社交

- [密钥保管库博客](http://aka.ms/kvblog)
- [密钥保管库论坛](http://aka.ms/kvforum)


## <a name="supporting-libraries"></a>支持库

- [Azure 密钥保管库核心库](http://www.nuget.org/packages/Microsoft.Azure.KeyVault.Core)提供 IKey 和 IKeyResolver 接口，用于通过标识符查找密钥，以及使用密钥执行操作。
- [Azure 密钥保管库扩展](http://www.nuget.org/packages/Microsoft.Azure.KeyVault.Extensions) 为 Azure 密钥保管库提供了扩展功能。

## <a name="other-key-vault-resources"></a>其他密钥保管库资源

<!--Update_Description: wording update-->