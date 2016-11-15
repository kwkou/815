<properties
   pageTitle="数据安全与加密最佳实践 | Microsoft Azure"
   description="本文提供一系列有关使用内置 Azure 功能实现数据安全与加密的最佳实践。"
   services="security"
   documentationCenter="na"
   authors="YuriDio"
   manager="swadhwa"
   editor="TomSh"/>  


<tags
   ms.service="security"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="08/16/2016"
   wacn.date="10/31/2016"
   ms.author="yuridio"/>  


#Azure 数据安全与加密最佳实践

在云中保护数据的关键问题之一是考虑数据可能将发生的状态，以及哪些控件适用于该状态。根据 Azure 数据安全和加密最佳实践的目的，相关建议将围绕以下数据状态：

- 静态：包括物理媒体（磁盘或光盘）上以静态方式存在的所有信息存储对象、容器和类型。

- 传输中：数据在组件、位置或程序之间发送时，例如通过网络、通过服务总线（从本地到云，反之亦然，包括诸如 ExpressRoute 的混合连接），或在输入/输出过程中，将被视为动态数据。

本文介绍一系列 Azure 数据安全与加密最佳实践。这些最佳实践衍生自我们的 Azure 数据安全与加密经验和客户经验。

对于每项最佳实践，我们将说明：

- 最佳实践是什么
- 为何要启用该最佳实践
- 如果你无法启用该最佳实践，可能的结果是什么
- 最佳实践的可能替代方案
- 如何学习启用最佳实践

这篇 Azure 数据安全与加密最佳实践以共识以及 Azure 平台功能和特性集（因为在编写本文时已存在）为基础。看法和技术将随着时间改变，本文将会定期更新以反映这些更改。

本文介绍的 Azure 数据安全与加密最佳实践包括：

- 实施多重身份验证
- 使用基于角色的访问控制 (RBAC)
- 加密 Azure 虚拟机
- 使用硬件安全模型
- 使用安全工作站进行管理
- 启用 SQL 数据加密
- 保护传输中的数据
- 实施文件级数据加密


## 实施多重身份验证

在 Microsoft Azure 中访问和控制数据的第一个步骤是对用户进行身份验证。[Azure 多重身份验证 (MFA)](/documentation/articles/multi-factor-authentication/) 是除了使用用户名与密码以外，还要求使用其他方法对用户标识进行身份验证的方法。这种身份验证方法可帮助保护对数据和应用程序的访问，同时可以满足用户对简单登录过程的需求。

如果针对用户启用 Azure MFA，则可为用户登录和事务增加第二层安全性。在此情况下，事务可能将访问位于文件服务器或 SharePoint Online 中的文档。Azure MFA 还可帮助 IT 部门减少使用透露的凭据访问企业数据的可能性。

例如：如果对用户实施 Azure MFA 并将它配置为使用电话呼叫或短信作为身份验证，那么，当用户的凭据透露时，攻击者无法访问任何资源，因为攻击者无权访问用户的电话。未添加这种额外标识保护层的组织将更容易受到凭据窃取攻击，从而导致数据泄漏。

有关 Azure MFA 的详细信息，请参阅 [Getting started with Azure Multi-Factor Authentication in the cloud](/documentation/articles/multi-factor-authentication-get-started-cloud/)（云中的 Azure 多重身份验证入门）。

