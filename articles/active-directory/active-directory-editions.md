<properties
    pageTitle="Azure Active Directory 版本 | Azure"
    description="本文介绍 Azure Active Directory 的免费版和付费版选项。Azure Active Directory 基本版、Azure Active Directory Premium P1 和 Azure Active Directory Premium P2 是付费版本。"
    services="active-directory"
    documentationcenter=""
    author="curtand"
    manager="femila"
    editor="" />
<tags
    ms.assetid="dcaf8939-7633-40a8-bd76-27dedbb6083a"
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/13/2017"
    wacn.date="03/07/2017"
    ms.author="curtand" />  


# Azure Active Directory 版本
所有 Microsoft Online 业务服务都依赖于 Azure Active Directory (Azure AD) 进行登录及满足其他标识需求。如果订阅了任何 Microsoft Online 业务服务（例如，Office 365 或 Azure），则表示已获得 Azure AD，可用于访问下述所有免费功能。

Azure Active Directory 是一个综合性的、高度可用的标识和访问管理云解决方案，它将核心目录服务、高级标识监管和应用程序访问管理结合起来。Azure Active Directory 还提供了功能丰富的基于标准的平台，该平台支持开发人员根据集中的策略和规则为应用程序提供访问控制。使用 Azure Active Directory 免费版，可以管理用户和组、与本地目录同步，以及在 Azure、Office 365 和数千种主流 SaaS 应用程序（如 Salesforce、Workday、Concur、DocuSign、Google Apps、Box、ServiceNow、Dropbox 等）上单一登录。若要了解有关 Azure Active Directory 的详细信息，请阅读[什么是 Azure AD？](/documentation/articles/active-directory-whatis/)

若要增强 Azure Active Directory，可以使用 Azure Active Directory 基本版、Premium P1 版和 Premium P2 版添加付费功能。Azure Active Directory 付费版建立在现有免费目录基础之上，提供企业级功能，包括自助服务、增强型监视、安全报告、多重身份验证 (MFA) 和移动工作者安全访问。

Office 365 订阅包括以下比较表中所述的其他 Azure Active Directory 功能。

> [AZURE.NOTE]
有关这些版本的定价选项，请参阅 [Azure Active Directory 定价](/pricing/details/identity/)。中国地区目前不支持 Azure Active Directory Premium P1 版、Premium P2 版和 Azure Active Directory 基本版。有关详细信息，请通过 Azure Active Directory 论坛与我们联系。
>
>

- **Azure Active Directory 基本版** - 面向具有云优先需求的任务工作者，此版本提供以云为中心的应用程序访问和自助标识管理解决方案。使用 Azure Active Directory 基本版，可以通过以下功能增强工作效率和缩减成本，例如基于组的访问管理、用于云应用程序的自助密码重置、Azure Active Directory 应用程序代理（使用 Azure Active Directory 发布本地 Web 应用程序），这都由 99.9％ 正常运行时间的企业级 SLA 作为坚实后盾。
- **Azure Active Directory Premium P1 版** - Azure Active Directory Premium 版添加了丰富的企业级标识管理功能，并允许各类用户无缝访问本地与云功能，旨在满足组织更加严苛的标识和访问管理需求。此版本为混合环境中的信息工作者和标识管理员提供一切必要的功能，让他们执行应用程序访问、自助标识和访问管理 (IAM)、标识保护以及实现云中的安全性。它支持动态组和自助服务组管理等高级管理与委派资源。它包含 Microsoft 标识管理器（一个本地标识与访问管理套件），并提供云写回功能，使本地用户能够使用自助密码重置等解决方案。
- **Azure Active Directory Premium P2** - 这款新产品包含 Azure AD Premium P1，以及新的 Identity Protection 和 Privileged Identity Management 中的所有功能，旨在为所有用户和管理员提供高级保护。Azure Active Directory Identity Protection 利用数十亿信号，针对应用程序和重要公司数据提供基于风险的条件访问。此外，我们还使用 Azure Active Directory Privileged Identity Management 帮助管理和保护特权帐户，方便发现、限制和监视管理员以及他们对资源的访问，并在需要时提供实时访问。

> [AZURE.NOTE]
许多 Azure Active Directory 功能以“即付即用”版本的形式提供：
> * 可以通过基于用户或基于身份验证的提供程序使用 Azure 多重身份验证。有关详细信息，请参阅[什么是 Azure 多重身份验证？](/documentation/articles/multi-factor-authentication/)
>
>

