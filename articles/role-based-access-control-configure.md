<properties
	pageTitle="Microsoft Azure 门户中基于角色的访问控制"
	description="介绍基于角色的访问控制的工作原理以及如何设置它"
	services=""
	documentationCenter=""
	authors="IHenkel"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="multiple"
	ms.date="08/14/2015"
	wacn.date="10/3/2015"/>

# Microsoft Azure 门户中基于角色的访问控制 

我们已在 Microsoft Azure 门户中添加对基于角色的访问控制 (RBAC) 的支持，以帮助组织简单、准确地满足其访问管理要求。[博客文章](http://go.microsoft.com/fwlink/?LinkId=511576)将为你提供该功能的简要介绍并帮助你入门。本主题详细介绍概念并介绍其他用例。


## Azure 中的 RBAC

每个 Azure 订阅都与 Azure Active Directory 相关联。使用 Microsoft Azure 管理门户或 Azure 资源管理器 API 访问订阅资源的用户和服务首先需要使用该 Azure Active Directory 进行身份验证。

![][1]

使用 Azure 基于角色的访问控制，可以通过在订阅、资源组或单个资源级别上为 Azure AD 用户、组和服务分配角色来向它们授予相应访问权限。分配的角色定义用户、组或服务对 Azure 资源的访问级别。

### 角色

角色是可以对 Azure 资源执行的操作的集合。如果为用户或服务分配了包含操作的角色，则允许该用户或服务对 Azure 资源执行该操作。有关内置角色及**其**操作和**不操作**属性的列表，请参阅[内置角色](#builtinroles)。

### 角色分配

通过在 Azure 资源上为 Azure AD 用户和服务分配相应角色，来向这些用户和服务授予访问权限。

#### Azure AD 安全主体

可以将角色分配给以下几种类型的 Azure AD 安全主体：

+ **用户**：可以将角色分配给 Azure AD 中与 Azure 订阅关联的组织用户。还可以通过在 Azure 门户中使用邀请操作为外部 Microsoft 帐户用户（例如 joe@outlook.com）分配角色为这些用户分配角色。为外部 Microsoft 帐户用户分配角色会导致在 Azure AD 中为其创建来宾帐户。如果在目录中禁用此来宾帐户，则不会允许外部用户访问已向该用户授予访问权限的任何 Azure 资源。
+ **组**：可以将角色分配给 Azure AD 安全组。如果用户成为具有访问权限的组的成员，则将自动向该用户授予对资源的访问权限。从组中删除该用户后，该用户也会自动失去对资源的访问权限。通过为组分配角色并将用户添加到这些组来通过组管理访问权限是最佳做法，而不是直接向用户分配角色。Azure RBAC 不允许将角色分配给分发列表。能够将角色分配给组，允许组织将其现有访问控制模型从其本地目录扩展到云中，因此可以重用已建立用于控制本地访问权限的安全组来控制对 Azure 门户中的资源的访问权限。有关用于从本地目录同步用户和组的不同选项的详细信息，请参阅[目录集成](http://technet.microsoft.com/zh-cn/library/jj573653.aspx)。Azure AD 高级版还提供了[委派组管理功能](http://msdn.microsoft.com/zh-cn/library/azure/dn641267.aspx)，使用该功能，可以将创建和管理组的功能委派给 Azure AD 中的非管理员用户。
+ **服务主体**：服务标识表示为目录中的服务主体。它们使用 Azure AD 进行身份验证，并安全地彼此通信。可以通过 Windows PowerShell 的 Azure 模块将角色分配给表示服务的 Azure AD 服务主体，来向这些服务授予对 Azure 资源的访问权限。

#### 资源作用域

不需要向整个订阅授予访问权限。此外，还可以为资源组以及单个资源分配角色。在 Azure RBAC 中，资源将从其父资源继承角色分配。因此，如果只向用户、组或服务授予对某个订阅中的一个资源组的访问权限，则他们将只能访问该资源组及该资源组中的资源，而不能访问该订阅中的其他资源组。另一个示例是，可以将安全组添加到某个资源组的读者角色中，但却添加到该资源组内数据库的参与者角色中。

![][2]

## RBAC 与订阅共同管理员共存

订阅管理员和共同管理员将继续对 Azure 门户和管理 API 具有完全访问权限。在 RBAC 模型中，在订阅级别为他们分配所有者角色。但是，Azure 门户和 Azure 资源管理器 API 只支持新的 RBAC 模型。分配了 RBAC 角色的用户和服务不能访问 Azure 管理门户和服务管理 API。将用户添加到 Azure 门户中某个订阅的所有者角色并不会使该用户成为整个 Azure 门户中订阅的共同管理员。

如果要授予用户对尚未可通过 Azure 门户管理的 Azure 资源的访问权限，应使用 Azure 管理门户将这些用户添加到订阅共同管理员中。服务总线和 StorSimple 是目前不能使用 RBAC 进行管理的资源示例。

## 管理授权与数据操作

只有 Azure 门户中的 Azure 资源的管理操作和 Azure 资源管理器 API 支持基于角色的访问控制。并不是 Azure 资源的所有数据级别操作都可通过 RBAC 授权。例如，存储帐户的创建/读取/更新/删除可通过 RBAC 进行控制，但存储帐户中的 Blob 或表的创建/读取/更新/删除尚不能通过 RBAC 进行控制。同样，SQL DB 的创建/读取/更新/删除可通过 RBAC 进行控制，但数据库中的 SQL 表的创建/读取/更新/删除尚不能通过 RBAC 进行控制。

## 如何添加和删除访问权限

让我们来看一下组织中的资源所有者如何管理访问权限的示例。在此方案中，有多个人员在使用 Azure 资源生成的各种测试和生产项目中工作。你需要按照最佳做法来授予访问权限。用户应对所需的所有资源具有访问权限，但没有其他访问权限。你要重用在进程和工具中所做的所有投资，以使用本地 Active Directory 中管理的安全组。以下部分介绍如何设置对这些资源的访问权限：

* [添加访问权限](#add-access)
* [删除访问权限](#remove-access)
* [添加或删除外部用户的访问权限](#add-or-remove-access-for-external-user)

### 添加访问权限

以下是访问权限要求以及如何在 Azure 中设置它们的摘要。

用户/组 | 访问权限要求 | 访问角色和作用域
------------- | -------------  | ------------
Jill Santos 的所有团队 | 读取所有 Azure 资源 | 将表示 Jill Santos 团队的 AD 组添加到 Azure 订阅的读者角色
Jill Santos 的所有团队 | 在 Test 资源组中创建和管理所有资源 | 将表示 Jill Santos 团队的 AD 组添加到 Test 资源组的参与者角色
Brock | 在 Prod 资源组中创建和管理所有资源 | 将 Brock 添加到 Prod 资源组的参与者角色


首先，让我们为订阅的所有资源添加读取访问权限。单击**“浏览”>“全部内容”>“订阅”**。

![][3]

单击*订阅的名称* ** >“读者”>“添加”**。从用户和组的列表中，选择或键入 Active Directory 组的名称。

![][4]

然后将同一个团队添加到 Test 资源组的参与者角色中。单击资源组以打开其属性边栏选项卡。在**“角色”**下，单击**“参与者”>“添加”**并键入团队的名称。

![][5]

若要将 Brock 添加到 Prod 资源组的参与者角色，请单击该资源组，单击**“参与者”>“添加”**并键入 Brock 的名称。

![][6]

也可以通过使用 Windows PowerShell 的 Microsoft Azure 模块来管理角色分配。以下是通过使用 New-AzureRoleAssignment cmdlet（而不是门户）添加 Brock 帐户的示例：

	PS C:\> New-AzureRoleAssignment -Mail brockh@contoso.com -RoleDefinitionName Contributor -ResourceGroupName ProdDB

有关使用 Windows PowerShell 来添加和删除访问权限的详细信息，请参阅<!--[-->使用 Windows PowerShell 管理基于角色的访问控制<!--](/documentation/articles/role-based-access-control-powershell)-->。

### 删除访问权限

你还可以轻松地删除分配。让我们假设你要从名为 TestDB 的资源组的读者角色中删除名为 Brad Adams 的用户。打开资源组边栏选项卡，单击**“读者”> Brad Adams >“删除”**。

![][7]

以下是如何通过使用 Remove-AzureRoleAssignment cmdlet 删除 Brad Adams 的示例：

	PS C:\> Remove-AzureRoleAssignment -Mail badams@contoso.com -RoleDefinitionName Reader -ResourceGroupName TestDB

### 添加或删除外部用户的访问权限

目录的**“配置”**选项卡包含用于控制外部用户访问权限的选项。只能在完全版 Azure 门户 UI 中由目录全局管理员更改这些选项（不支持 Windows PowerShell 或 API 方法）。若要在 Azure 门户中打开**“配置”**选项卡，请单击**“Active Directory”**，然后单击目录的名称。

![][10]

然后，便可以编辑用于控制外部用户访问权限的选项。

![][8]

默认情况下，来宾不能枚举目录的内容，因此他们在**“成员列表”**中看不到任何用户或组。他们可以通过键入用户的完整电子邮件地址来搜索用户，然后授予访问权限。针对来宾的默认限制组合为：

- 他们不能枚举目录中的用户和组。
- 如果他们知道用户的电子邮件地址，则可以查看该用户的有限详细信息。
- 如果他们知道组名，则可以查看该组的有限详细信息。

由于来宾可以查看有限的用户或组详细信息，因此可以邀请其他人，以及查看与他们协作的人员的某些详细信息。

让我们逐步完成为外部用户添加访问权限的过程。我们会将外部用户添加到 TestDB 资源组的同一个读者角色中，因此该用户可以帮助调试错误。打开资源组边栏选项卡，单击**“读者”>“添加”>“邀请”**，并键入要添加的用户的电子邮件地址。

![][9]

添加外部用户时，将在目录中创建来宾。此后，可以将该来宾添加到组中或将其从组中删除，也可以单独将该来宾添加到角色中或从角色中删除，就像对任何其他目录用户所做的那样。

你还可以从任何角色中删除来宾，就像删除任何用户一样。从资源的角色中删除来宾并不会从目录中删除该来宾。

## 如何跟踪对角色分配的更改

对角色分配的更改记录在[审核日志](http://azure.microsoft.com/updates/audit-logs-in-azure-preview-portal/)中，与其他事件类似。对角色分配更改所做的记录可以使用 [Azure PowerShell](https://msdn.microsoft.com/zh-cn/library/azure/jj156055.aspx) 或 [Azure 资源管理器 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn931927.aspx) 进行检索。

例如，若要针对整个订阅检索角色分配更改列表，请使用 Azure PowerShell 运行以下两个 cmdlet。第一个切换到 Azure 资源管理器模式。

`Switch-AzureMode -name AzureResourceManager`

`Get-AzureSubscriptionIdLog –DetailedOutput -StartTime '06-15-15' -EndTime '06-29-15'`

角色分配更改可在 ResourceProviderName 为 `Microsoft.Authorization` 的事件中捕获。在事件详细信息中捕获的实际分配详情：已分配什么主体，分配给何种角色、针对什么作用域进行了分配。在门户中浏览审核日志时，会显示角色分配更改情况，但该门户不显示事件详细信息。若要查看事件详细信息，必须使用 Azure PowerShell。

###事件详细信息

下面是进行某项角色分配更改时，事件详细信息的示例：

```
Authorization        :
                       Scope     : /subscriptions/ff945b8d-441a-41ef-a9db-7bd5fcc99978/resourceGroups/Default-Storage-W
                       estUS/providers/Microsoft.ClassicStorage/storageAccounts/authzwaes/providers/Microsoft.Authoriza
                       tion/roleAssignments/531f036a-37ff-40c1-9bb9-aa580ebe7e78
                       Action    : Microsoft.Authorization/roleAssignments/write
                       Role      : Subscription Admin
                       Condition :
Caller               : William.Hennum@contoso.com
Claims               :
                       aud            : https://management.core.windows.net/
                       iss            : https://sts.windows.net/72f988bf-86f1-41af-91ab-2d7cd011db47/
                       iat            : 1435333533
                       nbf            : 1435333533
                       exp            : 1435337433
                       ver            : 1.0
                       http://schemas.microsoft.com/identity/claims/tenantid: 72f988bf-86f1-41af-91ab-2d7cd011db47
                       http://schemas.microsoft.com/identity/claims/objectidentifier:
                       dda50086-5e3d-4a4b-b8bc-f54771104d89
                       http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn: William.Hennum@contoso.com
                       puid           : 10030000803CDC0B
                       http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier:
                       MJwntjqWaULfl30NJMiDRVSVCWMX5GzmMNU4oqitDXs
                       http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname: William
                       http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname: Hennum
                       name           : William Hennum
                       http://schemas.microsoft.com/claims/authnmethodsreferences: rsa,wia,mfa
                       _claim_names   : {"groups":"src1"}
                       _claim_sources : {"src1":{"endpoint":"https://graph.windows.net/72f988bf-86f1-41af-91ab-2d7cd011
                       db47/users/dda50086-5e3d-4a4b-b8bc-f54771104d89/getMemberObjects"}}
                       http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name: William.Hennum@contoso.com
                       onprem_sid     : S-1-5-21-1721254763-462695806-1538882281-3175325
                       appid          : c44b4083-3bb0-49c1-b47d-974e53cbdf3c
                       appidacr       : 2
                       http://schemas.microsoft.com/identity/claims/scope: user_impersonation
                       http://schemas.microsoft.com/claims/authnclassreference: 1
CorrelationId        : d724ffd0-31a4-4564-941b-f3a5d32ad8a4
Description          :
EventChannels        : Operation
EventDataId          : ed8e79b6-c7d1-4332-adcf-70d37546c5a6
EventName            : BeginRequest
EventSource          : Administrative
EventTimestamp       : 6/26/2015 3:53:34 PM
HttpRequest          :
                       ClientId        : F7272386-295A-4545-96BD-21F0856A43FE
                       Method          : PUT
                       Url             :
                       ClientIpAddress : 23.99.81.159
Id                   : /subscriptions/ff945b8d-441a-41ef-a9db-7bd5fcc99978/resourceGroups/Default-Storage-WestUS/provid
                       ers/Microsoft.ClassicStorage/storageAccounts/authzwaes/providers/Microsoft.Authorization/roleAss
                       ignments/531f036a-37ff-40c1-9bb9-aa580ebe7e78/events/ed8e79b6-c7d1-4332-adcf-70d37546c5a6/ticks/
                       635709308140011864
Level                : Informational
OperationId          : d724ffd0-31a4-4564-941b-f3a5d32ad8a4
OperationName        : Microsoft.Authorization/roleAssignments/write
Properties           :
                       requestbody    : {"Id":"531f036a-37ff-40c1-9bb9-aa580ebe7e78","Properties":{"PrincipalId":"dda50
                       086-5e3d-4a4b-b8bc-f54771104d89","RoleDefinitionId":"/subscriptions/ff945b8d-441a-41ef-a9db-7bd5
                       fcc99978/providers/Microsoft.Authorization/roleDefinitions/acdd72a7-3385-48ef-bd42-f606fba81ae7"
                       ,"Scope":"/subscriptions/ff945b8d-441a-41ef-a9db-7bd5fcc99978/resourceGroups/Default-Storage-Wes
                       tUS/providers/Microsoft.ClassicStorage/storageAccounts/authzwaes"}}
ResourceGroupName    : Default-Storage-WestUS
ResourceProviderName : Microsoft.Authorization
ResourceId           : /subscriptions/ff945b8d-441a-41ef-a9db-7bd5fcc99978/resourceGroups/Default-Storage-WestUS/provid
                       ers/Microsoft.ClassicStorage/storageAccounts/authzwaes/providers/Microsoft.Authorization/roleAss
                       ignments/531f036a-37ff-40c1-9bb9-aa580ebe7e78
Status               : Started
SubmissionTimestamp  : 6/26/2015 3:53:50 PM
SubscriptionId       : ff945b8d-441a-41ef-a9db-7bd5fcc99978
SubStatus            :`
```

该事件中的信息解释如下：

| 字段 | 值 | 详细信息 |
| --- | --- | --- |
| 调用方 |	`William.Hennum@contoso.com` | 进行角色分配的主体。主体可能是用户、组或服务主体。
| HttpRequest：方法 | `PUT` | 执行的操作。使用 PUT 来进行分配，使用 DELETE 来删除分配。 |
| 属性：PrincipalId | `dda50086-5e3d-4a4b-b8bc-f54771104d89` | 	已分配给该角色的主体的对象 ID。主体可能是用户、组或服务主体。你可以确定主体的名称和类型，只需在 Azure Active Directory 中使用 Azure PowerShell 进行查看即可。 |
| 属性：RoleDefinitionId |	`/subscriptions/ff945b8d-441a-41ef-a9db-7bd5fcc99978/providers/Microsoft.Authorization/roleDefinitions/acdd72a7-3385-48ef-bd42-f606fba81ae7	` | 已分配的角色。你可以使用 Azure PowerShell 确定角色的显示名称。 |
| 属性：Scope | `/subscriptions/ff945b8d-441a-41ef-a9db-7bd5fcc99978/resourceGroups/Default-Storage-WestUS/providers/Microsoft.ClassicStorage/storageAccounts/authzwaes` |	在其上进行了角色分配的资源。这可能是资源、资源组或订阅。 |

###示例 PowerShell 代码段

首先显示的一些示例性 Azure PowerShell 代码用于将 PrincipalId 映射到名称和类型：

```
# Sample - how to resolve a principal
function Get-PrincipalDetails($principalId)
{
    $principalDetails = "" | select Name, Type
    $user = Get-AzureADUser -ObjectId $principalId
    if ($user) {
        $principalDetails.Name = $user.DisplayName
        $principalDetails.Type = "User"
    } else {
        $group = Get-AzureADGroup -ObjectId $principalId
        if ($group) {
            $principalDetails.Name = $group.DisplayName
            $principalDetails.Type = "Group"
        } else {
            $servicePrincipal = Get-AZureADServicePrincipal -objectId $principalId
            if ($servicePrincipal) {
                $principalDetails.Name = $servicePrincipal.DisplayName
                $principalDetails.Type = "Service Principal"
            }
        }
    }

    $principalDetails
}
```

接下来显示的一些示例性 Azure PowerShell 代码用于将 Scope 映射到资源名称和类型：

```
# Sample - how to resolve a resource
function Get-ResourceDetails($resourceId)
{
    $resourceDetails = "" | select Name, Type
    $resource = Get-AzureResource -Id $resourceId -OutputObjectFormat New
    if ($resource) {
        if ($resource.ResourceName) {
            $resourceDetails.Name = $resource.ResourceName
            $resourceDetails.Type = "Resource"
        } elseif ($resource.ResourceGroupName) {
            $resourceDetails.Name = $resource.ResourceGroupName
            $resourceDetails.Type = "Resource Group"
        } elseif ($resource.SubscriptionId) {
            $resourceDetails.Name = $resource.SubscriptionId
            $resourceDetails.Type = "Subscription"
        }
    }
    $resourceDetails
}
```
接下来显示的一些示例性 Azure PowerShell 代码将 RoleDefinitionId 映射到角色的显示名称：

```

# Get the name of a role
function Get-AzureRoleDefinitionName($roleDefinitionId)
{
    if (!$Global:_azureRoleDefinitionCache) {
        $Global:_azureRoleDefinitionCache = @{}
        Get-AzureRoleDefinition | % { $Global:_azureRoleDefinitionCache[$_.Id] = $_; }
    }

    if ($Global:_azureRoleDefinitionCache[$roleDefinitionId]) {
        return $Global:_azureRoleDefinitionCache[$roleDefinitionId].Name
    } else {
        return ""
    }
}

```

###示例性 Azure PowerShell 脚本

下面是一个综合性的示例脚本，用于检索特定日期范围的角色分配事件，然后将这些事件作为表输出：

```
# Sample - how to resolve a principal
function Get-PrincipalDetails($principalId)
{
    $principalDetails = "" | select Name, Type
    $user = Get-AzureADUser -ObjectId $principalId
    if ($user) {
        $principalDetails.Name = $user.DisplayName
        $principalDetails.Type = "User"
    } else {
        $group = Get-AzureADGroup -ObjectId $principalId
        if ($group) {
            $principalDetails.Name = $group.DisplayName
            $principalDetails.Type = "Group"
        } else {
            $servicePrincipal = Get-AZureADServicePrincipal -objectId $principalId
            if ($servicePrincipal) {
                $principalDetails.Name = $servicePrincipal.DisplayName
                $principalDetails.Type = "Service Principal"
            }
        }
    }

    $principalDetails
}
# Sample - how to resolve a resource
function Get-ResourceDetails($resourceId)
{
    $resourceDetails = "" | select Name, Type
    $resource = Get-AzureResource -Id $resourceId -OutputObjectFormat New
    if ($resource) {
        if ($resource.ResourceName) {
            $resourceDetails.Name = $resource.ResourceName
            $resourceDetails.Type = "Resource"
        } elseif ($resource.ResourceGroupName) {
            $resourceDetails.Name = $resource.ResourceGroupName
            $resourceDetails.Type = "Resource Group"
        } elseif ($resource.SubscriptionId) {
            $resourceDetails.Name = $resource.SubscriptionId
            $resourceDetails.Type = "Subscription"
        }
    }
    $resourceDetails
}
# Get the name of a role
function Get-AzureRoleDefinitionName($roleDefinitionId)
{
    if (!$Global:_azureRoleDefinitionCache) {
        $Global:_azureRoleDefinitionCache = @{}
        Get-AzureRoleDefinition | % { $Global:_azureRoleDefinitionCache[$_.Id] = $_; }
    }

    if ($Global:_azureRoleDefinitionCache[$roleDefinitionId]) {
        return $Global:_azureRoleDefinitionCache[$roleDefinitionId].Name
    } else {
        return ""
    }
}
# Sample - output the list of role assignment events
function Get-AzureRBACAuditLog($startDateTime, $endDateTime)
{
    $log = Get-AzureSubscriptionIdLog -DetailedOutput -StartTime $startDateTime -EndTime $endDateTime
    $log = $log | ? { $_.ResourceProviderName -ieq "Microsoft.Authorization" }
    $startEvents = $log | ? { $_.httpRequest -and $_.Status -ieq "Started" }
    $endEvents = @{}
    $log | ? { $_.httpRequest -and $_.Status -ne "Started" } | % { $endEvents[$_.OperationId] = $_ }

    $startEvents | ? { $endEvents.ContainsKey($_.OperationId) } | % {
        $endEvent = $endEvents[$_.OperationId];
        $out = "" | select Timestamp, Caller, Action, PrincipalId, PrincipalName, PrincipalType, RoleName, Scope, ScopeName, ScopeType, RoleDefinitionId
        $out.Timestamp = $endEvent.EventTimestamp
        $out.Caller = $_.Caller
        if ($_.HttpRequest.Method -ieq "PUT") {
            $out.Action = "Granted"
            if ($_.Properties.Content.ContainsKey("requestbody")) {
                $messageBody = ConvertFrom-Json $_.Properties.Content["requestbody"]
            }
        }
        elseif ($_.HttpRequest.Method -ieq "DELETE") {
            $out.Action = "Revoked"
            if ($endEvent.Properties.Content.ContainsKey("responseBody")) {
                $messageBody = ConvertFrom-Json $endEvent.Properties.Content["responseBody"]
            }
        }

        if ($messageBody) {
            $out.PrincipalId = $messageBody.properties.principalId
            $pd = Get-PrincipalDetails $out.PrincipalId
            $out.PrincipalName = $pd.Name
            $out.PrincipalType = $pd.Type
            $out.RoleName = (Get-AzureRoleDefinitionName $messageBody.properties.roleDefinitionId)
            $out.Scope = $messageBody.properties.Scope
            $rd = Get-ResourceDetails $out.Scope
            $out.ScopeName = $rd.Name
            $out.ScopeType = $rd.Type
            $out.RoleDefinitionId = $messageBody.properties.roleDefinitionId
        }

        $out
    }
}

```

以下是用于运行该脚本的命令

```
$log = Get-AzureRBACAuditLog '2015-06-26' '2015-06-27'

$log | Format-Table
```

## 使用基于角色的访问控制时的已知问题

如果你在使用基于角色的访问控制功能时遇到问题，请参阅<!--[-->故障排除基于角色的访问控制<!--](/documentation/articles/role-based-access-control-troubleshooting)-->以了解任何可能与该问题相关的已知问题。


## 内置角色

Azure 基于角色的访问控制附带了以下内置角色，可以将这些角色分配给用户、组和服务。你不能修改内置角色的定义。在即将发布的 Azure RBAC 版本中，你将能够基于可在 Azure 资源上执行的可用操作列表编写一组操作来定义自定义角色。

单击相应的链接可查看角色定义的**操作**和**不操作**属性。**操作**属性指定对 Azure 资源允许的操作。操作字符串可以使用通配符。角色定义的**不操作**属性指定必须从允许的操作中排除的操作。


角色名称 | 说明
------------- | -------------  
[API 管理服务参与者](#api-management-service-contributor) | 允许你管理 API 管理服务，但不能访问它们。
[Application Insights 组件参与者](#application-insights-component-contributor) | 允许你管理 Application Insights 组件，但不能访问它们。
[BizTalk 参与者](#biztalk-contributor) | 允许你管理 BizTalk 服务，但不能访问它们。
[ClearDB MySQL DB 参与者](#cleardb-mysql-db-contributor) | 允许你管理 ClearDB MySQL 数据库，但不能访问它们。
[参与者](#contributor) | 参与者可以管理访问权限以外的所有内容。
[数据工厂参与者](#data-factory-contributor) | 允许你管理数据工厂，但不能访问它们。
[Document DB 帐户参与者](#document-db-account-contributor) | 允许你管理 DocumentDB 帐户，但不能访问它们。
[Intelligent Systems 帐户参与者](#intelligent-systems-account-contributor) | 允许你管理 Intelligent Systems 帐户，但不能访问它们。
[NewRelic APM 帐户参与者](#newrelic-apm-account-contributor) | 允许你管理 New Relic 应用程序性能管理帐户和应用程序，但不能访问它们。
[所有者](#owner) | 所有者可以管理所有内容（包括访问权限）。
[读者](#reader) | 读者可以查看所有内容，但不能进行更改。
[Redis Cache 参与者](#redis-cache-contributor) | 允许你管理 Redis Cache，但不能访问它们。
[SQL DB 参与者](#sql-db-contributor) | 允许你管理 SQL 数据库，但不能访问它们。此外，你也不能管理其与安全性相关的策略或其父 SQL 服务器。
[SQL 安全管理器](#sql-security-manager) | 允许你管理 SQL 服务器和数据库与安全性相关的策略，但不能访问它们。
[SQL Server 参与者](#sql-server-contributor) | 允许你管理 SQL 服务器和数据库，但不能访问它们，也不能访问与其安全性相关的策略。
[计划程序作业集合参与者](#scheduler-job-collections-contributor) | 允许你管理计划程序作业集合，但不能访问它们。
[搜索服务参与者](#search-service-contributor) | 允许你管理搜索服务，但不能访问它们。
[存储帐户参与者](#storage-account-contributor) | 允许你管理存储帐户，但不能访问它们。
[用户访问管理员](#user-access-administrator) | 允许你管理用户对 Azure 资源的访问权限。
[虚拟机参与者](#virtual-machine-contributor) | 允许你管理虚拟机，但不能访问它们，也不能访问虚拟机所连接到的虚拟网络或存储帐户。
[虚拟网络参与者](#virtual-network-contributor) | 允许你管理虚拟网络，但不能访问它们。
[Web 计划参与者](#web-plan-contributor) | 允许你管理网站的 Web 计划，但不能访问它们。
[网站参与者](#website-contributor) | 允许你管理网站（而非 Web 计划），但不能访问它们。


### API 管理服务参与者

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>Microsoft.ApiManagement/Services/*</td>
<td>创建和管理 API 管理服务</td>
</tr>
<tr>
<td>Microsoft.Authorization/*/read</td>
<td>读取角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/read</td>
<td>读取资源组</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/deployments/*</td>
<td>创建和管理资源组部署</td>
</tr>
<tr>
<td>Microsoft.Insights/alertRules/*</td>
<td>创建和管理警报规则</td>
</tr>
<tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
</table>

### Application Insights 组件参与者

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>Microsoft.Insights/components/*</td>
<td>创建和管理 Insights 组件</td>
</tr>
<tr>
<td>Microsoft.Insights/webtests/*</td>
<td>创建和管理 Web 测试</td>
</tr>
<tr>
<td>Microsoft.Authorization/*/read</td>
<td>读取角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/read</td>
<td>读取资源组</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/deployments/*</td>
<td>创建和管理资源组部署</td>
</tr>
<tr>
<td>Microsoft.Insights/alertRules/*</td>
<td>创建和管理警报规则</td>
</tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
</table>

### BizTalk 参与者

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>Microsoft.BizTalkServices/BizTalk/*</td>
<td>创建和管理 BizTalk 服务</td>
</tr>
<tr>
<td>Microsoft.Authorization/*/read</td>
<td>读取角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/read</td>
<td>读取资源组</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/deployments/*</td>
<td>创建和管理资源组部署</td>
</tr>
<tr>
<td>Microsoft.Insights/alertRules/*</td>
<td>创建和管理警报规则</td>
</tr>
<tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
</table>

### ClearDB MySQL DB 参与者

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>successbricks.cleardb/databases/*</td>
<td>创建和管理 ClearDB MySQL 数据库</td>
</tr>
<tr>
<td>Microsoft.Authorization/*/read</td>
<td>读取角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/read</td>
<td>读取资源组</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/deployments/*</td>
<td>创建和管理资源组部署</td>
</tr>
<tr>
<td>Microsoft.Insights/alertRules/*</td>
<td>创建和管理警报规则</td>
</tr>
<tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
</table>

### 参与者

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>*</td>
<td>创建和管理所有类型的资源</td>
</tr>
<tr>
<th colspan="2">不操作</th>
</tr>
<tr>
<td>Microsoft.Authorization/*/Write</td>
<td>无法创建角色和角色分配 </td>
</tr>
<tr>
<td>Microsoft.Authorization/*/Delete</td>
<td>无法删除角色和角色分配</td>
</tr>
</table>

### 数据工厂参与者

<table style=width:100%">
<tr>
<td>Microsoft.DataFactory/dataFactories/*</td>
<td>创建和管理数据工厂</td>
</tr>
<tr>
<td>Microsoft.Authorization/*/read</td>
<td>读取角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/read</td>
<td>读取资源组</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/deployments/*</td>
<td>创建和管理资源组部署</td>
</tr>
<tr>
<td>Microsoft.Insights/alertRules/*</td>
<td>创建和管理警报规则</td>
</tr>
<tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
</table>

### Document DB 帐户参与者

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>Microsoft.DocumentDb/databaseAccounts/*</td>
<td>创建和管理 DocumentDB 帐户</td>
</tr>
<tr>
<td>Microsoft.Authorization/*/read</td>
<td>读取角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/read</td>
<td>读取资源组</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/deployments/*</td>
<td>创建和管理资源组部署</td>
</tr>
<tr>
<td>Microsoft.Insights/alertRules/*</td>
<td>创建和管理警报规则</td>
</tr>
<tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
</table>

### Intelligent Systems 帐户参与者

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>Microsoft.IntelligentSystems/accounts/*</td>
<td>创建和管理 Intelligent Systems 帐户</td>
</tr>
<tr>
<td>Microsoft.Authorization/*/read</td>
<td>读取角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/read</td>
<td>读取资源组</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/deployments/*</td>
<td>创建和管理资源组部署</td>
</tr>
<tr>
<td>Microsoft.Insights/alertRules/*</td>
<td>创建和管理警报规则</td>
</tr>
<tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
</table>

### NewRelic APM 帐户参与者

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>NewRelic.APM/accounts/*</td>
<td>创建和管理 NewRelic 应用程序性能管理帐户</td>
</tr>
<tr>
<td>Microsoft.Authorization/*/read</td>
<td>读取角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/read</td>
<td>读取资源组</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/deployments/*</td>
<td>创建和管理资源组部署</td>
</tr>
<tr>
<td>Microsoft.Insights/alertRules/*</td>
<td>创建和管理警报规则</td>
</tr>
<tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
</table>

### 所有者

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>*</td>
<td>创建和管理所有类型的资源</td>
</tr>
</table>

### 读取器

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>*/read</td>
<td>读取所有类型的资源。但是，无法读取机密信息。</td>
</tr>
</table>

### Redis Cache 参与者

<table style=width:100%">
<tr>
<td>Microsoft.Cache/redis/*</td>
<td>创建和管理 Redis Cache</td>
</tr>
<tr>
<td>Microsoft.Authorization/*/read</td>
<td>读取角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/read</td>
<td>读取资源组</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/deployments/*</td>
<td>创建和管理资源组部署</td>
</tr>
<tr>
<td>Microsoft.Insights/alertRules/*</td>
<td>创建和管理警报规则</td>
</tr>
<tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
</table>

### SQL DB 参与者

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>Microsoft.Sql/servers/read</td>
<td>读取 SQL Server</td>
</tr>
<tr>
<td>Microsoft.Sql/servers/databases/*</td>
<td>创建和管理 SQL 数据库</td>
</tr>
<tr>
<td>Microsoft.Authorization/*/read</td>
<td>读取角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resources/read</td>
<td>读取订阅资源</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/read</td>
<td>读取资源组</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/resources/read</td>
<td>读取资源组资源</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/deployments/*</td>
<td>创建和管理资源组部署</td>
</tr>
<tr>
<td>Microsoft.Insights/alertRules/*</td>
<td>创建和管理警报规则</td>
</tr>
<tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
<tr>
<th colspan="2">不操作</th>
</tr>
<tr>
<td>Microsoft.Sql/servers/databases/auditingPolicies/*</td>
<td>无法管理 SQL 数据库审核策略</td>
</tr>
<tr>
<td>Microsoft.Sql/servers/databases/connectionPolicies/*</td>
<td>无法管理 SQL 数据库连接策略</td>
</tr>
<tr>
<td>Microsoft.Sql/servers/databases/dataMaskingPolicies/*</td>
<td>无法管理 SQL 数据库数据屏蔽策略</td>
</tr>
<tr>
<td>Microsoft.Sql/servers/databases/securityMetrics/*</td>
<td>无法管理 SQL 数据库安全指标</td>
</tr>
</table>

### SQL 安全管理器

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>Microsoft.Sql/servers/read</td>
<td>读取 SQL Server</td>
</tr>
<tr>
<td>Microsoft.Sql/servers/auditingPolicies/*</td>
<td>创建和管理 SQL Server 审核策略</td>
</tr>
<tr>
<td>Microsoft.Sql/servers/databases/read</td>
<td>读取 SQL 数据库</td>
</tr>
<tr>
<td>Microsoft.Sql/servers/databases/auditingPolicies/*</td>
<td>创建和管理 SQL 数据库审核策略</td>
</tr>
<tr>
<td>Microsoft.Sql/servers/databases/connectionPolicies/*</td>
<td>创建和管理 SQL 数据库连接策略</td>
</tr>
<tr>
<td>Microsoft.Sql/servers/databases/dataMaskingPolicies/*</td>
<td>创建和管理 SQL 数据库数据屏蔽策略</td>
</tr>
<tr>
<td>Microsoft.Sql/servers/databases/securityMetrics/*</td>
<td>创建和管理 SQL 数据库安全指标</td>
</tr>
<tr>
<td>Microsoft.Authorization/*/read</td>
<td>读取角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/read</td>
<td>读取资源组</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/deployments/*</td>
<td>创建和管理资源组部署</td>
</tr>
<tr>
<td>Microsoft.Insights/alertRules/*</td>
<td>创建和管理警报规则</td>
</tr>
<tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
</table>

### SQL Server 参与者

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>Microsoft.Sql/servers/*</td>
<td>创建和管理 SQL Server</td>
</tr>
<tr>
<td>Microsoft.Authorization/*/read</td>
<td>读取角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resources/read</td>
<td>读取订阅资源</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/read</td>
<td>读取资源组</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/resources/read</td>
<td>读取资源组资源</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/deployments/*</td>
<td>创建和管理资源组部署</td>
</tr>
<tr>
<td>Microsoft.Insights/alertRules/*</td>
<td>创建和管理警报规则</td>
</tr>
<tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
<tr>
<th colspan="2">不操作</th>
</tr>
<tr>
<td>Microsoft.Sql/servers/auditingPolicies/*</td>
<td>无法管理 SQL Server 审核策略</td>
</tr>
<tr>
<td>Microsoft.Sql/servers/databases/auditingPolicies/*</td>
<td>无法管理 SQL 数据库审核策略</td>
</tr>
<tr>
<td>Microsoft.Sql/servers/databases/connectionPolicies/*</td>
<td>无法管理 SQL 数据库连接策略</td>
</tr>
<tr>
<td>Microsoft.Sql/servers/databases/dataMaskingPolicies/*</td>
<td>无法管理 SQL 数据库数据屏蔽策略</td>
</tr>
<tr>
<td>Microsoft.Sql/servers/databases/securityMetrics/*</td>
<td>无法管理 SQL 数据库安全指标</td>
</tr>
</table>

### 计划程序作业集合参与者

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>Microsoft.Scheduler/jobcollections/*</td>
<td>创建和管理计划程序作业集合</td>
</tr>
<tr>
<td>Microsoft.Authorization/*/read</td>
<td>读取角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/read</td>
<td>读取资源组</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/deployments/*</td>
<td>创建和管理资源组部署</td>
</tr>
<tr>
<td>Microsoft.Insights/alertRules/*</td>
<td>创建和管理警报规则</td>
</tr>
<tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
</table>

### 搜索服务参与者

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>Microsoft.Search/searchServices/*</td>
<td>创建和管理搜索服务</td>
</tr>
<tr>
<td>Microsoft.Authorization/*/read</td>
<td>读取角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/read</td>
<td>读取资源组</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/deployments/*</td>
<td>创建和管理资源组部署</td>
</tr>
<tr>
<td>Microsoft.Insights/alertRules/*</td>
<td>创建和管理警报规则</td>
</tr>
<tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
</table>

### 存储帐户参与者

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>Microsoft.ClassicStorage/storageAccounts/*</td>
<td>创建和管理存储帐户</td>
</tr>
<tr>
<td>Microsoft.Authorization/*/read</td>
<td>读取角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/read</td>
<td>读取资源组</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/deployments/*</td>
<td>创建和管理资源组部署</td>
</tr>
<tr>
<td>Microsoft.Insights/alertRules/*</td>
<td>创建和管理警报规则</td>
</tr>
<tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
</table>

### 用户访问管理员

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>*/read</td>
<td>读取所有类型的资源</td>
</tr>
<tr>
<td>Microsoft.Authorization/*</td>
<td>创建和管理角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
</table>

### 虚拟机参与者

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>Microsoft.ClassicStorage/storageAccounts/read</td>
<td>读取存储帐户</td>
</tr>
<tr>
<td>Microsoft.ClassicStorage/storageAccounts/listKeys/action</td>
<td>检索存储帐户密钥</td>
</tr>
<tr>
<td>Microsoft.ClassicNetwork/virtualNetworks/read</td>
<td>读取虚拟网络</td>
</tr>
<tr>
<td>Microsoft.ClassicNetwork/virtualNetworks/join/action</td>
<td>加入虚拟网络</td>
</tr>
<tr>
<td>Microsoft.ClassicNetwork/reservedIps/read</td>
<td>读取保留 IP 地址</td>
</tr>
<tr>
<td>Microsoft.ClassicNetwork/reservedIps/link/action</td>
<td>链接到保留 IP 地址</td>
</tr>
<tr>
<td>Microsoft.ClassicCompute/domainNames/*</td>
<td>创建和管理云服务</td>
</tr>
<tr>
<td>Microsoft.ClassicCompute/virtualMachines/*</td>
<td>创建和管理虚拟机</td>
</tr>
<tr>
<td>Microsoft.Authorization/*/read</td>
<td>读取角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/read</td>
<td>读取资源组</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/deployments/*</td>
<td>创建和管理资源组部署</td>
</tr>
<tr>
<td>Microsoft.Insights/alertRules/*</td>
<td>创建和管理警报规则</td>
</tr>
<tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
</table>

### 虚拟网络参与者

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>Microsoft.ClassicNetwork/virtualNetworks/*</td>
<td>创建和管理虚拟网络</td>
</tr>
<tr>
<td>Microsoft.Authorization/*/read</td>
<td>读取角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/read</td>
<td>读取资源组</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/deployments/*</td>
<td>创建和管理资源组部署</td>
</tr>
<tr>
<td>Microsoft.Insights/alertRules/*</td>
<td>创建和管理警报规则</td>
</tr>
<tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
</table>

### Web 计划参与者

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>Microsoft.Web/serverFarms/*</td>
<td>创建和管理 Web 计划</td>
</tr>
<tr>
<td>Microsoft.Authorization/*/read</td>
<td>读取角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/read</td>
<td>读取资源组</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/deployments/*</td>
<td>创建和管理资源组部署</td>
</tr>
<tr>
<td>Microsoft.Insights/alertRules/*</td>
<td>创建和管理警报规则</td>
</tr>
<tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
</table>

### 网站参与者

<table style=width:100%">
<tr>
<th colspan="2">操作</th>
</tr>
<tr>
<td>Microsoft.Web/serverFarms/read</td>
<td>读取 Web 计划</td>
</tr>
<tr>
<td>Microsoft.Web/serverFarms/join/action</td>
<td>加入 Web 计划</td>
</tr>
<tr>
<td>Microsoft.Web/sites/*</td>
<td>创建和管理网站</td>
</tr>
<tr>
<td>Microsoft.Web/certificates/*</td>
<td>创建和管理网站证书</td>
</tr>
<tr>
<td>Microsoft.Authorization/*/read</td>
<td>读取角色和角色分配</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/read</td>
<td>读取资源组</td>
</tr>
<tr>
<td>Microsoft.Resources/subscriptions/resourceGroups/deployments/*</td>
<td>创建和管理资源组部署</td>
</tr>
<tr>
<td>Microsoft.Insights/alertRules/*</td>
<td>创建和管理警报规则</td>
</tr>
<tr>
<td>Microsoft.Support/*</td>
<td>创建和管理支持票证</td>
</tr>
</table>


## 如何提供反馈



## 后续步骤

下面是一些其他资源，可帮助你使用基于角色的访问控制：

+ <!--[-->使用 Windows PowerShell 管理基于角色的访问控制<!--](/documentation/articles/role-based-access-control-powershell)-->
+ <!--[-->使用 Azure CLI 管理基于角色的访问控制<!--](/documentation/articles/role-based-access-control-xplat-cli)-->
+ <!--[-->故障排除基于角色的访问控制<!--](/documentation/articles/role-based-access-control-troubleshooting)-->
+ [Azure Active Directory 高级版和基本版](/documentation/articles/active-directory-editions)
+ [Azure 订阅与 Azure AD 的关联方式](/documentation/articles/active-directory-how-subscriptions-associated-directory)
+ 有关安全组的自助服务组管理的简介，请参阅 [Active Directory 团队博客](http://blogs.technet.com/b/ad/archive/2014/02/24/more-preview-enhancements-for-windows-azure-ad-premium.aspx)

<!--Image references-->
[1]: ./media/role-based-access-control-configure/RBACSubAuthDir.png
[2]: ./media/role-based-access-control-configure/RBACAssignmentScopes.png
[3]: ./media/role-based-access-control-configure/RBACSubscriptionBlade.png
[4]: ./media/role-based-access-control-configure/RBACAddSubReader_NEW.png
[5]: ./media/role-based-access-control-configure/RBACAddRGContributor_NEW.png
[6]: ./media/role-based-access-control-configure/RBACAddProdContributor_NEW.png
[7]: ./media/role-based-access-control-configure/RBACRemoveRole.png
[8]: ./media/role-based-access-control-configure/RBACGuestAccessControls.png
[9]: ./media/role-based-access-control-configure/RBACInviteExtUser_NEW.png
[10]: ./media/role-based-access-control-configure/RBACDirConfigTab.png

<!---HONumber=71-->