<properties
   pageTitle="Azure Active Directory 中可配置的令牌生存期 | Azure"
   description="管理员和开发人员可以使用此功能指定 Azure AD 颁发的令牌的生存期。"
   services="active-directory"
   documentationCenter=""
   authors="billmath"
   manager="femila"
   editor=""/>  


<tags
   ms.service="active-directory"  
   ms.workload="identity"
   ms.tgt_pltfrm="na"
   ms.devlang="na"
   ms.topic="article"
   ms.date="10/06/2016"
   ms.author="billmath"
   wacn.date="10/31/2016"/>  



# Azure Active Directory 中可配置的令牌生存期（公共预览版）

>[AZURE.NOTE]
此功能目前以公共预览版提供。读者应准备好还原或删除所做的任何更改。在公共预览版推出期间，任何人都可以随意试用此功能，但是，在正式版推出后，某些方面可能需要配合 Azure AD 高级订阅使用。


## 介绍
管理员和开发人员可以使用此功能指定 Azure AD 颁发的令牌的生存期。可以针对租户中的所有应用、多租户应用程序或者租户中的特定服务主体配置令牌生存期。

在 Azure AD 中，策略对象表示针对租户中的单个应用程序或所有应用程序强制实施的一组规则。每种类型的策略都有一个唯一的结构，其中的一组属性将应用于它们所分配到的对象。

可将某个策略指定为租户的默认策略。然后，此策略将对该租户中的任何应用程序生效，只要此策略不被更高优先级的策略覆盖即可。此外，还可以将策略分配到特定的应用程序。优先顺序根据策略类型的不同而异。

可为刷新令牌、访问令牌、会话令牌和 ID 令牌配置令牌生存期策略。


### 访问令牌
客户端使用访问令牌来访问受保护的资源。访问令牌仅可用于用户、客户端和资源的特定组合。访问令牌不能吊销，在过期日期之前保持有效。获取了访问令牌的恶意行动者在访问令牌的生存期内一直可以使用它。调整访问令牌生存期的利弊是可以提高系统性能，但也会增加客户端在用户帐户禁用后保留访问权限的时间。通过减少客户端获取刷新访问令牌的次数可以实现系统性能的提升。

### 刷新令牌
当客户端获取访问令牌来访问受保护资源时，它会同时收到一个刷新令牌和一个访问令牌。当前访问令牌过期时，可以使用刷新令牌获取新的访问/刷新令牌对。刷新令牌绑定到用户和客户端的组合。刷新令牌可以吊销，每次使用它们时，系统都会检查其有效性。

必须在机密客户端与公共客户端之间做出区分。
机密客户端是能够安全存储客户端密码，可证实请求来自客户端应用程序而不是恶意行动者的应用程序。由于这些流程更加安全，因此颁发给这些流程的刷新令牌的默认生存期更长，无法使用策略更改。

由于运行应用程序的环境有限制，公共客户端无法安全存储客户端密码。可以针对资源设置策略，防止使用公共客户端中超过指定期限的刷新令牌获取新的访问权限/刷新令牌对（刷新令牌最大非活动时间）。此外，可以使用策略设置一个时间段，超出该时间段后，刷新令牌不再被接受（刷新令牌最大期限）。通过调整刷新令牌生存期，可以控制用户在使用公共客户端应用程序时，需要重新输入凭据而不是以无提示方式重新进行身份验证的时间和频率。


### ID 令牌
ID 令牌将传递给网站和本机客户端，包含有关用户的配置文件信息。ID 令牌绑定到用户和客户端的特定组合。在过期日期之前，ID 令牌保持有效。一般情况下，Web 应用程序会将应用程序中用户的会话生存期与针对该用户颁发的 ID 令牌的生存期进行匹配。通过调整 ID 令牌生存期，可以控制 Web 应用程序使应用程序会话过期，并要求用户在 Azure AD 上重新进行身份验证（以无提示方式或交互方式）的频率。

### 单一登录会话令牌
当用户在 Azure AD 上进行身份验证时，系统将在用户的浏览器与 Azure AD 之间建立单一登录会话。单一登录会话令牌采用 Cookie 形式，代表此会话。必须注意，SSO 会话令牌不会绑定到特定的资源/客户端应用程序。SSO 会话令牌可以吊销，每次使用它们时，系统都会检查其有效性。

