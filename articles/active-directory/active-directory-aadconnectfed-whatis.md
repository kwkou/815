<properties
    pageTitle="Azure AD Connect 和联合身份验证 | Azure"
    description="此页是与使用 Azure AD Connect 进行 AD FS 操作相关的所有文档的中央位置。"
    services="active-directory"
    documentationcenter=""
    author="anandyadavmsft"
    manager="femila"
    editor="" />
<tags
    ms.assetid="f9107cf5-0131-499a-9edf-616bf3afef4d"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="4/9/2016"
    wacn.date="05/22/2017"
    ms.author="anandy"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="e802054eec0f846f226a25293be299abd7b0ba91"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/12/2017" />

# <a name="azure-ad-connect-and-federation"></a>Azure AD Connect 和联合身份验证
Azure Active Directory (Azure AD) Connect 可让你配置使用本地 Active Directory 联合身份验证服务 (AD FS) 和 Azure AD 进行的联合身份验证。 通过联合身份验证登录，你可以让用户能够使用本地密码登录基于 Azure AD 的服务；使用公司网络时，无需再次输入密码就可登录服务。 使用 AD FS 的联合身份验证选项，你可以部署 AD FS 的新安装，也可以在 Windows Server 2012 R2 场中指定现有安装。

本主题是关于 Azure AD Connect 的联合身份验证相关功能的信息主页。 它列出了指向所有相关主题的链接。 有关 Azure AD Connect 的链接，请参阅 [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)。

## <a name="azure-ad-connect-federation-topics"></a>Azure AD Connect：联合身份验证主题
| 主题 | 涵盖的内容和阅读时机 |
|:--- |:--- |
| **Azure AD Connect 用户登录选项** | |
| [了解用户登录选项](/documentation/articles/active-directory-aadconnect-user-signin/) |了解各个用户登录选项以及这些选项如何影响 Azure 登录用户体验。 |
| **使用 Azure AD Connect 安装 AD FS** | |
| [先决条件](/documentation/articles/active-directory-aadconnect-get-started-custom/#ad-fs-configuration-pre-requisites/) |了解通过 Azure AD Connect 成功安装 AD FS 的先决条件。 |
| [配置 AD FS 场](/documentation/articles/active-directory-aadconnect-get-started-custom/#configuring-federation-with-ad-fs/) |使用 Azure AD Connect 安装新的 AD FS 场。 |
| [使用备用登录 ID 与 Azure AD 联合](/documentation/articles/active-directory-aadconnect-federation-management/#alternateid/) | 使用备用登录 ID 配置联合  |
| **修改 AD FS 配置** | |
| [修复信任](/documentation/articles/active-directory-aadconnect-federation-management/#repairthetrust/) |修复当前本地 AD FS 和 Office 365/Azure 之间的信任。 |
| [新增 AD FS 服务器](/documentation/articles/active-directory-aadconnect-federation-management/#addadfsserver/) |初始安装后，使用其他 AD FS 服务器扩展 AD FS 场。 |
| [新增 AD FS WAP 服务器](/documentation/articles/active-directory-aadconnect-federation-management/#addwapserver/) |初始安装后，使用其他 Web 应用程序代理 (WAP) 服务器扩展 AD FS 场。 |
| [添加新的联合域](/documentation/articles/active-directory-aadconnect-federation-management/#addfeddomain/) |添加其他要与 Azure AD 联合的域。 |
| [更新 SSL 证书](/documentation/articles/active-directory-aadconnectfed-ssl-update/)| 更新 AD FS 场的 SSL 证书。 |
| **其他联合配置** | |
| [将 Azure AD 的多个实例与 AD FS 的单个实例联合](/documentation/articles/active-directory-aadconnectfed-single-adfs-multitenant-federation/) | 联合多个 Azure AD 与单个 AD FS 场| 
| [添加自定义公司徽标/插图](/documentation/articles/active-directory-aadconnect-federation-management/#customlogo/) |通过指定 AD FS 登录页上显示的自定义徽标来修改登录体验。 |
| [添加登录说明](/documentation/articles/active-directory-aadconnect-federation-management/#addsignindescription/) |更改 AD FS 登录页上的登录说明。 |
| [修改 AD FS 声明规则](/documentation/articles/active-directory-aadconnect-federation-management/#modclaims/) |在 AD FS 中修改或添加对应于 Azure AD Connect 同步配置的声明规则。 |


## <a name="additional-resources"></a>其他资源
- [联合两个 Azure AD 与单个 AD FS](/documentation/articles/active-directory-aadconnectfed-single-adfs-multitenant-federation/)
- [Azure 中的 AD FS 部署](/documentation/articles/active-directory-aadconnect-azure-adfs/)
- [使用 Azure 流量管理器在 Azure 中部署高可用性跨地域 AD FS](/documentation/articles/active-directory-adfs-in-azure-with-azure-traffic-manager/)

<!---Update_Description: wording update -->