<properties
   pageTitle="Azure AD Reporting API 入门"
   description="如何开始使用 Azure Active Directory Reporting API"
   services="active-directory"
   documentationCenter=""
   authors="yossib"
   manager="mbaldwin"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="05/12/2015" 
   wacn.date="06/16/2015"/>


# Azure AD Reporting API 入门

Azure AD 包含一个 Reporting API，可让你以编程方式访问安全、活动和审核报告。它是基于 REST 的 API，允许你查询要在外部报告或法规遵从事务中使用的数据。

本文将引导你完成必要的步骤，以便对 Azure AD Reporting API 发出第一个身份验证 HTTP 请求。

你将需要：

- [Azure Active Directory](active-directory-whatis)
- 发出 HTTP GET 和 POST 请求的方法，可以是：
	- 具有 [curl](http://curl.haxx.se/) 的 bash shell
	- [Postman](https://www.getpostman.com/)
	- [用于发出 HTTP 请求的 PowerShell cmdlet](https://technet.microsoft.com/zh-cn/library/hh849901.aspx)



## 创建 Azure AD 应用程序以访问 API

为了对 Reporting API 进行身份验证，我们必须使用 OAuth 流，这需要将应用程序注册到 Azure AD。



### 创建应用程序
- 导航到 [Azure 管理门户](https://manage.windowsazure.cn/)
- 导航到你的目录
- 导航到应用程序
- 在底部栏上，单击“添加”。
	- 单击“添加我的组织正在开发的应用程序”。
	- **名称**：任意名称即可。建议输入类似于“Reporting API Application”的内容。
	- **类型**：选择“Web 应用程序和/或 Web API”
	- 单击箭头转到下一页。
	- **登录 URL**：```http://localhost```
	- **应用程序 ID URI**：```http://localhost```
	- 单击复选标记完成添加应用程序。

### 授予应用程序使用 API 的权限
- 导航到“应用程序”选项卡。
- 导航到新建的应用程序。
- 导航到“配置”选项卡。
- 在“针对其他应用程序的权限”部分中：
	- “添加 Microsoft Azure Active Directory”>“应用程序权限”> 启用“读取目录数据”
	- “添加 Microsoft Azure 服务管理 API”>“委托的权限”> 启用“访问 Azure 服务管理”
- 在底部栏上，单击“保存”。


### 获取目录 ID、客户端 ID 和客户端机密

查找应用程序的客户端 ID、客户端机密和你的目录 ID。将这些 ID 和 URL 复制到一个单独的位置；后面的步骤要用到它们。

#### 应用程序客户端 ID
- 导航到“应用程序”选项卡。
- 导航到新建的应用程序。
- 导航到“配置”选项卡。
- 应用程序的客户端 ID 列在“客户端 ID”字段中。

#### 应用程序客户端机密
- 导航到“应用程序”选项卡。
- 导航到新建的应用程序。
- 导航到“配置”选项卡。
- 通过在“密钥”部分中选择一个持续时间来为应用程序生成新密钥。
- 保存后，将显示该密钥。请确保复制该密钥，因为以后无法检索它。

#### 目录 ID
- 在登录 Azure 管理门户后，你可以在 URL 中找到你的目录 ID。
- 示例 URL：```https://manage.windowsazure.cn/@demo.partner.onmschina.cn#Workspaces/ActiveDirectoryExtension/Directory/<<YOUR-AZURE-AD-DIRECTORY-ID>/apps...```

将这些 ID 和 URL 复制到一个单独的位置；后面的步骤要用到它们。



## 从 Azure AD 检索 OAuth 访问令牌

接下来，我们需要生成一个授权代码，用于检索对 Reporting API 发出的请求的 OAuth 访问令牌。



### 获取授权代码

首先，需要一个授权代码。若要检索该代码，可以在浏览器中导航到特定 URL，以目录中的全局管理员身份登录，然后从你重定向到的 URL 中检索授权代码。

- 请将此 URL 替换为 Azure AD 目录 ID 和应用程序客户端 ID。
	- ```https://login.microsoftonline.com/<<INSERT-YOUR-AZURE-AD-DIRECTORY-ID-HERE>/oauth2/authorize?client_id=<<INSERT-YOUR-APPLICATION-CLIENT-ID-HERE>&response_type=code```
- 在填写字段后，打开一个浏览器窗口并导航到该 URL。浏览器将重定向到包含访问代码的 URL；不会有任何页面内容。这是正常的。 

- 出现提示时，请以目录中的全局管理员身份登录。
	- 如果遇到问题，你可能需要从 Azure 管理门户或 Office 门户注销，然后重试。
- 检查重定向页面的 URL。该 URL 包含授权代码。
	- ```http://localhost/?code=<<YOUR-AUTHORIZATION-CODE>&session_state=<<YOUR-SESSION-STATE>``` 
- 将 ```YOUR-AUTHORIZATION-CODE``` 复制到单独的位置；后面的步骤将要用到它。



### 获取 OAuth 访问令牌

接下来，你将通过使用授权令牌向 OAuth 终结点发出 HTTP 请求来检索访问令牌。对于本示例，我们将使用一个名为 [curl](http://curl.haxx.se/) 的小型 unix 库；你也可以使用 [Postman](https://www.getpostman.com/) 或者[能够发出 HTTP 请求的 PowerShell cmdlet](https://technet.microsoft.com/zh-cn/library/hh849901.aspx)。

- 首先，将 ```YOUR-AZURE-AD-DIRECTORY-ID``` 替换为你在上一步中检索的目录 ID。然后，将 ```YOUR-CLIENT-ID``` 替换为你在上一步中检索的客户端 ID。然后，将 ```YOUR-CLIENT-SECRET``` 替换为你在上一步中检索的客户端机密。最后，将 ```YOUR-AUTHORIZATION-CODE``` 替换为你在上一步中检索的授权代码。

```
curl -X POST https://login.microsoftonline.com/<<INSERT-YOUR-AZURE-AD-DIRECTORY-ID-HERE>/oauth2/token  \
	-F redirect_uri=http://localhost
	-F grant_type=authorization_code 
	-F resource=https://management.core.chinacloudapi.cn/
	-F client_id=<<INSERT-YOUR-CLIENT-ID-HERE>
	-F code=<<INSERT-YOUR-AUTHORIZATION-CODE-HERE>
	-F client_secret=<<INSERT-YOUR-CLIENT-SECRET-HERE>
```

- 使用所选的方法运行命令或发出请求。
- 你将收到一个包含访问令牌的 JSON 对象。

```
{
  "token_type": "Bearer",
  "expires_in": "3600",
  "expires_on": "1428701563",
  "not_before": "1428697663",
  "resource": "https://management.core.chinacloudapi.cn/",
  "access_token": "<<YOUR-ACCESS-TOKEN>",
  "refresh_token": "AAABA...WjCAA",
  "scope": "user_impersonation",
  "id_token": "eyJ0e...20ifQ."
}
```

- 将 JSON 对象中的访问令牌复制到单独的位置；我们将会使用它来调用 Reporting API。



## 向 Reporting API 发出请求

- 最后，在以下 curl 请求中，将 ```YOUR-ACCESS-TOKEN``` 替换为你的访问令牌。

```
curl -v https://graph.chinacloudapi.cn/<<INSERT-YOUR-DIRECTORY-ID-HERE>/reports/$metadata?api-version=beta
  -H "x-ms-version: 2013-08-01"
  -H "Authorization: Bearer <<INSERT-YOUR-ACCESS-TOKEN-HERE>"
```

- 使用所选的方法运行命令或发出请求。
- 你将收到一个包含审核报告内容的 OData 对象。

```
{
  "@odata.context":"https://graph.chinacloudapi.cn/<<DIRECTORY-ID>/reports/?api-version=beta/reports/$metadata#auditEvents",
  "value":[
    {
      "id":"SN2GR1RDS104.GRN001.msoprd.msft.net_4515449","timeStampOffset":"2015-04-13T21:27:55.1777659Z","actor":"thekenhoff_outlook.com#EXT#@kenhoffdemo.partner.onmschina.cn","action":"Add service principal","target":"04670e0d84264acb86dac2
ff1a94c9d7","actorDetail":"Other=284c417b-805e-493a-ad8e-328ce8d4b18e; UPN=thekenhoff_outlook.com#EXT#@kenhoffdemo.partner.onmschina.cn; PUID=1003BFFD8CD09753","targetDetail":"SPN=04670e0d84264acb86dac2ff1a94c9d7","tenantId":"c9b13f49-3c25-43
c0-a84f-57faf131dc2b"
    }
  ]
}
```

- 祝贺你！


## 后续步骤
- 想要知道可以使用哪些安全、审核和活动报告吗？ 查看 [Azure AD 安全、审核和活动报告](active-directory-view-access-usage-reports)
- 有关审核报告的详细信息，请参阅 [Azure AD 审核报告事件](active-directory-reporting-audit-events)
- 有关 Graph API REST 服务的详细信息，请参阅 [Azure AD 报告和事件（预览版）](https://msdn.microsoft.com/zh-cn/library/azure/mt126081.aspx)
- 有关在 Azure AD 中使用 curl 执行 OAuth 流程的详细信息：[Microsoft Azure REST API + OAuth 2.0](https://ahmetalpbalkan.com/blog/azure-rest-api-with-oauth2/)（外部链接）

<!---HONumber=60-->