有两种 SSO 会话令牌。浏览器将持久性会话令牌存储为持久性 Cookie，将非持久性会话令牌存储为会话 Cookie（关闭浏览器会破坏这些规则）。

非持久性会话令牌的生存期为 24 小时，持久性令牌的生存期为 180 天。每当在 SSO 会话令牌的有效期内使用它时，有效期会再次延长 24 小时或 180 天。如果 SSO 会话令牌在其有效期内未被使用，则将它视为过期，不再被系统接受。

颁发第一个会话令牌后，可以使用策略来设置一个时间段，超过该时间段后，该会话令牌不再被接受（会话令牌最大期限）。通过调整会话令牌生存期，可以控制用户在使用 Web 应用程序时，需要重新输入凭据而不是以无提示方式进行身份验证的时间和频率。

### 令牌生存期策略属性
令牌生存期策略是一种策略对象，其中包含令牌生存期规则。策略的属性用于控制指定的令牌生存期。如果未设置策略，系统将强制实施默认生存期值。


### 可配置的令牌生存期属性
属性|策略属性字符串|影响|默认|最小值|最大值|
----- | ----- | ----- |----- | ----- | ----- |
访问令牌生存期|	AccessTokenLifetime|访问令牌、ID 令牌、SAML2 令牌|1 小时|10 分钟|1 天|
刷新令牌最大非活动时间|	MaxInactiveTime |刷新令牌 |14 天|10 分钟|	90 天|
单因素刷新令牌最大期限|	MaxAgeSingleFactor |刷新令牌（适用于任何用户） |90 天|10 分钟 |直到吊销*|
多因素刷新令牌最大期限|	MaxAgeMultiFactor|	刷新令牌（适用于任何用户）|	90 天|10 分钟|直到吊销*|
单因素会话令牌最大期限 |MaxAgeSessionSingleFactor** |会话令牌（持久性和非持久性）|	直到吊销* |10 分钟 |直到吊销*|
多因素会话令牌最大期限|	MaxAgeSessionMultiFactor***|	会话令牌（持久性和非持久性）|	直到吊销*|	10 分钟|	直到吊销*



- *365 天是可针对这些属性设置的最大显式时间长短。
- **如果未设置 MaxAgeSessionSingleFactor，此值将采用 MaxAgeSingleFactor 值。如果未设置任一参数，该属性将采用默认值（直到吊销）。
- ***如果未设置 MaxAgeSessionMultiFactor，此值将采用 MaxAgeMultiFactor 值。如果未设置任一参数，该属性将采用默认值（直到吊销）。

### 异常
属性|影响|默认|
----- | ----- | ----- |
刷新令牌最大非活动时间（吊销信息不足的联合用户）|刷新令牌（针对吊销信息不足的联合用户颁发）|12 小时|
刷新令牌最大非活动时间（机密客户端）|	刷新令牌（针对机密客户端颁发）|90 天|
刷新令牌最大期限（针对机密客户端颁发） |	刷新令牌（针对机密客户端颁发） |直到吊销*

### 策略的优先级和评估

可以创建令牌生存期策略并将其分配到特定的应用程序、租户和服务主体。这意味着，可将多个策略应用到特定的应用程序。生效的令牌生存期策略遵循以下规则：


- 如果将某个策略显式分配到服务主体，将强制实施该策略。
- 如果未将策略显式分配到服务主体，则强制实施显式分配到服务主体的父租户的策略。
- 如果未将策略显式分配到服务主体或租户，则强制实施分配到应用程序的策略。
- 如果未将策略分配到服务主体、租户或应用程序对象，则强制实施默认值（请参阅上表）。

有关 Azure AD 中应用程序对象与服务主体对象之间的关系的详细信息，请参阅 [Application and service principal objects in Azure Active Directory](/documentation/articles/active-directory-application-objects/)（Azure Active Directory 中的应用程序对象和服务主体对象）。

使用令牌时，系统会评估其有效性。所访问的应用程序中具有最高优先级的策略将会生效。


