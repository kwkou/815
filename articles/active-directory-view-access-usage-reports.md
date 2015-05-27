<properties 
	pageTitle="查看访问和使用情况报告" 
	description="本主题说明如何查看访问和使用情况报告，以深入分析组织目录的完整性和安全性。" 
	services="active-directory" 
	documentationCenter="" 
	authors="Justinha" 
	manager="TerryLan" 
	editor="LisaToft"/>

<tags 
	ms.service="active-directory" 
	ms.workload="infrastructure-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="05/10/2015"
	wacn.date="05/26/2015" 
	ms.author="Justinha"/>

# 查看访问和使用情况报告

你可以使用 Azure Active Directory 的访问和使用情况报告来监控你所在组织的目录的完整性和安全性。使用此信息，目录管理员可以更好地确定哪里可能存在安全风险，以便制定相应的计划来降低风险。

在 Azure 管理门户中，报告按以下方式分类：

- 异常报告 - 包含我们发现存在异常的登录事件。我们的目标是让你知道这项活动并使你能够就事件是否可疑做出决定。 
- 集成应用程序报告 - 就你所在的组织如何使用云应用程序提供见解。Azure Active Directory 提供与数千个云应用程序的集成。 
- 错误报告 - 指示在为外部应用程序设置帐户时可能发生的错误。
- 用户特定的报告 - 显示特定用户的设备/登录活动数据。
- 活动日志 - 包含过去 24 小时、过去 7 天或过去 30 天内的所有已审核事件的记录，以及组活动更改记录、密码重置和注册活动记录。

