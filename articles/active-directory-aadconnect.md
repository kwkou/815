<properties 
	pageTitle="Azure AD Connect：将本地标识与 Azure Active Directory 集成 | Azure" 
	description="本页介绍 Azure AD Connect 是什么，以及为何要使用它。" 
	services="active-directory" 
	documentationCenter="" 
	authors="andkjell"
	manager="stevenpo" 
	editor="curtand"/>

<tags 
	ms.service="active-directory" 
	ms.date="12/02/2015"
	wacn.date="01/29/2016"/>


# 将本地标识与 Azure Active Directory 集成
Azure AD Connect 是用于集成本地标识系统（例如 Windows Server Active Directory）与 Azure Active Directory，并将用户连接到 Office 365、Azure 和 1000 多种 SaaS 应用程序的工具。本主题将全面指导你准备和部署必要的组件，让用户使用其目前用于访问现有公司应用的同一标识来访问云服务。

![什么是 Azure AD Connect](./media/active-directory-aadconnect/arch.png)

## 为何使用 Azure AD Connect
将本地目录与 Azure AD 集成后，可以让用户使用通用标识访问位于云中和本地的资源，从而提高他们的生产率。通过这种集成，用户和组织可以享受到以下好处：

- 用户可以使用单个标识来访问本地应用程序和云服务，例如 Office 365。

- 单个工具即可提供轻松同步和登录的部署体验。

- 为方案提供最新功能。Azure AD Connect 取代了 DirSync 和 Azure AD Sync 等早期版本的标识集成工具。有关详细信息，请参阅[目录集成工具比较](/documentation/articles/active-directory-aadconnect-get-started-tools-comparison)。


### Azure AD Connect 工作原理

Azure Active Directory Connect 由三个主要部分组成，分别是同步服务、可选的 Active Directory 联合身份验证服务功能，以及使用 [Azure AD Connect Health](/documentation/articles/active-directory-aadconnect-health) 实现的监视功能。

<center>![Azure AD Connect 堆栈](./media/active-directory-aadconnect-how-it-works/AADConnectStack2.png) </center>

- 同步 - 此部分由以前包含以前作为 [DirSync 和 Azure AD Sync](/documentation/articles/active-directory-aadconnect-get-started-tools-comparison) 发布的组件和功能组成。此部分负责创建用户和组。它还负责确保本地环境中有关用户和组的信息与云匹配。
- AD FS - 这是 Azure AD Connect 的可选部分，可用于使用本地 AD FS 基础结构设置混合环境。组织可以使用此部分来解决复杂的部署，包括域加入 SSO、实施 AD 登录策略和智能卡或第三方 MFA 等方案。
- 运行状况监视 - Azure AD Connect Health 能够可靠监视 AD FS 服务器，并在 Azure 门户中提供一个中心位置用于查看此活动。有关更多信息，请参阅 [Azure Active Directory Connect Health](/documentation/articles/active-directory-aadconnect-health)。

## 安装 Azure AD Connect

