<properties
	pageTitle="Azure Active Directory Reporting 通知"
	description="如何将 Azure Active Directory Reporting 通知用于可疑登录。"
	services="active-directory"
	documentationCenter=""
	authors="SSalahAhmed"
	manager="TerryLan"
	editor="LisaToft"/>

<tags
	ms.service="active-directory"
	ms.date="06/24/2015"
	wacn.date="08/29/2015"/>

# Azure Active Directory Reporting 通知

## 生成电子邮件通知的报表

目前，只有“异常登录活动”报表和“具有异常登录活动的用户”报表在使用电子邮件通知系统。

## 什么情况将触发电子邮件通知的发送？

默认情况下，Azure Active Directory 设置为自动向所有全局管理员发送电子邮件通知。对于各报表，满足以下条件时就会发送电子邮件。

对于“异常登录活动”报表：

- 未知源：10 个事件
- 多次失败：10 个事件
- 具有可疑活动的 IP 地址：10 个事件
- 感染设备：10 个事件

对于“具有异常登录活动的用户”报表：

- 未知源：10 个事件
- 多次失败：10 个事件
- 具有可疑活动的 IP 地址：10 个事件
- 感染设备：5 个事件
- 异常登录报表：15 个事件

如果在 30 天内或自上一封电子邮件发出后不到 30 天内满足上述任一条件，则会发送电子邮件。

“异常登录”是指已由机器学习算法基于异常登录位置、当天的时间和位置或这些因素的组合标识为“异常”的事件。这可能标名黑客已尝试使用此帐户登录。有关此报表的详细信息，请参见上表。

## 电子邮件通知的接收方是谁？

电子邮件将发送给已向其分配 Active Directory Premium 许可证的所有全局管理员。为确保已投递，我们还将其发送到管理员备用电子邮件地址。管理员应将 aad-alerts-noreply@mail.windowsazure.cn 添加到其安全发件人列表中，以防错过接收电子邮件。

## 这些电子邮件发送的频率是多久？

发送一封电子邮件以后，将仅在发送该电子邮件 30 天内遇到 10 个或更多新异常登录事件的情况下才发送下一封电子邮件。如何访问电子邮件中提到的报表？

单击链接即可重定向到 Azure 管理门户中的报表页。你需要同时具有以下身份才能访问报表：

- Azure 订阅的管理员或共同管理员
- 目录中的全局管理员，并已分配有 Active Directory Premium 许可证。有关详细信息，请参阅 Azure Active Directory 版本。

## 我是否可以关闭这些电子邮件?

可以，要关闭与 Azure 管理门户中的异常登录相关的通知，请单击“配置”，然后选择“通知”部分下的“禁用”。

## 后续步骤
- 想要知道可以使用哪些安全报表、审核报表和活动报表吗？ 查看 [Azure AD 安全报表、审核报表和活动报表](/documentation/articles/active-directory-view-access-usage-reports)
- [Azure Active Directory Premium 入门](/documentation/articles/active-directory-get-started-premium)
- [向“登录”和“访问面板”页添加公司品牌](/documentation/articles/active-directory-add-company-branding)

<!---HONumber=67-->