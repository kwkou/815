<properties 
	pageTitle="Azure AD Connect - SSL 证书要求" 
	description="配合 AD FS 使用的 Azure AD Connect SSL 证书所要满足的要求。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="swadhwa" 
	editor="curtand"/>

<tags 
	ms.service="active-directory"  
	ms.date="08/24/2015" 
	wacn.date="11/02/2015"/>

# Azure AD Connect - SSL 证书要求

**重要说明：**强烈建议在你的 AD FS 场的所有节点中以及所有 Web 应用程序代理服务器中使用相同的 SSL 证书。

- 此证书必须是 X509 证书。 
- 在测试实验室环境中，你可以在联合服务器上使用自签名证书；不过，对于生产环境，建议你从某个公共 CA 获取证书。 
	- 如果使用未公开受信任的证书，请确保每个 Web 应用程序代理服务器上安装的证书同时受本地服务器和所有联合服务器的信任 
- 不支持基于 CryptoAPI 下一代 (CNG) 密钥和密钥存储提供者的证书。这意味着，你必须使用基于 CSP（加密服务提供者）而非 KSP（密钥存储提供者）的证书。 
- 证书的标识必须与联合身份验证服务名称（例如 fs.contoso.com）匹配。 
	- 标识是类型为 dNSName 的使用者备用名称 (SAN) 扩展，或者是指定为公用名的使用者名称（当不存在 SAN 条目时）。  
	- 证书中可以存在多个 SAN 条目，但是它们中必须有一个与联合身份验证服务名称匹配。 
	- 如果计划使用“工作区加入”，则还额外需要一个 SAN，它具有值“enterpriseregistration.”，后跟你的组织的用户主体名称 (UPN) 后缀（例如 enterpriseregistration.contoso.com）。 

支持通配符证书。
 

<!---HONumber=67-->