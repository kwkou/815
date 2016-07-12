<properties
   pageTitle="Azure AD 图形 API 快速入门 | Aure"
   description="Azure Active Directory 图形 API 通过 OData REST API 终结点提供对 Azure AD 的编程访问权限。应用程序可以使用图形 API 对目录数据和对象执行创建、读取、更新和删除 (CRUD) 操作。"
   services="active-directory"
   documentationCenter="n/a"
   authors="JimacoMS"
   manager="msmbaldwin"
   editor=""
   tags=""/>
   <tags
      ms.service="active-directory"
      ms.date="03/28/2016"
      wacn.date="07/04/2016"/>

# Azure AD 图形 API 快速入门

Azure Active Directory (AD) 图形 API 通过 OData REST API 终结点提供对 Azure AD 的编程访问权限。应用程序可以使用图形 API 对目录数据和对象执行创建、读取、更新和删除 (CRUD) 操作。例如，可以使用图形 API 来创建新用户、查看或更新用户的属性、更改用户的密码、检查基于角色的访问的组成员身份、禁用或删除用户。有关图形 API 功能和应用方案的详细信息，请参阅 [Azure AD 图形 API](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog) 和 [Azure AD 图形 API 先决条件](https://msdn.microsoft.com/library/hh974476(Azure.100).aspx)。

> [AZURE.IMPORTANT] 也可以通过 [Microsoft Graph](https://graph.microsoft.io/) 访问 Azure AD 图形 API 功能。Microsoft Graph 是统一的 API，其中包含 Outlook、OneDrive、OneNote、Planner 和 Office Graph 等其他 Microsoft 服务中的 API，可通过单个终结点和单个访问令牌进行访问。

## 如何构造图形 API URL

在图形 API 中，若要访问你要对其执行 CRUD 操作的目录数据和对象（即资源或实体），可以使用基于开放数据 (OData) 协议的 URL。在图形 API 中使用的 URL 包括四个主要部分：服务根、租户标识符、资源路径和查询字符串选项：`https://graph.windows.net/{tenant-identifier}/{resource-path}?[query-parameters]`。以下面的 URL 为例：`https://graph.windows.net/contoso.com/groups?api-version=1.6`。

- **服务根**：在 Azure AD 图形 API 中，服务根始终是 https://graph.windows.net。
- **租户标识符**：可以是已验证（注册）的域名，在以上示例中为 contoso.com。也可以是租户对象 ID 或者“myorganiztion”或“me”别名。有关详细信息，请参阅[对图形 API 中的实体和操作进行寻址](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-operations-overview)。
- **资源路径**：URL 的此部分标识要交互的资源（用户、组、特定用户或特定组，等等）。 在以上示例中，它是用于对资源集寻址的顶级“组”。你也可以对特定的实体寻址，例如“users/{objectId}”或“users/userPrincipalName”。
- **查询参数**：? 用于分隔资源路径部分与查询参数部分。需要对图形 API 中的所有请求提供“Api 版本”查询参数。图形 API 还支持以下 OData 查询选项：**$filter**、**$orderby**、**$expand**、**$top** 和 **$format**。当前不支持以下查询选项：**$count**、**$inlinecount** 和 **$skip**。有关详细信息，请参阅 [Azure AD 图形 API 支持的查询、筛选和分页选项](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-supported-queries-filters-and-paging-options)。

## 图形 API 版本

可以在 api-version 查询参数中指定图形 API 请求的版本。对于版本 1.5 及更高版本，使用数字版本值：api-version=1.6。对于早期版本，使用遵循 YYYY-MM-DD 格式的日期字符串；例如，api-version=2013-11-08。对于预览功能，可使用字符串“beta”；例如，api-version=beta。有关图形 API 版本之间的差异的详细信息，请参阅 [Azure AD 图形 API 版本控制](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-versioning)。

## 图形 API 元数据

若要返回图形 API 元数据文件，请在 URL 中的租户标识符后面添加“$metadata”段。例如，以下 URL 将返回图形资源管理器使用的演示公司元数据：`https://graph.windows.net/GraphDir1.OnMicrosoft.com/$metadata?api-version=1.6`。可以在 Web 浏览器的地址栏中输入此 URL 来查看元数据。返回的 CSDL 元数据文档描述了实体和复杂类型及其属性，以及你请求的图形 API 版本公开的函数和操作。省略 api-version 参数将返回最新版本的元数据。

## 常见查询

[Azure AD 图形 API 常见查询](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-supported-queries-filters-and-paging-options#CommonQueries)列出了可与 Azure AD Graph 配合使用的常见查询，包括可用于访问目录中的顶层资源的查询，以及用于在目录中执行操作的查询。

例如，`https://graph.windows.net/contoso.com/tenantDetails?api-version=1.6` 将返回目录 contoso.com 的公司信息。

`https://graph.windows.net/contoso.com/users?api-version=1.6` 将列出目录 contoso.com 中的所有用户对象。

## 使用图形资源管理器

生成应用程序时，可以使用 Azure AD 图形 API 的图形资源管理器来查询目录数据。

> [AZURE.IMPORTANT] 图形资源管理器不支持在目录中写入或删除数据。使用图形资源管理器时只能对 Azure AD 目录执行读取操作。

下面是导航到图形资源管理器，选择“使用演示公司”，并输入 `https://graph.windows.net/GraphDir1.OnMicrosoft.com/users?api-version=1.6` 显示演示目录中所有用户时会看到的输出：

![Azure AD 图形 API 资源管理器](./media/active-directory-graph-api-quickstart/graph_explorer.png)

**加载图形资源管理器**：若要加载该工具，请导航到 [https://graphexplorer.cloudapp.net/](https://graphexplorer.cloudapp.net/)。单击“使用演示公司”针对示例租户中的数据运行图形资源管理器。无需凭据即可使用该演示公司。或者，你可以单击“登录”，并使用 Azure AD 帐户凭据登录，以针对租户运行图形资源资源器。如果你针对自己的租户运行图形资源资源器，则你或管理员需要在登录期间表示同意。如果你拥有 Office 365 订阅，则自然而然就拥有了 Azure AD 租户。用于登录 Office 365 的凭据事实上就是 Azure AD 帐户，你可以在图形资源资源器上使用这些凭据。

**运行查询**：若要运行查询，请在请求文本框中键入你的查询，然后单击“获取”或单击 **Enter** 键。结果将显示在响应框中。例如，`https://graph.windows.net/graphdir1.onmicrosoft.com /groups?api-version=1.6` 将列出演示目录中的所有组对象。

请注意，图形资源管理器具有以下功能与限制：
- 对资源集的自动完成功能。若要查看此功能，可单击“使用演示公司”，然后单击请求文本框（公司 URL 出现的位置）。可以从下拉列表中选择资源集。

- 支持“me”和“myorganization”寻址别名。例如，你可以使用 `https://graph.windows.net/me?api-version=1.6` 来返回登录用户的用户对象，或者使用 `https://graph.windows.net/myorganization/users?api-version=1.6` 来返回当前目录中的所有用户。请注意，对演示公司使用“me”别名将返回错误，因为没有任何登录的用户发出请求。

- 响应标头部分。这可以用来帮助排查运行查询时所发生的问题。

- 具有展开和折叠功能的 JSON 响应查看器。

- 不支持显示缩略图照片。

## 使用 Fiddler 写入目录

在本快速入门指南中，你可以使用 Fiddler Web 调试器来练习对 Azure AD 目录执行“写入”操作。有关如何安装 Fiddler 的详细信息，请参阅 [http://www.telerik.com/fiddler](http://www.telerik.com/fiddler)。

在以下示例中，你将使用 Fiddler Web 调试器在 Azure AD 目录中创建一个新的安全组“MyTestGroup”。

**获取访问令牌**：若要访问 Azure AD Graph，客户端需要先成功地向 Azure AD 进行身份验证。有关详细信息，请参阅 [Azure AD 的身份验证方案](/documentation/articles/active-directory-authentication-scenarios/)。

**编写和运行查询**：请完成以下步骤。

1. 打开 Fiddler Web 调试器并切换到“编辑器”选项卡。
2. 由于你要创建新的安全组，因此请从下拉菜单中选择“发布”作为 HTTP 方法。有关对组对象的操作和权限的详细信息，请参阅 [Azure AD Graph REST API 参考](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog)中的[组](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#GroupEntity)。
3. 在“发布”旁边的字段中，键入以下内容作为请求 URL：`https://graph.windows.net/mytenantdomain/groups?api-version=1.6`。

    > [AZURE.NOTE] 必须将 mytenantdomain 替换为你自己的 Azure AD 目录的域名。

4. 在紧靠在“发布”下拉列表下面的字段中，键入以下内容：


Host: graph.windows.net
Authorization: your access token
Content-Type: application/json

 > [AZURE.NOTE] 将 &lt;your access token&gt; 替换为你的 Azure AD 目录的访问令牌。

5. 在“请求正文”字段中键入以下内容：

    
        {
            "displayName":"MyTestGroup",
            "mailNickname":"MyTestGroup",
            "mailEnabled":"false",
            "securityEnabled": true
        }


    有关创建组的详细信息，请参阅[创建组](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/groups-operations#CreateGroup)。

有关 Graph 公开的 Azure AD 实体和类型的详细信息，以及有关可以使用 Graph 对它们执行的操作的信息，请参阅 [Azure AD Graph REST API 参考](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog)。

## 后续步骤

- 了解有关 [Azure AD 图形 API](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog) 的详细信息。
- 了解有关 [Azure AD 图形 API 权限范围](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-permission-scopes)的详细信息

<!---HONumber=Mooncake_0613_2016-->