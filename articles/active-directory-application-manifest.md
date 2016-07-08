<properties
   pageTitle="了解 Azure Active Directory 应用程序清单 | Azure"
   description="详细介绍如何使用 Azure Active Directory 应用程序清单，该清单表示 Azure AD 租户中的应用程序标识配置，并方便实现 OAuth 授权、许可体验和其他功能。"
   services="active-directory"
   documentationCenter=""
   authors="bryanla"
   manager="mbaldwin"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="03/16/2016"
   wacn.date="06/14/2016"/>

# 了解 Azure Active Directory 应用程序清单

与 Azure Active Directory (AD) 集成的应用程序必须向 Azure AD 租户注册，提供应用程序的持久性标识配置。在运行时查阅此配置，启用允许应用程序通过 Azure AD 外部和代理身份验证/授权的方案。有关 Azure AD 应用程序模型的详细信息，请参阅[添加、更新和删除应用程序][ADD-UPD-RMV-APP]一文。

## 更新应用程序的标识配置

实际上有多个可用的选项可以更新应用程序的标识配置属性，这些选项因功能与难度而有所不同，包括：

- **[Azure 经典管理门户][AZURE-CLASSIC-PORTAL]的 Web 用户界面**可让你更新应用程序的最常见属性。这是更新应用程序属性最快且最不容易出错的方法，但无法像下面两种方法一样提供对所有属性的完全访问权限。
- 对于需要在其中更新 Azure 经典管理门户中未公开的属性更高级方案，可以修改**应用程序清单**。这是本文的重点，将在下一部分中开始详细讨论。
- 你也可**编写使用[图形 API][GRAPH-API] 的应用程序**来更新应用程序，这是最费力的方法。如果要编写管理软件或需要自动定期更新应用程序属性，这可能是个不错的选择。

## 使用应用程序清单更新应用程序的标识配置
使用 [Azure 经典管理门户][AZURE-CLASSIC-PORTAL]，你可以通过下载和上载 JSON 文件表示形式（称为应用程序清单）管理应用程序的标识配置。不会将实际的文件存储在目录中 - 应用程序清单只是 Azure AD 图形 API 应用程序实体上的 HTTP GET 操作，上载是应用程序实体上的 HTTP PATCH 操作。

因此，若要了解应用程序清单的格式和属性，你需要参考图形 API [应用程序实体][APPLICATION-ENTITY]文档。可通过应用程序清单上载执行的更新示例包括：

- 声明 Web API 所公开的权限范围 (oauth2Permissions)。有关使用 oauth2Permissions 委派权限范围实现用户模拟的信息，请参阅[将应用程序与 Azure Active Directory 集成][INTEGRATING-APPLICATIONS-AAD]中的“向其他应用程序公开 Web API”。如前所述，图形 API [Entity and Complex Type reference（实体和复杂类型参考）][APPLICATION-ENTITY]一文中介绍了所有应用程序实体属性，包括属于 [OAuth2Permission][APPLICATION-ENTITY-OAUTH2-PERMISSION] 类型集合的 oauth2Permissions 属性。
- 声明应用公开的应用程序角色 (appRoles)。应用程序实体的 appRoles 属性是 [AppRole][APPLICATION-ENTITY-APP-ROLE] 类型的集合。请参阅[使用 Azure AD 在云应用程序中执行基于角色的访问控制][RBAC-CLOUD-APPS-AZUREAD]一文，以获取实现示例。
- 声明已知的客户端应用程序 (knownClientApplications)，可让你以逻辑方式将指定客户端应用程序的许可绑定到资源/Web API。
- 请求 Azure AD 对登录用户发出组成员资格声明 (groupMembershipClaims)。注意：可配置为额外发出有关用户目录角色成员资格的声明。请参阅[使用 AD 组在云应用程序中执行授权][AAD-GROUPS-FOR-AUTHORIZATION]一文，以获取实现示例。
- 允许应用程序支持 OAuth 2.0 隐式授权流 (oauth2AllowImplicitFlow)。这种类型的授权流可用于嵌入式 JavaScript 网页或单页应用程序 (SPA)。
- 允许使用 X509 证书作为机密密钥 (keyCredentials)。有关实现示例，请参阅文章 [Build service and daemon apps in Office 365（在 Office 365 中构建服务和守护程序应用）][O365-SERVICE-DAEMON-APPS]和 [Developer’s guide to auth with Azure Resource Manager API（使用 Azure Resource Manager API 进行身份验证的开发人员指南）][DEV-GUIDE-TO-AUTH-WITH-ARM]。

