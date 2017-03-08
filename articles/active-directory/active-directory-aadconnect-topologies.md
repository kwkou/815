<properties
    pageTitle="Azure AD Connect：支持的拓扑 | Azure"
    description="本主题详细说明 Azure AD Connect 的受支持和不受支持的拓扑"
    services="active-directory"
    documentationcenter=""
    author="AndKjell"
    manager="femila"
    editor="" />
<tags
    ms.assetid="1034c000-59f2-4fc8-8137-2416fa5e4bfe"
    ms.service="active-directory"
    ms.devlang="na"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.topic="article"
    ms.date="02/08/2017"
    wacn.date="03/07/2017"
    ms.author="billmath" />  


# Azure AD Connect 的拓扑
本文介绍使用 Azure AD Connect 同步作为关键集成解决方案的各种本地拓扑和 Azure Active Directory (Azure AD) 拓扑。此外，介绍支持和不支持的配置。

下面是本文中的图片图例：

| 说明 | 符号 |
| --- | --- |
| 本地 Active Directory 林 |![本地 Active Directory 林](./media/active-directory-aadconnect-topologies/LegendAD1.png) |
| 包含筛选导入的本地 Active Directory |![包含筛选导入的 Active Directory](./media/active-directory-aadconnect-topologies/LegendAD2.png) |
| Azure AD Connect 同步服务器 |![Azure AD Connect 同步服务器](./media/active-directory-aadconnect-topologies/LegendSync1.png) |
| Azure AD Connect 同步服务器“暂存模式” |![Azure AD Connect 同步服务器“暂存模式”](./media/active-directory-aadconnect-topologies/LegendSync2.png) |
| 装有 Forefront Identity Manager (FIM) 2010 或 Microsoft Identity Manager (MIM) 2016 的 GALSync |![使用 FIM 2010 或 MIM 2016 的 GALSync](./media/active-directory-aadconnect-topologies/LegendSync3.png) |
| Azure AD Connect 同步服务器（详细说明） |![Azure AD Connect 同步服务器（详细说明）](./media/active-directory-aadconnect-topologies/LegendSync4.png) |
| Azure AD |![Azure Active Directory](./media/active-directory-aadconnect-topologies/LegendAAD.png) |
| 不支持的方案 |![不支持的方案](./media/active-directory-aadconnect-topologies/LegendUnsupported.png) |

## 单个林，单个 Azure AD 租户 <a name="each-object-only-once-in-an-azure-ad-directory"></a>
![单个林和单个租户的拓扑](./media/active-directory-aadconnect-topologies/SingleForestSingleDirectory.png)  


最常见的拓朴是包含一个或多个域的单个本地林，以及单个 Azure AD 租户。Azure AD 身份验证使用密码同步。Azure AD Connect 的快速安装仅支持此拓扑。

### 单个林，多个同步服务器连接到一个 Azure AD 租户
![单个林不支持的筛选拓扑](./media/active-directory-aadconnect-topologies/SingleForestFilteredUnsupported.png)  


