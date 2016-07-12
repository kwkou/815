<properties
   pageTitle="Azure Active Directory 图形 API"
   description="有关可通过 REST API 终结点提供对 Azure AD 的编程访问权限的图形 API 的概述和快速入门指南。"
   services="active-directory"
   documentationCenter=""
   authors="msmbaldwin"
   manager="mbaldwin"
   editor="mbaldwin" />
<tags
   ms.service="active-directory"
   ms.date="03/18/2016"
   wacn.date="06/21/2016" />

# Azure Active Directory 图形 API

> [AZURE.IMPORTANT]也可以通过 [Microsoft Graph](https://graph.microsoft.io/) 访问 Azure AD 图形 API 功能。Microsoft Graph 是统一的 API，其中包含 Outlook、OneDrive、OneNote、Planner 和 Office Graph 等其他 Microsoft 服务中的 API，可通过单个终结点和单个访问令牌进行访问。

Azure Active Directory 图形 API 通过 REST API 终结点提供对 Azure AD 的编程访问权限。应用程序可以使用图形 API 对目录数据和对象执行创建、读取、更新和删除 (CRUD) 操作。例如，图形 API 支持对用户对象执行以下常见操作：

- 在目录中创建新用户

- 获取用户的详细属性，如其所属的组

- 更新用户的属性（如其位置和电话号码），或者更改其密码

- 检查用户进行基于角色的访问时使用的组成员身份

- 禁用用户的帐户或完全将其删除

除了用户对象外，你还可以对其他对象（如组和应用程序）执行类似操作。若要对目录调用图形 API，应用程序必须向 Azure AD 注册并配置为允许访问目录。这通常可以通过用户或管理员同意流来实现。

若要开始使用 Azure Active Directory 图形 API，请参阅<!-- [图形 API 快速入门指南](/documentation/articles/active-directory-graph-api-quickstart/)，或查看 --> [交互式图形 API 参考文档](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog)。


## 功能

图形 API 提供以下功能：

- **REST API 终结点**：图形 API 是一种支持 REST 的服务，该服务由使用标准 HTTP 请求访问的终结点组成。图形 API 支持对请求和响应使用 XML 或 Javascript 对象表示法 (JSON) 内容类型。有关详细信息，请参阅 [Azure AD Graph REST API 参考](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog)。

- **向 Azure AD 进行身份验证**：必须在请求的 Authorization 标头中追加 JSON Web 令牌 (JWT)，以便对向图形 API 发出的每个请求进行身份验证。可通过向 Azure AD 的令牌终结点发出请求并提供有效的凭据来获取此令牌。可以使用 OAuth 2.0 客户端凭据流或授权代码授予流来获取调用 Graph 所需的令牌。有关详细信息，请参阅 [Azure AD 中的 OAuth 2.0](https://msdn.microsoft.com/library/azure/dn645545.aspx)。

- **基于角色的授权 (RBAC)**：安全组用于在图形 API 中执行 RBAC。例如，如果要确定用户是否有权访问特定资源，应用程序可以调用[检查组成员身份（可传递）](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/groups-operations#FunctionsandactionsongroupsCheckmembershipinaspecificgrouptransitive)操作，该操作将返回 true 或 false。

- **差异查询**：如果要查看两个时间段之间对目录所做的更改，而不对图形 API 进行频繁的查询，可以发出差异查询请求。这种类型的请求将只返回在上一个差异查询请求与当前请求之间所做的更改。有关详细信息，请参阅 [Azure AD 图形 API 差异查询](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-differential-query)。

- **目录扩展**：如果你要开发需要读取或写入目录对象的唯一属性的应用程序，则可以通过图形 API 来注册并使用扩展值。例如，如果应用程序需要每个用户的 Skype ID 属性，则可以在目录中注册新属性，然后即可在每个用户对象上获取该属性。有关详细信息，请参阅 [Azure AD 图形 API 目录架构扩展](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-directory-schema-extensions)。

## 方案

图形 API 可实现许多应用程序方案。以下方案最常见：

- **业务线（单租户）应用程序**：在此方案中，一位企业开发人员为一个进行了 Office 365 订阅的组织工作。该开发人员要构建一个 Web 应用程序，该应用程序可与 Azure AD 交互以执行为用户分配许可证等任务。此任务需要访问图形 API，因此该开发人员在 Azure AD 中注册了单租户应用程序，并为图形 API 配置读取和写入权限。然后，该应用程序被配置为使用自己的凭据或当前登录用户的凭据来获取调用图形 API 所需的令牌。

- **“软件即服务”应用程序（多租户）：**在此方案中，独立软件供应商 (ISV) 要开发一个为使用 Azure AD 的其他组织提供用户管理功能的托管多租户 Web 应用程序。这些功能需要访问目录对象，因此该应用程序需要调用图形 API。开发人员在 Azure AD 中注册该应用程序，将它配置为需要对图形 API 的读取和写入权限，然后启用了外部访问，这样其他组织便可以同意在其目录中使用该应用程序。当其他组织中的用户首次向该应用程序进行身份验证时，他们会看到一个同意对话框，该对话框包含应用程序请求的权限。然后，授予许可将为该应用程序提供对用户目录中的图形 API 的请求权限。有关同意框架的详细信息，请参阅[同意框架概述](/documentation/articles/active-directory-integrating-applications/)。

## 另请参阅

[Azure AD 图形 API 快速入门指南](/documentation/articles/active-directory-graph-api-quickstart/)

[AD Graph REST 文档](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog)

[Azure Active Directory 开发人员指南](/documentation/articles/active-directory-developers-guide/)

<!---HONumber=Mooncake_0613_2016-->