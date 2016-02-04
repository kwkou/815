<properties
	pageTitle="更改默认配置的最佳做法"
	description="提供用于更改 Azure AD Connect Sync 的默认配置的最佳做法"
	services="active-directory"
	documentationCenter=""
	authors="markusvi"
	manager="swadhwa"
	editor=""/>

<tags   ms.service="active-directory"
	ms.date="07/27/2015"
	wacn.date="01/29/2016"/>


# Azure AD Connect Sync：用于更改默认配置的最佳做法

通过 Azure AD Connect 创建的配置无需更改即可适用于同步本地 Active Directory 与 Azure AD 的大多数环境。<br> 但是，在某些情况下，有必要将某些更改应用于配置，以满足特殊需求或要求。

虽然支持将更改应用于你的 Azure AD 配置，但是由于支持 Azure AD 尽可能接近设备，所以应小心应用这些更改。

下面是预期行为的列表：

- 将Azure AD Connect 升级到较新版本之后，大多数设置会重置为默认值。
- 在应用升级后，对“现成的”同步规则所做的更改将会丢失。
- 在升级到较新版本期间，会重新创建已删除的“现成”同步规则。
- 在升级到较新版本时，你创建的自定义同步规则将会保持不变。



如果需要更改默认配置，请执行以下操作：

- 如果需要修改某个“现成”同步规则的属性流，请不要更改该规则。而是新建一个具有较高优先级（编号值较小）的、包含所需属性流的同步规则。
- 使用同步规则编辑器导出自定义同步规则。这会提供一个 PowerShell 脚本，你可以在灾难恢复方案中使用该脚本轻松重新创建同步规则。
- 如果你需要在“现成”同步规则中更改范围或联接设置，请记下这些更改，并在升级到 Azure AD Sync 的较新版本之后重新应用更改。



**其他重要说明：**

- 如果你已配置基于属性的筛选和密码同步，请确保只有同步到 Azure AD 的对象在密码同步的范围内。 





## 其他资源

* [Azure AD Connect Sync：自定义同步选项](/documentation/articles/active-directory-aadconnectsync-whatis)
* [将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect)
 
<!--Image references-->

<!---HONumber=71-->