<properties 
   pageTitle="使用 Resource Manager API 进行授权 | Azure"
   description="指导开发人员如何使用 Azure Resource Manager API 和 Active Directory 来进行授权，以将应用集成到 Azure。"
   services="azure-resource-manager,active-directory"
   documentationCenter="na"
   authors="dushyantgill"
   manager="timlt"
   editor="tysonn" />
<tags 
   ms.service="azure-resource-manager"
   ms.date="04/18/2016"
   wacn.date="05/16/2016" />


# 使用 Azure Resource Manager API 进行授权的开发人员指南

如果软件开发人员想要将应用与 Azure 集成或要管理客户的 Azure 资源，则可以参考本主题，其中说明了如何使用 Azure Resource Manager API 进行身份验证。

应用可通过多种方式访问 Resource Manager API：

1. **用户 + 应用访问**：针对代表登录用户访问资源的应用使用此方法。这适用于仅处理“交互式管理”Azure 资源的应用，例如 Web 应用和命令行工具。
1. **仅限应用的访问**：必须授予应用直接访问资源的权限时，请使用此方法。此方法适用于需要长期“脱机访问”Azure 的守护程序服务和计划的作业。

本主题逐步说明如何创建一个可同时使用这两种授权方法的应用。

你构建的 Web 应用程序可以：

1. 将 Azure 用户登录
2. 代表用户查询 Resource Manager（用户 + 应用访问），以获取该用户拥有的 Azure 订阅列表
3. 可让用户将订阅“连接”到应用，从而向应用程序授予对订阅的直接访问权限
4. 以应用程序的形式访问 Resource Manager 以执行脱机操作（仅限应用的访问）

下面是你要编写的 Web 应用程序端到端流程。

![ARM 授权 - 应用注册 1](./media/resource-manager-api-authentication/ARM-Auth-Swim-Lane.png)

