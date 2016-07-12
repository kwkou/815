<properties 
	pageTitle="如何：更改或重置 Azure AD 密码 | Microsoft Azure" 
	description="了解如何注册密码重置、更改自己的密码，以及在忘记密码的情况下重置密码。" 
	services="active-directory" 
	documentationCenter="" 
	authors="asteen" 
	manager="kbrint" 
	editor="billmath"/>

<tags 
	ms.service="active-directory" 
	ms.date="02/16/2016" 
	wacn.date="06/27/2016"/>

# 如何更新自己的密码
如果你不确定如何管理自己的工作或学校帐户密码，来这里就对了！ 请阅读以下内容，了解如何执行常见的步骤，例如更改密码、重置密码或注册密码重置。

* [**如何注册密码重置**](#how-to-register-for-password-reset)
* [**如何从 Office 365 更改密码**](#how-to-change-your-password-from-o365)
* [**如何从访问面板更改密码**](#how-to-change-your-password-from-the-access-panel)
* [**如何重置密码**](#how-to-reset-your-password)
* [**如何解锁帐户**](#how-to-unlock-your-account)
* [**常见问题及其解决方法**](#common-problems-and-their-solutions)

## <a name="how-to-register-for-password-reset"></a>如何注册密码重置
若要注册密码重置，最快的方法是转到 http://aka.ms/ssprsetup。

 1. 导航到 http://aka.ms/ssprsetup。
 2. 输入你的用户名和密码。
 3. 单击“立即设置”选择注册选项。在本例中，我将演示如何注册**身份验证电话**。
 
    ![][011]
   
 4. 从下拉列表中选择国家代码，然后输入**完整电话号码 + 区号**。
 
    ![][012] 
    ![][013]
   
 5. 选择“向我发送短信”或“呼叫我”选项。在本例中，我将选择“向我发送短信”，这会将 6 位数代码发送到我的手机。等待手机收到该代码。
 
    ![][014]
 
 6. 收到代码后，将它输入到输入框中，然后单击“验证”。
 7. 当你看到“谢谢”时，任务即告完成。 现在，你可以使用注册的选项随时在 https://passwordreset.microsoftonline.com 上重置密码。
 
    ![][015]

 >[AZURE.IMPORTANT]如果管理员允许你注册多个选项，我们强烈建议你注册一个备用选项，以便在丢失电话或访问电子邮件时使用。

## <a name="how-to-change-your-password-from-o365"></a>如何从 O365 更改密码
请遵循以下步骤在 Office 365 中更改工作或学校帐户密码。如果你忘记了密码并想要重置，请遵循[此处](#how-to-reset-your-password)所述的步骤。

 1. 使用工作或学校帐户登录到 Office 365。
 2. 转到“设置”>“Office 365 设置”>“密码”>“更改密码”。
 3. 键入旧密码，然后键入新密码并确认。
 4. 单击“保存”。

可以在 [Office 365 文档中心](https://support.office.com/article/Change-my-password-in-Office-365-for-business-d1efbaee-63a7-4c08-ab1d-71bf932bbb5d)阅读更多相关信息。

## <a name="how-to-change-your-password-from-the-access-panel"></a> 如何从访问面板更改密码
请遵循以下步骤从[访问面板](https://myapps.microsoft.com)更改工作或学校帐户密码。如果你忘记了密码并想要重置，请遵循[此处](#how-to-reset-your-password)所述的步骤。

 1. 使用工作或学校帐户登录到 https://myapps.microsoft.com。
 2. 单击“配置文件”选项卡。
 3. 在屏幕右侧单击“更改我的密码”磁贴。
 4. 键入旧密码，然后键入新密码并确认。
 5. 单击“保存”。
 
 更改密码时遇到问题？ 请阅读[常见问题及其解决方法](#common-problems-and-their-solutions)。

## <a name="how-to-reset-your-password"></a>如何重置密码


请遵循以下步骤，从任何工作或学校帐户登录屏幕更改工作或学校帐户密码。

 >[AZURE.IMPORTANT]仅当你的管理员启用了此功能时，你才可以使用此功能。如果未启用，你将看到一条消息，指出没有为你的帐户启用此功能。在此情况下，你可以使用“联系管理员”链接来与管理员联系，以解锁你的帐户。<br><br>如果管理员为你启用了此功能，则你首先需要注册才可使用它。可以在此处完成注册：http://aka.ms/ssprsetup。
 

 1. 在任何工作或学校帐户登录页面上，单击“无法访问帐户?”或“忘记了密码?”链接，或直接导航到 https://passwordreset.microsoftonline.com。
 
    ![][001]
 
 2. 在“你是谁?”页面上，输入你的工作或学校帐户 ID，并通过身份验证来证明你不是机器人。
 
    ![][002]
 
 3. 单击“下一步”按钮。
 4. 选择重置密码的选项。依据管理员配置系统的方式，你可能会看到以下一个或多个选项：
     * **向我的备用电子邮件发送电子邮件** — 将包含 6 位数代码的电子邮件发送到你的**备用电子邮件**或**身份验证电子邮件**（由你选择）。
	 * **向我的移动电话发送短信** — 将包含 6 位数代码的短信发送到你的**移动电话**或**身份验证电子邮件**（由你选择）。
	 * **拨打我的移动电话** — 拨打你的**移动电话**或**身份验证电话**（由你选择）— 按 *#* 键确认呼叫。
	 * **拨打我的办公室电话** — 拨打你的**办公室电话** — 按 *#* 键确认呼叫。
	 * **解答我的安全性问题** — 显示预先注册要回答的安全性问题。

    ![][003]
	 
 5. 我们使用“向我的移动电话发短信”选项作为示例。如果你使用基于电话的选项，则在我们发送短信之前，你需要确认电话号码。请输入完整电话号码，单击“下一步”确认号码正确，然后发送短信。
 
    ![][004]
 
 6. 收到短信后，请确保使用短信正文中的验证码，而不是发送验证码的号码。可能需要等几分钟才能收到短信，你不妨休息片刻。
 
    ![][009]
 
 8. 现在，将手机上刚刚收到的验证码输入到页面上的输入框中。
 
    ![][005]
 
 9. 管理员可能要求完成另一个验证步骤，在这种情况下，请选择不同的选项重复步骤 4。
 10. 在“选择新密码”屏幕上，选择新密码并确认选择，然后单击“完成”。
 
    ![][006]
    ![][007]
  
 11. 看到成功页面表示大功告成！ 现在，你可以使用新密码登录。
 
    ![][008]
 
重置密码时遇到问题？ 请阅读[常见问题及其解决方法](#common-problems-and-their-solutions)。

## <a name="how-to-unlock-your-account"></a>如何解锁帐户
请遵循以下步骤，从任何工作或学校帐户登录屏幕解锁你的本地帐户。**注意：只能解锁本地锁定的帐户。**

 >[AZURE.IMPORTANT] 仅当你的管理员启用了此功能时，你才可以使用此功能。如果未启用，你将看到一条消息，指出没有为你的帐户启用此功能。在此情况下，你可以使用“请与管理员联系”链接来与管理员联系，以解锁你的帐户。<br><br>如果管理员为你启用了此功能，则你首先需要注册才可使用它。可以在此处完成注册：http://aka.ms/ssprsetup。


 1. 在任何工作或学校帐户登录页面上，单击“无法访问帐户?”或“忘记了密码?”链接，或直接导航到 https://passwordreset.microsoftonline.com。
 
    ![][001]
 
 2. 在“你是谁?”页面上，输入你的工作或学校帐户 ID，并通过身份验证来证明你不是机器人。
 
    ![][002]
 
 3. 单击“下一步”按钮。
 4. 选择解锁帐户的选项。依据管理员配置系统的方式，你可能会看到以下一个或多个选项：
     * **向我的备用电子邮件发送电子邮件** — 将包含 6 位数代码的电子邮件发送到你的**备用电子邮件**或**身份验证电子邮件**（由你选择）。
	 * **向我的移动电话发送短信** — 将包含 6 位数代码的短信发送到你的**移动电话**或**身份验证电子邮件**（由你选择）。
	 * **拨打我的移动电话** — 拨打你的**移动电话**或**身份验证电话**（由你选择）— 按 *#* 键确认呼叫。
	 * **拨打我的办公室电话** — 拨打你的**办公室电话** — 按 *#* 键确认呼叫。
	 * **解答我的安全性问题** — 显示预先注册要回答的安全性问题。

    ![][003]
	 
 5. 我们使用“向我的移动电话发短信”选项作为示例。如果你使用基于电话的选项，则在我们发送短信之前，你需要确认电话号码。请输入完整电话号码，单击“下一步”确认号码正确，然后发送短信。
 
    ![][004]
 
 6. 收到短信后，请确保使用短信正文中的验证码，而不是发送验证码的号码。可能需要等几分钟才能收到短信，你不妨休息片刻。
 
    ![][009]
 
 8. 现在，将手机上刚刚收到的验证码输入到页面上的输入框中。
 
    ![][005]
 
 9. 管理员可能要求完成另一个验证步骤，在这种情况下，请选择不同的选项重复步骤 4。
 
 11. 看到成功页面表示大功告成！ 你的本地帐户已解锁，现在你又可以登录了。
 
    ![][010]
  
 >[AZURE.IMPORTANT]请确保将所有设备更新为最新的密码，因为使用旧密码的恶意应用（例如手机电子邮件客户端）经常是帐户最初遭到锁定的罪魁祸首。
 
解锁帐户时遇到问题？ 请阅读[常见问题及其解决方法](#common-problems-and-their-solutions)。

## <a name="common-problems-and-their-solutions"></a>常见问题及其解决方法
以下是一些常见的错误案例及其解决方法：

<table>
          <tbody><tr>
            <td>
              <p>
                <strong>错误案例</strong>
              </p>
            </td>
            <td>
              <p>
                <strong>你看到了什么错误消息？</strong>
              </p>
            </td>
            <td>
              <p>
                <strong>解决方案</strong>
              </p>
            </td>
          </tr>
          <tr>
            <td>
              <p>在输入我的用户 ID 后，出现了“请联系管理员”页面</p>
            </td>
            <td>
              <p>请联系你的管理员 <br><br>我们检测到你的用户帐户密码不受 Microsoft 管理。因此，我们无法自动重置密码。<br><br>你需要联系管理员或客服人员，以寻求进一步的帮助。</p>
            </td>
            <td>
              <p>你之所以看到此消息，是因为管理员在本地环境中管理你的密码，而不允许你从“无法访问帐户链接”重置密码。<b></b><br><br> 若要重置密码，请直接向管理员寻求帮助，使其了解你想要从 Office 365 重置密码，从而为你启用此功能。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>在输入我的用户 ID 后，出现“你的帐户未针对密码重置进行启用”错误</p>
            </td>
            <td>
              <p>未针对密码重置启用帐户<br><br>很抱歉，你的管理员尚未将你的帐户设置为可使用此服务。<br><br> 如果你愿意，我们可以联系你所在组织的管理员为你重置密码。</p>
            </td>
            <td>
              <p>你之所以看到此消息，是因为管理员未针对组织启用“从无法访问帐户”链接重置密码的密码重置功能，或未授权你使用该功能。<b></b><br><br> 若要重置密码，请单击“联系管理员”链接以向公司管理员发送电子邮件，使其了解你想要从 Office 365 重置密码，从而为你启用此功能。<b></b></p>
            </td>
          </tr>
		  <tr>
            <td>
              <p>在输入我的用户 ID 后，出现了“我们无法验证你的帐户”错误</p>
            </td>
            <td>
              <p>我们无法验证你的帐户<br><br>如果你愿意，我们可以联系你所在组织的管理员为你重置密码。</p>
            </td>
            <td>
              <p>你之所以看到此消息，是因为已经为你启用了密码重置，但你并未注册使用此服务。若要注册密码重置，请在重新获取帐户访问权限后转到 http://aka.ms/ssprsetup。<br><br> 若要重置密码，请单击“联系管理员”链接以向公司管理员发送电子邮件。<b></b></p>
            </td>
          </tr>
        </tbody></table>
		

## 密码重置文档的链接
以下是所有 Azure AD 密码重置文档页面的链接：

* [**工作原理**](/documentation/articles/active-directory-passwords-how-it-works/) - 了解六个不同的服务组件及其功能
* [**入门**](/documentation/articles/active-directory-passwords-getting-started/) - 了解如何让用户重置及更改云密码或本地密码
* [**自定义**](/documentation/articles/active-directory-passwords-customize/) - 了解如何根据组织的需求自定义服务的外观和行为
* [**最佳实践**](/documentation/articles/active-directory-passwords-best-practices/) - 了解如何快速部署且有效管理组织的密码
* [**深入分析**](/documentation/articles/active-directory-passwords-get-insights/) - 了解集成式报告功能
* [**常见问题**](/documentation/articles/active-directory-passwords-faq/) - 获取常见问题的解答
* [**故障排除**](/documentation/articles/active-directory-passwords-troubleshoot/) - 了解如何快速排查服务的问题
* [**了解更多**](/documentation/articles/active-directory-passwords-learn-more/) - 深入探索服务工作原理的技术细节



[001]: ./media/active-directory-passwords-update-your-own-password/001.jpg "Image_001.jpg"
[002]: ./media/active-directory-passwords-update-your-own-password/002.jpg "Image_002.jpg"
[003]: ./media/active-directory-passwords-update-your-own-password/003.jpg "Image_003.jpg"
[004]: ./media/active-directory-passwords-update-your-own-password/004.jpg "Image_004.jpg"
[005]: ./media/active-directory-passwords-update-your-own-password/005.jpg "Image_005.jpg"
[006]: ./media/active-directory-passwords-update-your-own-password/006.jpg "Image_006.jpg"
[007]: ./media/active-directory-passwords-update-your-own-password/007.jpg "Image_007.jpg"
[008]: ./media/active-directory-passwords-update-your-own-password/008.jpg "Image_008.jpg"
[009]: ./media/active-directory-passwords-update-your-own-password/009.jpg "Image_009.jpg"
[010]: ./media/active-directory-passwords-update-your-own-password/010.jpg "Image_010.jpg"
[011]: ./media/active-directory-passwords-update-your-own-password/011.jpg "Image_011.jpg"
[012]: ./media/active-directory-passwords-update-your-own-password/012.jpg "Image_012.jpg"
[013]: ./media/active-directory-passwords-update-your-own-password/013.jpg "Image_013.jpg"
[014]: ./media/active-directory-passwords-update-your-own-password/014.jpg "Image_014.jpg"
[015]: ./media/active-directory-passwords-update-your-own-password/015.jpg "Image_015.jpg"
<!---HONumber=Mooncake_0620_2016-->