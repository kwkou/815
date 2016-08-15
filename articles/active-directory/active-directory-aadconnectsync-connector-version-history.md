<properties
   pageTitle="Azure AD Connect 同步：连接器版本发布历史记录 | Azure"
   description="本主题列出了 Forefront Identity Manager (FIM) 和 Microsoft Identity Manager (MIM) 的连接器的所有版本"
   services="active-directory"
   documentationCenter=""
   authors="AndKjell"
   manager="stevenpo"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.date="05/24/2016"
   wacn.date="07/12/2016"/>

# Azure AD Connect 同步：连接器版本发布历史记录
Forefront Identity Manager (FIM) 和 Microsoft Identity Manager (MIM) 的连接器会经常更新。

本文旨在帮助你跟踪已发布的版本，并了解你是否需要更新为最新版本。

相关链接：

- [下载最新连接器](http://go.microsoft.com/fwlink/?LinkId=717495)
- [通用 LDAP 连接器](/documentation/articles/active-directory-aadconnectsync-connector-genericldap/)参考文档
- [通用 SQL 连接器](/documentation/articles/active-directory-aadconnectsync-connector-genericsql/)参考文档
- [Web 服务连接器](http://go.microsoft.com/fwlink/?LinkID=226245)参考文档
- [PowerShell 连接器](/documentation/articles/active-directory-aadconnectsync-connector-powershell/)参考文档
- [Lotus Domino 连接器](/documentation/articles/active-directory-aadconnectsync-connector-domino/)参考文档

## 1\.1.117.0
发布时间：2016 年 3 月

**新连接器**  
[通用 SQL 连接器](/documentation/articles/active-directory-aadconnectsync-connector-genericsql/)的初始版本。

**新功能：**

- 通用 LDAP 连接器：
    - 添加了对通过 Isode 进行增量导入的支持。
- Web 服务连接器：
    - 更新了 csEntryChangeResult 活动和 setImportErrorCode 活动，以允许将对象级别错误返回到同步引擎。
    - 更新了 SAP6 和 SAP6User 模板，以使用新的对象级别错误功能。
- Lotus Domino 连接器：
    - 导出时，每个通讯簿需要一个认证者。现在可以对所有认证者使用同一密码以简化管理。

**已解决的问题：**

- 通用 LDAP 连接器：
    - 对于 IBM Tivoli DS，未正确检测到某些引用属性。
    - 对于增量导入过程中的 Open LDAP，截去了字符串开头和结尾的空格。
    - 对于 Novell 和 NetIQ，导出（在 OU/容器之间移动了对象，同时重命名了对象）失败。
- Web 服务连接器：
    - 如果 Web 服务对于同一绑定具有多个终结点，则连接器未正确发现这些终结点。
- Lotus Domino 连接器：
    - 将 fullName 属性导出到邮件数据库不正常工作。
    - 同时从组中添加和删除成员的导出仅导出了所添加的成员。
    - 如果 Notes Document 无效（isValid 属性设置为 false），则连接器将失败。

## 较旧版本
在 2016 年 3 月之前，连接器已发布为支持主题。

**通用 LDAP**

- [KB3078617](https://support.microsoft.com/kb/3078617) - 1.0.0597，2015 年 9 月
- [KB3044896](https://support.microsoft.com/kb/3044896) - 1.0.0549，2015 年 3 月
- [KB3031009](https://support.microsoft.com/kb/3031009) - 1.0.0534，2015 年 1 月
- [KB3008177](https://support.microsoft.com/kb/3008177) - 1.0.0419，2014 年 9 月
- [KB2936070](https://support.microsoft.com/kb/2936070) - 4.3.1082，2014 年 3 月

**WebServices**

- [KB3008178](https://support.microsoft.com/kb/3008178) - 1.0.0419，2014 年 9 月

**PowerShell**

- [KB3008179](https://support.microsoft.com/kb/3008179) - 1.0.0419，2014 年 9 月

**Lotus Domino**

- [KB3096533](https://support.microsoft.com/kb/3096533) - 1.0.0597，2015 年 9 月
- [KB3044895](https://support.microsoft.com/kb/3044895) - 1.0.0549，2015 年 3 月
- [KB2977286](https://support.microsoft.com/kb/2977286) - 5.3.0712，2014 年 8 月
- [KB2932635](https://support.microsoft.com/kb/2932635) - 5.3.1003，2014 年 2 月  
- [KB2899874](https://support.microsoft.com/kb/2899874) - 5.3.0721，2013 年 10 月
- [KB2875551](https://support.microsoft.com/kb/2875551) - 5.3.0534，2013 年 8 月

## 后续步骤
了解有关 [Azure AD Connect 同步](/documentation/articles/active-directory-aadconnectsync-whatis/)配置的详细信息。

了解有关[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)的详细信息。

<!---HONumber=Mooncake_0704_2016-->