<properties 
	pageTitle="深入分析：Azure AD 密码管理报告 | Windows Azure" 
	description="本文介绍如何使用报告来深入分析组织中的密码管理操作。" 
	services="active-directory" 
	documentationCenter="" 
	authors="asteen" 
	manager="kbrint" 
	editor="billmath"/>

<tags 
	ms.service="active-directory"  
	ms.date="06/08/2015" 
	wacn.date="08/29/2015"/>

# 如何使用密码管理报告获取操作见解
本部分介绍如何使用 Azure Active Directory 的密码管理报告来查看组织中用户对密码重置和更改的使用情况。

- [**密码管理报告概述**](#overview-of-password-management-reports)
- [**如何查看密码管理报告**](#how-to-view-password-management-reports)
- [**在组织中查看密码重置注册活动**](#view-password-reset-registration-activity)
- [**在组织中查看密码重置活动**](#view-password-reset-activity)

## <a name="overview-of-password-management-reports"></a>密码管理报告概述
一旦部署密码重置，最常见的后续步骤之一是查看其在组织中的使用情况。例如，你可能希望了解用户对密码重置的注册情况，或者在过去几天内进行了多少个密码重置。下面是一些常见问题，你将能够使用当今 [Azure 管理门户](https://manage.windowsazure.cn)中的密码管理报告来回答这些问题：

- 有多少人注册了密码重置？
- 谁注册了密码重置？
- 人们都注册哪些数据？
- 有多少人在过去 7 天内重置了他们的密码？
- 用户或管理员用于重置其密码的最常见方法是什么？
- 用户或管理员尝试使用密码重置时面临的常见问题是什么？
- 哪些管理员经常重置自己的密码？
- 密码重置时是否有任何可疑的活动？


##  <a name="how-to-view-password-management-reports"></a>如何查看密码管理报告
若要查找密码管理报告，请按照以下步骤操作：

1.	在 [Azure 管理门户](https://manage.windowsazure.cn)中单击 **Active Directory** 扩展。
2.	从门户显示的列表中选择你的目录。
3.	单击“报表”选项卡。
4.	查看“活动日志”部分的下方。
5.	选择**密码重置活动**报告或**密码重置注册活动**报告。

    ![][001]

##  <a name="view-password-reset-registration-activity"></a>查看密码重置注册活动

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

<table>
            <tbody><tr>
              <td>
                <p>
                  <strong>列</strong>
                </p>
              </td>
              <td>
                <p>
                  <strong>允许值及其含义</strong>
                </p>
              </td>
            </tr>
            <tr>
              <td>
                <p>已注册数据</p>
              </td>
              <td>
                <ul>
                  <li class="unordered">
                    <strong>备用电子邮件</strong> – 用户使用了备用电子邮件或身份验证电子邮件进行身份验证<br><br></li>
                </ul>
                <ul>
                  <li class="unordered">
                    <strong>办公电话</strong> – 用户使用了办公电话进行身份验证<br><br></li>
                </ul>
                <ul>
                  <li class="unordered">
                    <strong>移动电话</strong> – 用户使用了移动电话或身份验证电话进行身份验证<br><br></li>
                </ul>
                <ul>
                  <li class="unordered">
                    <strong>安全问题</strong> – 用户使用了安全问题进行身份验证<br><br></li>
                </ul>
                <ul>
                  <li class="unordered">
                    <strong>上述任一组合（例如，备用电子邮件 + 移动电话）</strong>– 指定两项关口策略时发生，并显示用户使用哪两种方法对其密码重置请求进行身份验证。<br><br></li>
                </ul>
              </td>
            </tr>
          </tbody></table>

##  <a name="view-password-reset-activity"></a>查看密码重置活动

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

<table>
            <tbody><tr>
              <td>
                <p>
                  <strong>列</strong>
                </p>
              </td>
              <td>
                <p>
                  <strong>允许值及其含义</strong>
                </p>
              </td>
            </tr>
            <tr>
              <td>
                <p>所用方法</p>
              </td>
              <td>
                <ul>
                  <li class="unordered">
                    <strong>备用电子邮件</strong> – 用户使用了备用电子邮件或身份验证电子邮件进行身份验证<br><br></li>
                </ul>
                <ul>
                  <li class="unordered">
                    <strong>办公电话</strong> – 用户使用了办公电话进行身份验证<br><br></li>
                </ul>
                <ul>
                  <li class="unordered">
                    <strong>移动电话</strong> – 用户使用了移动电话或身份验证电话进行身份验证<br><br></li>
                </ul>
                <ul>
                  <li class="unordered">
                    <strong>安全问题</strong> – 用户使用了安全问题进行身份验证<br><br></li>
                </ul>
                <ul>
                  <li class="unordered">
                    <strong>上述任一组合（例如，备用电子邮件 + 移动电话）</strong>– 指定两项关口策略时发生，并显示用户使用哪两种方法对其密码重置请求进行身份验证。<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <p>结果</p>
              </td>
              <td>
                <ul>
                  <li class="unordered">
                    <strong>已放弃</strong> – 用户启动了密码重置，但尚未完成便中途停止<br><br></li>
                </ul>
                <ul>
                  <li class="unordered">
                    <strong>已阻止</strong> – 用户帐户因在 24 小时内尝试使用密码重置页面或单个密码重置关口的次数过多而被禁止使用密码重置<br><br></li>
                </ul>
                <ul>
                  <li class="unordered">
                    <strong>已取消</strong> – 用户启动了密码重置，但却在中途单击取消按钮取消了会话 <br><br></li>
                </ul>
                <ul>
                  <li class="unordered">
                    <strong>已联系管理员</strong> – 用户在会话期间遇到无法解决的问题，因此用户单击了“联系管理员”链接而不是完成密码重置流程<br><br></li>
                </ul>
                <ul>
                  <li class="unordered">
                    <strong>已失败</strong> – 用户无法重置密码，可能原因是用户未被配置为使用该功能（例如，无许可证、缺少身份验证信息、密码在本地管理但写回处于关闭状态）。<br><br></li>
                </ul>
                <ul>
                  <li class="unordered">
                    <strong>已成功</strong> – 密码重置成功。<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <p>详细信息</p>
              </td>
              <td>
                <p>请参阅下表</p>
              </td>
            </tr>
          </tbody></table>

###  详细信息列的允许值
下面是你在使用密码重置活动报告时可能会遇到的结果类型列表：

<table>
            <tbody><tr>
              <td>
                <p>
                  <strong>详细信息</strong>
                </p>
              </td>
              <td>
                <p>
                  <strong>结果类型</strong>
                </p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在完成电子邮件验证选项后放弃</p>
              </td>
              <td>
                <p>已放弃</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在完成手机短信验证选项后放弃</p>
              </td>
              <td>
                <p>已放弃</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在完成移动语音呼叫验证选项后放弃</p>
              </td>
              <td>
                <p>已放弃</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在完成办公室语音呼叫验证选项后放弃</p>
              </td>
              <td>
                <p>已放弃</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在完成安全问题选项后放弃</p>
              </td>
              <td>
                <p>已放弃</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在输入其用户 ID 后放弃</p>
              </td>
              <td>
                <p>已放弃</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在启动电子邮件验证选项后放弃</p>
              </td>
              <td>
                <p>已放弃</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在启动手机短信验证选项后放弃</p>
              </td>
              <td>
                <p>已放弃</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在启动移动语音呼叫验证选项后放弃</p>
              </td>
              <td>
                <p>已放弃</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在启动办公室语音呼叫验证选项后放弃</p>
              </td>
              <td>
                <p>已放弃</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在启动安全问题选项后放弃</p>
              </td>
              <td>
                <p>已放弃</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在选择新密码之前放弃</p>
              </td>
              <td>
                <p>已放弃</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在选择新密码时放弃</p>
              </td>
              <td>
                <p>已放弃</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户输入的无效电子邮件验证代码过多，被阻止 24 小时</p>
              </td>
              <td>
                <p>已阻止</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户输入的无效短信验证代码过多，被阻止 24 小时</p>
              </td>
              <td>
                <p>已阻止</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户尝试移动电话语音验证的次数过多，被阻止 24 小时</p>
              </td>
              <td>
                <p>已阻止</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户尝试办公室电话语音验证的次数过多，被阻止 24 小时</p>
              </td>
              <td>
                <p>已阻止</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户尝试回答安全问题的次数过多，被阻止 24 小时</p>
              </td>
              <td>
                <p>已阻止</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户尝试电话号码验证的次数过多，被阻止 24 小时</p>
              </td>
              <td>
                <p>已阻止</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在传递必需的身份验证方法之前取消</p>
              </td>
              <td>
                <p>已取消</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在提交新密码之前取消</p>
              </td>
              <td>
                <p>已取消</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在尝试电子邮件验证选项后联系了管理员</p>
              </td>
              <td>
                <p>已联系管理员</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在尝试手机短信验证选项后联系了管理员</p>
              </td>
              <td>
                <p>已联系管理员</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在尝试移动语音呼叫验证选项后联系了管理员</p>
              </td>
              <td>
                <p>已联系管理员</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在尝试办公室语音呼叫验证选项后联系了管理员</p>
              </td>
              <td>
                <p>已联系管理员</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户在尝试安全问题验证选项后联系了管理员</p>
              </td>
              <td>
                <p>已联系管理员</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>未对此用户启用密码重置。启用配置选项卡下的密码重置以解决此问题</p>
              </td>
              <td>
                <p>已失败</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户没有许可证。你可以将许可证添加到用户以解决此问题</p>
              </td>
              <td>
                <p>已失败</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户试图在不启用 Cookie 的情况下从设备重置</p>
              </td>
              <td>
                <p>已失败</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户帐户已定义足够的身份验证方法。添加身份验证信息以解决此问题</p>
              </td>
              <td>
                <p>已失败</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户的密码在本地管理。你可以启用密码写回以解决此问题</p>
              </td>
              <td>
                <p>已失败</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>我们无法访问你的本地密码重置服务。检查同步计算机的事件日志</p>
              </td>
              <td>
                <p>已失败</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>我们在重置用户的本地密码时遇到问题。检查同步计算机的事件日志</p>
              </td>
              <td>
                <p>已失败</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>此用户不是密码重置用户组的成员。将此用户添加到该组以解决此问题。</p>
              </td>
              <td>
                <p>已失败</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>已对此租户完全禁用密码重置。请参阅 <a href="http://aka.ms/ssprtroubleshoot">http://aka.ms/ssprtroubleshoot</a> 来解决此问题。</p>
              </td>
              <td>
                <p>已失败</p>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户已成功重置密码</p>
              </td>
              <td>
                <p>已成功</p>
              </td>
            </tr>
          </tbody></table>

<br/> <br/> <br/>

**其他资源**


* [什么是密码管理](/documentation/articles/active-directory-passwords)
* [密码管理的工作原理](/documentation/articles/active-directory-passwords-how-it-works)
* [密码管理入门](/documentation/articles/active-directory-passwords-getting-started)
* [自定义密码管理](/documentation/articles/active-directory-passwords-customize)
* [密码管理最佳实践](/documentation/articles/active-directory-passwords-best-practices)
* [密码管理常见问题](/documentation/articles/active-directory-passwords-faq)
* [密码管理疑难解答](/documentation/articles/active-directory-passwords-troubleshoot)
* [了解详细信息](/documentation/articles/active-directory-passwords-learn-more)
* [MSDN 上的密码管理](https://msdn.microsoft.com/zh-cn/library/azure/dn510386.aspx)



[001]: ./media/active-directory-passwords-get-insights/001.jpg "Image_001.jpg"
[002]: ./media/active-directory-passwords-get-insights/002.jpg "Image_002.jpg"
[003]: ./media/active-directory-passwords-get-insights/003.jpg "Image_003.jpg"
 

<!---HONumber=67-->