>[AZURE.NOTE]
><p>
><p>示例
><p>
><p>用户想要访问 2 个 Web 应用程序：A 和 B。
><p>
><p>
><p>- 这两个应用程序在同一个父租户中。
><p>- 会话令牌最大期限为 8 小时的令牌生存期策略 1 设置为父租户的默认策略。
><p>- Web 应用程序 A 是一个常规用途的 Web 应用程序，未链接到任何策略。
><p>- Web 应用程序 B 用于高度敏感的流程，其服务主体已链接到会话令牌最大期限为 30 分钟的令牌生存期策略 2。
><p>
><p>下午 12:00，用户打开了新浏览器会话，尝试访问 Web 应用程序 A。用户已重定向到 Azure AD 并需要登录。这会在浏览器中留下一个包含会话令牌的 Cookie。用户使用可用于访问应用程序的 ID 令牌重定向回到 Web 应用程序 A。
><p>
><p>下午 12:15，用户尝试访问 Web 应用程序 B。浏览器重定向到 Azure AD 并检测会话 Cookie。Web 应用程序 B 的服务主体已链接到策略 1，但同时也属于使用默认策略 2 的父租户。由于链接到服务主体的策略优先级高于租户默认策略，因此策略 2 生效。会话令牌最初是在过去 30 分钟内颁发的，因此被视为有效。用户使用授予权限的 ID 令牌重定向回到 Web 应用程序 B。
><p>
><p>下午 1:00，用户尝试导航到 Web 应用程序 A。该用户已重定向到 Azure AD。Web 应用程序 A 未链接到任何策略，但由于它在使用默认策略 1 的租户中，因此，此策略将会生效。已检测到最初在过去 8 小时内颁发的会话 Cookie，用户已使用新 ID 令牌以无提示方式重定向回到 Web 应用程序 A，且无需身份验证。
><p>
><p>用户立即尝试访问 Web 应用程序 B。用户已重定向到 Azure AD。如前所述，策略 2 生效。由于颁发的令牌超过 30 分钟，系统提示用户重新输入其凭据，颁发新的会话和 ID 令牌。然后，用户可以访问 Web 应用程序 B。

## 可配置的策略属性：深入信息

### 访问令牌生存期

**字符串：**AccessTokenLifetime

**影响：**访问令牌、ID 令牌

**摘要：**此策略控制此资源的访问令牌和 ID 令牌有效期时间长短。减少访问令牌生存期可以降低恶意行动者长时间使用访问令牌或 ID 令牌的风险（因为这些令牌不可吊销），但同时会对性能造成不利影响，因为必须更频繁地更换令牌。

### 刷新令牌最大非活动时间

**字符串：**MaxInactiveTime

**影响：**刷新令牌

**摘要：**此策略控制客户端在尝试访问此资源时，不再可以使用超过多少时间的刷新令牌来检索新的访问/刷新令牌对。由于使用刷新令牌时通常会返回一个新的刷新令牌，因此，在此策略阻止访问之前，客户端在指定的时间段内不得使用当前刷新令牌访问任何资源。

此策略将强制客户端上处于非活动状态的用户重新进行身份验证，然后才能检索新的刷新令牌。

请务必注意，设置的“刷新令牌最大非活动时间”值必须必须小于“单因素令牌最大期限”和“多因素刷新令牌最大期限”。

### 单因素刷新令牌最大期限

**字符串：**MaxAgeSingleFactor

**影响：**刷新令牌

**摘要：**此策略控制用户在上次仅使用一个因素成功完成身份验证后，可以继续使用刷新令牌获取新访问/刷新令牌对的时间长短。用户完成身份验证并收到新刷新令牌后，即可在指定的时间段内使用刷新令牌流（前提是当前刷新令牌未吊销，并且未使用时间不超过非活动时间）。超过该时间后，用户必须重新完成身份验证才能接收新的会话令牌。

减少最大期限会强制用户更频繁地进行身份验证。由于单重身份验证的安全性不如多重身份验证，因此建议为此策略设置一个小于“多因素刷新令牌最大期限”策略的值。

### 多因素刷新令牌最大期限

**字符串：**MaxAgeMultiFactor