本主题的所有代码都将作为 Web 应用运行，你可以在 [http://vipswapper.azurewebsites.net/cloudsense](http://vipswapper.azurewebsites.net/cloudsense) 上试用。

身为用户，你可以选择要用于登录的帐户类型：

![选择登录](./media/resource-manager-api-authentication/ARM-Auth-Sample-App-Ux-1.png)

提供凭据。

![提供凭据](./media/resource-manager-api-authentication/ARM-Auth-Sample-App-Ux-2.png)

授予应用访问你的 Azure 订阅的权限：
 
 ![授予访问权限](./media/resource-manager-api-authentication/ARM-Auth-Sample-App-Ux-3.png)
 
将你的订阅连接到应用以进行监视：

![连接订阅](./media/resource-manager-api-authentication/ARM-Auth-Sample-App-Ux-4.png)

与应用断开连接或修复连接：

![断开连接订阅](./media/resource-manager-api-authentication/ARM-Auth-Sample-App-Ux-5.png)

## 将应用程序注册到 Azure Active Directory

首先，请将你的 Web 应用注册到 Azure Active Directory (AD)。应用注册将在 Azure AD 中为你的应用创建一个中心标识。该标识保留有关应用程序的基本信息，例如应用程序用来进行身份验证和访问 Azure Resource Manager API 的 OAuth 客户端 ID、回复 URL 和凭据。应用注册还会记录应用程序在代表用户访问 Microsoft API 时所需的各种委派权限。

[Create Active Directory application and service principal using portal](/documentation/articles/resource-group-create-service-principal-portal/)（使用门户创建 Active Directory 应用程序和服务主体）主题说明了设置应用程序所要执行的所有步骤。请参阅该主题来创建具有以下属性的应用程序：

- 名为 **CloudSense** 的 Web 应用程序
- 采用 **http://{domain_name_of_your_directory}/{name_of_the_app}** 格式的登录 URL 和应用 ID URI。
- 用于登录应用程序的身份验证密钥
- 为 **Azure 服务管理 API** 委派的权限**访问 Azure 服务管理**。为 **Azure Active Directory** 保留默认设置“启用单一登录并读取用户的配置文件”。
- 多租户应用程序

### 可选配置 - 证书凭据

Azure AD 还支持应用程序的证书凭据：创建自签名证书、保留私钥，以及将公钥添加到 Azure AD 应用程序注册。对于身份验证，应用程序会使用你的私钥将小负载发送到签名的 Azure AD，然后 Azure AD 使用注册的公钥来验证签名。

有关配置证书的信息，请参阅 [Build service and daemon apps in Office 365](https://msdn.microsoft.com/office/office365/howto/building-service-apps-in-office-365)（在 Office 365 中构建服务和守护程序应用）。标题为“配置应用程序的 X.509 公共证书”部分提供了有关设置证书的分步说明。或者，请参阅 [Authenticating a service principal with Azure Resource Manager](/documentation/articles/resource-group-authenticate-service-principal/)（通过 Azure 资源管理器对服务主体进行身份验证），获取有关通过 Azure PowerShell 或 Azure CLI 配置证书的示例。

## 对用户进行身份验证和获取访问令牌

现在你已做好一切准备，可以开始编写应用程序的代码。本主题提供端到端流程每个步骤的 REST API 示例，以及每个步骤的相关 C# 代码的链接。完整的 ASP.NET MVC 应用程序示例可在 [https://github.com/dushyantgill/VipSwapper/tree/master/CloudSense](https://github.com/dushyantgill/VipSwapper/tree/master/CloudSense) 中找到。

从用户确定要将其 Azure 订阅连接到应用程序时开始。

你必须要求用户提供两项信息：

1. **目录域名**：与用户的 Azure 订阅关联的 Azure Active Directory 域名。必须将 OAuth 2.0 授权请求发送到此 Azure AD。用户可通过导航到 Azure 门户并选择右上角的帐户，来找到其 Azure AD 的域名。你可以向用户提供类似于下面的可视化说明： 

     ![](./media/resource-manager-api-authentication/show-directory.png)
   
1. **Microsoft 帐户与工作帐户**：确定用户是使用 Microsoft 帐户（也称为 Live Id）还是工作帐户（也称为组织帐户）来管理其 Azure 订阅。如果使用 Microsoft 帐户，你的应用程序会使用查询字符串参数 (&domain\_hint=live.com) 将用户重定向到 Azure Active Directory 登录页，其中会指示 Azure AD 要将用户直接转到 Microsoft 帐户登录页。针对任一帐户类型接收的授权代码和令牌将以相同的方式处理。

然后，应用程序使用 OAuth 2.0 授权请求将用户重定向到 Azure AD - 以验证用户的凭据并取回授权代码。应用程序将使用授权代码来访问 Resource Manager 的令牌。

### 授权请求 (OAuth 2.0)

将 Open ID Connect/OAuth2.0 授权请求发送到 Azure AD 授权终结点：

    http://login.microsoftonline.com/{directory_domain_name}/OAuth2/Authorize

[Authorization Code Grant Flow](https://msdn.microsoft.com/zh-cn/library/azure/dn645542.aspx)（授权代码授予流）主题中介绍了适用于此请求的查询字符串参数。

以下示例演示如何请求 OAuth2.0 授权：

    https://login.windows.net/dushyantgill.com/OAuth2/Authorize?client_id=a0448380-c346-4f9f-b897-c18733de9394&response_mode=query&response_type=code&redirect_uri=http%3a%2f%2fwww.vipswapper.com%2fcloudsense%2fAccount%2fSignIn&resource=https%3a%2f%2fgraph.windows.net%2f&domain_hint=live.com

Azure AD 对用户进行身份验证，并根据需要请求用户向应用授予权限。它会将授权代码返回到应用程序的回复 URL。Azure AD 根据请求的 response\_mode，将数据发回到查询字符串，或作为发布数据发送。

    code=AAABAAAAiL****FDMZBUwZ8eCAA&session_state=2d16bbce-d5d1-443f-acdf-75f6b0ce8850

### 授权请求 (Open ID Connect)

如果你不只想要代表用户访问 Azure Resource Manager，而且还要允许用户使用其 Azure AD 帐户登录你的应用程序，请发出 Open ID Connect 授权请求。使用 Open ID Connect，应用程序也可以从 Azure AD 接收 id\_token，应用可以使用它来将用户登录。

OAuth2.0 授权请求查询字符串参数为：

| QS 参数 | 值
|----|----
| client\_id | 应用程序的客户端 ID
| response\_mode | **form\_post** 或 **query**
| response\_type | **code+id\_token**
| redirect\_uri | 应用程序的采用 URL 编码的回复 URL。例如：http://www.vipswapper.com/cloudsense/Account/SignIn |
| resource | Azure 服务管理 API 的 URL 编码标识符：https://management.core.chinacloudapi.cn/ |
| 作用域 | openid+profile
| nonce | 数据片段，用于将授权请求绑定到返回的 id\_token，以确保授权响经过请求且未重播。
| domain\_hint | live.com <br />注意：仅当用户使用 Microsoft 帐户管理其 Azure 订阅时，才可使用 domain\_hint 参数。
| state | 选择性地指定你希望 Azure AD 与响应一起返回的任何状态数据。

下面是一个示例 Open ID Connect 请求：

     https://login.windows.net/dushyantgill.com/OAuth2/Authorize?client_id=a0448380-c346-4f9f-b897-c18733de9394&response_mode=form_post&response_type=code+id_token&redirect_uri=http%3a%2f%2fwww.vipswapper.com%2fcloudsense%2fAccount%2fSignIn&resource=https%3a%2f%2fgraph.windows.net%2f&scope=openid+profile&nonce=63567Dc4MDAw&domain_hint=live.com&state=M_12tMyKaM8

Azure AD 对用户进行身份验证，并根据需要请求用户向应用授予权限。它会将授权代码返回到应用程序的回复 URL。Azure AD 根据请求的 response\_mode，将数据发回到查询字符串，或作为发布数据发送。

下面是一个示例 Open ID Connect 响应：

    code=AAABAAAAiL*****I4rDWd7zXsH6WUjlkIEQxIAA&id_token=eyJ0eXAiOiJKV1Q*****T3GrzzSFxg&state=M_12tMyKaM8&session_state=2d16bbce-d5d1-443f-acdf-75f6b0ce8850

### 验证 id\_token

在应用程序将用户登录之前，必须先验证 id\_token。令牌验证是一个微妙的主题，我建议针对开发平台使用标准 JSON Web 令牌处理程序库（请参阅 [.NET Azure AD JWT Handler source code](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet/blob/master/src/System.IdentityModel.Tokens.Jwt/JwtSecurityTokenHandler.cs)（.NET Azure AD JWT 处理程序源代码））。也就是说，你需要负责应用程序的安全性，因此，请确保用于处理 id\_token 的库可以正确地验证令牌的以下方面：

- **令牌计时**：检查 nbf 和 exp 声明，以确保令牌不会太新或太旧。惯常的做法是保留一些宽限时间（5 分钟）以适应时间偏差。
- **颁发者**：检查 iss 声明以确保令牌颁发者是 Azure Active Directory：https://sts.windows.net/{tenant_id_of_the_directory}
- **受众**：检查 aud 声明以确保为应用程序构建了令牌。该值必须是应用程序的客户端 ID。
- **Nonce**：检查 nonce 声明以检查在授权请求中发送的 nonce 数据，确保应用程序已请求响应且未重播令牌。
- **签名**：应用必须验证令牌是否已由 Azure Active Directory 签名。Azure AD 签名密钥经常滚动更新，因此应用必须每日轮询刷新的密钥，或在签名验证失败时签入刷新的密钥。有关详细信息，请参阅 [Important Information About Signing Key Rollover in Azure AD](https://msdn.microsoft.com/zh-cn/library/azure/dn641920.aspx)（有关 Azure AD 中签名密钥滚动更新的重要信息）。

验证 **id\_token** 后，使用 oid 声明值作为用户的不变且不可重复使用的标识符。将 **unique\_name** 或 upn/email 声明用作用户可读的用户显示名称。也可以针对显示目的使用可选的 given\_name/family\_name 声明。

### 令牌请求（OAuth2.0 代码授予流）

既然应用程序已从 Azure AD 收到授权代码，现在你可以获取 Azure Resource Manager 的访问令牌。将 OAuth2.0 代码授予令牌请求发布到 Azure AD 令牌终结点：

    http://login.microsoftonline.com/{directory_domain_name}/OAuth2/Token

[Authorization Code Grant Flow](https://msdn.microsoft.com/zh-cn/library/azure/dn645542.aspx)（授权代码授予流）主题中介绍了适用于此请求的查询字符串参数。

以下示例演示如何使用密码凭据来请求代码授予令牌：

    POST https://login.windows.net/7fe877e6-a150-4992-bbfe-f517e304dfa0/oauth2/token HTTP/1.1

    Content-Type: application/x-www-form-urlencoded
    Content-Length: 1012

    grant_type=authorization_code&code=AAABAAAAiL9Kn2Z*****L1nVMH3Z5ESiAA&redirect_uri=http%3A%2F%2Flocalhost%3A62080%2FAccount%2FSignIn&client_id=a0448380-c346-4f9f-b897-c18733de9394&client_secret=olna84E8*****goScOg%3D

使用证书凭据时，请使用应用程序证书凭据的私钥来创建 JSON Web 令牌 (JWT) 并签名 (RSA SHA256)。[Authorization Code Grant Flow](https://msdn.microsoft.com/zh-cn/library/azure/dn645542.aspx)（授权代码授予流）中说明了该令牌的声明类型。请参考 [Active Directory Auth Library (.NET) code](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/blob/master/src/ADAL.NET/CryptographyHelper.cs)（Active Directory 身份验证库 (.NET) 代码）来为客户端断言 JWT 令牌签名。

有关客户端身份验证的详细信息，请参阅 [Open ID Connect spec](http://openid.net/specs/openid-connect-core-1_0.html#ClientAuthentication)（Open ID Connect 规范）。下面是一个[示例客户端断言 JWT 令牌](https://www.authnauthz.com/OAuth/ParseJWTToken?token=eyJhbGciOiJSUzI1NiIsIng1dCI6IlFwcXdKZnJNZ003ekJ4M1hkM2NSSFdkYVFsTSJ9.eyJhdWQiOiJodHRwczpcL1wvbG9naW4ud2luZG93cy5uZXRcL2FhbHRlc3RzLm9ubWljcm9zb2Z0LmNvbVwvb2F1dGgyXC90b2tlbiIsImV4cCI6MTQyODk2Mjk5MSwiaXNzIjoiOTA4M2NjYjgtOGE0Ni00M2U3LTg0MzktMWQ2OTZkZjk4NGFlIiwianRpIjoiMmYyMjczMzQtZGQ3YS00NzZkLWFlOTYtYzg4NDQ4YTkxZGM0IiwibmJmIjoxNDI4OTYyMzkxLCJzdWIiOiI5MDgzY2NiOC04YTQ2LTQzZTctODQzOS0xZDY5NmRmOTg0YWUifQ.UXQE9H-FlwxYQmRVG0-p7pAX9TFgiRXcYr7GhbcC7ndIPHKpZ5tfHWPEgBl3ZVRvF2l8uA7HEV86T7t2w7OHhHwLBoW7XTgj-17hnV1CY21MwjrebPjaPIVITiilekKiBASfW2pmss3MjeOYcnBV2MuUnIgt4A_iUbF_-opRivgI4TFT4n17_3VPlChcU8zJqAMpt3TcAxC3EXXfh10Mw0qFfdZKqQOQxKHjnL8y7Of9xeB9BBD_b22JNRv0m7s0cYRx2Cz0cUUHw-ipHhWaW7YwhVRMfK6BMkaDUgaie4zFkcgHb7rm1z0rM1CvzIqP-Mwu3oEqYpY9cYo8nEjMyA)。

以下示例演示如何使用证书凭据来请求代码授予令牌：

	POST https://login.windows.net/7fe877e6-a150-4992-bbfe-f517e304dfa0/oauth2/token HTTP/1.1
	
	Content-Type: application/x-www-form-urlencoded
	Content-Length: 1012
	
	grant_type=authorization_code&code=AAABAAAAiL9Kn2Z*****L1nVMH3Z5ESiAA&redirect_uri=http%3A%2F%2Flocalhost%3A62080%2FAccount%2FSignIn&client_id=a0448380-c346-4f9f-b897-c18733de9394&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer&client_assertion=eyJhbG*****Y9cYo8nEjMyA

代码授予令牌的示例响应：

    HTTP/1.1 200 OK

    {"token_type":"Bearer","expires_in":"3599","expires_on":"1432039858","not_before":"1432035958","resource":"https://management.core.chinacloudapi.cn/","access_token":"eyJ0eXAiOiJKV1Q****M7Cw6JWtfY2lGc5A","refresh_token":"AAABAAAAiL9Kn2Z****55j-sjnyYgAA","scope":"user_impersonation","id_token":"eyJ0eXAiOiJKV*****-drP1J3P-HnHi9Rr46kGZnukEBH4dsg"}

#### 处理代码授予令牌响应

成功的令牌响应将包含 Azure Resource Manager 的（用户 + 应用）访问令牌。应用程序将使用此访问令牌来代表用户访问 Resource Manager。Azure AD 颁发的访问令牌生存期为一小时。Web 应用程序不太可能需要更新（用户 + 应用）访问令牌 - 但如果需要，你可以使用应用程序在令牌响应中收到的刷新令牌。将 OAuth2.0 令牌请求发布到 Azure AD 令牌终结点：

    http://login.microsoftonline.com/{directory_domain_name}/OAuth2/Token

[Authorization Code Grant Flow](https://msdn.microsoft.com/zh-cn/library/azure/dn645542.aspx)（授权代码授予流）中介绍了要在刷新请求中使用的参数。

以下示例演示如何使用刷新令牌：

    POST https://login.windows.net/7fe877e6-a150-4992-bbfe-f517e304dfa0/oauth2/token HTTP/1.1

    Content-Type: application/x-www-form-urlencoded
    Content-Length: 1012

    grant_type=refresh_token&refresh_token=AAABAAAAiL9Kn2Z****55j-sjnyYgAA&client_id=a0448380-c346-4f9f-b897-c18733de9394&client_secret=olna84E8*****goScOg%3D

请注意，尽管刷新令牌可用于获取 Azure Resource Manager 的新访问令牌，但它们并不适合应用程序脱机访问。刷新令牌生存期有限，且刷新令牌绑定到用户。如果用户离开了组织，使用该刷新令牌的应用程序将失去访问权限。此方法不适合团队用来管理其 Azure 资源的应用程序。

## 列出可用订阅

现在，应用程序已获取令牌，可代表用户访问 Azure Resource Manager。

体验的下一步是允许用户将其 Azure 订阅连接到你的应用，因此，即使该用户不存在，应用程序也可以管理这些订阅（长期脱机访问）。向用户显示其可以管理访问权限的 Azure 订阅列表，并允许用户将 RBAC 角色直接分配到应用程序的标识。

![ARM 授权 - 示例应用用户体验 4](./media/resource-manager-api-authentication/ARM-Auth-Sample-App-Ux-4-full.png)

### 列出用户拥有任何访问权限的订阅

我们应该先调用 [Resource Manager 列出订阅](https://msdn.microsoft.com/zh-cn/library/azure/dn790531.aspx) API 来列出用户拥有任何类型的访问权限的所有订阅。然后，我们将识别用户可管理访问权限的订阅。

ASP.net MVC 示例应用的 [GetUserSubscription](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L79) 方法可实现此调用。

下面是列出订阅的示例请求：

    GET https://management.chinacloudapi.cn/subscriptions?api-version=2014-04-01-preview HTTP/1.1

    Authorization: Bearer eyJ0eXAiOiJKV1QiLC***lwO1mM7Cw6JWtfY2lGc5A

下面是列出订阅的示例响应：

    HTTP/1.1 200 OK

    {"value":[{"id":"/subscriptions/34370e90-ac4a-4bf9-821f-85eeedeae1a2","subscriptionId":"34370e90-ac4a-4bf9-821f-85eeedeae1a2","displayName":"Sandbox","state":"Enabled","subscriptionPolicies":{"locationPlacementId":"Public_2014-09-01","quotaId":"PayAsYouGo_2014-09-01"}},{"id":"/subscriptions/c276fc76-9cd4-44c9-99a7-4fd71546436e","subscriptionId":"c276fc76-9cd4-44c9-99a7-4fd71546436e","displayName":"Production","state":"Enabled","subscriptionPolicies":{"locationPlacementId":"Public_2014-09-01","quotaId":"PayAsYouGo_2014-09-01"}}]}

### 获取用户对订阅的权限

只能为用户可管理访问权限的订阅显示连接/断开连接操作。对于每个订阅，可调用 [Resource Manager 列出权限](https://msdn.microsoft.com/zh-cn/library/azure/dn906889.aspx) API 来确定用户是否拥有订阅的“访问管理”权限。

ASP.net MVC 示例应用的 [UserCanManagerAccessForSubscription](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L132) 方法可实现此调用。

以下示例演示如何请求用户对订阅的权限。83cfe939-2402-4581-b761-4f59b0a041e4 是订阅的 ID。

    GET https://management.chinacloudapi.cn/subscriptions/83cfe939-2402-4581-b761-4f59b0a041e4/providers/microsoft.authorization/permissions?api-version=2014-07-01-preview HTTP/1.1

    Authorization: Bearer eyJ0eXAiOiJKV1QiLC***lwO1mM7Cw6JWtfY2lGc5A

下面是请求获取用户对订阅的权限后返回的示例响应：

    HTTP/1.1 200 OK

    {"value":[{"actions":["*"],"notActions":["Microsoft.Authorization/*/Write","Microsoft.Authorization/*/Delete"]},{"actions":["*/read"],"notActions":[]}]}

权限 API 将返回多个权限。每个权限包括允许的操作 (actions) 和禁止的操作 (notactions)。如果某个操作出现在任何权限的允许操作列表中，并且不在该权限的禁止操作列表中，则用户可执行该操作。**microsoft.authorization/roleassignments/write** 是授予访问管理权限的操作。应用程序必须分析权限结果，以便在每个权限的 actions 和 notactions 中的此操作字符串上查找 regex 匹配项。

### 可选：列出存在用户帐户的目录

用户的帐户可以在多个 Azure Active Directory 中。用户最初可能未指定正确的目录名称 - 在此情况下，所需的订阅不会显示在列表中。

[Resource Manager 列出租户](https://msdn.microsoft.com/zh-cn/library/azure/dn790536.aspx) API 可列出包含用户帐户的所有目录的标识符列表。你可以调用该 API 来确定用户帐户是否出现在多个目录中，并选择性地向用户显示如下所示的消息：“找不到所需的订阅? 它可能位于你所属的其他 Azure Active Directory 中。请单击此处切换目录。”

ASP.NET MVC 示例应用的 [GetUserOrganizations](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L20) 方法可实现此调用。

下面是列出目录的示例请求：

    GET https://management.chinacloudapi.cn/tenants?api-version=2014-04-01-preview HTTP/1.1

    Authorization: Bearer eyJ0eXAiOiJKV1Qi****8DJf1UO4a-ZZ_TJmWFlwO1mM7Cw6JWtfY2lGc5A

下面是列出目录的示例响应：

    HTTP/1.1 200 OK

    {"value":[{"id":"/tenants/7fe877e6-a150-4992-bbfe-f517e304dfa0","tenantId":"7fe877e6-a150-4992-bbfe-f517e304dfa0"},{"id":"/tenants/62e173e9-301e-423e-bcd4-29121ec1aa24","tenantId":"62e173e9-301e-423e-bcd4-29121ec1aa24"}]}

## 将订阅连接到应用程序

现在，你已获取可由用户连接到应用程序的 Azure 订阅列表。下一步是为用户提供一个命令来创建连接。当用户选择“连接”时，你的应用将会：

1. 将相应的 RBAC 角色分配到订阅上的应用程序标识。
2. 通过查询订阅上的应用程序权限或使用仅限应用的令牌访问 Resource Manager，来验证访问权限分配。
1. 记录应用程序“已连接的订阅”数据结构中的连接 - 保存订阅的 ID。

让我们更深入地分析第一个步骤：若要将相应的 RBAC 角色分配到应用程序标识，你必须确定：

- 用户 Azure Active Directory 中的应用程序标识的对象 ID
- 应用程序需要在订阅上拥有的 RBAC 角色的标识符

我们将深入探讨第一个部分：当应用程序首次从 Azure AD 对用户进行身份验证时，将在该 Azure AD 中创建应用程序的服务主体对象。Azure 允许将 RBAC 角色分配到服务主体，以将直接访问权限授予 Azure 资源上的相应应用程序。这正是我们要做的 - 让我们开始查询 Azure AD 图形 API，以确定应用程序在已登录用户的 Azure AD 中的服务主体标识符。

你只有 Azure Resource Manager 的访问令牌 - 需要获取新的访问令牌来调用 Azure AD 图形 API。Azure AD 中的每个应用程序都有权查询其本身的服务主体对象，因此我们不需要用户 + 应用访问令牌就能执行此操作，仅限应用的访问令牌已足够。

<a id="app-azure-ad-graph"></a>
### 获取 Azure AD 图形 API 的仅限应用的访问令牌

若要对应用进行身份验证并获取 Azure AD 图形 API 的令牌，请向 Azure AD 令牌终结点发出客户端凭据授予 OAuth2.0 流令牌请求 (**http://login.microsoftonline.com/{directory\_domain\_name}/OAuth2/Token**)。

ASP.net MVC 示例应用程序的 [GetObjectIdOfServicePrincipalInOrganization](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureADGraphAPIUtil.cs#L73) 方法的第 73-77 行使用适用于 .NET 的 Active Directory 身份验证库来获取图形 API 的仅限应用的访问令牌。

客户端凭据将授予令牌请求数据：

| 元素 | 值
|----|----
| grant\_type | **client\_credentials**
| client\_id | 应用程序的客户端 ID
| resource | 正在针对访问令牌请求的资源的 URL 编码标识符。在本例中，Azure AD 图形 API 的标识符为：**https://graph.windows.net/**
| client\_secret 或 client\_assertion\_type + client\_assertion | 如果你的应用程序使用密码凭据，请使用 client\_secret。如果你的应用程序使用证书凭据，请使用 client\_assertion。

客户端凭据授予令牌的示例请求：

    POST https://login.windows.net/62e173e9-301e-423e-bcd4-29121ec1aa24/oauth2/token HTTP/1.1
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 187</pre>
    <pre>grant_type=client_credentials&client_id=a0448380-c346-4f9f-b897-c18733de9394&resource=https%3A%2F%2Fgraph.windows.net%2F &client_secret=olna8C*****Og%3D

客户端凭据授予令牌的示例响应：

    HTTP/1.1 200 OK

    {"token_type":"Bearer","expires_in":"3599","expires_on":"1432039862","not_before":"1432035962","resource":"https://graph.windows.net/","access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNBVGZNNXBPWWlKSE1iYTlnb0VLWSIsImtpZCI6Ik1uQ19WWmNBVGZNNXBPWWlKSE1iYTlnb0VLWSJ9.eyJhdWQiOiJodHRwczovL2dyYXBoLndpbmRv****G5gUTV-kKorR-pg"}

### 获取用户 Azure AD 中应用程序服务主体的 ObjectId

现在，请使用仅限应用的访问令牌来查询 [Azure AD Graph 服务主体](https://msdn.microsoft.com/zh-cn/library/Azure/Ad/Graph/api/entity-and-complex-type-reference#ServicePrincipalEntity) API，以确定目录中应用程序服务主体的对象 ID。

ASP.net MVC 示例应用程序的 [GetObjectIdOfServicePrincipalInOrganiation](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureADGraphAPIUtil.cs#L66) 方法可实现此调用。

以下示例演示如何请求应用程序的服务主体。a0448380-c346-4f9f-b897-c18733de9394 是应用程序的客户端 ID。

    GET https://graph.windows.net/62e173e9-301e-423e-bcd4-29121ec1aa24/servicePrincipals?api-version=1.5&$filter=appId%20eq%20'a0448380-c346-4f9f-b897-c18733de9394' HTTP/1.1

    Authorization: Bearer eyJ0eXAiOiJK*****-kKorR-pg

以下示例演示了请求应用程序的服务主体后返回的响应

    HTTP/1.1 200 OK

    {"odata.metadata":"https://graph.windows.net/62e173e9-301e-423e-bcd4-29121ec1aa24/$metadata#directoryObjects/Microsoft.DirectoryServices.ServicePrincipal","value":[{"odata.type":"Microsoft.DirectoryServices.ServicePrincipal","objectType":"ServicePrincipal","objectId":"9b5018d4-6951-42ed-8a92-f11ec283ccec","deletionTimestamp":null,"accountEnabled":true,"appDisplayName":"CloudSense","appId":"a0448380-c346-4f9f-b897-c18733de9394","appOwnerTenantId":"62e173e9-301e-423e-bcd4-29121ec1aa24","appRoleAssignmentRequired":false,"appRoles":[],"displayName":"CloudSense","errorUrl":null,"homepage":"http://www.vipswapper.com/cloudsense","keyCredentials":[],"logoutUrl":null,"oauth2Permissions":[{"adminConsentDescription":"Allow the application to access CloudSense on behalf of the signed-in user.","adminConsentDisplayName":"Access CloudSense","id":"b7b7338e-683a-4796-b95e-60c10380de1c","isEnabled":true,"type":"User","userConsentDescription":"Allow the application to access CloudSense on your behalf.","userConsentDisplayName":"Access CloudSense","value":"user_impersonation"}],"passwordCredentials":[],"preferredTokenSigningKeyThumbprint":null,"publisherName":"vipswapper"quot;,"replyUrls":["http://www.vipswapper.com/cloudsense","http://www.vipswapper.com","http://vipswapper.com","http://vipswapper.azurewebsites.net","http://localhost:62080"],"samlMetadataUrl":null,"servicePrincipalNames":["http://www.vipswapper.com/cloudsense","a0448380-c346-4f9f-b897-c18733de9394"],"tags":["WindowsAzureActiveDirectoryIntegratedApp"]}]}

### 获取 Azure RBAC 角色标识符

必须将适当的 RBAC 角色分配到所选订阅上的应用程序服务主体。为此，你必须确定 Azure RBAC 角色的标识符。

应用程序的正确 RBAC 角色：

- 如果应用程序只监视订阅而不进行任何更改，它只需要对订阅的读取者权限。请分配**读取者**角色。
- 如果应用程序管理 Azure 订阅并创建/修改/删除实体，它将需要一个参与者权限。
  - 若要管理特定类型的资源，请分配资源的特定参与者角色（虚拟机参与者、虚拟网络参与者、存储帐户参与者，等等）
  - 若要管理任何资源类型，请分配**参与者**角色。

应用程序的角色分配将向用户显示，因此请选择最低必要权限。

请调用 [Resource Manager 角色定义 API](https://msdn.microsoft.com/zh-cn/library/azure/dn906879.aspx) 列出所有 Azure RBAC 角色，然后搜索并逐一查看结果，按名称找到所需的角色定义。

ASP.net MVC 示例应用的 [GetRoleId](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L354) 方法可实现此调用。

以下请求示例演示如何获取 Azure RBAC 角色标识符。09cbd307-aa71-4aca-b346-5f253e6e3ebb 是订阅的 ID。

    GET https://management.chinacloudapi.cn/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/Microsoft.Authorization/roleDefinitions?api-version=2014-07-01-preview HTTP/1.1

    Authorization: Bearer eyJ0eXAiOiJKV*****fY2lGc5

响应格式如下：

    HTTP/1.1 200 OK

    {"value":[{"properties":{"roleName":"API Management Service Contributor","type":"BuiltInRole","description":"Lets you manage API Management services, but not access to them.","scope":"/","permissions":[{"actions":["Microsoft.ApiManagement/Services/*","Microsoft.Authorization/*/read","Microsoft.Resources/subscriptions/resources/read","Microsoft.Resources/subscriptions/resourceGroups/read","Microsoft.Resources/subscriptions/resourceGroups/resources/read","Microsoft.Resources/subscriptions/resourceGroups/deployments/*","Microsoft.Insights/alertRules/*","Microsoft.Support/*"],"notActions":[]}]},"id":"/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/Microsoft.Authorization/roleDefinitions/312a565d-c81f-4fd8-895a-4e21e48d571c","type":"Microsoft.Authorization/roleDefinitions","name":"312a565d-c81f-4fd8-895a-4e21e48d571c"},{"properties":{"roleName":"Application Insights Component Contributor","type":"BuiltInRole","description":"Lets you manage Application Insights components, but not access to them.","scope":"/","permissions":[{"actions":["Microsoft.Insights/components/*","Microsoft.Insights/webtests/*","Microsoft.Authorization/*/read","Microsoft.Resources/subscriptions/resources/read","Microsoft.Resources/subscriptions/resourceGroups/read","Microsoft.Resources/subscriptions/resourceGroups/resources/read","Microsoft.Resources/subscriptions/resourceGroups/deployments/*","Microsoft.Insights/alertRules/*","Microsoft.Support/*"],"notActions":[]}]},"id":"/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/Microsoft.Authorization/roleDefinitions/ae349356-3a1b-4a5e-921d-050484c6347e","type":"Microsoft.Authorization/roleDefinitions","name":"ae349356-3a1b-4a5e-921d-050484c6347e"}]}

请注意，你不需要持续调用此 API。一旦确定角色定义的已知 GUID，便可以将角色定义 ID 构造为：

    /subscriptions/{subscription_id}/providers/Microsoft.Authorization/roleDefinitions/{well-known-role-guid}

下面是常用内置角色的已知 GUID：

| 角色 | Guid |
| ----- | ------ |
| 读取器 | acdd72a7-3385-48ef-bd42-f606fba81ae7
| 参与者 | b24988ac-6180-42a0-ab88-20f7382dd24c
| 虚拟机参与者 | d73bb868-a0df-4d4d-bd69-98a00b01fccb
| 虚拟网络参与者 | b34d265f-36f7-4a0d-a4d4-e158ca92e90f
| 存储帐户参与者 | 86e8f5dc-a6e9-4c67-9d15-de283e8eac25
| 网站参与者 | de139f84-1756-47ae-9be6-808fbbe84772
| Web 计划参与者 | 2cc479cb-7b4d-49a8-b449-8c00fd0f0a4b
| SQL Server 参与者 | 6d8ee4ec-f05a-4a1d-8b00-a9b17e38b437
| SQL DB 参与者 | 9b7fa17d-e63e-47b0-bb0a-15c516ac86ec

### 将 RBAC 角色分配到应用程序

你已做好一切准备，现在可使用 [Resource Manager 创建角色分配](https://msdn.microsoft.com/zh-cn/library/azure/dn906887.aspx) API，将相应的 RBAC 角色分配到所选订阅上的应用程序服务主体。

ASP.net MVC 示例应用的 [GrantRoleToServicePrincipalOnSubscription](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L269) 方法可实现此调用。

将 RBAC 角色分配到应用程序的示例请求：

    PUT https://management.chinacloudapi.cn/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/microsoft.authorization/roleassignments/4f87261d-2816-465d-8311-70a27558df4c?api-version=2014-10-01-preview HTTP/1.1

    Authorization: Bearer eyJ0eXAiOiJKV1QiL*****FlwO1mM7Cw6JWtfY2lGc5
    Content-Type: application/json
    Content-Length: 230

    {"properties": {"roleDefinitionId":"/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/Microsoft.Authorization/roleDefinitions/acdd72a7-3385-48ef-bd42-f606fba81ae7","principalId":"c3097b31-7309-4c59-b4e3-770f8406bad2"}}

在请求中使用以下值：

| Guid | 说明 |
| ------ | --------- |
| 09cbd307-aa71-4aca-b346-5f253e6e3ebb | 订阅的 ID
| c3097b31-7309-4c59-b4e3-770f8406bad2 | 应用程序服务主体的对象 ID
| acdd72a7-3385-48ef-bd42-f606fba81ae7 | 读取者 RBAC 角色的已知 GUID
| 4f87261d-2816-465d-8311-70a27558df4c | 为新角色分配创建的新 GUID

响应格式如下：

    HTTP/1.1 201 Created

    {"properties":{"roleDefinitionId":"/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/Microsoft.Authorization/roleDefinitions/acdd72a7-3385-48ef-bd42-f606fba81ae7","principalId":"c3097b31-7309-4c59-b4e3-770f8406bad2","scope":"/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb"},"id":"/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/Microsoft.Authorization/roleAssignments/4f87261d-2816-465d-8311-70a27558df4c","type":"Microsoft.Authorization/roleAssignments","name":"4f87261d-2816-465d-8311-70a27558df4c"}

### 获取 Azure Resource Manager 的仅限应用的访问令牌

下一步是验证该应用是否对订阅拥有所需的访问权限。为此，你应该使用 Azure Resource Manager 的仅限应用的令牌在订阅上执行测试任务。测试任务应该验证你的应用程序是否确实对订阅拥有所需的访问权限，以执行脱机监视/管理。

要获取 Azure Resource Manager 的仅限应用的访问令牌，请根据[获取 Azure AD 图形 API 的仅限应用的访问令牌](#app-azure-ad-graph)中的说明为资源参数使用不同的值：

    https://management.core.windows.net/

ASP.net MVC 示例应用程序的 [ServicePrincipalHasReadAccessToSubscription](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L203) 方法的第 210-214 行使用适用于 .NET 的 Active Directory 身份验证库来获取 Azure Resource Manager 的仅限应用的访问令牌。

#### 获取应用程序对订阅的权限

若要检查应用程序是否对 Azure 订阅拥有所需的访问权限，你也可以调用 [Resource Manager 列出权限](https://msdn.microsoft.com/zh-cn/library/azure/dn906889.aspx) API，以类似于确定用户是否拥有对订阅的“访问管理”权限的方式进行检查。不过，这次请使用上一步骤中收到的仅限应用的访问令牌来调用权限 API。

ASP.NET MVC 示例应用的 [ServicePrincipalHasReadAccessToSubscription](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L203) 方法可实现此调用。

## 管理连接的订阅

将相应的 RBAC 角色分配到订阅上的应用程序服务主体后，应用程序可以使用 Azure Resource Manager 的仅限应用的访问令牌来持续进行监视/管理。

如果订阅所有者使用 Azure 管理门户或命令行工具删除应用程序的角色分配，应用程序将再也无法访问该订阅。在此情况下，你应该通知用户，与订阅的连接是通过应用程序外部提供的，并为他们提供“修复”连接的选项。“修复”只是重新创建脱机删除的角色分配。


## 断开连接订阅

如同允许用户将其订阅连接到你的应用程序一样，你必须允许用户断开连接订阅。从访问管理的观点来讲，断开连接意味着删除应用程序服务主体在订阅上的角色分配。（可选）也可能删除订阅的任何应用程序状态。只有对订阅拥有访问管理权限的用户才能断开连接订阅。

ASP.net MVC 示例应用的 [RevokeRoleFromServicePrincipalOnSubscription 方法](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L303)可实现此调用。

大功告成 - 用户现在可以使用你的应用程序来轻松连接和管理其 Azure 订阅。


<!---HONumber=Mooncake_0509_2016-->