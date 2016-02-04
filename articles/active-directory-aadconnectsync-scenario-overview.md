<properties
	pageTitle="Azure AD Connect Sync：多林方案概述"
    description="本主题的目标是介绍一些常见的方案以及它们在 Azure AD Connect Sync 的同步服务中的表示方式。"
	services="active-directory"
	documentationCenter=""
	authors="markusvi"
	manager="swadhwa"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="07/27/2015"
	wacn.date="01/29/2016"/>


# Azure AD Connect Sync：多林方案概述

许多组织具有包含多个本地 Active Directory 林的环境。有多种原因导致部署多个本地 Active Directory 林。典型示例是具有帐户资源林、合并和收购相关的林或用于外包数据的林的设计。

Microsoft 为你提供适用于单林方案的解决方案 DirSync 以及适用于多林方案的解决方案 FIM。但是，配置 FIM 非常具有挑战性，并且它会占用相当长的时间。

使用 Azure AD Connect Sync，你可以显著地简化配置并使其更有预测性。

在 Azure AD Connect Sync 提供的默认配置中，进行以下假设：

1. 用户只有一个已启用的帐户并且此帐户所在的林用于联合用户。
2. 用户只有一个邮箱。
3. 托管用户邮箱的林具有 Exchange 全局地址列表 (GAL) 中可见属性的最佳数据质量。如果用户没有邮箱，则任何林都可以用于提供这些属性值。


本主题的目标是介绍一些常见的方案以及它们在 Azure AD Connect Sync 的同步服务中的表示方式：

1. 单独的技术 
2. 具有可选 GALSync 的完全网络 
3. 帐户资源林 


## 单独的技术

在此环境中，本地的所有林都被视为单独的实体，并且没有用户出现在任何其他林中。<br> 每个林都有其自己的 Exchange 组织，并且林之间没有任何 GALSync。合并/收购之后或在其中每个业务单位在相互隔离的情况下运营的组织中，可能出现这种情况。

![单独的技术][1]

在此图中，每个林中的每个对象会在 metaverse 中出现一次，并在目标 Azure AD 目录中聚合。将一个 DirSync 服务器连接到每个源 AD 林时，这将是相同的最终结果。
 




## 具有可选 GALSync 的完全网络

利用完全网格拓扑，用户和资源能够位于任何林中，并且通常林之间会有双向信任。

如果 Exchange 存在于多个林中，则可以根据需要提供将一个林中的用户表示为每个其他林中的联系人的 GALSync 解决方案。

在此方案中，标识对象通常使用 mail 属性进行联接。因此，一个林中具有邮箱的用户与其他林中的联系人进行联接。分发和安全组可以在每个林中找到，并且可以包含用户、联系人和 FSP（外部安全主体）的组合。

下图概述了此方案。

![具有可选 GALSync 的完全网络][2]


## 帐户资源林
在帐户资源林拓扑中，你拥有带活动用户帐户的一个或多个帐户林。<br> 此方案包括一个信任所有帐户林的林。此林通常具有带 Exchange 和 Lync 的扩展 AD 架构。所有 Exchange 和 Lync 服务以及其他共享服务都位于此林中。用户在此林中具有已禁用用户帐户，并且邮箱已链接到帐户林。

下图只使用一个帐户概述了这种方案。

![帐户资源林][3]


## 其他资源

* [Azure AD Connect Sync：自定义同步选项](/documentation/articles/active-directory-aadconnectsync-whatis)
* [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect)

 
<!--Image references-->
[1]: ./media/active-directory-aadsync-scenario-overview/ic750599.png
[2]: ./media/active-directory-aadsync-scenario-overview/ic750600.png
[3]: ./media/active-directory-aadsync-scenario-overview/ic750601.png

<!---HONumber=71-->