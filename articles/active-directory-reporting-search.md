<properties
	pageTitle="Azure Active Directory Reporting 搜索"
	description="如何搜索自己的 Azure Active Directory 安全报表、活动报表和审核报表"
	services="active-directory"
	documentationCenter=""
	authors="kenhoff"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="07/07/2015"
	wacn.date="08/29/2015"/>

# Azure Active Directory Reporting 搜索

Azure Active Directory 提供目录管理员在多个报表中搜索用户安全性、活动和审核事件的能力。

要查找搜索面板中，则导航到“Azure 管理门户”->“你的 Azure Active Directory”->“报表”。 可以在报表列表的顶部找到面板。

要搜索特定用户的活动或审核事件，则从“自”和“至”字段选择日期范围，键入用户的“UPN”或“显示名称”，并单击复选标记按钮。

## 搜索中包括的报表

并非所有报表都包含在搜索结果中。此表指示了具体包含的报表。

|	报表 |	附送 |
|	------												|	--------			|
|	来自未知源的登录 |	否 |
|	多次失败后的登录 |	否 |
|	来自多个地理区域的登录 |	否 |
|	从具有可疑活动的 IP 地址登录 |	否 |
|	从可能受感染的设备登录 |	否 |
|	异常登录活动 |	否 |
|	具有异常登录活动的用户 |	否 |
|	具有已泄漏凭据的用户 |	否 |
|	审核报告 |	是 |
|	密码重置活动 |	是 |
|	查看密码重置注册活动 |	是 |
|	自助服务组活动 |	是 |
|	应用程序使用情况 |	否 |
|	帐户设置活动 |	是 |
|	密码滚动更新状态 |	否 |
|	帐户设置错误 |	否 |
|	RMS 使用情况 |	否 |
|	最活跃的 RMS 用户 |	否 |
|	RMS 设备使用情况 |	否 |

## 了解详细信息

 - [Azure Active Directory 报表](/documentation/articles/active-directory-view-access-usage-reports)
 - [Azure Active Directory Reporting 审核事件](/documentation/articles/active-directory-reporting-audit-events)

<!---HONumber=67-->