使用应用程序清单还能很好地跟踪应用程序注册状态。由于它可以 JSON 格式提供，因此文件表示形式可以签入源代码管理，以及应用程序的源代码。

## 分步示例
现在，让我们逐步了解通过应用程序清单更新应用程序标识配置的步骤。我们将重点演示上述示例之一，其中说明了如何在资源应用程序中声明新的权限范围：

1. 导航到 [Azure 经典管理门户][AZURE-CLASSIC-PORTAL]并使用具有服务管理员或协同管理员权限的帐户登录。


2. 在经过身份验证后，向下滚动并选择左侧导航面板中的 Azure“Active Directory”扩展 (1)，然后单击应用程序注册所在的 Azure AD 租户 (2)。

	![选择 Azure AD 租户][SELECT-AZURE-AD-TENANT]


3. 出现“目录”页时，单击页面顶部的“应用程序”(1) 以查看在租户中注册的应用程序列表。然后找到要在列表中更新的应用程序并单击它 (2)。

	![选择 Azure AD 租户][SELECT-AZURE-AD-APP]


4. 选择应用程序的主页后，注意页面底部的“管理清单”功能 (1)。如果你单击此链接，系统将提示下载或上载 JSON 清单文件。单击“下载清单”(2)，随后即会出现下载确认对话框，提示你单击“下载清单”(3) 进行确认。然后，请在本地打开或保存文件 (4)。

	![管理清单，下载选项][MANAGE-MANIFEST-DOWNLOAD]

	![下载清单][DOWNLOAD-MANIFEST]


5. 在此示例中，我们已在本地保存了文件，因此可以在编辑器中打开该文件，对 JSON 进行更改，然后再次保存。下面是 JSON 结构在 Visual Studio JSON 编辑器中的外观。请注意，它遵循[应用程序实体][APPLICATION-ENTITY]的架构，如前所述：

	![更新清单 JSON][UPDATE-MANIFEST]

    例如，假设要在资源应用程序 (API) 中实现/公开名为“Employees.Read.All”的新权限，你只需将新的/第二个元素添加到 oauth2Permissions 集合，如下所示：

        "oauth2Permissions": [
        {
        "adminConsentDescription": "Allow the application to access MyWebApplication on behalf of the signed-in user.",
        "adminConsentDisplayName": "Access MyWebApplication",
        "id": "aade5b35-ea3e-481c-b38d-cba4c78682a0",
        "isEnabled": true,
        "type": "User",
        "userConsentDescription": "Allow the application to access MyWebApplication on your behalf.",
        "userConsentDisplayName": "Access MyWebApplication",
        "value": "user_impersonation"
        },
        {
        "adminConsentDescription": "Allow the application to have read-only access to all Employee data.",
        "adminConsentDisplayName": "Read-only access to Employee records",
        "id": "2b351394-d7a7-4a84-841e-08a6a17e4cb8",
        "isEnabled": true,
        "type": "User",
        "userConsentDescription": "Allow the application to have read-only access to your Employee data.",
        "userConsentDisplayName": "Read-only access to your Employee records",
        "value": "Employees.Read.All"
        }
        ],

    条目必须唯一，因此你必须为 `"id"` 属性生成新的全局唯一 ID (GUID)。在本例中，由于我们指定了 `"type": "User"`，此权限可由资源/API 应用程序注册所在的 Azure AD 租户所验证的任何帐户同意，授予代表帐户进行访问的客户端应用程序权限。说明和显示名称字符串将在同意期间使用，并显示在 Azure 经典管理门户中。

6. 完成更新清单后，请返回到 Azure 经典管理门户中的 Azure AD 应用程序页，再次单击“管理清单”功能 (1)，但这次请选择“上载清单”选项 (2)。与下载时类似，你将看到另一个对话框，提示你选择 JSON 文件的位置。单击“浏览文件...”(3)，使用“选择要上载的文件”对话框选择 JSON 文件 (4)，然后按下“打开”。对话框消失后，请选择“确定”复选标记 (5)，随即将会上载你的清单。

    ![管理清单，上载选项][MANAGE-MANIFEST-UPLOAD]

    ![上载清单 JSON][UPLOAD-MANIFEST]

    ![上载清单 JSON - 确认][UPLOAD-MANIFEST-CONFIRM]

