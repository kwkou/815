<properties
	pageTitle="Azure AD Connect 同步：了解用户和联系人 | Azure"
	description="介绍 Azure AD Connect 同步中的用户和联系人。"
	services="active-directory"
	documentationCenter=""
	authors="markusvi"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="05/03/2016"
	wacn.date="06/14/2016"/>


# Azure AD Connect 同步：了解用户和联系人

有几个不同的原因导致你会有多个 Active Directory 林，并且还有几个不同的部署拓扑。常见的模型包括合并和收购之后的帐户-资源部署和 GAL 同步的林。但即使有纯模型，混合模型也是常见的模型。Azure AD Connect 同步中的默认配置不会假定任何特定模型，但具体取决于安装指南中如何选择用户匹配，可以观察到不同的行为。

在本主题中，我们将讨论默认配置在某些拓扑中的行为方式。我们将讨论配置，并且同步规则编辑器可用于查看配置。

有几个配置假定的一般规则：

- 不管按什么顺序从源 Active Directory 导入，最终结果始终相同。
- 活动帐户会始终提供登录信息，包括 **userPrincipalName** 和 **sourceAnchor**。
- 如果找不到活动的帐户，已禁用帐户会提供 userPrincipalName 和 sourceAnchor，除非该帐户为已链接邮箱。
- 具有已链接邮箱的帐户永远不会用于 userPrincipalName 和 sourceAnchor。据推测，更高版本中将会找到有效帐户。
- 可能为 Azure AD 设置联系人对象，作为联系人或用户。在处理完所有源 Active Directory 林之前，你确实不会知道。

## 联系人

合并和收购之后，不同林中具有表示用户的联系人很常见，其中，GALSync 解决方案对两个或多个 Exchange 林桥接。联系人对象始终使用邮件属性从连接器空间联接到 metaverse。如果已存在具有相同邮件地址的联系人对象或用户对象，则会将这些对象联接在一起。这在规则 **In from AD – Contact Join** 中进行配置。另外，还有一条名为 **In from AD – Contact Common** 的规则，该规则具有到包含常量 **Contact** 的 metaverse 属性 **sourceObjectType** 的属性流。如果将任何用户对象联接到相同的 metaverse 对象，则此规则的优先级非常低，并且 **In from AD – User Common** 规则会为此属性提供值 User。在使用此规则的情况下，如果没有联接任何用户，此属性则会具有值 Contact，如果至少找到了一个用户，则会具有值 User。

对于为 Azure AD 设置对象，如果将 metaverse 属性 **sourceObjectType** 设置为 **Contact**，出站规则 **Out to AAD – Contact Join** 则会创建联系人对象。如果将此属性设置为 **User**，**Out to AAD – User Join** 规则则会改为创建用户对象。
当导入和同步更多源 Active Directory 时，对象很可能由 Contact 提升为 User。

例如，在 GALSync 拓扑中，当我们导入第一个林时，我们会在第二个林中发现每个的联系人对象。这将会在 AAD 连接器中暂存新的联系人对象。当我们之后导入并同步第二个林时，我们会找到实际用户并将他们联接到现有的 metaverse 对象。然后我们会删除 AAD 中的联系人对象，并改为创建新的用户对象。

如果你有用户表示为联系人的拓扑，请确保你的选择匹配安装指南中 mail 属性上的用户。如果选择另一个选项，则会具有依赖于顺序的配置。联系人对象始终会联接 mail 属性，但如果安装指南中选择了此选项，则用户对象只会联接 mail 属性。如果在用户对象之前已导入联系人对象，那么具有相同 mail 属性的 metaverse 中可能最终会有两个不同的对象。在导出到 Azure AD 期间，会引发错误。此行为是设计使然，并且会指示错误数据或者在安装过程中未正确标识拓扑。

## 已禁用帐户

已禁用帐户也会同步到 Azure AD。已禁用帐户在 Exchange 中表示资源是很常见的，例如会议室。例外情况是具有已链接邮箱的用户；如前文所述，这些用户永远不会将帐户设置到 Azure AD。

假设是，如果找到已禁用的用户帐户，那么之后我们将找不到另一个有效帐户，并且在找到 userPrincipalName 和 sourceAnchor 的情况下，对象会设置到 Azure AD。如果另一个有效帐户联接到相同的 metaverse 对象，则会使用其 userPrincipalName 和 sourceAnchor。

## 更改 sourceAnchor

当对象已导出到 Azure AD 时，则不再允许更改 sourceAnchor。当已导出对象时，则采用 Azure AD 接受的 **sourceAnchor** 值设置 metaverse 属性 **cloudSourceAnchor**。如果更改了 **sourceAnchor**，且不匹配 **cloudSourceAnchor**，规则 **Out to AAD – User Join** 将引发错误“sourceAnchor 属性已更改”。在这种情况下，必须更正配置或数据，以便相同的 sourceAnchor 再次在 metaverse 中出现，然后才能再次同步对象。

## 其他资源

* [Azure AD Connect Sync：自定义同步选项](/documentation/articles/active-directory-aadconnectsync-whatis/)
* [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)

 
<!--Image references-->

<!---HONumber=Mooncake_0606_2016-->