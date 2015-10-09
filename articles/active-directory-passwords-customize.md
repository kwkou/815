<properties 
	pageTitle="自定义：Azure AD 密码管理 | Windows Azure" 
	description="如何根据需要在 Azure AD 中自定义密码管理外观、行为和通知。" 
	services="active-directory" 
	documentationCenter="" 
	authors="asteen" 
	manager="kbrint" 
	editor="billmath"/>

<tags 
	ms.service="active-directory"  
	ms.date="06/08/2015" 
	wacn.date="08/29/2015"/>

# 自定义密码管理以符合组织的需求
为了尽可能地向用户提供最佳体验，我们建议你了解并试用你可以使用的所有密码管理配置选项。事实上，你可以通过转到 [Azure 管理门户](https://manage.windowsazure.cn)中“Active Directory 扩展”的配置选项卡立即开始了解。本主题将指导你完成所有不同的密码管理自定义，你可以从 [Azure 管理门户](https://manage.windowsazure.cn)中你目录的“配置”选项卡以管理员身份进行这些自定义，这些自定义包括：

- [**自定义密码管理外观**](#password-managment-look-and-feel)
- [**自定义用户密码管理行为**](#password-management-behavior)
- [**自定义密码管理通知**](#password-management-notifications)

## <a name="password-managment-look-and-feel"></a>密码管理外观
下表说明了每个控件对注册密码重置并重置密码的用户体验有何影响。你可以在 [Azure 管理门户](https://manage.windowsazure.cn)内你目录的“配置”选项卡的“目录属性”部分下方配置这些选项。

<table>
            <tbody><tr>
              <td>
                <p>
                  <strong>策略控件</strong>
                </p>
              </td>
              <td>
                <p>
                  <strong>描述</strong>
                </p>
              </td>
              <td>
                <p>
                  <strong>影响？</strong>
                </p>
              </td>
            </tr>
            <tr>
              <td>
                <p>目录名称</p>
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
												确定发件人地址的友好名称，例如“Microsoft 代表 <strong>Wingtip Toys</strong>”<br><br></li>
                  <li class="unordered">
												确定该电子邮件的主题名，例如“<strong>Wingtip Toys</strong> 帐户电子邮件验证码”<br><br></li>
                </ul>
                <p>
                  <strong>密码重置电子邮件：</strong>
                </p>
                <ul>
                  <li class="unordered">
												确定发件人地址的友好名称，例如“Microsoft 代表 <strong>Wingtip Toys</strong>”<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <p>登录页和访问面板页的外观</p>
              </td>
              <td>
                <p>确定访问密码重置页的用户是否能够看到 Microsoft 徽标或你自己的自定义徽标。此配置项还会将你的品牌添加到访问面板和登录页中。</p>
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

## <a name="password-management-behavior"></a>密码管理行为
下表说明了每个控件对注册密码重置并重置密码的用户体验有何影响。你可以在 [Azure 管理门户](https://manage.windowsazure.cn)内你目录的“配置”选项卡的“用户密码重置策略”部分下方配置这些选项。

> [AZURE.NOTE]你使用的管理员帐户必须分配有 AAD Premium 许可证才能查看这些策略控件。<br><br>这些策略控件仅适用于重置其密码的最终用户，而不适用于管理员。**Microsoft 为管理员指定了默认的备用电子邮件和/或移动电话策略，这些策略无法更改。**

<table>
            <tbody><tr>
              <td>
                <p>
                  <strong>策略控件</strong>
                </p>
              </td>
              <td>
                <p>
                  <strong>描述</strong>
                </p>
              </td>
              <td>
                <p>
                  <strong>影响？</strong>
                </p>
              </td>
            </tr>
            <tr>
              <td>
                <p>可进行密码重置的用户</p>
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
												如果设置为“否”，则没有用户能够注册自己的质询数据。<br><br></li>
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
												如果设置为“否”，用户会看到一条消息，指示必须联系管理员来重置他们的密码。<br><br></li>
                  <li class="unordered">
												如果设置为“是”，则用户可通过转到 <a href="http://passwordreset.microsoftonline.com">http://passwordreset.microsoftonline.com</a> 或单击任何组织 ID 登录页上的“无法访问你的帐户”<strong></strong>链接来自动重置他们的密码。<br><br></li>
                  <li class="unordered">
                    <strong>注意：</strong>用户必须分配了 Azure AD Premium 或 Basic 许可证才能重置其密码。<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <p>限制对密码重置的访问</p>
              </td>
              <td>
                <p>确定是否只允许一组特定用户使用密码重置。（仅当“可进行密码重置的用户”<strong></strong>设置为“是”<strong></strong>时可见）。</p>
              </td>
              <td>
                <p>
                  <strong>注册门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果设置为“否”，则目录中的所有最终用户均可在 <a href="http://aka.ms/ssprsetup">http://aka.ms/ssprsetup</a> 注册密码重置<br><br></li>
                  <li class="unordered">
												如果设置为“是”，则只有在“可以执行密码重置的组”<strong></strong>控件中指定的最终用户才能在 <a href="http://aka.ms/ssprsetup">http://aka.ms/ssprsetup</a> 注册密码重置<br><br></li>
                </ul>
                <p>
                  <strong>密码重置门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果设置为“否”，则目录中的所有最终用户均可重置他们的密码。<br><br></li>
                  <li class="unordered">
												如果设置为“是”，则只有在“可以执行密码重置的组”<strong></strong>控件中指定的最终用户才能重置他们的密码。<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <p>可以执行密码重置的组</p>
              </td>
              <td>
                <p>确定允许哪些最终用户组使用密码重置。</p>
                <p>
                  
                </p>
                <p>（仅当“限制对密码重置的访问”<strong></strong>设置为“是”<strong></strong>时可见）。</p>
              </td>
              <td>
                <p>
                  <strong>注意：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果未指定组并且单击“保存”<strong></strong>，则将为你创建一个名为 <strong>SSPRSecurityGroupUsers</strong> 的空组。<br><br></li>
                  <li class="unordered">
												如果你想要指定自己的组，可以提供自己的显示名称。<br><br></li>
                </ul>
                <p>
                  <strong>注册门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果“限制对密码重置的访问”<strong></strong>设置为“是”<strong></strong>，则只有此组中的最终用户能够注册密码重置。<br><br></li>
                </ul>
                <p>
                  <strong>密码重置门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果“限制对密码重置的访问”<strong></strong>设置为“是”<strong></strong>，则只有此组中的最终用户能够重置他们的密码。<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <p>用户可使用的身份验证方法</p>
              </td>
              <td>
                <p>确定允许用户用于重置其密码的质询。</p>
                <p>
                  
                </p>
                <p>（仅当“可进行密码重置的用户”<strong></strong>设置为“是”<strong></strong>时可见）。</p>
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
												“办公电话”对应于目录中用户对象的“办公电话”<strong></strong>属性。<br><br></li>
                  <li class="unordered">
												“移动电话”对应于目录中用户对象的“身份验证移动”<strong></strong>属性（非公开可见）或“移动电话”<strong></strong>属性（公开可见）。服务首先检查“身份验证电话”<strong></strong>数据，如果不存在，则回退到“移动电话”<strong></strong>属性。<br><br></li>
                  <li class="unordered">
												“备用电子邮件地址”对应于目录中用户对象的“身份验证电子邮件”<strong></strong>属性（非公开可见）或“备用电子邮件”<strong></strong>属性。服务首先检查“身份验证电子邮件”<strong></strong>数据，如果不存在，则回退到“备用电子邮件”<strong></strong>属性。<br><br></li>
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
												确定用户可用作给定验证步骤的质询的身份验证方法。例如，如果用户在 Azure Active Directory 的“办公电话”<strong></strong>和“身份验证电话”<strong></strong>字段中都输入了数据，则他（她）可以使用其中任一身份验证方法来恢复自己的密码。<br><br></li>
                  <li class="unordered">
                    <strong>注意：</strong>当且仅当用户在你以管理员身份启用的身份验证方法中输入了数据时，他们才能重置自己的密码。<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <p>所需身份验证方法的数量</p>
              </td>
              <td>
                <p>确定为重置他或她的密码用户必须通过的可用身份验证方法数量的下限。</p>
                <p>
                  
                </p>
                <p>（仅当“可进行密码重置的用户”<strong></strong>设置为“是”<strong></strong>时可见）。</p>
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
												影响在能够重置密码前用户必须完成的验证步骤数量。验证步骤是指用户使用一条身份验证信息（如呼叫他们的办公电话或向他们的备用电子邮件发送电子邮件）验证他们的帐户。<br><br></li>
                  <li class="unordered">
                    <strong>注意：</strong>如果某个用户的帐户中尚未定义按照你所设定的策略成功重置其密码所需的身份验证信息数量，他或她将看到一个错误页面，该页面将指示该用户请求管理员来重置其密码。 <br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <p>注册所需的问题数量</p>
              </td>
              <td>
                <p>确定用户在注册安全问题选项时必须回答的问题数量的下限。</p>
                <p>（仅当启用了“安全问题”<strong></strong>复选框时可见）。</p>
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
												我们建议将注册所需的问题数量设置得高于重置所需的问题数量，以使用户可以更灵活地重置他们的密码。这种配置也更为安全，因为我们将从用户已注册的所有问题中为用户随机选择要回答的问题。<br><br></li>
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
                <p>重置所需的问题数量 </p>
              </td>
              <td>
                <p>确定用户在重置密码时必须回答的问题数量下限。</p>
                <p>
                  
                </p>
                <p>（仅当启用了“安全问题”<strong></strong>复选框时可见）。</p>
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
                <p>安全问题</p>
              </td>
              <td>
                <p>定义用户在注册密码重置以及重置他们的密码时可以选择的安全问题。</p>
                <p>
                  
                </p>
                <p>（仅当启用了“安全问题”<strong></strong>复选框时可见）。</p>
              </td>
              <td>
                <p>
                  <strong>注意：</strong>
                </p>
                <ul>
                  <li class="unordered">
												最多可以定义 20 个问题。<br><br></li>
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
												目前还不支持为不同的区域设置定义不同的问题，但将来会实现。<br><br></li>
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
                <p>登录到访问面板时要求用户注册？</p>
                <p>
                  
                </p>
              </td>
              <td>
                <p>确定在下次登录到访问面板时是否要求用户注册联系人数据以进行密码重置。</p>
                <p>
                  
                </p>
                <p>（仅当“可进行密码重置的用户”<strong></strong>设置为“是”<strong></strong>时可见）。</p>
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
												用户还可以通过单击访问面板中“个人资料”选项卡下的“注册密码重置”<strong></strong>链接转到注册门户。<br><br></li>
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
                <p>用户必须确认其联系人数据前的天数</p>
              </td>
              <td>
                <p>启用“要求用户注册”<strong></strong>时，此设置将确定用户必须再次确认其数据前可经过的时间段。</p>
                <p>
                  
                </p>
                <p>（仅当“登录到访问面板时要求用户注册”<strong></strong>设置为“是”<strong></strong>时可见）。</p>
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
                <p>自定义“联系你的管理员”链接？</p>
              </td>
              <td>
                <p>控制发生错误时或用户对指向自定义 URL 或电子邮件地址的操作等待太长时间时是否在密码重置门户上显示“联系你的管理员”链接（显示在左边）。</p>
                <p>
                  
                </p>
                <p>（仅当“可进行密码重置的用户”<strong></strong>设置为“是”<strong></strong>时可见）。</p>
              </td>
              <td>
                <p>
                  <strong>注意：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果启用此设置，你必须通过立即填写此设置下的<strong>“自定义电子邮件地址或 URL”</strong>字段来选择自定义 URL 或电子邮件地址。<br><br></li>
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
														如果没有用户管理员，则将电子邮件发送给所有全局管理员，最多可发送给 100 名收件人。<br><br></li>
                    </ul>
                  </li>
                  <li class="unordered">
												如果设置为“是”，则此设置会将上方突出显示的链接的行为自定义为指向自定义 URL 或电子邮件地址，用户可导航到这些 URL 或电子邮件地址以获得有关密码重置的帮助。<br><br></li>
                  <li class="unordered">
												如果指定的是 URL，它将在新选项卡中打开。<br><br></li>
                  <li class="unordered">
												如果指定的是电子邮件地址，我们将创建一个指向该电子邮件地址的 mailto 链接。<br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <p>自定义电子邮件地址或 URL</p>
              </td>
              <td>
                <p>控制“联系你的管理员”<strong></strong>链接指向的电子邮件地址或 URL。</p>
                <p>
                  
                </p>
                <p>（仅当“自定义‘联系你的管理员’链接”<strong></strong>设置为“是”<strong></strong>时可见）。</p>
              </td>
              <td>
                <p>
                  <strong>注意：</strong>
                </p>
                <ul>
                  <li class="unordered">
												必须是有效的 Intranet 或 Extranet URL 或电子邮件地址。<br><br></li>
                </ul>
                <p>
                  <strong>密码重置门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												更改“联系你的管理员”<strong></strong>链接所指向的位置。<br><br></li>
                  <li class="unordered">
												如果你提供的是电子邮件地址，此链接将成为指向该电子邮件地址的“mailto”链接。<br><br></li>
                  <li class="unordered">
												如果你提供的是 URL，此链接将成为指向该 URL 的标准 href 并将在一个新选项卡中打开。 <br><br></li>
                </ul>
              </td>
            </tr>
            <tr>
              <td>
                <p>将密码写回到本地目录</p>
              </td>
              <td>
                <p>控制是否为此目录启用密码写回功能，如果启用写回，该项将指示本地写回服务的状态。</p>
                <p>
                  
                </p>
                <p>如果你由于中断原因希望临时禁用该服务，这会很有用。</p>
              </td>
              <td>
                <p>
                  
                </p>
                <p>
                  <strong>注意：</strong>
                </p>
                <ul>
                  <li class="unordered">
												仅当你已通过下载 Azure AD Connect 最新版本并启用“可选功能”<strong></strong>选择屏幕下方的“密码写回”<strong></strong>选项安装了密码写回时，此控件才会出现。<br><br></li>
                  <li class="unordered">
												如果你启用了密码写回功能，但认为存在服务配置问题，可以进入此选项卡并查看“密码写回服务状态”<strong></strong>标签，看看是否有问题。<br><br></li>
                  <li class="unordered">
												可能显示的状态如下：<br><br><ul><li class="unordered"><strong>已配置</strong> – 所有内容均按预期工作<br><br></li><li class="unordered"><strong>未配置</strong> – 已安装写回功能，但我们不能访问此服务，请检查并确保你未阻止到 443 的出站连接；如果问题仍然存在，请尝试重新安装该服务。<br><br></li></ul></li>
                </ul>
                <p>
                  <strong>注册门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果写回功能已部署并配置，但此开关设置为<strong>“否”</strong>，则将禁用写回，因此联合用户和已进行密码哈希同步的用户将不能注册密码重置。<br><br></li>
                  <li class="unordered">
												如果此开关设置为“是”<strong></strong>，则将启用写回，因此联合用户和已进行密码哈希同步的用户将能够重置他们的密码。<br><br></li>
                </ul>
                <p>
                  <strong>密码重置门户：</strong>
                </p>
                <ul>
                  <li class="unordered">
												如果写回功能已部署并配置，但此开关已设置为“否”<strong></strong>，则将禁用写回，因此联合用户和已进行密码哈希同步的用户将不能重置他们的密码。<br><br></li>
                  <li class="unordered">
												如果此开关设置为“是”<strong></strong>，则将启用写回，因此联合用户和已进行密码哈希同步的用户将能够重置他们的密码。<br><br></li>
                </ul>
              </td>
            </tr>
          </tbody></table>

## <a name="password-management-notifications"></a>密码管理通知
下表说明了每个控件对接收密码重置通知的用户和管理员的体验有何影响。你可以在 [Azure 管理门户](https://manage.windowsazure.cn)内你目录的“配置”选项卡的“通知”部分配置这些选项。

<table>
            <tbody><tr>
              <td>
                <p>
                  <strong>策略控件</strong>
                </p>
              </td>
              <td>
                <p>
                  <strong>描述</strong>
                </p>
              </td>
              <td>
                <p>
                  <strong>影响？</strong>
                </p>
              </td>
            </tr>
            <tr>
              <td>
                <p>当其他管理员重置自己的密码时通知管理员</p>
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
                <p>当用户和管理员自己的密码已被重置时通知他们</p>
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

**其他资源**


* [什么是密码管理](/documentation/articles/active-directory-passwords)
* [密码管理的工作原理](/documentation/articles/active-directory-passwords-how-it-works)
* [密码管理入门](/documentation/articles/active-directory-passwords-getting-started)
* [密码管理最佳实践](/documentation/articles/active-directory-passwords-best-practices)
* [密码管理常见问题](/documentation/articles/active-directory-passwords-faq)
* [密码管理疑难解答](/documentation/articles/active-directory-passwords-troubleshoot)
* [了解详细信息](/documentation/articles/active-directory-passwords-learn-more)
* [MSDN 上的密码管理](https://msdn.microsoft.com/zh-cn/library/azure/dn510386.aspx) 

<!---HONumber=67-->