保存清单后，可以将已注册的客户端应用程序访问权限提供给前面添加的新权限，但这次可以使用 Azure 经典管理门户的 Web UI，而无需编辑客户端应用程序的清单：

1. 首先，转到要添加新 API 的访问权限的客户端应用程序的“配置”页，然后单击“添加应用程序”按钮。
2. 屏幕中会显示租户中已注册的资源应用程序 (API) 的列表。单击资源应用程序名称旁边的加号 (+) 进行选择。  
3. 然后，单击右下角的复选标记。
4. 当你返回到客户端配置页的“添加应用程序”部分时，可在列表中看到新的资源应用程序。如果将光标转到该行右侧的“委派的权限”部分上方，可以看到显示的下拉列表。单击该列表，然后选择新权限，以将其添加到客户端请求的权限列表。注意：此新权限将存储在客户端应用程序标识配置的“requiredResourceAccess”集合属性中。

![针对其他应用程序的权限][PERMS-TO-OTHER-APPS]

![针对其他应用程序的权限][PERMS-SELECT-APP]

![针对其他应用程序的权限][PERMS-SELECT-PERMS]

就这么简单。现在，你的应用程序将使用其新标识配置来运行。

## 后续步骤
欢迎使用以下 DISQUS 意见部分提供反馈，以帮助我们改进与制作内容。

<!--Image references-->
[DOWNLOAD-MANIFEST]: ./media/active-directory-application-manifest/download-manifest.png
[MANAGE-MANIFEST-DOWNLOAD]: ./media/active-directory-application-manifest/manage-manifest-download.png
[MANAGE-MANIFEST-UPLOAD]: ./media/active-directory-application-manifest/manage-manifest-upload.png
[PERMS-SELECT-APP]: ./media/active-directory-application-manifest/portal-perms-select-app.png
[PERMS-SELECT-PERMS]: ./media/active-directory-application-manifest/portal-perms-select-perms.png
[PERMS-TO-OTHER-APPS]: ./media/active-directory-application-manifest/portal-perms-to-other-apps.png
[SELECT-AZURE-AD-APP]: ./media/active-directory-application-manifest/select-azure-ad-application.png
[SELECT-AZURE-AD-TENANT]: ./media/active-directory-application-manifest/select-azure-ad-tenant.png
[UPDATE-MANIFEST]: ./media/active-directory-application-manifest/update-manifest.png
[UPLOAD-MANIFEST]: ./media/active-directory-application-manifest/upload-manifest.png
[UPLOAD-MANIFEST-CONFIRM]: ./media/active-directory-application-manifest/upload-manifest-confirm.png

<!--article references -->
[AAD-GROUPS-FOR-AUTHORIZATION]: http://www.dushyantgill.com/blog/2014/12/10/authorization-cloud-applications-using-ad-groups/

[APPLICATION-ENTITY]: https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#ApplicationEntity
[APPLICATION-ENTITY-APP-ROLE]: https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#AppRoleType
[APPLICATION-ENTITY-OAUTH2-PERMISSION]: https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#OAuth2PermissionType
[AZURE-CLASSIC-PORTAL]: https://manage.windowsazure.cn
[DEV-GUIDE-TO-AUTH-WITH-ARM]: http://www.dushyantgill.com/blog/2015/05/23/developers-guide-to-auth-with-azure-resource-manager-api/
[GRAPH-API]: active-directory-graph-api.md
[INTEGRATING-APPLICATIONS-AAD]: https://azure.microsoft.com/documentation/articles/active-directory-integrating-applications/
[O365-PERM-DETAILS]: https://msdn.microsoft.com/office/office365/HowTo/application-manifest
[O365-SERVICE-DAEMON-APPS]: https://msdn.microsoft.com/office/office365/howto/building-service-apps-in-office-365
[RBAC-CLOUD-APPS-AZUREAD]: http://www.dushyantgill.com/blog/2014/12/10/roles-based-access-control-in-cloud-applications-using-azure-ad/

<!---HONumber=Mooncake_0606_2016-->