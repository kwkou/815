<properties
	pageTitle="访问防火墙后面的密钥保管库 | Azure"
	description="了解如何从防火墙后面的应用程序访问密钥保管库"
	services="key-vault"
	documentationCenter=""
	authors="amitbapat"
	manager="mbaldwin"
	tags="azure-resource-manager"/>  


<tags
	ms.service="key-vault"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="hero-article"
	ms.date="09/13/2016"
	wacn.date="10/19/2016"
	ms.author="ambapat"/>  


# 访问防火墙后面的密钥保管库
### 问：我的密钥保管库客户端应用程序需要位于防火墙后面，我应该开放哪些端口/主机/IP 地址应才能访问密钥保管库？

若要访问密钥保管库，密钥保管库客户端应用程序需要能够访问多个终结点才能使用各种功能。

- 身份验证（通过 Azure Active Directory）
- 通过 Azure Resource Manager 管理密钥保管库（其中包括创建/读取/更新/删除以及设置访问策略）
- 通过密钥保管库特定终结点（例如 https://yourvaultname.vault.chinacloudapi.cn ），访问和管理密钥保管库自身存储的对象（密钥和密码）。

根据配置和环境，会有一些变化。

## 端口

针对全部 3 个功能（身份验证、管理和数据平面访问）的所有密钥保管库流量都会通过 HTTPS：端口 443 传递。但是，对于 CRL，偶尔会有 HTTP（端口 80）流量。支持 OCSP 的客户端不应到达 CRL，但可能偶尔会到达 [http://cdp1.public-trust.com/CRL/Omniroot2025.crl](http://cdp1.public-trust.com/CRL/Omniroot2025.crl)。

## 身份验证

密钥保管库客户端应用程序需要访问 Azure Active Directory 终结点进行身份验证。使用的终结点取决于 AAD 租户配置、主体类型（用户主体、服务主体）和帐户类型（例如 Microsoft 帐户或组织 ID）。

| 主体类型 | 终结点：端口 |
|----------------|---------------|
| 使用 Microsoft 帐户的用户<br>（例如 user@hotmail.com） | **全球：**<br>login.microsoftonline.com:443<br><br> **Azure China：**<br>login.chinacloudapi.cn:443<br><br>**Azure US Government：**<br>login-us.microsoftonline.com:443<br><br>**Azure Germany：**<br>login.microsoftonline.de:443<br><br> 和 <br>login.live.com:443 |
| 使用具有 AAD 的组织 ID 的用户/服务主体（例如 user@contoso.com） | **全球：**<br>login.microsoftonline.com:443<br><br> **Azure China：**<br>login.chinacloudapi.cn:443<br><br>**Azure US Government：**<br>login-us.microsoftonline.com:443<br><br>**Azure Germany：**<br>login.microsoftonline.de:443 |
| 使用组织 ID + ADFS 或其他联合终结点的用户/服务主体（例如 user@contoso.com） | 所有上述组织 ID 终结点加上 ADFS 或其他联合终结点 |

还有其他可能的复杂情况。有关其他信息，请参阅 [Azure Active Directory 身份验证流](/documentation/articles/active-directory-authentication-scenarios/)、[将应用程序与 Azure Active Directory 集成](/documentation/articles/active-directory-integrating-applications/)和 [Active Directory 身份验证协议](/documentation/articles/active-directory-developers-guide/)。

## 密钥保管库管理

对于密钥保管库管理（CRUD 和设置访问策略），密钥保管库客户端应用程序需要访问 Azure Resource Manager 终结点。

| 操作类型 | 终结点：端口 |
|----------------|---------------|
| 通过 Azure Resource Manager 的密钥保管库控制平面操作<br> | **全球：**<br>management.azure.com:443<br><br> **Azure China：**<br>management.chinacloudapi.cn:443<br><br> **Azure US Government：**<br>management.usgovcloudapi.net:443<br><br> **Azure Germany：**<br>management.microsoftazure.de:443 |
| Azure Active Directory 图形 API | **全球：**<br>graph.chinacloudapi.cn:443<br><br> **Azure China：**<br>graph.chinacloudapi.cn:443<br><br> **Azure US Government：**<br>graph.chinacloudapi.cn:443<br><br> **Azure Germany：**<br>graph.cloudapi.de:443 |

## 密钥保管库操作

对于所有密钥保管库对象（密钥和密码）管理和加密操作，密钥保管库客户端需要访问密钥保管库终结点。根据密钥保管库的位置，终结点 DNS 后缀会有所不同。密钥保管库终结点的格式为 <vault-name>.<region-specific-dns-suffix>，如下表中所述。

| 操作类型 | 终结点：端口 |
|----------------|---------------|
| 密钥保管库操作，如加密密钥操作，创建/读取/更新/删除密钥和密码、设置/获取标记和密钥保管库对象（密钥/密码）的其他属性 | **全球：**<br> &lt;vault-name&gt;.vault.chinacloudapi.cn:443<br><br> **Azure China：**<br> &lt;vault-name&gt;.vault.azure.cn:443<br><br> **Azure US Government：**<br> &lt;vault-name&gt;.vault.usgovcloudapi.net:443<br><br> **Azure Germany：**<br> &lt;vault-name&gt;.vault.microsoftazure.de:443 |

## IP 地址范围

密钥保管库服务会接着使用其他 Azure 资源（如 PaaS 基础结构），因此不可能提供密钥保管库服务终结点在任意给定时间会具有的特定 IP 地址范围。但是，如果防火墙仅支持 IP 地址范围，请参阅 [Azure 数据中心 IP 范围](https://www.microsoft.com/en-us/download/details.aspx?id=41653)文档。对于身份验证和标识 (Azure Active Directory)，应用程序必须能够连接到[身份验证和标识地址](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2)中所述的终结点。

## 后续步骤

- 如果有关于密钥保管库的问题，请访问 [Azure 密钥保管库论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=AzureKeyVault)

<!---HONumber=Mooncake_1010_2016-->
