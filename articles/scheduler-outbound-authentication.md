<properties 
 pageTitle="计划程序出站身份验证" 
 description="" 
 services="scheduler" 
 documentationCenter=".NET" 
 authors="krisragh" 
 manager="dwrede" 
 editor=""/>
<tags 
 ms.service="scheduler" 
 ms.date="03/09/2016"
 wacn.date="04/11/2016"/>
 
# 计划程序出站身份验证

计划程序作业可能需要调用要求进行身份验证的服务。这样，被调用的服务可以确定计划程序作业是否可以访问其资源。其中的某些服务包括其他 Azure 服务、Salesforce.com、Facebook 和安全的自定义网站。

## 添加和删除身份验证

将身份验证添加到计划程序作业非常简单 – 只需在创建或更新作业时将一个 JSON 子元素 `authentication` 添加到 `request` 元素。在 PUT、PATCH 或 POST 请求中（作为 `authentication` 对象的一部分）传递给计划程序服务的机密永远不会在响应中返回。在响应中，机密信息将设置为 null，或者包含一个表示已经过身份验证的实体的公共令牌。

若要删除身份验证，请对作业显式执行 PUT 或 PATCH，并将 `authentication` 对象设置为 null。响应中不会传回任何身份验证属性。

目前，支持的身份验证类型仅包括 `ClientCertificate` 模型（表示使用 SSL/TLS 客户端证书）、`Basic` 模型（表示基本身份验证）和 `ActiveDirectoryOAuth` 模型（表示 Active Directory OAuth 身份验证）。

## ClientCertificate 身份验证的请求正文

使用 `ClientCertificate` 模型添加身份验证时，请在请求正文中指定以下附加元素。

|元素|说明|
|:---|:---|
|authentication（父元素）|用于使用 SSL 客户端证书的身份验证对象。|
|type|必需。身份验证的类型。对于 SSL 客户端证书，该值必须是 `ClientCertificate`。|
|pfx|必需。PFX 文件的 Base64 编码内容。|
|password|必需。用于访问 PFX 文件的密码。|


## ClientCertificate 身份验证的响应正文

发送包含身份验证信息的请求时，响应将包含以下与身份验证相关的元素。

|元素 | 说明 |
|:--|:--|
|authentication（父元素） |用于使用 SSL 客户端证书的身份验证对象。|
|type |身份验证的类型。对于 SSL 客户端证书，该值为 `ClientCertificate`。|
|certificateThumbprint |证书的指纹。|
|certificateSubjectName |证书的使用者可分辨名称。|
|certificateExpiration |证书的过期日期。|

## 基本身份验证的请求正文

使用 `Basic` 模型添加身份验证时，请在请求正文中指定以下附加元素。

|元素|说明|
|:--|:--|
|authentication（父元素） |用于使用基本身份验证的身份验证对象。|
|type |必需。身份验证的类型。对于基本身份验证，该值必须是 `Basic`。|
|username |必需。要进行身份验证的用户名。|
|password |必需。要进行身份验证的密码。|

## 基本身份验证的响应正文

发送包含身份验证信息的请求时，响应将包含以下与身份验证相关的元素。

|元素|说明|
|:--|:--|
|authentication（父元素） |用于使用基本身份验证的身份验证对象。|
|type |身份验证的类型。对于基本身份验证，该值为 `Basic`。|
|username |经过身份验证的用户名。|

## ActiveDirectoryOAuth 身份验证的请求正文

使用 `ActiveDirectoryOAuth` 模型添加身份验证时，请在请求正文中指定以下附加元素。

|元素 |说明 |
|:--|:--|
|authentication（父元素） |用于使用 ActiveDirectoryOAuth 身份验证的身份验证对象。|
|type |必需。身份验证的类型。对于 ActiveDirectoryOAuth 身份验证，该值必须是 `ActiveDirectoryOAuth`。|
|tenant |必需。Azure AD 租户的租户标识符。|
|audience |必需。此元素设置为 https://management.core.chinacloudapi.cn/.|。
|clientId |必需。为 Azure AD 应用程序提供客户端标识符。|
|secret |必需。正在请求令牌的客户端的机密。|

### 确定你的租户标识符

可以通过在 Azure PowerShell 中运行 `Get-AzureAccount`，找到 Azure AD 租户的租户标识符。

## ActiveDirectoryOAuth 身份验证的响应正文

发送包含身份验证信息的请求时，响应将包含以下与身份验证相关的元素。

|元素 |说明 |
|:--|:--|
|authentication（父元素） |用于使用 ActiveDirectoryOAuth 身份验证的身份验证对象。|
|type |身份验证的类型。对于 ActiveDirectoryOAuth 身份验证，该值为 `ActiveDirectoryOAuth`。|
|tenant |Azure AD 租户的租户标识符。|
|audience |此元素设置为 https://management.core.chinacloudapi.cn/.|。
|clientId |AD 应用程序的客户端标识符。|

## 另请参阅
 

 [计划程序是什么？](/documentation/articles/scheduler-intro/)
 
 [Azure 计划程序的概念、术语和实体层次结构](/documentation/articles/scheduler-concepts-terms/)
 
 [开始在管理门户中使用计划程序](/documentation/articles/scheduler-get-started-portal/)
 
 [Azure 计划程序中的计划和计费](/documentation/articles/scheduler-plans-billing/)
 [Azure 计划程序 REST API 参考](https://msdn.microsoft.com/zh-CN/library/dn528946)
 
 [Azure 计划程序 PowerShell cmdlet 参考](/documentation/articles/scheduler-powershell-reference/)
 [Azure 计划程序高可用性和可靠性](/documentation/articles/scheduler-high-availability-reliability/)
 
 [Azure 计划程序的限制、默认值和错误代码](/documentation/articles/scheduler-limits-defaults-errors/)


  

 
  

<!---HONumber=Mooncake_0405_2016-->