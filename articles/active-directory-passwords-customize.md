<properties 
	pageTitle="自定义：Azure AD 密码管理 | Azure" 
	description="如何根据需要在 Azure AD 中自定义密码管理外观、行为和通知。" 
	services="active-directory" 
	documentationCenter="" 
	authors="asteen" 
	manager="kbrint" 
	editor="billmath"/>

<tags 
	ms.service="active-directory"  
	ms.date="02/16/2016" 
	wacn.date="06/27/2016"/>

# 自定义密码管理以符合组织的需求
为了尽可能地向用户提供最佳体验，我们建议你了解并试用你可以使用的所有密码管理配置选项。事实上，你可以转到 [Azure 经典管理门户](https://manage.windowsazure.cn)的“Active Directory 扩展”中的配置选项卡，立即开始探索。本主题将引导你从 [Azure 经典管理门户](https://manage.windowsazure.cn)中目录的“配置”选项卡，完成管理员可以进行的不同密码管理自定义，包括：

| 主题 | |
| --------- | --------- |
| 如何启用或禁用密码重置？ | [设置：为用户启用了密码重置](#users-enabled-for-password-reset) |
| 如何将密码重置的范围限定为一组特定的用户？ | [将密码重置限定于特定用户](#restrict-access-to-password-reset) |
| 如何更改支持的身份验证方法？ | [设置：用户可用的身份验证方法](#authentication-methods-available-to-users) |
| 如何更改所需身份验证方法的数量？ | [设置：所需身份验证方法的数量](#number-of-authentication-methods-required) |
| 如何设置自定义安全提问？ | [设置：自定义安全提问](#custom-security-questions) |
| 如何设置预先编写的本地化安全提问？ | [设置：基于知识的安全提问](#knowledge-based-security-questions) |
| 如何更改所需的安全提问数？ | [设置：注册或重置的安全提问数](#number-of-questions-required-to-register) |
| 如何强制用户在登录时注册？ | [基于强制注册的密码重置推广](#require-users-to-register-when-signing-in) |
| 如何强制用户定期重新确认注册？ | [设置：用户必须在几天后重新确认其身份验证数据](#number-of-days-before-users-must-confirm-their-contact-data) |
| 如何自定义用户联系管理员的方式？ | [设置：自定义“联系管理员”链接](#customize-the-contact-your-administrator-link) |
| 如何让用户直接解锁 AD 帐户而不必重置密码？ | [设置：让用户直接解锁 AD 帐户而不必重置密码](#allow-users-to-unlock-accounts-without-resetting-their-password) |
| 如何为用户启用密码重置通知？ | [设置：在用户的密码重置时通知用户](#notify-users-and-admins-when-their-own-password-has-been-reset) |
| 如何为管理员启用密码重置通知？ | [设置：在管理员重置其密码时通知其他管理员](#notify-admins-when-other-admins-reset-their-own-passwords) |



## 密码管理外观
下表说明了每个控件对注册密码重置并重置密码的用户体验有何影响。你可以在 [Azure 经典管理门户](https://manage.windowsazure.cn)中目录的“配置”选项卡的“目录属性”部分下配置这些选项。

<table>
            <tbody><tr>
              <td>
                <p>
                  <strong>策略控制</strong>
                </p>
              </td>
              <td>
                <p>
                  <strong>说明</strong>
                </p>
              </td>
              <td>
                <p>
                  <strong>影响</strong>
                </p>
              </td>
            </tr>
            <tr>
              <td>
                <div id="directory-name">
                  <p>目录名称</p>
                </div>
              </td>
              <td>
                <p>确定用户或管理员可以在密码重置电子邮件通信中看到的组织名称</p>
              </td>
              <td>
                <p>
                  <strong>“联系你的管理员”电子邮件：</strong>
                </p>
                <ul>
                  <li class="unordered">
												确定发件人地址的友好名称，例如，“Microsoft 代表 <strong>Wingtip Toys</strong>”<br><br></li>
                  <li class="unordered">
												确定该电子邮件的主题名，例如，“<strong>Wingtip Toys</strong> 帐户电子邮件验证码”<br><br></li>
                </ul>
                <p>
                  <strong>密码重置电子邮件：</strong>
                </p>
                <ul>
                  <li class="unordered">
												确定发件人地址的友好名称，例如，“Microsoft 代表 <strong>Wingtip Toys</strong>”<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <div id="sign-in-and-access-panel-page-appearance">
                  <p>登录并访问面板页面外观</p>
                </div>
              </td>
              <td>
                <p>确定访问密码重置页面的用户是否能看到 Microsoft 徽标或你自己的自定义徽标。此配置项还会将你的品牌添加到访问面板和登录页中。</p>
                <p>
                  
                </p>
                <p>如需了解有关租户品牌和自定义功能的详细信息，请参阅<a href="https://technet.microsoft.com/zh-cn/library/dn532270.aspx">向“登录”和“访问面板”页添加公司品牌</a>。</p>
              </td>
              <td>
                <p>
                  <strong>密码重置门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												确定你的徽标（而非默认 Microsoft 徽标）是否显示在密码重置门户顶部。<br><br></li>
                  <li class="unordered">
                    <strong>注意：</strong>如果你直接转到密码重置页，则在密码重置门户的第一页可能看不到你的徽标。用户输入其用户 ID 并单击“下一步”后，你的徽标将显示出来。你可以通过将 whr 参数传递到密码重置页，在页面加载时强制显示你的徽标，如下所示：<a href="https://passwordreset.microsoftonline.com?whr=wingtiptoysonline.com">https://passwordreset.microsoftonline.com?whr=wingtiptoysonline.com</a><br><br></li>
                </ul>
                <p>
                  <strong>“联系你的管理员”电子邮件：</strong>
                </p>
                <ul>
                  <li class="unordered">
												确定当用户通过单击密码重置 UI 上的“联系你的管理员”链接与你联系时，发送给管理员的电子邮件底部是否显示你的徽标。<br><br></li>
                </ul>
                <p>
                  <strong>密码重置电子邮件：</strong>
                </p>
                <ul>
                  <li class="unordered">
												确定当用户重置密码时发送给他们的电子邮件底部是否显示你的徽标。<br><br></li>
                </ul>
              </td>
            </tr>
          </tbody></table>

## 密码管理行为
下表说明了每个控件对注册密码重置并重置密码的用户体验有何影响。可以在 [Azure 经典管理门户](https://manage.windowsazure.cn)中目录的“配置”选项卡的“用户密码重置策略”部分下配置这些选项。

> [AZURE.NOTE]你所使用的管理员帐户必须分配了 AAD Premium 许可证才能看到这些策略控件。<br><br>这些策略控件仅适用于重置密码的最终用户，而不是管理员。**Microsoft 为管理员指定了默认的备用电子邮件和/或移动电话策略，这些策略无法更改。**

<table>
            <tbody><tr>
              <td>
                <p>
                  <strong>策略控制</strong>
                </p>
              </td>
              <td>
                <p>
                  <strong>说明</strong>
                </p>
              </td>
              <td>
                <p>
                  <strong>影响</strong>
                </p>
              </td>
            </tr>
            <tr>
              <td>
                <div id="users-enabled-for-password-reset">
                  <p>可进行密码重置的用户</p>
                </div>
              </td>
              <td>
                <p>确定用户是否可在此目录中进行密码重置。</p>
              </td>
              <td>
                <p>
                  <strong>注册门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果设置为“无”，则没有用户能够注册自己的质询数据。<br><br></li>
                  <li class="unordered">
												如果设置为“是”，则目录中的所有最终用户均可通过转至位于 <a href="http://aka.ms/ssprsetup">http://aka.ms/ssprsetup</a> 的注册门户注册质询数据。<br><br></li>
                  <li class="unordered">
                    <strong>注意：</strong>用户必须分配了 Azure AD Premium 或 Basic 许可证才能注册密码重置。<br><br></li>
                </ul>
                <p>
                  <strong>密码重置门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果设置为“无”，用户会看到一条消息，指示必须联系管理员来重置他们的密码。<br><br></li>
                  <li class="unordered">
												如果设置为“是”，则用户可通过转到 <a href="http://passwordreset.microsoftonline.com">http://passwordreset.microsoftonline.com</a> 或单击任何组织 ID 登录页面上的“无法访问你的帐户”链接来自动重置他们的密码<strong></strong>。<br><br></li>
                  <li class="unordered">
                    <strong>注意：</strong>用户必须分配有 Azure AD Premium 或 Basic 许可证才能重置其密码。<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <div id="restrict-access-to-password-reset">
                  <p>限制对密码重置的访问</p>
                </div>
              </td>
              <td>
                <p>确定是否只允许一组特定用户使用密码重置。（仅当“可进行密码重置的用户”设置为“是”时可见）<strong></strong><strong></strong>。</p>
              </td>
              <td>
                <p>
                  <strong>注册门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果设置为“否”，则目录中的所有最终用户均可在 <a href="http://aka.ms/ssprsetup">http://aka.ms/ssprsetup</a> 注册密码重置<br><br></li>
                  <li class="unordered">
												如果设置为“是”，则只有在“可以执行密码重置的组”<strong></strong>中指定的最终用户才能在 <a href="http://aka.ms/ssprsetup">http://aka.ms/ssprsetup</a> 注册密码重置<br><br></li>
                </ul>
                <p>
                  <strong>密码重置门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果设置为“否”，则目录中的所有最终用户均可重置他们的密码。<br><br></li>
                  <li class="unordered">
												如果设置为“是”，则只有在“可以执行密码重置的组”中指定的最终用户才能重置他们的密码<strong></strong>。<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <div id="group-that-can-perform-password-reset">
                  <p>可以执行密码重置的组</p>
                </div>
              </td>
              <td>
                <p>确定允许哪些最终用户组使用密码重置。</p>
                <p>
                  
                </p>
                <p>（仅当“限制对密码重置的访问”设置为“是”时可见）<strong></strong><strong></strong>。</p>
              </td>
              <td>
                <p>
                  <strong>注意：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果未指定组并且单击“保存” <strong></strong>，则将为你创建一个名为 <strong>SSPRSecurityGroupUsers</strong> 的空组。<br><br></li>
                  <li class="unordered">
												如果你想要指定自己的组，可以提供自己的显示名称。<br><br></li>
                </ul>
                <p>
                  <strong>注册门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果“限制对密码重置的访问”设置为“是”，则只有此组中的最终用户能够注册密码重置<strong></strong><strong></strong>。<br><br></li>
                </ul>
                <p>
                  <strong>密码重置门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果“限制对密码重置的访问”设置为“是”，则只有此组中的最终用户能够注册他们的密码<strong></strong><strong></strong>。<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <div id="authentication-methods-available-to-users">
                  <p>用户可使用的身份验证方法</p>
                </div>
              </td>
              <td>
                <p>确定允许用户用于重置其密码的质询。</p>
                <p>
                  
                </p>
                <p>（仅当“可进行密码重置的用户”设置为“是”时可见）<strong></strong><strong></strong>。</p>
              </td>
              <td>
                <p>
                  
                </p>
                <p>
                  
                </p>
                <p>
                  <strong>注意：</strong>
                </p>
                <ul>
                  <li class="unordered">
												必须至少选择一个选项。<br><br></li>
                  <li class="unordered">
												我们强烈建议启用至少 2 个选项，以为重置密码的用户提供最大灵活性。<br><br></li>
                  <li class="unordered">
												如果你使用的是安全问题，我们强烈建议将它们与另一种身份验证方法配合使用，因为安全问题的安全性不如基于电话或电子邮件的密码重置方法。<br><br></li>
                </ul>
                <p>
                  <strong>使用目录中的哪些字段？</strong>
                </p>
                <ul>
                  <li class="unordered">
												“办公电话”对应于目录中用户对象的“办公电话”属性<strong></strong>。<br><br></li>
                  <li class="unordered">
												“移动电话”对应于目录中用户对象的“身份验证移动”属性（非公开可见）或“移动电话”属性（公开可见）<strong></strong><strong></strong>。服务首先检查“身份验证电话”数据，如果不存在，则回退到“移动电话”属性<strong></strong><strong></strong>。<br><br></li>
                  <li class="unordered">
												“备用电子邮件地址”对应于目录中用户对象的“身份验证电子邮件”属性（非公开可见）或“备用电子邮件”属性<strong></strong><strong></strong>。服务首先检查“身份验证电子邮件”数据，如果不存在，则回退到“备用电子邮件”属性<strong></strong><strong></strong>。<br><br></li>
                  <li class="unordered">
												“安全问题”以专用、安全的方式存储在目录的用户对象上，并且只能由用户在注册期间进行回答。出于安全目的，管理员目前无法编辑或查看这些答案。<br><br></li>
                  <li class="unordered">
                    <strong>注意：</strong>默认情况下，只有“办公电话”和“移动电话”两个云属性才会从本地目录同步到云目录。若要详细了解哪些本地属性同步到云，请参阅<a href="https://msdn.microsoft.com/zh-cn/library/azure/dn764938.aspx">同步到 Azure AD 的属性</a>。<br><br></li>
                </ul>
                <p>
                  <strong>注册门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												影响在用户注册时显示的身份验证方法。如果你没有启用给定的身份验证方法，则用户将无法自行注册该身份验证方法。<br><br></li>
                  <li class="unordered">
                    <strong>注意：</strong>目前用户无法注册自己的办公电话号码；该身份验证方法必须由他们的管理员定义。<br><br></li>
                </ul>
                <p>
                  <strong>密码重置门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												确定用户可用作给定验证步骤的质询的身份验证方法。例如，如果用户在 Azure Active Directory 的“办公电话”和“身份验证电话”字段中都输入了数据，则他（她）可以使用其中任一身份验证方法来恢复自己的密码<strong></strong><strong></strong>。<br><br></li>
                  <li class="unordered">
                    <strong>注意：</strong>当且仅当用户在你以管理员身份启用的身份验证方法中输入了数据时，他们才能重置自己的密码。<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <div id="number-of-authentication-methods-required">
                  <p>所需身份验证方法的数量</p>
                </div>
              </td>
              <td>
                <p>确定为重置他或她的密码用户必须通过的可用身份验证方法数量的下限。</p>
                <p>
                  
                </p>
                <p>（仅当“可进行密码重置的用户”设置为“是”时可见）<strong></strong><strong></strong>。</p>
              </td>
              <td>
                <p>
                  <strong>注意：</strong>
                </p>
                <ul>
                  <li class="unordered">
												可以设置 1 或 2 个所需身份验证方法。<br><br></li>
                </ul>
                <p>
                  <strong>注册门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												确定在能够完成注册体验前用户必须注册的身份验证方法数量下限。<br><br></li>
                </ul>
                <p>
                  <strong>密码重置门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												影响在能够重置密码前用户必须完成的验证步骤数量。一个验证步骤是指用户使用一条身份验证信息（如呼叫他们的办公电话或向他们的备用电子邮件发送电子邮件）验证他们的帐户。<br><br></li>
                  <li class="unordered">
                    <strong>注意：</strong>如果某个用户的帐户中尚未定义按照你所设定的策略成功重置其密码所需的身份验证信息数量，他或她将看到一个错误页面，该页面将指示该用户请求管理员来重置其密码。 <br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <div id="number-of-questions-required-to-register">
                  <p>注册所需的问题数量</p>
                </div>
              </td>
              <td>
                <p>确定用户在注册安全问题选项时必须回答的问题数量的下限。</p>
                <p>（仅当启用了“安全问题”复选框时可见）<strong></strong>。</p>
              </td>
              <td>
                <p>
                  <strong>注意：</strong>
                </p>
                <ul>
                  <li class="unordered">
												可以设置 3 到 5 个注册所需问题。<br><br></li>
                  <li class="unordered">
												注册所需的问题数量必须大于或等于重置所需的问题数量。<br><br></li>
                  <li class="unordered">
												我们建议将注册所有的问题数量设置得高于重置所需的问题数量，以使用户可以更灵活地重置他们的密码。这种配置也更为安全，因为我们将从用户已注册的所有问题中为用户随机选择要回答的问题。<br><br></li>
                </ul>
                <p>
                  <strong>注册门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												确定用户在被视为已完全注册密码重置之前必须回答的问题数量下限。<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <div id="number-of-questions-required-to-reset">
                  <p>重置所需的问题数量 </p>
                </div>
              </td>
              <td>
                <p>确定用户在重置密码时必须回答的问题数量下限。</p>
                <p>
                  
                </p>
                <p>（仅当启用了“安全问题”复选框时可见）<strong></strong>。</p>
              </td>
              <td>
                <p>
                  <strong>注意：</strong>
                </p>
                <ul>
                  <li class="unordered">
												可以设置 3 到 5 个重置所需问题。<br><br></li>
                  <li class="unordered">
												重置所需的问题数量必须小于或等于注册所需的问题数量。<br><br></li>
                </ul>
                <p>
                  <strong>密码重置门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												确定用户在重置密码前必须回答的问题数量下限。<br><br></li>
                  <li class="unordered">
												重置密码时，将从用户的已注册问题总列表中随机选择此问题数量。例如，如果用户已注册了 5 个问题，则当用户重置密码时，将从这 5 个问题中随机选择 3 个问题。然后，用户必须正确回答所有这些问题才能重置密码。<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <div id="knowledge-based-security-questions">
                  <p>基于知识的安全提问</p>
                </div>
              </td>
              <td>
                <p>定义用户在注册密码重置以及重置他们的密码时可以选择的预先编写的安全问题。</p>
                <p>
                  
                </p>
                <p>（仅当启用了“安全问题”复选框时可见）<strong></strong>。</p>
              </td>
              <td>
                <p>
                  <strong>注意：</strong>
                </p>
                <ul>
                  <li class="unordered">
												将会根据用户的浏览器区域设置，将所有基于知识的问题本地化为完整的 O365 语言集。<br><br></li>
                  <li class="unordered">
												总共最多可以定义 20 个问题（自定义问题和基于知识的问题的总和）。<br><br></li>
                 <li class="unordered">
												答案包含的字符数量下限为 3 个字符。<br><br></li>
                  <li class="unordered">
												答案包含的字符数量上限为 40 个字符。<br><br></li>
                  <li class="unordered">
												用户不能对同一问题回答两次。<br><br></li>
                  <li class="unordered">
												用户不能为两个不同的问题提供相同的答案。<br><br></li>
                  <li class="unordered">
												可以使用任何字符集来定义答案（包括 Unicode 字符）。<br><br></li>
                  <li class="unordered">
												所定义问题的数量必须大于或等于注册所需问题数量。<br><br></li>
                </ul>
                <p>
                  <strong>注册门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												确定用户在注册密码重置时可以回答的问题。<br><br></li>
                </ul>
                <p>
                  <strong>密码重置门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												确定用户在重置密码时可以使用的问题。<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <div id="custom-security-questions">
                  <p>自定义安全提问</p>
                </div>
              </td>
              <td>
                <p>定义用户在注册密码重置以及重置他们的密码时可以选择的安全问题。</p>
                <p>
                  
                </p>
                <p>（仅当启用了“安全问题”复选框时可见）<strong></strong>。</p>
              </td>
              <td>
                <p>
                  <strong>注意：</strong>
                </p>
                <ul>
                  <li class="unordered">
												总共最多可以定义 20 个问题（自定义问题和基于知识的问题的总和）。<br><br></li>
                  <li class="unordered">
												问题包含的字符数量上限为 200 个字符。<br><br></li>
                  <li class="unordered">
												答案包含的字符数量下限为 3 个字符。<br><br></li>
                  <li class="unordered">
												答案包含的字符数量上限为 40 个字符。<br><br></li>
                  <li class="unordered">
												用户不能对同一问题回答两次。<br><br></li>
                  <li class="unordered">
												用户不能为两个不同的问题提供相同的答案。<br><br></li>
                  <li class="unordered">
												可以使用任何字符集来定义问题和答案（包括 Unicode 字符）。<br><br></li>
                  <li class="unordered">
												所定义问题的数量必须大于或等于注册所需问题数量。<br><br></li>
                  <li class="unordered">
												不支持为不同的区域设置定义不同的自定义问题。所有自定义问题的显示语言是你在管理 UI 中输入这些问题时所用的语言，即使用户浏览器的区域设置与此不同。如果你需要本地化这些问题，请改用“基于知识的”问题。<br><br></li>
                </ul>
                <p>
                  <strong>注册门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												确定用户在注册密码重置时可以回答的问题。<br><br></li>
                </ul>
                <p>
                  <strong>密码重置门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												确定用户在重置密码时可以使用的问题。<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <div id="require-users-to-register-when-signing-in">
                  <p>要求用户登录时注册？</p>
                </div>
                <p>
                  
                </p>
              </td>
              <td>
                <p>确定在下次登录时是否要求用户注册联系人数据以进行密码重置。 
                </p>
                <p>此功能适用于使用工作或学校帐户的任何登录页面。此类页面包括所有 Office 365、Azure 经典管理门户、访问面板和任何使用 Azure AD 登录的联合或自定义开发应用程序。
                </p>
                <p>
                  
                </p>
                <p>强制注册只适用于启用了密码重置的用户，因此，如果你已使用“限制访问密码重置”功能并将密码重置的范围限定为一组特定的用户，则只有该组中的用户需要在登录时注册密码重置。</p>
                <p>
                  
                </p>
                <p>（仅当“可进行密码重置的用户”设置为“是”时可见）<strong></strong><strong></strong>。</p>
              </td>
              <td>
                <p>
                  
                </p>
                <p>
                  
                </p>
                <p>
                  <strong>注意：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果禁用此功能，你也可以手动将用户发送至 <a href="http://aka.ms/ssprsetup">http://aka.ms/ssprsetup</a> 以注册其联系人数据。 <br><br></li>
                  <li class="unordered">
												用户也可通过单击访问面板中配置文件选项卡下的“注册进行密码重置”链接来访问注册门户<strong></strong>。<br><br></li>
                  <li class="unordered">
												可通过单击取消按钮或关闭窗口忽略此注册方法，但如果用户未进行注册，则等到登录时就会非常麻烦。<br><br></li>
                </ul>
                <p>
                  <strong>注册门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												此设置不会影响注册门户本身的行为，然而，它将确定用户登录到访问面板时是否为他们显示注册门户。<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <div id="number-of-days-before-users-must-confirm-their-contact-data">
                  <p>用户必须确认其联系人数据前的天数</p>
                </div>
              </td>
              <td>
                <p>启用“要求用户注册”时，此设置将确定用户必须再次确认其数据前可经过的时间段<strong></strong>。</p>
                <p>
                  
                </p>
                <p>（仅当“登录到访问面板时要求用户注册”设置为“是”时可见）<strong></strong><strong></strong>。</p>
              </td>
              <td>
                <p>
                  
                </p>
                <p>
                  
                </p>
                <p>
                  <strong>注意：</strong>
                </p>
                <ul>
                  <li class="unordered">
												可接受 0-730 天以内的值，0 天表示永远不会要求用户再次确认其联系人数据。<br><br></li>
                </ul>
                <p>
                  <strong>注册门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												此设置不会影响注册门户本身的行为，然而，它将确定需要再次确认用户的联系人数据时是否为他们显示注册门户。<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <div id="customize-the-contact-your-administrator-link">
                  <p>自定义“联系你的管理员”链接？</p>
                </div>
              </td>
              <td>
                <p>控制发生错误时或用户对指向自定义 URL 或电子邮件地址的操作等待太长时间时是否在密码重置门户上显示“联系你的管理员”链接（显示在左边）。</p>
                <p>
                  
                </p>
                <p>（仅当“可进行密码重置的用户”设置为“是”时可见）<strong></strong><strong></strong>。</p>
              </td>
              <td>
                <p>
                  <strong>注意：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果启用此设置，你必须通过立即填写此设置下的“自定义电子邮件地址或 URL”字段来选择自定义 URL 或电子邮件地址<strong></strong>。<br><br></li>
                </ul>
                <p>
                  <strong>密码重置门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果设置为“否”，用户单击突出显示的链接将会分发一封电子邮件到所有租户管理员的主电子邮件地址，要求重置其密码。此电子邮件遵循以下逻辑顺序分发：<br><br></li>
                  <li class="unordered">
                    <ul>
                      <li class="unordered">
														如果有密码管理员，则将电子邮件发送给所有密码管理员，最多可发送给 100 名收件人。<br><br></li>
                      <li class="unordered">
														如果没有密码管理员，则将电子邮件发送给所有用户管理员，最多可发送给 100 名收件人。<br><br></li>
                      <li class="unordered">
														如果没有用户管理员，则将电子邮件发送给所有全球管理员，最多可发送给 100 名收件人。<br><br></li>
                    </ul>
                  </li>
                  <li class="unordered">
												如果设置为“是”，则此设置会将上方突出显示的链接的行为自定义为指向自定义 URL 或电子邮件地址，用户可导航到这些 URL 或电子邮件地址以获得有关密码重置的帮助。<br><br></li>
                  <li class="unordered">
												如果指定的是 URL，它将在新选项卡中打开。<br><br></li>
                  <li class="unordered">
												如果指定的是电子邮件地址，我们将创建一个指向该电子邮件地址的链接。<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <div id="custom-email-address-or-URL">
                  <p>自定义电子邮件地址或 URL</p>
                </div>
              </td>
              <td>
                <p>控制“联系你的管理员”链接所指向的电子邮件地址或 URL<strong></strong>。</p>
                <p>
                  
                </p>
                <p>（仅当“自定义“联系你的管理员”链接”设置为“是”时可见）<strong></strong><strong></strong>。</p>
              </td>
              <td>
                <p>
                  <strong>注意：</strong>
                </p>
                <ul>
                  <li class="unordered">
												必须是有效的 Intranet 或 Extranet URL 电子邮件地址。<br><br></li>
                </ul>
                <p>
                  <strong>密码重置门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												更改“联系你的管理员”链接所指向的位置<strong></strong>。<br><br></li>
                  <li class="unordered">
												如果你提供的是电子邮件地址，此链接将成为指向该电子邮件地址的“mailto”链接。<br><br></li>
                  <li class="unordered">
												如果你提供的是 URL，此链接将成为指向该 URL 的标准 href 并将在一个新选项卡中打开。 <br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <div id="write-back-passwords-to-on-premises-directory">
                  <p>将密码写回到本地目录</p>
                </div>
              </td>
              <td>
                <p>控制是否为此目录启用密码写回功能，如果启用写回，该项将指示本地写回服务的状态。</p>
                <p>
                  
                </p>
                <p>如果你想要暂时禁用服务而不重新配置 Azure AD Connect，此设置将很有用。</p>
              </td>
              <td>
                <p>
                  
                </p>
                <p>
                  <strong>注意：</strong>
                </p>
                <ul>
                  <li class="unordered">
												仅当你已通过下载 Azure AD Connect 最新版本并启用“可选功能”选择屏幕下方的“密码写回”选项安装了密码写回时，此控件才会出现<strong></strong><strong></strong>。<br><br></li>
                  <li class="unordered">
												如果你启用了密码写回功能，但认为存在服务配置问题，可以进入此选项卡并查看“密码写回服务状态”标签，看看是否有问题<strong></strong>。<br><br></li>
                  <li class="unordered">
												可能显示的状态如下：<br><br><ul><li class="unordered"><strong>已配置</strong> - 所有内容均按预期工作<br><br></li><li class="unordered"><strong>未配置</strong> – 已安装写回功能，但我们不能访问此服务，请检查并确保你未阻止到 443 的出站连接；如果问题仍然存在，请尝试重新安装该服务。<br><br></li></ul></li>
                </ul>
                <p>
                  <strong>注册门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果写回功能已部署并配置，但此开关设置为“无”，则将禁用写回，因此联合用户和已进行密码哈希同步的用户将不能注册密码重置<strong></strong>。<br><br></li>
                  <li class="unordered">
												如果此开关设置为“是”，则将启用写回，因此联合用户和已进行密码哈希同步的用户将能够重置他们的密码<strong></strong>。<br><br></li>
                </ul>
                <p>
                  <strong>密码重置门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果写回功能已部署并配置，但此开关已设置为“无”，则将禁用写回，因此联合用户和已进行密码哈希同步的用户将不能重置他们的重置<strong></strong>。<br><br></li>
                  <li class="unordered">
												如果此开关设置为“是”，则将启用写回，因此联合用户和已进行密码哈希同步的用户将能够重置他们的密码<strong></strong>。<br><br></li>
                </ul>
              </td>
            </tr>
             <tr>
              <td>
                <div id="allow-users-to-unlock-accounts-without-resetting-their-password">
                  <p>允许用户在不重置密码的情况下解锁帐户</p>
                </div>
              </td>
              <td>
              
              <p>指定是否应为浏览密码重置门户的用户提供选项，让他们在不重置密码的情况下解锁本地 Active Directory 帐户。默认情况下，Azure AD 在执行密码重置时始终会解锁帐户，此设置可让你区分这两项操作。</p>
              
              <p>如果设置为“是”，将提供用户重置其密码以解锁帐户的选项，或者在不重置密码的情况下解锁的选项。</p>
              
              <p>如果设置为“否”，用户只能同时执行密码重置和帐户解锁的操作。</p>

              </td>
              <td>
                <p>
                  <strong>注意：</strong>
                </p>
                <ul>
                  <li class="unordered">
												若要使用此功能，必须安装 2015 年 8 月或更高版本的 Azure AD Connect（v.1.0.8667.0 或更高版本）。<br><br><a href="http://www.microsoft.com/download/details.aspx?id=47594">单击此处下载最新版本的 Azure AD Connect。</a></li>
                        
                  <li class="unordered">
                    <strong>注意：</strong>若要测试此功能，需要启用密码写回，并使用源自本地（例如联合或密码同步的用户）的帐户，并且有一个已锁定的帐户。非本地并且没有锁定帐户的用户将看不到解锁其帐户的选项。</li>
                </ul>
                <p>
                  <strong>密码重置门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												启用此选项后，当本地帐户被锁定的用户打开密码重置门户时，将提供他（她）在不重置密码的情况下解锁其帐户的选项。<br><br>请注意，如果你使用密码写回，则密码重置时帐户就已自动解锁，此选项只是区分这些操作。<br><br>如果你发现许多技术支持来电都是帐户解锁请求，则启用此选项特别有用。</li>
                </ul>
              </td>
            </tr>
          </tbody></table>

## <a name="password-management-notifications"></a>密码管理通知
下表介绍每个控件对接收密码重置通知的用户和管理员的体验有何影响。可以在 [Azure 经典管理门户](https://manage.windowsazure.cn)中目录“配置”选项卡的“通知”部分下方配置这些选项。

<table>
            <tbody><tr>
              <td>
                <p>
                  <strong>策略控制</strong>
                </p>
              </td>
              <td>
                <p>
                  <strong>说明</strong>
                </p>
              </td>
              <td>
                <p>
                  <strong>影响</strong>
                </p>
              </td>
            </tr>
            <tr>
              <td>
                <div id="notify-admins-when-other-admins-reset-their-own-passwords">
                  <p>当其他管理员重置自己的密码时通知管理员</p>
                </div>
              </td>
              <td>
                <p>确定当其他任何类型的管理员重置他或她自己的密码时，是否通过向所有全球管理员的主电子邮件地址发送电子邮件来通知他们。</p>
              </td>
              <td>
                <p>
                  <strong>密码重置门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果设置为“否”，则将不发送任何通知。<br><br></li>
                  <li class="unordered">
												如果设置为“是”，则当任何一个管理员重置他（她）自己的密码时都将通知所有其他全局管理员。<br><br></li>
                  <li class="unordered">
												此通知将以电子邮件形式发送到组织中的所有其他全局管理员的主电子邮件地址。<br><br></li>
                </ul>
                <p>
                  <strong>示例：</strong>
                </p>
                <ul>
                  <li class="unordered">
												启用此选项后，如果管理员 A 重置了其密码，并且租户 B、C 和 D 中还有 3 位管理员，则管理员 B、C 和 D 将会收到一封电子邮件，指示管理员 A 重置了其密码。<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <div id="notify-users-and-admins-when-their-own-password-has-been-reset">
                  <p>当用户和管理员自己的密码已被重置时通知他们</p>
                </div>
              </td>
              <td>
                <p>确定其密码被重置的最终用户或管理员是否将收到通知其密码已被重置的电子邮件通知。</p>
              </td>
              <td>
                <p>
                  <strong>密码重置门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果设置为“否”，则将不发送任何通知。<br><br></li>
                  <li class="unordered">
												如果设置为“是”，则当某用户或管理员自己的密码被重置时，他（她）将会收到一个指示他或她的密码已被重置的通知。<br><br></li>
                  <li class="unordered">
												此通知将以电子邮件形式发送到其密码被重置的用户的主电子邮件地址和备用（或身份验证）电子邮件地址。<br><br></li>
                </ul>
              </td>
            </tr>
          </tbody></table>


<br/> <br/> <br/>

## 密码重置文档的链接
以下是所有 Azure AD 密码重置文档页面的链接：

* [**重置自己的密码**](/documentation/articles/active-directory-passwords-update-your-own-password/) — 了解如何以系统用户的身份重置或更改自己的密码
* [**工作原理**](/documentation/articles/active-directory-passwords-how-it-works/) - 了解六个不同的服务组件及其功能
* [**入门**](/documentation/articles/active-directory-passwords-getting-started/) - 了解如何让用户重置及更改云密码或本地密码
* [**最佳实践**](/documentation/articles/active-directory-passwords-best-practices/) - 了解如何快速部署且有效管理组织的密码
* [**深入分析**](/documentation/articles/active-directory-passwords-get-insights/) - 了解集成式报告功能
* [**常见问题**](/documentation/articles/active-directory-passwords-faq/) - 获取常见问题的解答
* [**故障排除**](/documentation/articles/active-directory-passwords-troubleshoot/) - 了解如何快速排查服务的问题
* [**了解更多**](/documentation/articles/active-directory-passwords-learn-more/) - 深入探索服务工作原理的技术细节


[001]: ./media/active-directory-passwords-customize/001.jpg "Image_001.jpg"

<!---HONumber=Mooncake_0620_2016-->