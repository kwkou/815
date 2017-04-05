<properties
    pageTitle="连接器版本发行历史记录 | Azure"
    description="本主题列出了 Forefront Identity Manager (FIM) 和 Microsoft Identity Manager (MIM) 的连接器的所有版本"
    services="active-directory"
    documentationcenter=""
    author="AndKjell"
    manager="femila"
    editor="" />
<tags
    ms.assetid="6a0c66ab-55df-4669-a0c7-1fe1a091a7f9"
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="03/08/2017"
    wacn.date="04/05/2017"
    ms.author="billmath" />

# 连接器版本发行历史记录
Forefront Identity Manager (FIM) 和 Microsoft Identity Manager (MIM) 的连接器会经常更新。

> [AZURE.NOTE]
本主题仅适用于 FIM 和 MIM。Azure AD Connect 不支持这些连接器。

本主题列出所有已发布的连接器版本。

相关链接：

- [下载最新连接器](http://go.microsoft.com/fwlink/?LinkId=717495)
- [泛型 LDAP 连接器](/documentation/articles/active-directory-aadconnectsync-connector-genericldap/)参考文档
- [泛型 SQL 连接器](/documentation/articles/active-directory-aadconnectsync-connector-genericsql/)参考文档
- [Web 服务连接器](http://go.microsoft.com/fwlink/?LinkID=226245)参考文档
- [PowerShell 连接器](/documentation/articles/active-directory-aadconnectsync-connector-powershell/)参考文档
- [Lotus Domino 连接器](/documentation/articles/active-directory-aadconnectsync-connector-domino/)参考文档

## 1\.1.443.0

发布时间：2017 年 3 月

### 增强功能
- 泛型 SQL：</br>
**方案症状：**SQL 连接器有一个众所周知的限制，即只允许引用一个对象类型，并且需要对成员进行交叉引用。</br> 
**解决方案说明：**在选择了“*”选项的引用的处理步骤中，对象类型的所有组合都会返回给同步引擎。

	>[AZURE.IMPORTANT]
	- 这将创建许多占位符
	- 必须确保命名跨对象类型唯一。


- 泛型 LDAP：</br>
**方案：**
如果在特定分区中只选择了数个容器，则会在整个分区中进行搜索。特定的将通过同步服务进行筛选，但不通过 MA 进行筛选，后者可能导致性能下降。</br>

 **解决方案说明：**更改了 GLDAP 连接器的代码，以便浏览所有容器，在每个容器中搜索对象，不必在整个分区进行搜索。


- Lotus Domino：

  **方案：**支持 Domino 邮件删除功能，允许在导出过程中进行个人删除。</br>**解决方案：**支持配置邮件删除功能，允许在导出过程中进行个人删除。

### 已解决的问题：
- 泛型 Web 服务：
 - 通过 WebService 配置工具在默认 SAP wsconfig 项目中更改服务 URL 时，会发生以下错误：找不到部分路径

      ``'C:\Users\cstpopovaz\AppData\Local\Temp\2\e2c9d9b0-0d8a-4409-b059-dceeb900a2b3\b9bedcc0-88ac-454c-8c69-7d6ea1c41d17\cfg.config\cloneconfig.xml'. ``  


- 泛型 LDAP：
 - 修复了泛型 SQL 水印增量导入时多值属性无法导入的 Bug
 - GLDAP 连接器看不到 AD LDS 中的所有属性
 - 从 LDAP 目录架构中检测不到 UPN 属性时，向导中断
 - 在完全导入过程中，当未选择“objectclass”属性时，增量导入失败，但未显示发现错误
 - “配置分区和层次结构”配置页未显示任何对象的类型与泛型 LDAP MA 中 Novel 服务器的分区相同。仅显示了 RootDSE 分区中的对象。


- 泛型 SQL：
 - 导出多值属性的已删除\\添加值时，这些值不是在数据源中删除\\添加的。


- Lotus Notes：
 - 特定字段“全名”在 Metaverse 中显示正确，但在导出到 Notes 时，属性的值为 Null 或空。
 - 修复了“认证者重复”错误
 - 如果在 Lotus Domino 连接器上选择了没有任何数据的对象以及其他对象，则在执行完全导入时会收到发现错误。
 - 如果在 Lotus Domino 连接器上运行增量导入，则在运行结束时，Microsoft.IdentityManagement.MA.LotusDomino.Service.exe 服务有时会返回应用程序错误。
 - 组成员身份总体运行正常，并且可以进行保留，但在通过运行导出操作尝试从成员身份中删除用户时，会显示更新成功，但用户并未从 Lotus Notes 的成员身份中实际删除。
 - 在 Lotus MA 的配置 GUI 中添加了一个选项，允许选择“追加底部的项”作为导出模式，以便在多值属性的导出过程中追加底部的新项。
 - 连接器会添加所需的逻辑，以便从邮件文件夹和 ID 保管库中删除文件。
 - 删除不适用于跨 NAB 成员的成员身份。
 - 应可从多值属性中成功删除值

## 1\.1.117.0
发布时间：2016 年 3 月

**新连接器** [泛型 SQL 连接器](/documentation/articles/active-directory-aadconnectsync-connector-genericsql/)的初始版本。

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

- [KB3078617](https://support.microsoft.com/zh-cn/kb/3078617) - 1.0.0597，2015 年 9 月
- [KB3044896](https://support.microsoft.com/zh-cn/kb/3044896) - 1.0.0549，2015 年 3 月
- [KB3031009](https://support.microsoft.com/zh-cn/kb/3031009) - 1.0.0534，2015 年 1 月
- [KB3008177](https://support.microsoft.com/zh-cn/kb/3008177) - 1.0.0419，2014 年 9 月
- [KB2936070](https://support.microsoft.com/zh-cn/kb/2936070) - 4.3.1082，2014 年 3 月

**WebServices**

- [KB3008178](https://support.microsoft.com/zh-cn/kb/3008178) - 1.0.0419，2014 年 9 月

**PowerShell**

- [KB3008179](https://support.microsoft.com/zh-cn/kb/3008179) - 1.0.0419，2014 年 9 月

**Lotus Domino**

- [KB3096533](https://support.microsoft.com/zh-cn/kb/3096533) - 1.0.0597，2015 年 9 月
- [KB3044895](https://support.microsoft.com/zh-cn/kb/3044895) - 1.0.0549，2015 年 3 月
- [KB2977286](https://support.microsoft.com/zh-cn/kb/2977286) - 5.3.0712，2014 年 8 月
- [KB2932635](https://support.microsoft.com/zh-cn/kb/2932635) - 5.3.1003，2014 年 2 月
- [KB2899874](https://support.microsoft.com/zh-cn/kb/2899874) - 5.3.0721，2013 年 10 月
- [KB2875551](https://support.microsoft.com/zh-cn/kb/2875551) - 5.3.0534，2013 年 8 月

## 后续步骤
了解有关 [Azure AD Connect 同步](/documentation/articles/active-directory-aadconnectsync-whatis/)配置的详细信息。

了解有关[将本地标识与 Azure Active Directory 集成](/documentation/articles/active-directory-aadconnect/)的详细信息。

<!---HONumber=Mooncake_0327_2017-->
<!---Update_Description: wording update -->