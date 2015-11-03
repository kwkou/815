<properties 
	pageTitle="目录集成工具比较" 
	description="本页将会提供比较各种目录集成工具的综合表。" 
	services="active-directory" 
	documentationCenter="" 
	authors="billmath" 
	manager="stevenpo" 
	editor="curtand"/>

<tags 
	ms.service="active-directory"  
	ms.date="08/24/2015" 
	wacn.date="11/02/2015"/>

# 目录集成工具比较

过去数年以来，集成工具已得到发展和演进。本文档旨在帮助提供这些工具的合并视图，并比较每个工具提供的功能。

>[AZURE.NOTE]Azure AD Connect 整合了以前作为 DirSync 和 AAD Sync 发布的组件和功能。这些工具不再单独发布，将来所做的改进将包含在 Azure AD Connect 更新中，因此你始终知道从何处获取最新功能。
>
>目前仍支持 Dirsync，但以后会弃用它。弃用后，只会对其提供短暂的支持。这段短暂的时间过后，将不再提供对 Dirsync 的支持。

每个表中的代号解释如下。

● = 现已发布</br>
FR = 未来版本</br>
PP = 公共预览版本</br>


## 本地到云的同步

| 功能 | Azure Active Directory Connect | Azure Active Directory 同步服务 (AAD Sync) | Azure Active Directory 同步工具 (DirSync)| Forefront Identity Manager 2010 R2 (FIM) |
| :-------- |:--------:|:--------:|:--------:|:--------:
| 连接到单个本地 AD 林 | ● | ● | ● | ● |
| 连接到多个本地 AD 林 |● | ● | | ● |
| 连接到多个本地 Exchange 组织 | ● | | | |
| 连接到单个本地 LDAP 目录 | FR | | | ● |
| 连接到多个本地 LDAP 目录 |FR | | | ● |
| 连接到本地 AD 和本地 LDAP 目录 |FR | | | ● |
| 连接到自定义系统（例如 SQL、Oracle、MySQL 等） | FR | | | ● |
| 同步客户定义的属性（目录扩展） | ● | | | |

## 云到本地的同步

| 功能 | Azure Active Directory Connect | Azure Active Directory 同步服务 | Azure Active Directory 同步工具 (DirSync) | Forefront Identity Manager 2010 R2 (FIM) |
| :-------- |:--------:|:--------:|:--------:|:--------:
| 设备写回 | ● | | ● | |
| 属性写回（适用于 Exchange 混合部署） | ● | ● | ● | ● |
| 用户和组对象写回 | ●| | | |
| 密码写回（通过自助密码重置 (SSPR) 和密码更改） | ● | ● | | |



## 身份验证功能支持

| 功能 | Azure Active Directory Connect | Azure Active Directory 同步服务 | Azure Active Directory 同步工具 (DirSync) | Forefront Identity Manager 2010 R2 (FIM) |
| :-------- |:--------:|:--------:|:--------:|:--------:
| 单个本地 AD 林的密码同步 | ● | ● | ● | |
| 多个本地 AD 林的密码同步 | ●| ● | | |
| 使用联合身份验证的单一登录 | ● | ● | ● | ● |
| 密码写回（通过 SSPR 和密码更改） |● | ● | | |



## 设置和安装

| 功能 | Azure Active Directory Connect | Azure Active Directory 同步服务 | Azure Active Directory 同步工具 (DirSync) | Forefront Identity Manager 2010 R2 (FIM) |
| :-------- |:--------:|:--------:|:--------:|:--------:
| 支持在域控制器上安装 | ● | ● | ● | |
| 支持使用 SQL Express 安装 | ● | ● | ● | |
| 从 DirSync 轻松升级 |● | | | |
| Windows Server 本地化语言 | FR | ● | ● | |
| 对 Windows Server 2008 和 Windows Server 2008 R2 的支持 | ● 支持同步，不支持联合身份验证| ● | ● | ● |
| 对 Windows Server 2012 和 Windows Server 2012 R2 的支持 | ● | ● | ● | 仅限 2012 |


## 筛选和配置

功能 | Azure Active Directory Connect | Azure Active Directory 同步服务 | Azure Active Directory 同步工具 (DirSync) | Forefront Identity Manager 2010 R2 (FIM)  
:-------- |:--------:|:--------:|:--------:|:--------:|
筛选域和组织单位 | ● | ● | ● | ●  
筛选对象的属性值 | ● | ● | ● | ● 
允许同步极少量的属性 (MinSync) | ● | ● | |   
允许对属性流应用不同的服务模板 |● | ● | |   
允许删除从 AD 流向 Azure AD 的属性 | ● | ● | |  
允许对属性流进行高级自定义 | ● | ● | | ●  

<!---HONumber=67-->