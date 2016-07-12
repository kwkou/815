<properties 
	pageTitle="深入分析：Azure AD 密码管理报告 | Azure" 
	description="本文介绍如何使用报告来深入分析组织中的密码管理操作。" 
	services="active-directory" 
	documentationCenter="" 
	authors="asteen" 
	manager="kbrint" 
	editor="billmath"/>

<tags 
	ms.service="active-directory"  
	ms.date="02/16/2016" 
	wacn.date="06/27/2016"/>

# 如何使用密码管理报告获取 Operational Insights
本部分介绍如何使用 Azure Active Directory 的密码管理报告来查看组织中用户对密码重置和更改的使用情况。

- [**密码管理报告概述**](#overview-of-password-management-reports)
- [**如何查看密码管理报告**](#how-to-view-password-management-reports)
- [**在组织中查看密码重置注册活动**](#view-password-reset-registration-activity)
- [**在组织中查看密码重置活动**](#view-password-reset-activity)

## <a name="overview-of-password-management-reports"></a>密码管理报告概述
一旦部署密码重置，最常见的后续步骤之一是查看其在组织中的使用情况。例如，你可能希望了解用户对密码重置的注册情况，或者在过去几天内进行了多少个密码重置。下面是一些常见问题，你将能够使用当今 [Azure 经典管理门户](https://manage.windowsazure.cn)中的密码管理报告来回答这些问题：

- 有多少人已注册了密码重置？
- 谁已经注册了密码重置？
- 人们都注册哪些数据？
- 有多少人在过去 7 天内重置了他们的密码？
- 用户或管理员用于重置其密码的最常见方法是什么？
- 用户或管理员尝试使用密码重置时面临的常见问题是什么？
- 哪些管理员经常重置其自己的密码？
- 密码重置时是否有任何可疑的活动？


编写有效的脚本后，接下来请根据你的方案，检查可以检索的密码重置和注册事件。

- [SsprActivityEvent](https://msdn.microsoft.com/library/azure/mt126081.aspx#BKMK_SsprActivityEvent)：列出密码重置事件可用的列
- [SsprRegistrationActivityEvent](https://msdn.microsoft.com/library/azure/mt126081.aspx#BKMK_SsprRegistrationActivityEvent)：列出密码重置注册事件可用的列

## <a name="view-password-reset-registration-activity"></a> 查看密码重置注册活动

密码重置注册活动报告显示你的组织中已发生的所有密码重置注册。对于已在密码重置注册门户 ([http://aka.ms/ssprsetup](http://aka.ms/ssprsetup)) 成功注册身份验证信息的所有用户，密码重置注册都显示在此报告中。

- **最大时间范围**：1 个月
- **最大行数**：无限制
- **可下载**：是，通过 CSV 文件

    ![][002]

### 报告列的说明
以下列表详细说明每个报告列：

- **用户** – 尝试了密码重置注册操作的用户。
- **角色** – 该用户在目录中的角色。
- **日期和时间** – 尝试日期和时间。
- **已注册数据** – 用户在密码重置注册期间提供的身份验证数据。

### 报告值的说明
下表描述了每个列的不同允许值：

列|允许值及其含义
---|---
已注册数据| **备用电子邮件** – 用户使用了备用电子邮件或身份验证电子邮件进行身份验证<p><p>**办公电话** – 用户使用了办公室电话进行身份验证<p>**移动电话** – 用户使用了移动电话或身份验证电话进行身份验证<p>**安全问题** – 用户使用了安全问题进行身份验证<p>**上述任一组合（例如，备用电子邮件 + 移动电话）** – 指定双门槛策略时发生，并显示用户使用哪两种方法对其密码重置请求进行身份验证。

## <a name="view-password-reset-activity"></a> 查看密码重置活动

此报告显示你的组织中发生的所有密码重置尝试。

- **最大时间范围**：1 个月
- **最大行数**：无限制
- **可下载**：是，通过 CSV 文件

    ![][003]

###  报告列的说明
以下列表详细说明每个报告列：

1. **用户** – 尝试了密码重置操作的用户（基于用户重置密码时提供的“用户 ID”字段）。
2. **角色** – 该用户在目录中的角色。
3. **日期和时间** – 尝试日期和时间。
4. **所用方法** – 用户针对此重置操作使用的身份验证方法。
5. **结果** – 密码重置操作的最终结果。
6. **详细信息** – 密码重置为什么导致这样值的详细信息。此外，还包括你为解决意外错误而可能采取的任何缓解步骤。

###  报告值的说明
下表描述了每个列的不同允许值：

列|允许值及其含义
---|---
所用方法|**备用电子邮件** – 用户使用了备用电子邮件或身份验证电子邮件进行身份验证<p>**办公电话** – 用户使用了办公室电话进行身份验证<p>**移动电话** – 用户使用了移动电话或身份验证电话进行身份验证<p>**安全问题** – 用户使用了安全问题进行身份验证<p>**上述任一组合（例如，备用电子邮件 + 移动电话）** – 指定双门槛策略时发生，并显示用户使用哪两种方法对其密码重置请求进行身份验证。
结果|**已放弃** – 用户启动了密码重置，但尚未完成便中途停止<p>**已阻止** – 用户帐户因在 24 小时内尝试使用密码重置页面或单个密码重置入口的次数过多而被禁止使用密码重置<p>**已取消** – 用户启动了密码重置，但却在中途单击取消按钮取消了会话<p>**已联系管理员** – 用户在会话期间遇到无法解决的问题，因此用户单击了“联系管理员”链接而不是完成密码重置流程<p>**已失败** – 用户无法重置密码，可能原因是用户未被配置为使用该功能（例如，无许可证、缺少身份验证信息、密码在本地管理但写回处于关闭状态）。<p>**已成功** – 密码重置成功。
详细信息|请参阅下表

### 详细信息列的允许值
下面是你在使用密码重置活动报告时可能会遇到的结果类型列表：

详细信息 | 结果类型
----|----
用户在完成电子邮件验证选项后放弃 | 已放弃
用户在完成手机短信验证选项后放弃|已放弃
用户在完成移动语音呼叫验证选项后放弃 | 已放弃
用户在完成办公室语音呼叫验证选项后放弃 | 已放弃
用户在完成安全问题选项后放弃|已放弃
用户在输入其用户 ID 后放弃| 已放弃
用户在启动电子邮件验证选项后放弃|已放弃
用户在启动手机短信验证选项后放弃|已放弃
用户在启动移动语音呼叫验证选项后放弃|已放弃
用户在启动办公室语音呼叫验证选项后放弃|已放弃
用户在启动安全问题选项后放弃| 已放弃
用户在选择新密码之前放弃| 已放弃
用户在选择新密码时放弃| 已放弃
用户输入的无效短信验证代码过多，被阻止 24 小时|已阻止
用户尝试移动电话语音验证的次数过多，被阻止 24 小时|已阻止
用户尝试办公室电话语音验证的次数过多，被阻止 24 小时 |已阻止
用户尝试回答安全问题的次数过多，被阻止 24 小时| 已阻止
用户尝试电话号码验证的次数过多，被阻止 24 小时|已阻止
用户在传递必需的身份验证方法之前取消|已取消
用户在提交新密码之前取消|已取消
用户在尝试电子邮件验证选项后联系了管理员 |已联系管理员
用户在尝试手机短信验证选项后联系了管理员|已联系管理员
用户在尝试移动语言呼叫验证选项后联系了管理员|已联系管理员
用户在尝试办公室语言呼叫验证选项后联系了管理员 |已联系管理员
用户在尝试安全问题验证选项后联系了管理员|已联系管理员
未对此用户启用密码重置。启用配置选项卡下的密码重置以解决此问题| 已失败
用户没有许可证。你可以将许可证添加到用户以解决此问题|已失败
用户试图从设备重置而不启用 Cookie| 已失败
用户帐户已定义足够的身份验证方法。添加身份验证信息以解决此问题|已失败
用户的密码在本地管理。你可以启用密码写回以解决此问题|已失败
我们无法访问你的本地密码重置服务。检查同步计算机的事件日志|已失败
我们在重置用户的本地密码时遇到问题。检查同步计算机的事件日志 | 已失败
此用户不是密码重置用户组的成员。将此用户添加到该组以解决此问题。|已失败
已对此租户完全禁用密码重置。若要解决此问题，请参阅[此文](/documentation/articles/active-directory-passwords-getting-started/#enable-users-to-reset-their-azure-ad-passwords)。 | 已失败
用户已成功重置密码|已成功

## 密码重置文档的链接
以下是所有 Azure AD 密码重置文档页面的链接：

* [**重置自己的密码**](/documentation/articles/active-directory-passwords-update-your-own-password/) - 了解如何以系统用户的身份重置或更改自己的密码
* [**工作原理**](/documentation/articles/active-directory-passwords-how-it-works/) - 了解六个不同的服务组件及其功能
* [**入门**](/documentation/articles/active-directory-passwords-getting-started/) - 了解如何让用户重置及更改云密码或本地密码
* [**自定义**](/documentation/articles/active-directory-passwords-customize/) - 了解如何根据组织的需求自定义服务的外观和行为
* [**最佳实践**](/documentation/articles/active-directory-passwords-best-practices/) - 了解如何快速部署且有效管理组织的密码
* [**常见问题**](/documentation/articles/active-directory-passwords-faq/) - 获取常见问题的解答
* [**故障排除**](/documentation/articles/active-directory-passwords-troubleshoot/) - 了解如何快速排查服务的问题
* [**了解更多**](/documentation/articles/active-directory-passwords-learn-more/) - 深入探索服务工作原理的技术细节



[001]: ./media/active-directory-passwords-get-insights/001.jpg "Image_001.jpg"
[002]: ./media/active-directory-passwords-get-insights/002.jpg "Image_002.jpg"
[003]: ./media/active-directory-passwords-get-insights/003.jpg "Image_003.jpg"
 
<!---HONumber=Mooncake_0620_2016-->