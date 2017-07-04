<properties
    pageTitle="访问防火墙后面的密钥保管库 | Azure"
    description="了解如何从防火墙后面的应用程序访问 Azure 密钥保管库"
    services="key-vault"
    documentationcenter=""
    author="amitbapat"
    manager="mbaldwin"
    tags="azure-resource-manager" />
<tags
    ms.assetid="50d21774-2ee1-4212-8995-570c9de603c5"
    ms.service="key-vault"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="hero-article"
    ms.date="01/07/2017"
    wacn.date="04/11/2017"
    ms.author="ambapat" />  


# 访问防火墙后面的 Azure 密钥保管库
### 问：我需要将密钥保管库客户端应用程序置于防火墙后。我应该打开什么样的端口、主机或 IP 地址来启用密钥保管库访问权限？
若要访问密钥保管库，密钥保管库客户端应用程序必须访问多个终结点才能使用各种功能：

- 通过 Azure Active Directory \(Azure AD\) 进行身份验证。
- 管理 Azure 密钥保管库。这包括通过 Azure资源管理器创建、读取、更新、删除和设置访问策略。
- 通过密钥保管库特定终结点（例如 https://yourvaultname.vault.azure.cn），访问和管理密钥保管库自身存储的对象（密钥和机密）。

根据配置和环境，会有一些变化。

## 端口
针对全部三项功能（身份验证、管理和数据平面访问）的所有密钥保管库流量都会通过 HTTPS：端口 443 传递。但是，对于 CRL，偶尔会有 HTTP（端口 80）流量。支持 OCSP 的客户端不应到达 CRL，但可能偶尔会到达 [http://cdp1.public-trust.com/CRL/Omniroot2025.crl](http://cdp1.public-trust.com/CRL/Omniroot2025.crl)。

## 身份验证
密钥保管库客户端应用程序需要访问 Azure Active Directory 终结点进行身份验证。使用的终结点取决于 Azure AD 租户配置、主体类型（用户主体或服务主体）和帐户类型--例如 Microsoft 帐户或者工作或学校帐户。

| 主体类型 | 终结点：端口 |
| --- | --- |
| 使用 Microsoft 帐户的用户<br>（例如 user@hotmail.com） |**全球：**<br>login.microsoftonline.com:443<br><br> **Azure China：**<br>login.chinacloudapi.cn:443<br><br>**Azure US Government：**<br>login-us.microsoftonline.com:443<br><br>**Azure Germany：**<br>login.microsoftonline.de:443<br><br> 和 <br>login.live.com:443 |
| 通过 Azure AD 使用工作或学校帐户（例如 user@contoso.com）的用户或服务主体 |**全球：**<br>login.microsoftonline.com:443<br><br> **Azure China：**<br>login.chinacloudapi.cn:443<br><br>**Azure US Government：**<br>login-us.microsoftonline.com:443<br><br>**Azure Germany：**<br>login.microsoftonline.de:443 |
| 使用工作或学校帐户以及 Active Directory 联合身份验证服务 \(AD FS\) 或其他联合终结点（例如 user@contoso.com）的用户或服务主体 |适用于工作或学校帐户的所有终结点，以及 AD FS 或其他联合终结点 |

还有其他可能的复杂情况。有关其他信息，请参阅 [Azure Active Directory 身份验证流](/documentation/articles/active-directory-authentication-scenarios/)、[将应用程序与 Azure Active Directory 集成](/documentation/articles/active-directory-integrating-applications/)以及 [Active Directory 身份验证协议](/documentation/articles/active-directory-developers-guide/)。

## 密钥保管库管理
对于密钥保管库管理（CRUD 和设置访问策略），密钥保管库客户端应用程序需要访问 Azure资源管理器终结点。

| 操作类型 | 终结点：端口 |
| --- | --- |
| 通过 Azure资源管理器进行的密钥保管库控制平面操作 |**全球：**<br>management.azure.com:443<br><br> **Azure China：**<br>management.chinacloudapi.cn:443<br><br> **Azure US Government：**<br>management.usgovcloudapi.net:443<br><br> **Azure Germany：**<br>management.microsoftazure.de:443 |
| Azure Active Directory 图形 API |**全球：**<br>graph.chinacloudapi.cn:443<br><br> **Azure China：**<br>graph.chinacloudapi.cn:443<br><br> **Azure US Government：**<br>graph.chinacloudapi.cn:443<br><br> **Azure Germany：**<br>graph.cloudapi.de:443 |

## 密钥保管库操作
对于所有密钥保管库对象（密钥和机密）管理和加密操作，密钥保管库客户端需要访问密钥保管库终结点。根据密钥保管库的位置，终结点 DNS 后缀会有所不同。密钥保管库终结点的格式为 *vault-name*.*region-specific-dns-suffix*，如下表所述。

| 操作类型 | 终结点：端口 |
| --- | --- |
| 操作包括：对密钥的加密操作；创建、读取、更新和删除密钥和机密；设置或获取密钥保管库对象（密钥或机密）上的标记和其他属性 |**全球：**<br> &lt;vault-name&gt;.vault.azure.net:443<br><br> **Azure China：**<br> &lt;vault-name&gt;.vault.azure.cn:443<br><br> **Azure US Government：**<br> &lt;vault-name&gt;.vault.usgovcloudapi.net:443<br><br> **Azure Germany：**<br> &lt;vault-name&gt;.vault.microsoftazure.de:443 |

## IP 地址范围
密钥保管库服务使用其他 Azure 资源，例如 PaaS 基础结构。因此，不可能提供密钥保管库服务终结点在任意特定时间会有的特定 IP 地址范围。如果防火墙仅支持 IP 地址范围，请参阅 [Azure Datacenter IP Ranges](https://www.microsoft.com/en-us/download/details.aspx?id=42064)（Azure 数据中心 IP 范围）文档。对于身份验证和标识 \(Azure Active Directory\)，应用程序必须能够连接到[身份验证和标识地址](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2)中所述的终结点。

## 后续步骤
如果在密钥保管库方面有任何问题，请访问 [Azure 密钥保管库论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=AzureKeyVault)。

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: wording update-->