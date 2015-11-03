<properties 
	pageTitle="更改 Azure AD Connect 的默认配置" 
	description="了解如何更改 Azure AD Connect 的默认配置。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtand"/>

<tags 
	ms.service="active-directory" 
	ms.date="08/24/2015" 
	wacn.date="11/02/2015"/>

# 更改 Azure AD Connect 的默认配置 


在大多数情况下，使用 Azure AD Connect 的默认配置便足以将本地目录扩展到云中。但是，在某些情况下，你可能需要修改默认配置，并根据组织的业务逻辑对其进行自定义。在这种情况下，你可以修改默认配置，不过，在这样做之前，需要注意几个事项。

如果需要更改默认配置，请执行以下操作：

- 如果需要修改某个“现成”同步规则的属性流，请不要更改该规则。而是新建一个具有较高优先级（编号值较小）的、包含所需属性流的同步规则。
- 使用同步规则编辑器导出自定义同步规则。这会提供一个 PowerShell 脚本，你可以在灾难恢复方案中使用该脚本轻松重新创建同步规则。
- 如果你需要在“现成”同步规则中更改范围或联接设置，请记下这些更改，并升级到 Azure AD Connect 的较新版本之后重新应用更改。 

<!---HONumber=76-->