## 使用基于角色的访问控制 (RBAC)
根据[需要知道](https://en.wikipedia.org/wiki/Need_to_know)和[最低权限](https://en.wikipedia.org/wiki/Principle_of_least_privilege)安全策略限制访问权限。对于想要实施数据访问安全策略的组织，这是必须要做的事。Azure 基于角色的访问控制 (RBAC) 可用于向特定范围的用户、组和应用程序分配权限。角色分配的范围可以是订阅、资源组或单个资源。

可以利用 Azure 中[内置的 RBAC](/documentation/articles/role-based-access-built-in-roles/) 角色向用户分配权限。考虑将*存储帐户参与者*用于需要管理存储帐户的云操作者，并使用*经典存储帐户参与者*角色来管理经典存储帐户。对于需要管理 VM 和存储帐户的云操作者，请考虑将他们添加到*虚拟机参与者*角色。

未使用 RBAC 等功能实施数据访问控制的组织可能会给其用户分配超过需要的权限。一开始就允许某些用户访问他们不应有权访问的数据可能会导致数据泄漏。

有关 Azure RBAC 的详细信息，请阅读 [Azure Role-Based Access Control](/documentation/articles/role-based-access-control-configure/)（Azure 基于角色的访问控制）一文。

## 使用硬件安全模块

行业加密解决方案使用机密密钥来加密数据。因此，必须安全存储这些密钥。密钥管理已变成数据保护不可或缺的部分，因为它将用于存储加密数据所用的机密密钥。

Azure 磁盘加密使用 [Azure 密钥保管库](/home/features/key-vault/)来帮助控制和管理密钥保管库订阅中的磁盘加密密钥和机密，同时确保虚拟机磁盘中的所有数据可在 Azure 存储中静态加密。应使用 Azure 密钥保管库来审核密钥和策略用法。

若未采用适当的安全控件来保护用于加密数据的机密密钥，将有许多相关的固有风险。如果攻击者可以访问机密密钥，就可以将数据解密，并可能访问机密信息。

有关 Azure 中证书管理的一般建议，请阅读 [Certificate Management in Azure: Do’s and Don’ts](https://blogs.msdn.microsoft.com/azuresecurity/2015/07/13/certificate-management-in-azure-dos-and-donts/)（Azure 中的证书管理：必做与禁忌事项）。

有关 Azure 密钥保管库的详细信息，请阅读 [Get started with Azure Key Vault](/documentation/articles/key-vault-get-started/)（Azure 密钥保管库入门）。

## 使用安全工作站进行管理

因为绝大多数的攻击以用户为目标，所以终结点将成为主要攻击点之一。如果攻击者入侵终结点，则可以利用用户的凭据来访问组织的数据。大多数终结点攻击能够使用用户就是其本地工作站的管理员的这一事实。

可以使用安全的管理工作站来降低这些风险。建议使用[特权访问工作站 (PAW)](https://technet.microsoft.com/zh-cn/library/mt634654.aspx) 来减小工作站的受攻击面。这些安全的管理工作站可帮助减轻其中一些攻击，以确保数据更为安全。请务必使用 PAW 来强化并锁定工作站。这是重要的步骤，可为敏感帐户、任务和数据保护提供高安全保证。

缺少终结点保护将使数据面临风险，请务必在用于使用数据的所有设备上实施安全策略（无论数据位于云中还是本地）。

有关特权访问工作站的详细信息，请阅读 [Securing Privileged Access](https://technet.microsoft.com/zh-cn/library/mt631194.aspx)（保护特权访问）一文。

## 启用 SQL 数据加密

[Azure SQL 数据库透明数据加密](https://msdn.microsoft.com/zh-cn/library/dn948096.aspx) (TDE) 无需更改应用程序，即可对静态数据库、关联的备份和事务日志执行实时加密和解密，帮助防止恶意活动的威胁。TDE 使用称为数据库加密密钥的对称密钥来加密整个数据库的存储。

即使整个存储都已加密，也一定要加密数据库本身。这是数据保护的深度防御方法实现。如果使用 [Azure SQL 数据库](https://msdn.microsoft.com/zh-cn/library/0bf7e8ff-1416-4923-9c4c-49341e208c62.aspx)，并且想要保护敏感数据（例如信用卡号或社会安全号码），可以使用 FIPS 140-2 验证的 256 位 AES 加密来加密数据库，它符合许多行业标准（例如 HIPAA、PCI）的要求。

请务必了解，使用 TDE 加密数据库时，[缓冲池扩展](https://msdn.microsoft.com/zh-cn/library/dn133176.aspx) (BPE) 相关文件不会加密。必须对 BPE 相关文件使用文件系统级别的加密工具，例如 BitLocker 或[加密文件系统](https://technet.microsoft.com/zh-cn/library/cc700811.aspx) (EFS)。

因为经过授权的用户（如安全管理员或数据库管理员）可以访问数据，所以即使已使用 TDE 将数据库加密，也应遵循以下建议：

- 在数据库级别进行 SQL 身份验证
- 使用 RBAC 角色进行 Azure AD 身份验证
- 用户和应用程序应使用不同的帐户进行身份验证。这样，可以限制授予用户和应用程序的权限，并降低恶意活动的风险
- 使用固定的数据库角色（例如 db\_datareader 或 db\_datawriter）实现数据库级别安全，或者为应用程序创建自定义角色，以向选定的数据库对象授予显式权限

不使用数据库级别加密的组织可能更容易受到攻击，使 SQL 数据库中的数据面临透露的风险。

有关 SQL TDE 加密的详细信息，请阅读 [Transparent Data Encryption with Azure SQL Database](https://msdn.microsoft.com/zh-cn/library/0bf7e8ff-1416-4923-9c4c-49341e208c62.aspx)（使用 Azure SQL 数据库的透明数据加密）一文。

## 保护传输中的数据

保护传输中的数据应该是数据保护策略中不可或缺的部分。由于数据将从许多位置来回移动，一般将建议始终使用 SSL/TLS 协议来交换不同位置的数据。在某些情况下，建议使用虚拟专用网络 (VPN) 隔离本地与云基础结构之间的整个通信通道。

对于在本地基础结构与 Azure 之间移动的数据，应该考虑适当的防护措施，例如 HTTPS 或 VPN。

对于需要从位于本地的多个工作站安全访问 Azure 的组织而言，请使用 [Azure 站点到站点 VPN](/documentation/articles/vpn-gateway-site-to-site-create/)。

对于需要从位于本地的一个工作站安全访问 Azure 的组织而言，请使用[点到站点 VPN](/documentation/articles/vpn-gateway-point-to-site-create/)。

可以通过专用高速 WAN 链接（例如 [ExpressRoute](https://www.azure.cn/home/features/expressroute/)）移动较大的数据集。如果选择使用 ExpressRoute，则还可以使用 [SSL/TLS](https://support.microsoft.com/zh-cn/kb/257591) 或其他协议，在应用程序级别加密数据，以提供额外的保护。

如果通过 Azure 门户与 Azure 存储空间交互，则所有事务都将通过 HTTPS 发生。也可以使用基于 HTTPS 的[存储 REST API](https://msdn.microsoft.com/zh-cn/library/azure/dd179355.aspx) 来与 [Azure 存储](/home/features/storage/)和 [Azure SQL 数据库](/home/features/sql-database/)交互。

无法保护传输中数据的组织比较容易遭受[中间人攻击](https://technet.microsoft.com/zh-cn/library/gg195821.aspx)、[窃听](https://technet.microsoft.com/zh-cn/library/gg195641.aspx)和会话劫持。这些攻击可能是获取机密数据访问权限的第一步。

有关 Azure VPN 选项的详细信息，请阅读 [Planning and design for VPN Gateway](/documentation/articles/vpn-gateway-plan-design/)（规划和设计 VPN 网关）一文。

<!---HONumber=Mooncake_1024_2016-->