> [AZURE.NOTE]
> 
- 仅当你已启用 Azure Active Directory 高级版时，一些高级异常报告和资源使用情况报告才可用。高级报告可帮助你提高访问安全性、应对潜在威胁以及获得对设备访问和应用程序使用情况进行分析。
- 在中国，使用 Azure Active Directory 全球实例的客户可以使用 Azure Active Directory 高级和基本版。由中国 21Vianet 运营的 Microsoft Azure 服务目前不支持 Azure Active Directory 高级和基本版。有关详细信息，请在 [Azure Active Directory 论坛](http://feedback.azure.com/forums/169401-azure-active-directory)与我们联系。


## 异常报告

报告名称  | 适用的版本    	
------------- | -------------  
[来自未知源的登录](#sign-ins-from-unknown-sources) | 免费和高级版
[多次失败后的登录](#sign-ins-after-multiple-failures) | 免费和高级版
[来自多个地理区域的登录](sign-ins-from-multiple-geographies) | 免费和高级版
从具有可疑活动的 IP 地址登录| 高级版
异常登录活动 | 高级版
从可能感染的设备登录 | 高级版
具有异常登录活动的用户 | 高级版
应用程序使用情况：摘要 | 高级版
应用程序使用情况：详情 | 高级版
[应用程序仪表板](#application-dashboard) | 免费和高级版
[帐户设置错误](#account-provisioning-errors) | 免费和高级版
设备 | 高级版
[活动](#activity) | 免费和高级版
[审核报告](#audit-report) | 免费和高级版
组活动报告 | 高级版
密码重置注册活动报告 | 高级版
密码重置活动 | 高级版
 

### 来自未知源的登录

<table style=width:100%">
<tr>
<td>说明</td>
<td>报告位置</td>
</tr>
<tr>
<td><p>此报告指示在被分配了 Microsoft 已识别为匿名代理 IP 地址的客户端 IP 地址时已成功登录到你的目录的用户。这些代理通常由隐藏了自己计算机的 IP 地址的用户使用，可能用于恶意目的 - 有时黑客使用这些代理。</p><p>来自此报告的结果将显示用户从该地址和代理的 IP 地址成功登录到你的目录的次数。</p></td>
<td>"目录">"报告"选项卡</td>
</tr>
</table>

### 多次失败后的登录

<table style=width:100%">
<tr>
<td>说明</td>
<td>报告位置</td>
</tr>
<tr>
<td>此报告指示在多次连续登录尝试失败后成功登录的用户。可能的原因包括： <ul><li>用户忘记了自己的密码</li><li>用户是成功密码猜测暴力攻击的受害者</li></ul><p>来自此报告的结果将显示在成功登录前连续失败的登录次数和与第一次成功登录关联的时间戳。</p><p><b>报告设置</b>：你可以配置在报告中显示连续登录尝试失败次数之前必须至少发生的连续登录尝试失败次数。对此设置进行更改时，必须注意这些更改不会应用于当前报告中显示的任何现有失败登录。但是，这些更改将应用于所有将来的登录。对此报告的更改只能由经过许可的管理员进行。</p></td>
<td>"目录">"报告"选项卡</td>
</tr>
</table>

### 来自多个地理区域的登录

<table style=width:100%">
<tr>
<td>说明</td>
<td>报告位置</td>
</tr>
<tr>
<td><p>此报告包括某个用户的成功登录活动，其中两次登录显示为来自不同区域且这两次登录间隔的时间不足以让用户从其中一个区域来到另一个区域。可能的原因包括：</p>
<ul><li>用户共享其密码</li><li>用户使用远程桌面启动 Web 浏览器进行登录</li><li>黑客已从不同国家/地区登录到该用户帐户。</li></ul><p>来自此报告的结果将为你显示成功的登录活动以及两次登录间隔的时间、这些登录似乎源自的区域以及在这两个区域之间往返所需的估计时间。</p><p>显示的往返时间仅是估计值，可能不同于在这两个地点之间往返所需的实际时间。而且，不为两个邻近区域之间的登录生成事件。</p></td>
<td>"目录">"报告"选项卡</td>
</tr>
</table>

## 集成应用程序报告

### 应用程序仪表板

<table style=width:100%">
<tr>
<td>说明</td>
<td>报告位置</td>
</tr>
<tr>
<td>此报告指示在所选时间间隔内你所在组织的用户累积登录该应用程序的次数。面板页面上的图表将帮助你识别该应用程序全部使用情况的趋势。</td>
<td>"目录">"应用程序">"仪表板"选项卡</td>
</tr>
</table>

## 错误报告

### 帐户设置错误

<table style=width:100%">
<tr>
<td>说明</td>
<td>报告位置</td>
</tr>
<tr>
<td>使用此报告监视在从 SaaS 应用程序将帐户同步到 Azure Active Directory 期间发生的错误。 </td>
<td>"目录">"报告"选项卡</td>
</tr>
</table>

## 用户特定的报告

### 活动

<table style=width:100%">
<tr>
<td>说明</td>
<td>报告位置</td>
</tr>
<tr>
<td>显示用户的登录活动。该报告包含登录的应用程序、所使用的设备、IP 地址和位置等信息。我们不收集使用 Microsoft 帐户登录的用户的历史记录。
</td>
<td>"目录">"用户">"<i>用户</i>">"活动"选项卡</td>
</tr>
</table>

#### 用户活动报告中包含的登录事件

用户活动报告中只包括特定类型的登录事件。

| 事件类型										| 是否包括？		|
| ----------------------								| ---------		|
| 登录[访问面板](http://myapps.microsoft.com/)				| 是			|
| 登录 [Azure 管理门户](https://manage.windowsazure.cn/)		| 是			|
| 登录 [Microsoft Azure 门户](https://manage.windowsazure.cn/)			| 是			|
| 登录 [Office 365 门户](https://login.partner.microsoftonline.cn)			| 是			|
| 登录本机应用程序，例如 Outlook（参阅以下例外情况）			| 是			|
| 通过访问面板登录联合/设置的应用程序，例如 Salesforce	| 是			|
| 通过访问面板登录基于密码的应用程序，例如 Twitter		| 否（即将包括）	|
| 登录已添加到目录中的自定义业务应用程序		| 否（即将包括）	|

> 注意：为了减少报告中的干扰信息量，将不会显示 [Lync/Skype for Business](http://products.office.com/zh-cn/skype-for-business/online-meetings) 本机应用程序登录，以及通过 [Microsoft Online Services 登录助手](http://community.office365.com/en-us/w/sso/534.aspx)进行的登录。

## 活动日志

### 审核报告

<table style=width:100%">
<tr>
<td>说明</td>
<td>报告位置</td>
</tr>
<tr>
<td><p>显示在过去 24 小时、过去 7 天或过去 30 天内发生的所有已审核事件的记录。 </p><p>有关详细信息，请参阅 [Azure Active Directory 审核报告事件](active-directory-reporting-audit-events.md)</p>
</td>
<td>"目录">"报告"选项卡</td>
</tr>
</table>

## 当你怀疑有安全漏洞时要考虑的事项

如果你怀疑某个用户帐户可能已泄露或者出现可能导致云中的目录数据产生安全漏洞的任何可疑用户活动，你可能需要考虑采取以下一个或多个措施：

- 联系该用户以核实该活动
- 重置用户密码
- [启用多重身份验证](https://msdn.microsoft.com/zh-cn/library/azure/7a9c56cf-72f1-4797-8e86-a9a2d9569ef6#enableuser)以提高安全性

## 查看或下载报告

1. 在 Azure 管理门户中，单击"Active Directory"，单击组织目录的名称，然后单击"报告"。
2. 在"报告"页面上，单击你要查看和/或下载的报告。
    > [AZURE.NOTE] 如果这是你第一次使用 Azure Active Directory 的报告功能，你将看到一条"选择加入"的消息。如果同意，请单击复选标记图标以继续。
3. 单击"间隔"旁边的下拉菜单，然后从以下时间范围选择其中一个作为生成此报告应使用的时间间隔：
    - 过去 24 小时
    - 过去 7 天
    - 过去 30 天
4. 单击复选标记图标以运行报告。
5. 如果适用，请单击"下载"将报告下载到逗号分隔值 (CSV) 格式的压缩文件中，以便脱机查看或进行存档。

## 忽略事件

如果你正在查看任何异常报告，则可能会注意到你可以忽略相关报告中出现的各种事件。若要忽略某个事件，只需在报告中突出显示该事件，然后单击"忽略"。"忽略"按钮会从报告中永久地删除突出显示的事件，但此按钮只能由获得许可的全局管理员使用。

## 后续步骤

<!--- [Getting started with Azure Active Directory Premium](active-directory-get-started-premium.md)-->
- [向"登录"和"访问面板"页添加公司品牌](active-directory-add-company-branding)

<!--HONumber=57-->