## 比较正式推出的功能
> [AZURE.NOTE]
有关此数据的不同视图，请参阅 [Azure Active Directory 功能](https://www.microsoft.com/en/server-cloud/products/azure-active-directory/features.aspx)。
>
>

**常用功能**

- [目录对象](#directory-objects)
- [用户/组管理（添加/更新/删除）/基于用户的预配、设备注册](#usergroup-management-addupdatedelete-user-based-provisioning-device-registration)
- [单一登录 (SSO)](#single-sign-on-sso)
- [云用户的自助密码更改](#self-service-password-change-for-cloud-users)
- [Connect（将本地目录扩展到 Azure Active Directory 的同步引擎）](#connect-sync-engine-that-extends-on-premises-directories-to-azure-active-directory)
- [安全性/使用情况报告](#securityusage-reports)

**Basic 功能**

- 基于组的访问管理/预配
- [云用户的自助密码重置](#self-service-password-reset-for-cloud-users)
- [公司品牌（登录页/访问面板自定义）](#company-branding-logon-pagesaccess-panel-customization)
- [应用程序代理](#application-proxy)
- [SLA 99.9%](#sla-999)

**Premium P1 功能**

- [自助组和应用管理/自助应用程序添加件/动态组](#self-service-group)
- [通过本地写回实现自助密码重置/更改/解锁](#self-service-password-resetchangeunlock-with-on-premises-write-back)
- [多重身份验证（云和本地（MFA 服务器））](#multi-factor-authentication-cloud-and-on-premises-mfa-server)
- [MIM CAL + MIM 服务器](#mim-cal-mim-server)
- Cloud App Discovery
- Connect Health
- [组帐户的自动密码滚动更新](#automatic-password-rollover-for-group-accounts)

**高级 P2 功能**

- 标识保护
- Privileged Identity Management

## 常用功能
#### 目录对象 <a name="directory-objects"></a>
**类型：**常用功能

默认使用配额为 150,000 个对象。对象是目录服务中的一个实体，由其唯一可分辨名称表示。举例来说，对象就是用于身份验证的用户条目。如果需要超过此默认配额，请与支持人员联系。50 万个对象限制不适用于 Office 365、Microsoft Intune 或任何其他依赖 Azure Active Directory 提供目录服务的 Microsoft 付费联机服务。

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| 最多 500,000 个对象 |无对象限制 |无对象限制 |Office 365 用户帐户没有对象限制 |

#### 用户/组管理（添加/更新/删除）/基于用户的预配、设备注册 <a name="usergroup-management-addupdatedelete-user-based-provisioning-device-registration"></a>
**类型：**常用功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| ![勾选标记][12] |![勾选标记][12] |![勾选标记][12] |![勾选标记][12] |

**更多详细信息：**

- [管理 Azure AD 目录](/documentation/articles/active-directory-administer/)
#### 单一登录 (SSO) <a name="single-sign-on-sso"></a>
**类型：**常用功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| 每个用户 10 个应用 (1) |每个用户 10 个应用 (1) |无限制 (2) |每个用户 10 个应用 (1) |

1. 使用 Azure AD 免费版和 Azure AD 基本版，最终用户可以通过单一登录访问多达 10 个应用程序。
2. 使用应用程序库菜单中提供的模板，自助集成支持 SAML、SCIM 或基于窗体的身份验证的任何应用程序。

**更多详细信息：**

- [使用 Azure Active Directory (AD) 管理应用程序](/documentation/articles/active-directory-enable-sso-scenario/)

#### 云用户的自助密码更改 <a name="self-service-password-change-for-cloud-users"></a>
**类型：**常用功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| ![勾选标记][12] |![勾选标记][12] |![勾选标记][12] |![勾选标记][12] |

**更多详细信息：**

- [如何更新自己的密码](/documentation/articles/active-directory-passwords-update-your-own-password/)

#### Connect（将本地目录扩展到 Azure Active Directory 的同步引擎）<a name="connect-sync-engine-that-extends-on-premises-directories-to-azure-active-directory"></a>
**类型：**常用功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| ![勾选标记][12] |![勾选标记][12] |![勾选标记][12] |![勾选标记][12] |

**更多详细信息：**

- [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)

#### 安全/使用情况报告 <a name="securityusage-reports"></a>
**类型：**常用功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| 3 个基本报告 |3 个基本报告 |高级报告 |3 个基本报告 |



#### 云用户的自助密码重置 <a name="self-service-password-reset-for-cloud-users"></a>
**类型：**基本功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| &nbsp; |![勾选标记][12] |![勾选标记][12] | ![勾选标记][12] |

**更多详细信息：**

- [用户和管理员的 Azure AD 密码重置](/documentation/articles/active-directory-passwords/)

#### 公司品牌（登录页/访问面板自定义）<a name="company-branding-logon-pagesaccess-panel-customization"></a>
**类型：**基本功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| &nbsp; |![勾选标记][12] |![勾选标记][12] | ![勾选标记][12] |

**更多详细信息：**

- [向“登录”和“访问面板”页添加公司品牌](/documentation/articles/active-directory-add-company-branding/)

#### 应用程序代理 <a name="application-proxy"></a>
**类型：**基本功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| &nbsp; |![勾选标记][12] | ![勾选标记][12] | &nbsp; |

#### SLA 99.9% <a name="sla-999"></a>
**类型：**基本功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| &nbsp; |![勾选标记][12] |![勾选标记][12] | ![勾选标记][12] |

**更多详细信息：**

- [服务级别协议](/support/legal/sla/)

## Premium 功能


#### <a name="self-service-group"></a>自助组和应用管理/自助应用程序添加/动态组
**类型：**高级功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| &nbsp; | &nbsp; | ![勾选标记][12]  
| &nbsp; |

#### 通过本地写回实现自助密码重置/更改/解锁 <a name="self-service-password-resetchangeunlock-with-on-premises-write-back"></a>
**类型：**高级功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| &nbsp; | &nbsp; | ![勾选标记][12] | &nbsp; |

#### 多重身份验证（云和本地（MFA 服务器））<a name="multi-factor-authentication-cloud-and-on-premises-mfa-server"></a>
**类型：**高级功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| &nbsp; | &nbsp; |![勾选标记][12] |对于 Office 365 应用限制为云 |

**更多详细信息：**

- [什么是 Azure 多重身份验证？](/documentation/articles/multi-factor-authentication/)


#### <a name="mim-cal-mim-server"></a>MIM CAL + MIM 服务器
Microsoft 标识管理器服务器软件权限随 Windows Server 许可证（任意版本）一起授予。由于 Microsoft 标识管理器在 Windows Server 操作系统上运行，只要服务器正在运行有效的、经过许可的 Windows Server 副本，就能在该服务器上安装和使用 Microsoft 标识管理器。Microsoft 标识管理器服务器不需要其他单独的许可证。

**类型：**高级功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| &nbsp; | &nbsp; |![勾选标记][12] | &nbsp; |

#### Cloud App Discovery
**类型：**高级功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| &nbsp; | &nbsp; | ![勾选标记][12] | &nbsp; |



#### Azure AD Connect Health
**类型：**高级功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| &nbsp; | &nbsp; | ![勾选标记][12] | &nbsp; |

#### 组帐户的自动密码滚动更新
**类型：**高级功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| &nbsp; | &nbsp; | ![勾选标记][12] | &nbsp; |

#### 标识保护
**类型：**高级功能

| 免费版 | 基本版 | Premium P2 版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| &nbsp; | &nbsp; | ![勾选标记][12] | &nbsp; |

#### Privileged Identity Management
**类型：**高级功能

| 免费版 | 基本版 | Premium P2 版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| &nbsp; | &nbsp; | ![勾选标记][12] | &nbsp; |

## Azure Active Directory Join - 仅适用于 Windows 10 的相关功能
#### 让设备加入 Azure AD、Desktop SSO、Microsoft Passport for Azure AD 和 Administrator Bitlocker 恢复
**类型：**Azure Active Directory Join - 仅适用于 Windows 10 的相关功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| ![勾选标记][12] |![勾选标记][12] |![勾选标记][12] |![勾选标记][12] |


#### <a name="mdm-auto-enrollment"></a>MDM 自动注册、自助 Bitlocker 恢复、通过 Azure AD Join 将其他本地管理员加入 Windows 10 设备
**类型：**Azure Active Directory Join - 仅适用于 Windows 10 的相关功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| &nbsp; | &nbsp; | ![勾选标记][12] | &nbsp; |

#### 企业状态漫游
**类型：**Azure Active Directory Join - 仅适用于 Windows 10 的相关功能

**可用性：**

| 免费版 | 基本版 | Premium（P1 和 P2）版 | 仅限 Office 365 应用 |
|:---:|:---:|:---:|:---:|
| &nbsp; | &nbsp; | ![勾选标记][12] | &nbsp; |


## Azure AD 预览功能
除免费版、基本版和 Premium（P1 和 P2）版正式推出的功能外，Azure AD 还提供一系列预览功能。可以通过预览功能了解不久的将来可用的功能，并判断这些功能能否帮助改善你的环境。

**可用的预览功能：**

- [管理单元](/documentation/articles/active-directory-administrative-units-management/)
- [iOS 上基于证书的身份验证](/documentation/articles/active-directory-certificate-based-authentication-ios/)
- [Android 上基于证书的身份验证](/documentation/articles/active-directory-certificate-based-authentication-android/)

## 后续步骤
- [向“登录”和“访问面板”页添加公司品牌](/documentation/articles/active-directory-add-company-branding/)

<!--Image references-->
[12]: ./media/active-directory-editions/ic195031.png

<!---HONumber=Mooncake_0227_2017-->
<!---Update_Description: wording update -->