不支持多个 Azure AD Connect 同步服务器连接到同一个 Azure AD 租户（[暂存服务器](#staging-server)除外）。即使将这些服务器配置为与一组互斥对象同步，也不支持这种拓扑。如果无法从单个服务器连接到林中的所有域，或者想要将负载分散到多个服务器，则应该考虑这种拓扑。

## 多个林，单个 Azure AD 租户 <a name="multiple-forests-single-azure-ad-tenant"></a>
![多个林和单个租户的拓扑](./media/active-directory-aadconnect-topologies/MultiForestSingleDirectory.png)  


许多组织具有包含多个本地 Active Directory 林的环境。有多种原因导致出现多个本地 Active Directory 林。典型示例是使用帐户资源林的设计，以及合并和收购之后采用的设计。

如果使用多个林，所有林必须可由单个 Azure AD Connect 同步服务器访问。不需要将服务器加入域。如果需要访问所有林，可将服务器放在外围网络（也称为 DMZ、外围安全区域或屏蔽子网）中。

Azure AD Connect 安装向导提供多个选项用于合并多个林中显示的用户。目标是一个用户只在 Azure AD 中显示一次。可以在安装向导的自定义安装路径中配置某些常见拓扑。在“唯一标识你的用户”页上选择表示拓扑的相应选项。只对用户配置合并。复制的组不会与默认配置合并。

有关[独立的拓扑](#multiple-forests-separate-topologies)、[完整网格](#multiple-forests-full-mesh-with-optional-galsync)和[帐户资源拓扑](#multiple-forests-account-resource-forest)的部分讨论了常见拓扑。

Azure AD Connect 同步中的默认配置假设：

- 每个用户只有一个已启用的帐户并且此帐户所在的林用于对用户进行身份验证。这种假设适用于密码同步和联合。UserPrincipalName 和 sourceAnchor/immutableID 来自此林。
- 每个用户只有一个邮箱。
- 托管用户邮箱的林具有 Exchange 全局地址列表 (GAL) 中可见属性的最佳数据质量。如果用户没有邮箱，则任何林都可以用于提供这些属性值。
- 如果有链接邮箱，则还有其他林中的某个帐户用于登录。

如果环境不符合这些假设，将发生以下情况：

- 如果使用多个活动帐户或多个邮箱，同步引擎将选择其中一个并忽略其他帐户或邮箱。
- 没有其他活动帐户的链接邮箱不会导出到 Azure AD。用户帐户不会显示为任何组中的成员。DirSync 中的链接邮箱始终显示为普通邮箱。这项更改是有意而为的，目的是使用不同的行为来更好地支持多林方案。

可在[了解默认配置](/documentation/articles/active-directory-aadconnectsync-understanding-default-configuration/)中找到更多详细信息。

### 多个林，多个同步服务器连接到单个 Azure AD 租户
![多个林和多个同步服务器不支持的拓扑](./media/active-directory-aadconnect-topologies/MultiForestMultiSyncUnsupported.png)  


不支持多个 Azure AD Connect 同步服务器连接到单个 Azure AD 租户。使用[暂存服务器](#staging-server)时例外。

### 多个林，独立的拓扑 <a name="multiple-forests-separate-topologies"></a>
![表示用户在所有目录中只出现一次的选项](./media/active-directory-aadconnect-topologies/MultiForestUsersOnce.png)  


![描述多个林和独立的拓扑](./media/active-directory-aadconnect-topologies/MultiForestSeperateTopologies.png)  


在此环境中，所有本地林都被视为独立的实体。没有用户出现在任何其他林中。每个林都有其自己的 Exchange 组织，并且林之间没有任何 GALSync。合并/收购之后或者如果组织中的每个业务单位独立运营，可能会出现这种拓扑。在 Azure AD 中，这些林位于相同的组织中并与统一的 GAL 一起出现。在上图中，每个林中的每个对象会在 Metaverse 中出现一次，并在目标 Azure AD 租户中聚合。

### 多个林：匹配用户
对于所有这些方案，一种常见情况是分发组和安全组可以包含用户、联系人和外部安全主体 (FSP) 的混合形式。可在 Active Directory 域服务 (AD DS) 中使用 FSP 来表示安全组中来自其他林的成员。在 Azure AD 中，所有 FSP 解析为实际对象。

### 多个林：包含可选 GALSync 的完整网格 <a name="multiple-forests-full-mesh-with-optional-galsync"></a>
![当用户标识跨多个目录存在时使用 mail 属性进行匹配的选项](./media/active-directory-aadconnect-topologies/MultiForestUsersMail.png)  


![多个林的完整网格拓扑](./media/active-directory-aadconnect-topologies/MultiForestFullMesh.png)  


完整网格拓扑允许用户和资源位于任何林中。通常，林之间建立了双向信任。

如果 Exchange 存在于多个林中，则可以选择使用本地 GALSync 解决方案。这样，每个用户将表示为其他所有林中的联系人。GALSync 通常是使用 FIM 2010 或 MIM 2016 实现的。Azure AD Connect 无法用于本地 GALSync。

在此方案中，标识对象通过 mail 属性进行联接。一个林中具有邮箱的用户与其他林中的联系人进行联接。

### 多个林：帐户资源林 <a name="multiple-forests-account-resource-forest"></a>
![当用户标识跨多个目录存在时使用 ObjectSID 和 msExchMasterAccountSID 属性进行匹配的选项](./media/active-directory-aadconnect-topologies/MultiForestUsersObjectSID.png)  


![多个林的帐户资源林拓扑](./media/active-directory-aadconnect-topologies/MultiForestAccountResource.png)  


在帐户资源林拓扑中，有一个或多个包含活动用户帐户的*帐户*林。此外，还有一个或多个包含已禁用帐户的*资源*林。

在此方案中，一个（或多个）资源林信任所有帐户林。资源林通常包含装有 Exchange 和 Lync 的扩展 Active Directory 架构。所有 Exchange 和 Lync 服务以及其他共享服务都位于此林中。用户在此林中具有一个禁用的用户帐户，并且邮箱被链接到帐户林。

## Office 365 和拓扑注意事项
某些 Office 365 工作负荷对支持的拓扑实施一些限制：

| 工作负载 | 限制 |
--------- | ---------
| Exchange Online | 如果有多个本地 Exchange 组织（即 Exchange 已部署到多个林），必须使用 Exchange 2013 SP1 或更高版本。有关详细信息，请参阅[包含多个 Active Directory 林的混合部署](https://technet.microsoft.com/zh-cn/library/jj873754.aspx)。 |
| Skype for Business | 使用多个本地林时，只支持帐户资源林拓扑。有关详细信息，请参阅 [Skype for Business Server 2015 的环境要求](https://technet.microsoft.com/zh-cn/library/dn933910.aspx)。 |


## 暂存服务器 <a name="staging-server"></a>
![拓扑中的暂存服务器](./media/active-directory-aadconnect-topologies/MultiForestStaging.png)  


Azure AD Connect 支持以*暂存模式*安装第二个服务器。使用此模式的服务器从所有已连接的目录读取数据，但不会向已连接的目录写入任何数据。它使用普通的同步周期，因此具有标识数据的更新副本。

在主服务器发生故障的灾难事件中，可以故障转移到暂存服务器。在 Azure AD Connect 向导中执行此操作。可将第二个服务器定位在不同的数据中心，因为没有和主服务器共享基础结构。必须手动将主服务器上所做的任何配置更改复制到第二台服务器。

可以使用暂存服务器来测试新的自定义配置及其对数据造成的影响。你可以预览更改并调整配置。如果满意新的配置，可让暂存服务器成为活动服务器，将旧的活动服务器设置为暂存模式。

还可以使用此方法替换活动的同步服务器。准备新的服务器，并将其设置为暂存模式。确保它处于良好状态、禁用暂存模式（使之成为活动服务器），然后关闭当前活动的服务器。

如果想要在不同的数据中心拥有多个备份，也可以配置多个暂存服务器。

## 多个 Azure AD 租户
我们建议组织在 Azure AD 中部署单个租户。

![多个林和多个租户的拓扑](./media/active-directory-aadconnect-topologies/MultiForestMultiDirectory.png)  


Azure AD Connect 同步服务器与 Azure AD 租户之间不存在一对一的关系。在每个 Azure AD 租户中，需要安装一个 Azure AD Connect 同步服务器。Azure AD 租户实例在设计上是隔离的。也就是说，一个租户中的用户看不到另一个租户中的用户。如果想要这种隔离，可以使用这种受支持的配置。否则，应使用单一 Azure AD 租户模型。

### 每个对象只在 Azure AD 租户中运行一次 <a name="each-object-only-once-in-an-azure-ad-tenant"></a>
![单个林的筛选拓扑](./media/active-directory-aadconnect-topologies/SingleForestFiltered.png)  


在此拓扑中，一个 Azure AD Connect 同步服务器连接到每个 Azure AD 租户。Azure AD Connect 同步服务器必须设置筛选，让它们都有一组对象的互斥集可运行。例如，将每个服务器的范围设置为特定域或组织单位。

DNS 域只能在单个 Azure AD 租户中注册。本地 Active Directory 实例中的用户 UPN 也必须使用独立的命名空间。例如，在上图中，三个独立 UPN 后缀都注册在本地 Active Directory 实例中：contoso.com、fabrikam.com 和 wingtiptoys.com。每个本地 Active Directory 域中的用户使用不同的命名空间。

Azure AD 租户实例之间没有任何 GALSync。Exchange Online 和 Skype for Business 中的通讯簿只在相同的租户中显示用户。

另外，此拓扑对支持的方案实施以下限制：

- 只有一个 Azure AD 租户可以使用本地 Active Directory 实例启用 Exchange 混合部署。
- Windows 10 设备只能与一个 Azure AD 租户相关联。
- 用于密码同步和传递身份验证的单一登录 (SSO) 选项只能用于一个 Azure AD 租户。

对象互斥集的要求也适用于写回。此拓扑不支持某些写回功能，因为这些功能采用单个本地配置。这些功能包括：

- 使用默认配置进行组写回。
- 设备写回。

### 每个对象在 Azure AD 租户中运行多次
![单个林和多个租户不支持的拓扑](./media/active-directory-aadconnect-topologies/SingleForestMultiDirectoryUnsupported.png) ![单个林和多个连接器不支持的拓扑](./media/active-directory-aadconnect-topologies/SingleForestMultiConnectorsUnsupported.png)

不支持以下任务：

- 将同一用户同步到多个 Azure AD 租户。
- 进行配置更改，使一个 Azure AD 租户中的用户显示为另一个 Azure AD 租户中的联系人。
- 将 Azure AD Connect 同步修改为连接到多个 Azure AD 租户。

### 使用写回的 GALSync
![多个林和多个目录不支持的拓扑，其中的 GALSync 重点用于 Azure AD](./media/active-directory-aadconnect-topologies/MultiForestMultiDirectoryGALSync1Unsupported.png) ![多个林和多个目录不支持的拓扑，其中的 GALSync 重点用于本地 Active Directory](./media/active-directory-aadconnect-topologies/MultiForestMultiDirectoryGALSync2Unsupported.png)

Azure AD 租户在设计上是隔离的。不支持以下任务：

- 将 Azure AD Connect 同步更改为从另一个 Azure AD 租户读取数据。
- 使用 Azure AD Connect 同步将用户作为联系人导出到另一个本地 Active Directory 实例。

### 使用本地同步服务器的 GALSync
![多个林和多个目录的拓扑中的 GALSync](./media/active-directory-aadconnect-topologies/MultiForestMultiDirectoryGALSync.png)  


可以使用本地 FIM 2010 或 MIM 2016 在两个 Exchange 组织之间同步用户（通过 GALSync）。一个组织中的用户将显示为另一组织中的外部用户/联系人。这些不同的本地 Active Directory 实例可与其自身的 Azure AD 租户同步。

## 后续步骤
若要了解如何为这些方案安装 Azure AD Connect，请参阅[Azure AD Connect 的自定义安装](/documentation/articles/active-directory-aadconnect-get-started-custom/)。

了解有关 [Azure AD Connect 同步](/documentation/articles/active-directory-aadconnectsync-whatis/)配置的详细信息。

了解有关[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)的详细信息。

<!---HONumber=Mooncake_0227_2017-->
<!---Update_Description: wording update -->