**影响：**刷新令牌

**摘要：**此策略控制用户在上次仅使用多个因素成功完成身份验证后，可以继续使用刷新令牌获取新访问/刷新令牌对的时间长短。用户完成身份验证并收到新刷新令牌后，即可在指定的时间段内使用刷新令牌流（前提是当前刷新令牌未吊销，并且未使用时间不超过非活动时间）。超过该时间后，用户必须重新完成身份验证才能接收新的会话令牌。

减少最大期限会强制用户更频繁地进行身份验证。由于单重身份验证的安全性不如多重身份验证，因此建议为此策略设置一个大于“单因素刷新令牌最大期限”策略的值。

### 单因素会话令牌最大期限

**字符串：**MaxAgeSessionSingleFactor

**影响：**会话令牌（持久性和非持久性）

**摘要：**此策略控制用户在上次仅使用一个因素成功完成身份验证后，可以继续使用会话令牌获取新 ID 和会话令牌的时间长短。用户完成身份验证并收到新会话令牌后，即可在指定的时间段内使用会话令牌流（前提是当前会话令牌未吊销或过期）。超过该时间后，用户必须重新完成身份验证才能接收新的会话令牌。

减少最大期限会强制用户更频繁地进行身份验证。由于单重身份验证的安全性不如多重身份验证，因此建议为此策略设置一个小于“多因素会话令牌最大期限”策略的值。

### 多因素会话令牌最大期限

**字符串：**MaxAgeSessionMultiFactor

**影响：**会话令牌（持久性和非持久性）

**摘要：**此策略控制用户在上次仅使用多个因素成功完成身份验证后，可以继续使用会话令牌获取新 ID 和会话令牌的时间长短。用户完成身份验证并收到新会话令牌后，即可在指定的时间段内使用会话令牌流（前提是当前会话令牌未吊销或过期）。超过该时间后，用户必须重新完成身份验证才能接收新的会话令牌。

减少最大期限会强制用户更频繁地进行身份验证。由于单重身份验证的安全性不如多重身份验证，因此建议为此策略设置一个大于“单因素会话令牌最大期限”策略的值。

## 示例令牌生存期策略

如果能够创建和管理应用、服务主体和整个租户的令牌生存期，就可以在 Azure AD 中实现各种新的方案。下面逐步讲解一些常见的策略方案，帮助针对以下属性实施新规则：


- 令牌生存期
- 令牌最大非活动时间
- 令牌最大期限

逐步讲解的方案包括：


- 管理租户的默认策略
- 为 Web 登录创建策略
- 为调用 Web API 的本机应用创建策略
- 管理高级策略

### 先决条件
以下示例方案针对应用、服务主体和总个租户创建、更新、链接和删除策略。在学习这些示例之前，Azure AD 的新手应该先阅读[此文](/documentation/articles/active-directory-howto-tenant/)帮助自己入门。