可以在 [Microsoft 下载中心](http://go.microsoft.com/fwlink/?LinkId=615771)找到 Azure AD Connect 的下载文件。


| 解决方案 | 方案 |
| ----- | ----- |
| 开始之前 | <li>[Azure AD Connect：硬件和先决条件](/documentation/articles/active-directory-aadconnect-prerequisites)</li> |
| [快速设置](/documentation/articles/active-directory-aadconnect-get-started-express) | <li>适用于单林 AD 的推荐默认选项。</li> <li>使用密码同步以同一密码进行用户登录。</li>
| [自定义设置](/documentation/articles/active-directory-aadconnect-get-started-custom) | <li>有多个林时使用。支持许多[本地拓扑](/documentation/articles/active-directory-aadconnect-topologies)。</li> <li>自定义登录选项，例如用于联合身份验证的 ADFS，或使用第三方标识提供者。</li> <li>自定义同步功能，例如筛选和写回。</li>
| [从 DirSync 升级](/documentation/articles/active-directory-aadconnect-dirsync-upgrade-get-started) | <li>如果你有已在运行的现有 DirSync 服务器。</li>
| 从 Azure AD Sync 升级 | <li>无缝的就地升级。</li>


[安装后](/documentation/articles/active-directory-aadconnect-whats-next)，你应该验证程序是否按预期工作，并将许可证分配给用户。

### Azure AD Connect 安装后续步骤

| 主题 | |
| --------- | --------- |
| 下载 Azure AD Connect | [下载 Azure AD Connect](http://go.microsoft.com/fwlink/?LinkId=615771) |
| 使用快速设置安装 | [Azure AD Connect 的快速安装](/documentation/articles/active-directory-aadconnect-get-started-express) |
| 使用自定义设置安装 | [Azure AD Connect 的自定义安装](/documentation/articles/active-directory-aadconnect-get-started-custom) |
| 从 DirSync 升级 | [从 Azure AD 同步工具 (DirSync) 升级](/documentation/articles/active-directory-aadconnect-dirsync-upgrade-get-started) |
| 安装后 | [验证安装并分配许可证](/documentation/articles/active-directory-aadconnect-whats-next) |

### 了解有关安装 Azure AD Connect 的详细信息

| 主题 | |
| --------- | --------- |
| 支持的拓扑 | [Azure AD Connect 的拓扑](/documentation/articles/active-directory-aadconnect-topologies) |
| 设计概念 | [Azure AD Connect 设计概念](/documentation/articles/active-directory-aadconnect-design-concepts) |
| 用于安装的帐户 | [有关 Azure AD Connect 凭据和权限的更多信息](/documentation/articles/active-directory-aadconnect-accounts-permissions) |
| 操作规划 | [Azure AD Connect 同步：操作任务和注意事项](/documentation/articles/active-directory-aadconnectsync-operations) |
| 用户登录选项 | [Azure AD Connect 用户登录选项](/documentation/articles/active-directory-aadconnect-user-signin) |

## 配置功能
Azure AD Connect 随附了多个可以选择启用或已按默认启用的功能。在某些情况下，有些功能可能需要在特定方案和拓扑中进行其他配置。

如果你要限制可将哪些对象同步到 Azure AD，可以使用[筛选](/documentation/articles/active-directory-aadconnectsync-configure-filtering)。默认情况下，会同步所有用户、联系人、组和 Windows 10 计算机，但你可以根据域、OU 或属性限制这些对象。

[密码同步](/documentation/articles/active-directory-aadconnectsync-implement-password-synchronization)可将 Active Directory 中的密码哈希同步到 Azure AD。这样，用户便可以在本地与云中使用相同的密码，且只需在一个位置管理此密码。由于它使用本地 Active Directory，因此你还可以使用自己的密码策略。

[密码写回](/documentation/articles/active-directory-passwords-getting-started)可让用户在云中更改和重置其密码，及应用本地密码策略。

[设备写回](/documentation/articles/active-directory-aadconnect-get-started-custom-device-writeback)可将 Azure AD 中注册的设备写回到本地 Active Directory，以便可以使用该设备进行条件性访问。

[防止意外删除](/documentation/articles/active-directory-aadconnectsync-feature-prevent-accidental-deletes)功能默认为打开，它可以保护云目录，避免同时进行多次删除。默认情况下，它允许每次执行 500 次删除，你可以根据组织的大小更改此值。

### 功能配置后续步骤

| 主题 | |
| --------- | --------- |
| 配置筛选 | [Azure AD Connect 同步：配置筛选](/documentation/articles/active-directory-aadconnectsync-configure-filtering) |
| 密码同步 | [Azure AD Connect 同步：实现密码同步](/documentation/articles/active-directory-aadconnectsync-implement-password-synchronization) |
| 密码写回 | [密码管理入门](/documentation/articles/active-directory-passwords-getting-started) |
| 设备写回 | [在 Azure AD Connect 中启用设备写回](/documentation/articles/active-directory-aadconnect-get-started-custom-device-writeback) |
| 防止意外删除 | [Azure AD Connect 同步：防止意外删除](/documentation/articles/active-directory-aadconnectsync-feature-prevent-accidental-deletes) |

## 自定义 Azure AD Connect 同步
Azure AD Connect 同步随附一个适用于大部分客户和拓扑的默认配置。但总会有一些情况使得默认配置不适用，因此你必须进行调整。你可以根据本部分和链接主题中所述进行更改。

。Azure AD Connect 是在 MIIS2003、ILM2007 和 FIM2010 基础上演进而来的。即使有些功能相同，但改变的部分也有很多。

[默认配置](/documentation/articles/active-directory-aadconnectsync-understanding-default-configuration)假设配置中可能存在多个林。在这些拓扑中，用户对象可能表示为另一个林中的联系人。用户还可能在另一个资源林中具有链接的邮箱。[用户和联系人](/documentation/articles/active-directory-aadconnectsync-understanding-users-and-contacts)中介绍了默认配置的行为。

同步的配置模型称为[声明性预配](/documentation/articles/active-directory-aadconnectsync-understanding-declarative-provisioning-expressions)。高级属性流程使用[函数](/documentation/articles/active-directory-aadconnectsync-functions-reference)来表示属性转换。你可以使用 Azure AD Connect 随附的工具来检查整个配置。如果需要对配置进行更改，请确保遵循[最佳实践](/documentation/articles/active-directory-aadconnectsync-best-practices-changing-default-configuration)，因为当有新版本可用时，可以更轻松地采用这些版本。

### Azure AD Connect 同步自定义后续步骤

| 主题 | |
| --------- | --------- |
| 了解用户和联系人 | [Azure AD Connect 同步：了解用户和联系人](/documentation/articles/active-directory-aadconnectsync-understanding-users-and-contacts) |
| 声明性预配 | [Azure AD Connect Sync：了解声明性设置表达式](/documentation/articles/active-directory-aadconnectsync-understanding-declarative-provisioning-expressions) |
| 声明性预配函数参考 | [Azure AD Connect 同步：函数参考](/documentation/articles/active-directory-aadconnectsync-functions-reference) |
| 最佳实践 | [更改默认配置的最佳做法](/documentation/articles/active-directory-aadconnectsync-best-practices-changing-default-configuration) |

## 详细信息和参考

| 主题 | |
| --------- | --------- |
| 版本历史记录 | [版本历史记录](/documentation/articles/active-directory-aadconnect-version-history) |
| 比较 DirSync、Azure ADSync 和 Azure AD Connect | [目录集成工具比较](/documentation/articles/active-directory-aadconnect-get-started-tools-comparison) |
| 同步的属性 | [同步的属性](/documentation/articles/active-directory-aadconnectsync-attributes-synchronized) |
| 使用 Azure AD Connect Health 进行监视 | [Azure AD Connect Health](/documentation/articles/active-directory-aadconnect-health) |
| 常见问题 | [Azure AD Connect 常见问题](/documentation/articles/active-directory-aadconnect-faq) |



**其他资源**


有关将本地目录扩展到云的 Ignite 2015 演示文稿。

<!---HONumber=Mooncake_0118_2016-->