1. 若要开始，请下载最新的 [Azure AD PowerShell Cmdlet 预览版](https://www.powershellgallery.com/packages/AzureADPreview)。
2.	获取 Azure AD PowerShell Cmdlet 后，运行 Connect 命令登录到 Azure AD 管理员帐户。每次启动新会话都需要执行此操作。
        
        Connect-AzureAD -Confirm

3.	运行以下命令，查看租户中创建的所有策略。在以下方案中执行大多数操作之后，都应该使用此命令。此命令还可帮助获取策略的**对象 ID**。
        
        Get-AzureADPolicy

### 示例：管理租户的默认策略

本示例将创建一个策略，使用户不需要太频繁地在整个租户中登录。

为此，可以针对应用到整个租户的单因素刷新令牌创建一个令牌生存期策略。此策略将应用到租户中的每个应用程序，以及尚未设置策略的每个服务主体。

1.	**创建令牌生存期策略。**

将单因素刷新令牌设置为“直到吊销”，这意味着在吊销访问权限之前它都不会过期。要创建的策略定义如下：
        
        @("{
          `"TokenLifetimePolicy`":
              {
                 `"Version`":1, 
                 `"MaxAgeSingleFactor`":`"until-revoked`"
              }
        }")

然后，运行以下命令创建此策略。

		
    New-AzureADPolicy -Definition @("{`"TokenLifetimePolicy`":{`"Version`":1, `"MaxAgeSingleFactor`":`"until-revoked`"}}") -DisplayName TenantDefaultPolicyScenario -IsTenantDefault $true -Type TokenLifetimePolicy
		
若要查看新策略并获取其 ObjectID，请运行以下命令。

	Get-AzureADPolicy
&nbsp;&nbsp;2.**更新策略**

假设第一个策略不像服务要求的那样严格，单因素刷新令牌应该在 2 天后过期。运行以下命令。
		
	Set-AzureADPolicy -ObjectId <ObjectID FROM GET COMMAND> -DisplayName TenantDefaultPolicyUpdatedScenario -Definition @("{`"TokenLifetimePolicy`":{`"Version`":1,`"MaxAgeSingleFactor`":`"2.00:00:00`"}}")
		
&nbsp;&nbsp;3.**大功告成！**

### 示例：为 Web 登录创建策略

本示例创建一个要求用户更频繁地在 Web 应用中进行身份验证的策略。此策略将会针对 Web 应用的服务主体设置访问/ID 令牌的生存期以及多因素会话令牌的最大期限。

1.	**创建令牌生存期策略。**

这个用于 Web 登录的策略将访问/ID 令牌生存期和单因素会话令牌最大期限设置为 2 小时。

	New-AzureADPolicy -Definition @("{`"TokenLifetimePolicy`":{`"Version`":1,`"AccessTokenLifetime`":`"02:00:00`",`"MaxAgeSessionSingleFactor`":`"02:00:00`"}}") -DisplayName WebPolicyScenario -IsTenantDefault $false -Type TokenLifetimePolicy

若要查看新策略并获取其 ObjectID，请运行以下命令。

	Get-AzureADPolicy
&nbsp;&nbsp;2.**将策略分配到服务主体。**

接下来，将此新策略链接到服务主体。还需要通过某种方式访问服务主体的 **ObjectId**。可以查询 [Microsoft Graph](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#serviceprincipal-entity) 或转到 [Graph Explorer Tool](https://graphexplorer.cloudapp.net/)，然后登录到 Azure AD 帐户，查看租户的所有服务主体。

获取 **ObjectId** 后，运行以下命令。
        
	Add-AzureADServicePrincipalPolicy -ObjectId <ObjectID of the Service Principal> -RefObjectId <ObjectId of the Policy>
&nbsp;&nbsp;3.**大功告成！**

 

### 示例：为调用 Web API 的本机应用创建策略

>[AZURE.NOTE]
将策略链接到应用程序的功能目前已禁用。我们正在努力，有望很快就会启用。一旦有新功能推出，此页面就会更新。

在本示例中创建的策略不要求用户太频繁地进行身份验证，这会延长用户可保持非活动状态、不必再次身份验证的时间。该策略将应用到 Web API，因此，当本机应用以资源形式请求 Web API 时，将应用此策略。

1.	**创建令牌生存期策略。**

此命令将为 Web API 创建一个严格的策略。
        
	New-AzureADPolicy -Definition @("{`"TokenLifetimePolicy`":{`"Version`":1,`"MaxInactiveTime`":`"30.00:00:00`",`"MaxAgeMultiFactor`":`"until-revoked`",`"MaxAgeSingleFactor`":`"180.00:00:00`"}}") -DisplayName WebApiDefaultPolicyScenario -IsTenantDefault $false -Type TokenLifetimePolicy
         
若要查看新策略并获取其 ObjectID，请运行以下命令。

	Get-AzureADPolicy

&nbsp;&nbsp;2.**将策略分配到 Web API**。

接下来，将此新策略链接到应用程序。还需要通过某种方式访问应用程序的 **ObjectId**。查找应用的 **ObjectId** 的最佳方式是使用 [Azure 门户预览](https://portal.azure.cn/)。

获取 **ObjectId** 后，运行以下命令。

	Add-AzureADApplicationPolicy -ObjectId <ObjectID of the App> -RefObjectId <ObjectId of the Policy>

&nbsp;&nbsp;3.**大功告成！**

### 示例：管理高级策略 

本示例创建一些策略来演示优先级系统的工作原理，以及如何管理已应用到多个对象的多个策略。本示例可让我们深入了解上述策略的优先级，同时有助于管理更复杂的方案。

1.	**创建令牌生存期策略**

现在一切都很简单。我们已经创建了一个将单因素刷新令牌生存期设置为 30 天的租户默认策略。

	New-AzureADPolicy -Definition @("{`"TokenLifetimePolicy`":{`"Version`":1,`"MaxAgeSingleFactor`":`"30.00:00:00`"}}") -DisplayName ComplexPolicyScenario -IsTenantDefault $true -Type TokenLifetimePolicy
若要查看新策略并获取其 ObjectID，请运行以下命令。
 
	Get-AzureADPolicy

&nbsp;&nbsp;2.**将策略分配到服务主体**

现在，已针对整个租户设置了一个策略。假设我们要为特定的服务主体保留这个 30 天策略，但要将租户默认策略更改为上限“直到吊销”。

首先，将此新策略链接到服务主体。还需要通过某种方式访问服务主体的 **ObjectId**。可以查询 [Microsoft Graph](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#serviceprincipal-entity) 或转到 [Graph Explorer Tool](https://graphexplorer.cloudapp.net/)，然后登录到 Azure AD 帐户，查看租户的所有服务主体。

获取 **ObjectId** 后，运行以下命令。

	Add-AzureADServicePrincipalPolicy -ObjectId <ObjectID of the Service Principal> -RefObjectId <ObjectId of the Policy>

&nbsp;&nbsp;3.**使用以下命令将 IsTenantDefault 标志设置为 false**。

	Set-AzureADPolicy -ObjectId <ObjectId of Policy> -DisplayName ComplexPolicyScenario -IsTenantDefault $false
&nbsp;&nbsp;4.**创建新的租户默认策略**

	New-AzureADPolicy -Definition @("{`"TokenLifetimePolicy`":{`"Version`":1,`"MaxAgeSingleFactor`":`"until-revoked`"}}") -DisplayName ComplexPolicyScenarioTwo -IsTenantDefault $true -Type TokenLifetimePolicy

&nbsp;&nbsp;5.**大功告成！**

现在，已将原始策略链接到服务主体，已将新策略设置为租户默认策略。请务必记住，应用到服务主体的策略优先级高于租户默认策略。


## Cmdlet 参考

### 管理策略
以下 cmdlet 可用于管理策略。</br></br>



#### New-AzureADPolicy
创建新策略。

	New-AzureADPolicy -Definition <Array of Rules> -DisplayName <Name of Policy> -IsTenantDefault <boolean> -Type <Policy Type> 

参数|说明|示例|
-----| ----- |-----|
-Definition| 包含所有策略规则的字符串化 JSON 的数组。|-Definition @("{`"TokenLifetimePolicy`":{`"Version`":1,`"MaxInactiveTime`":`"20:00:00`"}}") 
-DisplayName|策略名称的字符串|-DisplayName MyTokenPolicy
-IsTenantDefault|如果为 true，则将策略设置为租户的默认策略；如果为 false，则不执行任何操作|-IsTenantDefault $true
-Type|策略类型，对于令牌生存期，始终使用“TokenLifetimePolicy”|-Type TokenLifetimePolicy
-AlternativeIdentifier [可选]|设置策略的备用 ID。|-AlternativeIdentifier myAltId
</br>

#### Get-AzureADPolicy         
获取所有 AzureAD 策略或指定的策略
        
	Get-AzureADPolicy 

参数|说明|示例|
-----| ----- |-----|
-ObjectId [可选]|要获取的策略的对象 ID。 |-ObjectId &lt;策略的 ObjectID&gt; 
</br>

#### Get-AzureADPolicyAppliedObject         
获取已链接到策略的所有应用和服务主体
        
	Get-AzureADPolicyAppliedObject -ObjectId <object id of policy> 

Parameters|说明|示例|
-----| ----- |-----|
-ObjectId|要获取的策略的对象 ID。|-ObjectId &lt;策略的 ObjectID&gt;
</br>

#### Set-AzureADPolicy
更新现有策略
        
	Set-AzureADPolicy -ObjectId <object id of policy> -DisplayName <string> 
 
参数|说明|示例|
-----| ----- |-----|
-ObjectId|要获取的策略的对象 ID。|-ObjectId &lt;策略的 ObjectID&gt;
-DisplayName|策略名称的字符串|-DisplayName MyTokenPolicy
-Definition [可选]|包含所有策略规则的字符串化 JSON 的数组。|-Definition @("{`"TokenLifetimePolicy`":{`"Version`":1,`"MaxInactiveTime`":`"20:00:00`"}}") 
-IsTenantDefault [可选]|如果为 true，则将策略设置为租户的默认策略；如果为 false，则不执行任何操作|-IsTenantDefault $true
-Type [可选]|策略类型，对于令牌生存期，始终使用“TokenLifetimePolicy”|-Type TokenLifetimePolicy
-AlternativeIdentifier [可选]|设置策略的备用 ID。|-AlternativeIdentifier myAltId
</br>

#### Remove-AzureADPolicy         
删除指定的策略

	Remove-AzureADPolicy -ObjectId <object id of policy>

参数|说明|示例|
-----| ----- |-----|
-ObjectId|要获取的策略的对象 ID。|-ObjectId &lt;策略的 ObjectID&gt;
</br>

### 应用程序策略
以下 cmdlet 可用于应用程序策略。</br></br>

#### Add-AzureADApplicationPolicy         
将指定的策略链接到应用程序

	Add-AzureADApplicationPolicy -ObjectId <object id of application> -RefObjectId <object id of policy>

参数|说明|示例|
-----| ----- |-----|
-ObjectId|应用程序的对象 ID。|-ObjectId &lt;应用程序的 ObjectID&gt; 
-RefObjectId|策略的对象 ID。 |-RefObjectId &lt;策略的 ObjectID&gt;
</br>

#### Get-AzureADApplicationPolicy        
获取已分配到应用程序的策略

	Get-AzureADApplicationPolicy -ObjectId <object id of application>

参数|说明|示例|
-----| ----- |-----|
-ObjectId|应用程序的对象 ID。|-ObjectId &lt;应用程序的 ObjectID&gt; 
</br>

#### Remove-AzureADApplicationPolicy        
从应用程序中删除策略

	Remove-AzureADApplicationPolicy -ObjectId <object id of application> -PolicyId <object id of policy>

参数|说明|示例|
-----| ----- |-----|
-ObjectId|应用程序的对象 ID。|-ObjectId &lt;应用程序的 ObjectID&gt; 
-PolicyId| 策略的 ObjectId。|-PolicyId &lt;策略的 ObjectID&gt;
</br>

### 服务主体策略
以下 cmdlet 可用于服务主体策略。</br></br>

#### Add-AzureADServicePrincipalPolicy         
将指定的策略链接到服务主体

	Add-AzureADServicePrincipalPolicy -ObjectId <object id of service principal> -RefObjectId <object id of policy>

参数|说明|示例|
-----| ----- |-----|
-ObjectId|应用程序的对象 ID。|-ObjectId &lt;应用程序的 ObjectID&gt; 
-RefObjectId|策略的对象 ID。 |-RefObjectId &lt;策略的 ObjectID&gt;
</br>

#### Get-AzureADServicePrincipalPolicy        
获取已链接到指定服务主体的任何策略

	Get-AzureADServicePrincipalPolicy -ObjectId <object id of service principal>
 
参数|说明|示例|
-----| ----- |-----|
-ObjectId|应用程序的对象 ID。|-ObjectId &lt;应用程序的 ObjectID&gt; 
</br>

#### Remove-AzureADServicePrincipalPolicy         
从指定的服务主体中删除策略

	Remove-AzureADServicePrincipalPolicy -ObjectId <object id of service principal>  -PolicyId <object id of policy>
 
参数|说明|示例|
-----| ----- |-----|
-ObjectId|应用程序的对象 ID。|-ObjectId &lt;应用程序的 ObjectID&gt; 
-PolicyId| 策略的 ObjectId。|-PolicyId &lt;策略的 ObjectID&gt;

<!---HONumber=Mooncake_1024_2016-->
