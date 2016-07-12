<properties 
	pageTitle="故障排除：Azure AD 密码管理 | Microsoft Azure" 
	description="针对 Azure AD 密码管理的常见故障排除步骤，包括重置、更改、写回、注册，以及寻求帮助时要提供的信息。" 
	services="active-directory" 
	documentationCenter="" 
	authors="asteen" 
	manager="kbrint" 
	editor="billmath"/>

<tags 
	ms.service="active-directory"  
	ms.date="02/16/2016" 
	wacn.date="06/27/2016"/>

# 如何排查密码管理问题
如果你有密码管理方面的问题，我们随时可提供帮助。你可能会遇到的大多数问题都可以通过以下简单故障排除步骤来解决，请阅读以下内容来排查你的部署问题：

* [**你需要帮助时应提供的信息**](#information-to-include-when-you-need-help)
* [**Azure 经典管理门户中的密码管理配置问题**](#troubleshoot-password-reset-configuration-in-the-azure-management-portal)
* [**密码重置注册门户的问题**](#troubleshoot-the-password-reset-registration-portal)
* [**密码重置门户的问题**](#troubleshoot-the-password-reset-portal)
* [**密码写回的问题**](#troubleshoot-password-writeback)
  - [密码写回事件日志错误代码](#password-writeback-event-log-error-codes)
  - [密码写回连接的问题](#troubleshoot-password-writeback-connectivity)

如果你已尝试过以下故障排除步骤，但仍遇到问题，请将问题发布到 [Azure AD 论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=windowsazurezhchs)，或者与支持人员联系，我们将尽快解答你的问题。

## <a name="information-to-include-when-you-need-help"></a>你需要帮助时应包含的信息

如果根据以下指导仍然无法解决问题，请联系我们的支持工程师。与他们联系时，建议你包含以下信息：

 - **错误的一般描述** – 用户实际看到的错误消息。 如果没有任何错误消息，请详细描述你所发现的意外行为。
 - **页面** – 你看到错误时所在的页面（包括 URL）。
 - **日期/时间/时区** – 你看到错误时的精确日期和时间（包括时区）。
 - **支持代码** – 用户看到错误时所生成的支持代码（若要找到支持代码，请再现错误，然后单击屏幕底部的“支持代码”链接，将生成的 GUID 发送给支持工程师）。 
   - 如果所在页面的底部没有支持代码，请按 F12，搜索 SID 和 CID，然后将这两个结果发送给支持工程师。

    ![][001]

 - **用户 ID** – 看到错误的用户的 ID（例如 user@contoso.com)?）
 - **有关用户的信息** – 用户是否已联合、密码哈希是否已同步、是否只在云中？ 用户是否已获 AAD Premium 或 AAD Basic 授权？
 - **应用程序事件日志** – 如果你使用密码写回，而且错误位于你的本地基础结构中，请将 Azure AD Connect 服务器中的应用程序事件日志副本进行压缩，然后连同请求一起发送。

包含这些信息将有助于我们尽快为你解决问题。


##  <a name="troubleshoot-password-reset-configuration-in-the-azure-management-portal"></a>在 Azure 经典管理门户中排查密码重置配置问题
如果你在配置密码重置时遇到错误，可以遵循以下故障排除步骤来解决错误：

<table>
          <tbody><tr>
            <td>
              <p>
                <strong>错误案例</strong>
              </p>
            </td>
            <td>
              <p>
                <strong>用户看到的错误</strong>
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
              <p>我在 Azure 经典管理门户的“配置”选项卡下未看到“用户密码重置策略”部分<strong></strong><strong></strong>。</p>
            </td>
            <td>
              <p>Azure 经典管理门户的“配置”选项卡中未显示“用户密码重置策略”部分<strong></strong><strong></strong>。</p>
            </td>
            <td>
              <p>如果未向执行此操作的管理员分配 AAD Premium 或 AAD Basic 许可证，则会发生这种情况。</p>
              <p>若要纠正此问题，请导航到“许可证”选项卡，将 AAD Premium 或 AAD Basic 许可证分配给遇到问题的管理员帐户，然后重试<strong></strong>。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>我在文档中所述的“用户密码重置策略”部分下未看到任何配置选项<strong></strong>。</p>
            </td>
            <td>
              <p>“用户密码重置策略”部分可见，但它的下方只显示了“可进行密码重置的用户”标志<strong></strong><strong></strong>。</p>
            </td>
            <td>
              <p>将“可进行密码重置的用户”标志切换到“是”时，即可显示用户界面的其余部分<strong></strong><strong></strong>。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>我看不到某个特定的配置选项。</p>
            </td>
            <td>
              <p>例如，当滚动浏览“用户密码重置策略”部分时，我看不到“用户必须确认其联系人数据前的天数”选项（或属于相同问题的其他示例）<strong></strong><strong></strong>。</p>
            </td>
            <td>
              <p>UI 的许多元素都是隐藏的，直到需要它们时才会显示。如果你希望看到它们，请尝试启用页面上的所有选项。</p>
              <p>有关可供你使用的所有控件的更多信息，请参阅<a href="/documentation/articles/active-directory-passwords-customize/#password-management-behavior">自定义用户密码策重置略</a>。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>我看不到“将密码写回到本地”配置选项<strong></strong></p>
            </td>
            <td>
              <p>“将密码写回到本地”选项未显示在 Azure 经典管理门户中的“配置”选项卡下方<strong></strong><strong></strong></p>
            </td>
            <td>
              <p>仅当你已下载 Azure AD Connect 并配置了密码写回时，此选项才可见。完成此操作后，此选项才会出现，并允许你启用或禁用从云写回功能。</p>
              <p>有关如何执行此操作的详细信息，请参阅<a href="/documentation/articles/active-directory-passwords-getting-started/#enable-users-to-reset-or-change-their-ad-passwords">如何启用/禁用密码写回</a>。</p>
            </td>
          </tr>
        </tbody></table>

## <a name="troubleshoot-password-writeback"></a>排查密码重置注册门户问题
如果你在针对密码重置注册用户时遇到错误，也许能够通过执行以下故障排除步骤来解决错误：

<table>
          <tbody><tr>
            <td>
              <p>
                <strong>错误案例</strong>
              </p>
            </td>
            <td>
              <p>
                <strong>用户看到的错误</strong>
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
              <p>未针对密码重置启用目录</p>
            </td>
            <td>
              <p>你的管理员没有允许你使用此功能。</p>
            </td>
            <td>
              <p>将“可进行密码重置的用户”标志切换为“是”，并点击 Azure 经典管理门户目录配置选项卡中的“保存”<strong></strong><strong></strong><strong></strong>。你必须向执行此操作的管理员分配 Azure AD Premium 或 Azure AD Basic 许可证。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>用户未分配有 AAD Premium 或 AAD Basic 许可证</p>
            </td>
            <td>
              <p>你的管理员没有允许你使用此功能。</p>
            </td>
            <td>
              <p>在 Azure 经典管理门户中的“许可证”选项卡下方向此用户分配 Azure AD Premium 或 Azure AD Basic 许可证<strong></strong>。你必须向执行此操作的管理员分配 Azure AD Premium 或 Azure AD Basic 许可证。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>处理请求时出错</p>
            </td>
            <td>
              <p>用户看到了内容如下的错误：</p>
              <p>
                
              </p>
              <p>处理请求时出错 </p>
              <p>尝试重置密码时。</p>
            </td>
            <td>
              <p>许多问题都可能会导致此错误，但此错误通常是由服务中断或者无法解决的配置问题导致的。</p>
              <p>如果你看到了此错误并且它影响你的业务，请与支持人员联系，我们会尽快帮你解决。若要了解你应当向支持工程师提供哪些信息来帮助快速解决问题，请参阅<a href="#information-to-include-when-you-need-help">你需要帮助时应提供的信息</a>。</p>
            </td>
          </tr>
        </tbody></table>

## <a name="troubleshoot-the-password-reset-portal"></a>排查密码重置门户问题
如果在为用户重置密码时遇到错误，可通过遵循以下故障排除步骤来解决：

<table>
          <tbody><tr>
            <td>
              <p>
                <strong>错误案例</strong>
              </p>
            </td>
            <td>
              <p>
                <strong>用户看到的错误</strong>
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
              <p>未针对密码重置启用目录</p>
            </td>
            <td>
              <p>未针对密码重置启用帐户</p>
              <p>很抱歉，你的管理员尚未将你的帐户设置为可使用此服务。</p>
              <p>
                
              </p>
              <p>如果你愿意，我们可以联系你所在组织的管理员为你重置密码。</p>
            </td>
            <td>
              <p>将“可进行密码重置的用户”标志切换为“是”，并点击 Azure 经典管理门户目录配置选项卡中的“保存”<strong></strong><strong></strong><strong></strong>。你必须向执行此操作的管理员分配 Azure AD Premium 或 Azure AD Basic 许可证。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>用户未分配有 AAD Premium 或 AAD Basic 许可证</p>
            </td>
            <td>
              <p>虽然我们无法自动重置非管理员帐户密码，但我们可以联系你所在组织的管理员为你执行这一操作。</p>
            </td>
            <td>
              <p>在 Azure 经典管理门户中的“许可证”选项卡下方向此用户分配 Azure AD Premium 或 Azure AD Basic 许可证<strong></strong>。你必须向执行此操作的管理员分配 Azure AD Premium 或 Azure AD Basic 许可证。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>已针对密码重置启用了目录，但用户的身份验证信息缺失或格式错误</p>
            </td>
            <td>
              <p>未针对密码重置启用帐户</p>
              <p>很抱歉，你的管理员尚未将你的帐户设置为可使用此服务。</p>
              <p>
                
              </p>
              <p>如果你愿意，我们可以联系你所在组织的管理员为你重置密码。</p>
            </td>
            <td>
              <p>确保用户在目录下的文件中有格式正确的联系人数据，然后继续。有关如何在目录中配置身份验证信息以避免用户遇到此错误的信息，请参阅<a href="/documentation/articles/active-directory-passwords-learn-more/#what-data-is-used-by-password-reset">密码重置使用哪些数据</a>。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>已针对密码重置启用目录，但用户在文件中只有一个联系人数据，而策略要求进行两个验证步骤</p>
            </td>
            <td>
              <p>未针对密码重置启用帐户</p>
              <p>很抱歉，你的管理员尚未将你的帐户设置为可使用此服务。</p>
              <p>
                
              </p>
              <p>如果你愿意，我们可以联系你所在组织的管理员为你重置密码。</p>
            </td>
            <td>
              <p>确保用户至少有两个正确配置的联系方式（例如，移动电话和办公室电话），然后继续。有关如何在目录中配置身份验证信息以避免用户遇到此错误的信息，请参阅<a href="/documentation/articles/active-directory-passwords-learn-more/#what-data-is-used-by-password-reset">密码重置使用哪些数据</a>。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>已针对密码重置启用目录并正确配置用户，但无法联系到用户 </p>
            </td>
            <td>
              <p>哎呀! 我们联系你时遇到了意外错误。</p>
            </td>
            <td>
              <p>这可能是因临时服务错误或错误配置的联系人数据导致的，我们无法正确检测到这些错误。如果用户等待 10 秒，将出现“重试”和“联系你的管理员”链接。单击“重试”将重新进行呼叫，而单击“联系你的管理员”将向用户、密码管理员或全球管理员（按此优先顺序）发送表格邮件，请求为该用户帐户执行密码重置。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>用户一直收不到密码重置短信或电话呼叫</p>
            </td>
            <td>
              <p>用户单击了“向我发送短信”或“呼叫我”，但一直收不到任何信息。</p>
            </td>
            <td>
              <p>这可能是由于目录中的电话号码格式不正确导致的。请确保电话号码的格式为“+ccc xxxyyyzzzzXeeee”。若要了解有关设置电话号码格式以用于重置的详细信息，请参阅<a href="/documentation/articles/active-directory-passwords-learn-more/#what-data-is-used-by-password-reset">密码重置使用哪些数据</a>。</p>
              <p>如果你需要一个分机才能路由到相关用户，请注意密码重置不支持分机，即使你在目录中指定了分机（它们将在呼叫前被剥离）。请尝试使用不带分机的号码，或者在你的 PBX 中将分机集成到电话号码中。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>用户一直收不到密码重置电子邮件</p>
            </td>
            <td>
              <p>用户单击了“向我发送电子邮件”，但一直收不到任何信息。</p>
            </td>
            <td>
              <p>此问题的最常见原因是消息被垃圾邮件筛选器拒绝。请检查你的垃圾邮件或已删除邮件文件夹中是否有电子邮件。</p>
              <p>另请确保你在正确的电子邮箱中检查是否有该邮件，许多人都有类似的电子邮件地址，结果在错误的收件箱中检查是否有该邮件。如果这些选项都不起作用，还可能是目录中的电子邮件地址不正确，请进行检查以确保电子邮件地址正确，然后重试。若要了解有关设置电子邮件地址格式以用于重置的详细信息，请参阅<a href="/documentation/articles/active-directory-passwords-learn-more/#what-data-is-used-by-password-reset">密码重置使用哪些数据</a>。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>我已经设置了密码重置策略，但当管理员帐户使用密码重置时，该策略未应用</p>
            </td>
            <td>
              <p>无论在“配置”选项卡的“用户密码重置策略”下方设置了何种策略，重置密码的管理员帐户所看到的为密码重置、电子邮件和移动电话启用的选项都是相同的<strong></strong><strong></strong>。</p>
            </td>
            <td>
              <p>在“配置”选项卡的“用户密码重置策略”下方配置的选项仅适用于你组织的最终用户<strong></strong><strong></strong>。</p>
              <p>为确保最高级别的安全性，管理员密码重置策略由 Microsoft 进行管理和控制</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>一天中阻止用户尝试密码重置的次数太多</p>
            </td>
            <td>
              <p>用户看到了内容如下的错误：</p>
              <p>
                
              </p>
              <p>请使用其他选项。</p>
              <p>你在过去 1 小时内尝试验证你的帐户的次数太多。出于安全原因，你将需要等待 24 小时，然后才能再次尝试。</p>
              <p>如果你愿意，我们可以联系你所在组织的管理员为你重置密码。</p>
            </td>
            <td>
              <p>我们实施了一种自动扼制机制来阻止用户在短时间内过多地尝试重置其密码。此错误发生在以下情况下：</p>
              <ol class="ordered">
                <li>
										用户在一小时内 5 次尝试验证某个电话号码。<br\><br\></li>
                <li>
										用户在一小时内 5 次尝试使用安全问题入口。<br\><br\></li>
                <li>
										用户在一小时内 5 次尝试为同一用户帐户重置密码。<br\><br\></li>
              </ol>
              <p>若要解决此问题，请指示用户自最后一次尝试后等待 24 小时，然后用户将能够重置其密码。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>用户在验证其电话号码时看到了一个错误</p>
            </td>
            <td>
              <p>当尝试验证要用作身份验证方法的电话号码时，用户看到了内容如下的错误：</p>
              <p>
                
              </p>
              <p>指定了错误的电话号码。</p>
            </td>
            <td>
              <p>当输入的电话号码与文件上的电话号码不匹配时，将发生此错误。</p>
              <p>当尝试使用基于电话的方法进行密码重置时，请确保用户输入了完整的电话号码（包括区域和国家/地区代码）。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>处理请求时出错</p>
            </td>
            <td>
              <p>用户看到了内容如下的错误：</p>
              <p>
                
              </p>
              <p>处理请求时出错 </p>
              <p>尝试重置密码时。</p>
            </td>
            <td>
              <p>许多问题都可能会导致此错误，但此错误通常是由服务中断或者无法解决的配置问题导致的。</p>
              <p>如果你看到了此错误并且它影响你的业务，请与支持人员联系，我们会尽快帮你解决。若要了解你应当向支持工程师提供哪些信息来帮助快速解决问题，请参阅<a href="#information-to-include-when-you-need-help">你需要帮助时应提供的信息</a>。</p>
            </td>
          </tr>
        </tbody></table>

## <a name="troubleshoot-password-writeback"></a>排查密码写回问题
如果在启用、禁用或使用密码写回时遇到错误，可以通过执行以下故障排除步骤来解决错误：

<table>
          <tbody><tr>
            <td>
              <p>
                <strong>错误案例</strong>
              </p>
            </td>
            <td>
              <p>
                <strong>用户看到的错误</strong>
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
              <p>常规登记和启动失败</p>
            </td>
            <td>
              <p>密码重置服务在本地未启动，并且在 Azure AD Connect 计算机的应用程序事件日志中出现错误 6800。</p>
              <p>
                
              </p>
              <p>在登记后，联合的或密码哈希同步的用户无法重置其密码。</p>
            </td>
            <td>
              <p>当启用了密码写回时，同步引擎将调用写回库通过与云登记服务进行通信来执行配置（登记）。在登记期间或者为密码写回启动 WCF 终结点时遇到任何错误都将导致在 Azure AD Connect 计算机的事件日志中生成错误。</p>
              <p>在重新启动 ADSync 服务期间，如果配置了写回，则 WCF 终结点将启动。但是，如果终结点启动失败，我们将只是记录事件 6800 并允许同步服务启动。存在此事件意味着密码写回终结点未启动。此事件 (6800) 的事件日志详细信息以及 PasswordResetService 组件生成的事件日志条目将指明终结点无法启动的原因。请查看这些事件日志错误，如果密码写回仍不能正常工作，请尝试重新启动 Azure AD Connect。如果问题仍然存在，请尝试禁用并重新启用密码写回。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>安装 Azure AD Connect 期间配置写回时出错。</p>
            </td>
            <td>
              <p>在 Azure AD Connect 安装过程的最后一步，你看到了一个错误，它指出无法配置密码写回。</p>
              <p>
                
              </p>
              <p>Azure AD Connect 应用程序事件日志包含错误 32009，其文本为“获取身份验证令牌时出错”。</p>
            </td>
            <td>
              <p>在以下两种情况下会发生此错误：</p>
              <ul>
                <li class="unordered">
										你为在 Azure AD Connect 安装过程开始时指定的全局管理员帐户指定了错误的密码。<br\><br\></li>
              </ul>
              <ul>
                <li class="unordered">
										你试图将联合用户用于在 Azure AD Connect 安装过程开始时指定的全局管理员帐户。<br\><br\></li>
              </ul>
              <p>若要解决此错误，请确保你没有将联合帐户用于在 Azure AD Connect 安装过程开始时指定的全局管理员帐户，并且指定的密码正确无误。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>安装 Azure AD Connect 期间配置写回时出错。</p>
            </td>
            <td>
              <p>Azure AD Connect 计算机事件日志包含由 PasswordResetService 引发的错误 32002。</p>
              <p>
                
              </p>
              <p>错误如下：“连接到服务总线时出错，令牌提供程序无法提供安全令牌...”</p>
              <p>
                
              </p>
            </td>
            <td>
              <p>发生此错误的根本原因是在你的本地环境中运行的密码重置服务无法连接到云中的服务总线终结点。此错误通常是由于防火墙规则阻止了到特定端口或 web 地址的出站连接导致的。</p>
              <p>
                
              </p>
              <p>请确保你的防火墙允许以下各项的出站连接：</p>
              <ul>
                <li class="unordered">
										所有基于 TCP 443 (HTTPS) 的通信<br\><br\></li>
              </ul>
              <ul>
                <li class="unordered">
										出站连接 <br\><br\></li>
              </ul>
              <p>
                
              </p>
              <p>在更新这些规则后，重新启动 AAD Sync 计算机，密码写回应当会再次开始工作。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>本地的密码写回终结点不可访问</p>
            </td>
            <td>
              <p>在工作一段时间后，联合用户或密码哈希同步的用户无法重置其密码。</p>
            </td>
            <td>
              <p>在某些极少见的情况下，当 Azure AD Connect 已重新启动时，密码写回服务可能无法重新启动。在这些情况下，首先，请检查是否已在本地启用了密码写回。这可以通过使用 Azure AD Connect 向导或 powershell 来完成（请参阅上面的“操作说明”部分）。如果此功能已启用，请尝试再次通过 UI 或 PowerShell 启用或禁用此功能。有关如何执行此操作的详细信息，请参阅<a href="/documentation/articles/active-directory-passwords-getting-started/#enable-users-to-reset-or-change-their-ad-passwords">如何启用/禁用密码写回</a>中的“步骤 2：在 Directory Sync 计算机上启用密码写回并配置防火墙规则”。</p>
              <p>
                
              </p>
              <p>如果这不起作用，请尝试完全卸载并重新安装 Azure AD Connect。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>权限错误</p>
            </td>
            <td>
              <p>尝试重置其密码的联合用户或密码哈希同步的用户在提交密码后看到了一个错误，该错误指示存在服务问题。</p>
              <p>
                
              </p>
              <p>除此之外，在密码重置期间，你可能会在你的本地事件日志中看到有关管理代理被拒绝访问的消息。</p>
            </td>
            <td>
              <p>如果在你的事件日志中看到这些错误，请确认 AD MA 帐户（在配置时在向导中指定的帐户）具有进行密码写回所需的权限。</p>
              <p>
                
              </p>
              <p>请注意，在授予此权限后，权限可能需要最多 1 小时来通过 DC 上的 sdprop 后台任务进行渗透。</p>
              <p>要使密码重置工作，需要将此权限标记在为其重置密码的用户对象的安全描述符上。在此权限显示在用户对象上之前，密码重置将一直因为访问被拒绝而失败。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>从 Azure AD Sync 配置向导配置密码写回时出错 </p>
            </td>
            <td>
              <p>启用/禁用密码写回时在向导中出现“找不到 MA”错误</p>
            </td>
            <td>
              <p>这是 Azure AD Connect 的已发行版本中的一个已知 bug，在以下情况下会出现：</p>
              <ol class="ordered">
                <li>
										你使用凭据为租户 abc.com（已验证的域）配置 Azure AD Connect。这将导致创建名为 "abc.com - AAD" 的 AAD 连接器。<br\><br\></li>
                <li>
										然后，你将连接器的 AAD 凭据（使用旧的同步 UI）更改为 （注意，它是同一租户但不同的域名）。<br\><br\></li>
                <li>
										现在，你尝试启用/禁用密码写回。向导将使用凭据将连接器的名称构造为“abc.onmicrosoft.com - AAD”并将其传递给密码写回 cmdlet。此操作将会失败，因为没有使用此名称创建的连接器。<br\><br\></li>
              </ol>
              <p>此问题在我们的最新内部版本中已修复。如果你具有较早的内部版本，一个解决方法是使用 powershell cmdlet 来启用/禁用该功能。有关如何执行此操作的详细信息，请参阅<a href="/documentation/articles/active-directory-passwords-getting-started/#enable-users-to-reset-or-change-their-ad-passwords">如何启用/禁用密码写回</a>中的“步骤 2：在 Directory Sync 计算机上启用密码写回并配置防火墙规则”。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>无法为特殊组（例如，“域管理员”/“企业管理员”等等）中的用户重置密码。</p>
            </td>
            <td>
              <p>属于受保护的组并且尝试重置其密码的联合用户或密码哈希同步的用户在提交密码后看到了一个错误，该错误指示存在服务问题。</p>
            </td>
            <td>
              <p>Active Directory 中的特权用户是使用 AdminSDHolder 进行保护的。有关详细信息，请参阅 <a href="https://technet.microsoft.com/magazine/2009.09.sdadminholder.aspx">http://technet.microsoft.com/magazine/2009.09.sdadminholder.aspx</a>。</p>
              <p>
                
              </p>
              <p>这意味着将定期检查这些对象上的安全描述符是否与 AdminSDHolder 中指定的安全描述符匹配，如果它们不同，将对前者进行重置。因此，执行密码写回所需的其他权限不会渗透到这类用户。这可能会导致不会为这类用户执行密码写回。因此，我们不支持为这些组内的用户管理密码，因为这会破坏 AD 安全模型。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>重置操作因为找不到对象而失败</p>
            </td>
            <td>
              <p>尝试重置其密码的联合用户或密码哈希同步的用户在提交密码后看到了一个错误，该错误指示存在服务问题。</p>
              <p>
                
              </p>
              <p>除此之外，在密码重置期间，你可能会在 Azure AD Connect 服务的事件日志中看到一个表示“找不到对象”错误的错误。</p>
            </td>
            <td>
              <p>此错误通常表示同步引擎无法找到 AAD 连接器空间中的用户对象或者无法找到链接的 MV 或 AD 连接器空间对象。</p>
              <p>
                
              </p>
              <p>若要解决此问题，请确保用户确实已通过当前的 Azure AD Connect 实例从本地同步到了 AAD，并检查连接器空间和 MV 中的对象的状态。通过“Microsoft.InfromADUserAccountEnabled.xxx”规则确认 AD CS 对象是 MV 对象的连接器。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>重置操作因为发生找到多个匹配项错误而失败</p>
            </td>
            <td>
              <p>尝试重置其密码的联合用户或密码哈希同步的用户在提交密码后看到了一个错误，该错误指示存在服务问题。</p>
              <p>
                
              </p>
              <p>除此之外，在密码重置期间，你可能会在 Azure AD Connect 服务的事件日志中看到一个表示“找到多个匹配项”错误的错误。</p>
            </td>
            <td>
              <p>这表示同步引擎通过“Microsoft.InfromADUserAccountEnabled.xxx”检测到 MV 对象连接到了多个 AD CS 对象。这意味着用户在多个林具有已启用的帐户。</p>
              <p>
                
              </p>
              <p>当前，密码写回不支持此方案。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>密码操作因为发生配置错误而失败。</p>
            </td>
            <td>
              <p>密码操作因为发生配置错误而失败。应用程序事件日志包含 AAD 同步错误 6329，文本为：0x8023061f (操作失败，因为未在此管理代理上启用密码同步。)</p>
            </td>
            <td>
              <p>如果在启用密码写回功能之后将 AAD Sync 配置更改为添加新的 AD 林（或者更改为删除现有林<strong>之后</strong>再重新添加），则会发生这种情况。在此类新添加的林中，用户的密码操作将失败。若要解决此问题，请在林配置更改完成后，先禁用密码写回功能，然后再重新启用它。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>由用户重置的密码写回功能可以正常使用，但由用户更改的或由管理员为用户重置的密码写回功能无法正常使用。</p>
            </td>
            <td>
              <p>尝试通过 Azure 经典管理门户代表用户重置密码时，你将看到一条消息：“在本地环境中运行的密码重置服务不支持管理员重置用户密码。请升级到最新版本的 AAD Sync 以解决此问题。”</p>
            </td>
            <td>
              <p>当同步引擎的版本不支持所使用的特定密码写回操作时，将会发生这种情况。1.0.0419.0911 以上的 AAD Sync 版本支持所有密码管理操作，包括密码重置写回、密码更改写回，以及管理员通过 Azure 经典管理门户启动的密码重置写回。&#160; 1.0.6862 以上的 DirSync 版本仅支持密码重置写回。若要解决此问题，强烈建议你安装最新版本的 AAD Sync 或 Azure Active Directory Connect（有关详细信息，请参阅<a href="/documentation/articles/active-directory-aadconnect/#download-azure-ad-connect">目录集成工具</a>），这样既能解决此问题，又可以在你的组织中充分利用密码写回功能。</p>
            </td>
          </tr>
        </tbody></table>


## <a name="password-writeback-event-log-error-codes"></a>密码写回事件日志错误代码
在排查密码写回问题时，最佳做法是检查 Azure AD Connect 计算机上的该应用程序事件日志。此事件日志将包含来自与密码写回相关的两个源的事件。PasswordResetService 源将描述与密码写回操作相关的操作和问题。ADSync 源将描述与在你的 AD 环境中设置密码相关的操作和问题。

<table>
          <tbody><tr>
            <td>
              <p>
                <strong>代码</strong>
              </p>
            </td>
            <td>
              <p>
                <strong>名称/消息</strong>
              </p>
            </td>
            <td>
              <p>
                <strong>源</strong>
              </p>
            </td>
            <td>
              <p>
                <strong>说明</strong>
              </p>
            </td>
          </tr>
          <tr>
            <td>
              <p>6329</p>
            </td>
            <td>
              <p>BAIL: MMS(4924) 0x80230619 -“某个限制阻止将密码更改为当前指定的密码。”</p>
            </td>
            <td>
              <p>ADSync</p>
            </td>
            <td>
              <p>当密码写回服务尝试在你的本地目录中设置的密码不符合域在密码期限、历史记录、复杂度或筛选方面的要求时，将发生此事件。</p>
              <ul>
                <li class="unordered">
										如果你有最短密码期限，并且最近在此时间窗口内已更改过密码，你将不能再次更改密码，直到它达到你的域中指定的期限。对于测试目的，最短期限应设置为 0。<br\><br\></li>
              </ul>
              <ul>
                <li class="unordered">
										如果你启用了密码历史记录要求，则必须选择在最近 N 次未使用过的密码，其中 N 是密码历史记录设置。如果你选择了在最近 N 次中使用过的密码，则在此情况下将会失败。对于测试目的，历史记录应设置为 0。<br\><br\></li>
              </ul>
              <ul>
                <li class="unordered">
										如果你有密码复杂性要求，则当用户尝试更改或重置密码时会强制实施所有这些要求。<br\><br\></li>
              </ul>
              <ul>
                <li class="unordered">
										如果你启用了密码筛选器，并且用户选择了不满足筛选条件的密码，则重置或更改操作将失败。<br\><br\></li>
              </ul>
            </td>
          </tr>
          <tr>
            <td>
              <p>HR 8023042</p>
            </td>
            <td>
              <p>同步引擎返回了错误：hr = 80230402，消息 = 由于存在使用相同定位点的重复条目，尝试获取对象失败</p>
            </td>
            <td>
              <p>ADSync</p>
            </td>
            <td>
              <p>在多个域中启用同一用户 ID 时会发生此事件。例如，如果你正在同步帐户/资源林，并且每个林中存在并启用了同一个用户 ID，则可能会发生此错误。 </p>
              <p>如果你使用了不唯一的定位点属性（如别名或 UPN），并且两个用户共享了这同一个定位点属性，则也可能发生此错误。</p>
              <p>若要解决此问题，请确保你的域中没有任何重复的用户，并且每个用户使用唯一的定位点属性。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>31001</p>
            </td>
            <td>
              <p>PasswordResetStart </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示本地服务检测到从云中发起了联合用户或密码哈希同步的用户的密码重置请求。此事件是每个密码重置写回操作中的第一个事件。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>31002</p>
            </td>
            <td>
              <p>PasswordResetSuccess </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示用户在密码重置操作过程中选择了一个新密码，我们确定该密码满足公司密码要求，并且该密码已成功写回到本地 AD 环境。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>31003</p>
            </td>
            <td>
              <p>PasswordResetFail </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示用户选择了一个密码，并且该密码已成功到达本地环境，但当我们尝试在本地 AD 环境中设置该密码时出现错误。这可能有以下几个原因：</p>
              <ul>
                <li class="unordered">
										用户的密码不满足域在期限、历史记录、复杂性或筛选器方面的要求。若要解决此问题，请尝试使用一个全新的密码。<br\><br\></li>
              </ul>
              <ul>
                <li class="unordered">
										MA 服务帐户没有合适的权限在相关的用户帐户上设置新密码。<br\><br\></li>
              </ul>
              <ul>
                <li class="unordered">
										该用户的帐户位于不允许密码设置操作的受保护组中（例如，域管理员或企业管理员）。<br\><br\></li>
              </ul>
              <p>若要了解会导致此错误的其他情况的详细信息，请参阅<a href="#troubleshoot-password-writeback">排查密码写回问题</a>。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>31004</p>
            </td>
            <td>
              <p>OnboardingEventStart </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>如果你为 Azure AD Sync 启用了密码写回，则会发生此事件，它表示我们已开始将你的组织登记到密码写回 web 服务。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>31005</p>
            </td>
            <td>
              <p>OnboardingEventSuccess </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示登记过程成功，并且密码写回功能已就绪可用。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>31006</p>
            </td>
            <td>
              <p>ChangePasswordStart </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示本地服务检测到从云中发起了联合用户或密码哈希同步的用户的密码更改请求。此事件是每个密码更改写回操作中的第一个事件。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>31007</p>
            </td>
            <td>
              <p>ChangePasswordSuccess </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示用户在密码更改操作过程中选择了一个新密码，我们确定该密码满足公司密码要求，并且该密码已成功写回到本地 AD 环境。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>31008</p>
            </td>
            <td>
              <p>ChangePasswordFail </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示用户选择了一个密码，并且该密码已成功到达本地环境，但当我们尝试在本地 AD 环境中设置该密码时出现错误。这可能有以下几个原因：</p>
              <ul>
                <li class="unordered">
										用户的密码不满足域在期限、历史记录、复杂性或筛选器方面的要求。若要解决此问题，请尝试使用一个全新的密码。<br\><br\></li>
              </ul>
              <ul>
                <li class="unordered">
										MA 服务帐户没有合适的权限在相关的用户帐户上设置新密码。<br\><br\></li>
              </ul>
              <ul>
                <li class="unordered">
										该用户的帐户位于不允许密码设置操作的受保护组中（例如，域管理员或企业管理员）。<br\><br\></li>
              </ul>
              <p>若要了解会导致此错误的其他情况的详细信息，请参阅<a href="#troubleshoot-password-writeback">排查密码写回问题</a>。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>31009</p>
            </td>
            <td>
              <p>ResetUserPasswordByAdminStart </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>本地服务检测到管理员代表某个用户发起了联合用户或密码哈希同步的用户的密码重置请求。此事件是每个由管理员启动的密码重置写回操作中的第一个事件。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>31010</p>
            </td>
            <td>
              <p>ResetUserPasswordByAdminSuccess </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>管理员在管理员启动的密码重置操作过程中选择了一个新密码，我们确定该密码满足公司密码要求，并且该密码已成功写回到本地 AD 环境。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>31011</p>
            </td>
            <td>
              <p>ResetUserPasswordByAdminFail </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>管理员代表用户选择了一个密码，并且该密码已成功到达本地环境，但当我们尝试在本地 AD 环境中设置该密码时出现错误。这可能有以下几个原因：</p>
              <ul>
                <li class="unordered">
										用户的密码不满足域在期限、历史记录、复杂性或筛选器方面的要求。若要解决此问题，请尝试使用一个全新的密码。<br\><br\></li>
              </ul>
              <ul>
                <li class="unordered">
										MA 服务帐户没有合适的权限在相关的用户帐户上设置新密码。<br\><br\></li>
              </ul>
              <ul>
                <li class="unordered">
										该用户的帐户位于不允许密码设置操作的受保护组中（例如，域管理员或企业管理员）。<br\><br\></li>
              </ul>
              <p>若要了解会导致此错误的其他情况的详细信息，请参阅<a href="#troubleshoot-password-writeback">排查密码写回问题</a>。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>31012</p>
            </td>
            <td>
              <p>OffboardingEventStart </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>如果你为 Azure AD Sync 禁用了密码写回，则会发生此事件，它表示我们已开始将你的组织卸载到密码写回 web 服务。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>31013</p>
            </td>
            <td>
              <p>OffboardingEventSuccess </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示卸载过程成功并且已成功地禁用了密码写回功能。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>31014</p>
            </td>
            <td>
              <p>OffboardingEventFail </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示卸载过程没有成功。这可能是由于在配置期间指定的云管理员帐户或本地管理员帐户存在权限错误导致的，也可能是因为你在禁用密码写回时尝试使用联合的云全局管理员。若要解决此问题，请检查你的管理权限并且确保你在配置密码写回功能时未使用任何联合帐户。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>31015</p>
            </td>
            <td>
              <p>WriteBackServiceStarted </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示密码写回服务已成功启动，并且已就绪可接受来自云的密码管理请求。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>31016</p>
            </td>
            <td>
              <p>WriteBackServiceStopped </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示密码写回服务已停止并且来自云环境的任何密码管理请求都不会成功。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>31017</p>
            </td>
            <td>
              <p>AuthTokenSuccess </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示为启动卸载或登记过程，我们已成功检索到在设置 Azure AD Connect 期间指定的全局管理员的授权令牌。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>31018</p>
            </td>
            <td>
              <p>KeyPairCreationSuccess </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示我们已成功创建了密码加密密钥，该密钥将用来对从云发送到你的本地环境的密码进行加密。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>32000</p>
            </td>
            <td>
              <p>UnknownError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示在执行密码管理操作期间遇到未知的错误。有关更多详细信息，请查看事件中的异常文本。如果有任何问题，请尝试禁用并重新启用密码写回。如果这没有帮助，请将你的事件日志的副本以及内部指定的跟踪 ID 提供给你的支持工程师。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>32001</p>
            </td>
            <td>
              <p>ServiceError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示连接到云密码重置服务时发生错误，并且通常在本地服务无法连接到密码重置 web 服务时发生。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>32002</p>
            </td>
            <td>
              <p>ServiceBusError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示连接到你的租户的服务总线实例时发生错误。发生此错误可能是因为你在本地环境中阻止了出站连接。请检查你的防火墙以确保你允许基于 TCP 443 的连接或者到 <a href="https://ssprsbprodncu-sb.accesscontrol.chinacloudapi.cn/">https://ssprsbprodncu-sb.accesscontrol.chinacloudapi.cn/</a> 的连接，然后重试。如果仍然有问题，请尝试禁用再重新启用密码写回。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>32003</p>
            </td>
            <td>
              <p>InPutValidationError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示传递给我们的 web 服务 API 的输入无效。请重试操作。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>32004</p>
            </td>
            <td>
              <p>DecryptionError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示解密从云到达的密码时出错。这可能是因为云服务与本地环境之间存在解密密钥不匹配问题。若要解决此问题，请在你的本地环境中禁用再重新启用密码写回。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>32005</p>
            </td>
            <td>
              <p>ConfigurationError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>在登记期间，我们将特定于租户的信息保存到你的本地环境中的配置文件中。此事件表示保存此文件时出错或者启动服务时读取此文件时出错。若要解决此问题，请尝试禁用并重新启用密码写回以强制重新写入此配置文件。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>32006</p>
            </td>
            <td>
              <p>EndPointConfigurationError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>已废弃 - 此事件在 Azure AD Connect 中不存在，只存在于支持写回的非常早的 DirSync 内部版本中。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>32007</p>
            </td>
            <td>
              <p>OnBoardingConfigUpdateError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>在登记期间，我们将数据从云发送到本地密码重置服务。然后，在将数据发送到同步服务以安全地在磁盘上存储此信息之前，会将数据写入到一个内存中文件中。此事件表示在内存中写入或更新该数据时出现问题。若要解决此问题，请尝试禁用并重新启用密码写回以强制重新写入此配置。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>32008</p>
            </td>
            <td>
              <p>ValidationError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示我们从密码重置 web 服务收到了无效的响应。若要解决此问题，请尝试禁用再重新启用密码写回。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>32009</p>
            </td>
            <td>
              <p>AuthTokenError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示我们无法获取在设置 Azure AD Sync 期间指定的全局管理员帐户的授权令牌。此错误可能是由于为全局管理员帐户指定了错误的用户名或密码导致的，也可能是因为指定的全局管理员帐户是联合的。若要解决此问题，请使用正确的用户名和密码重新运行配置，并确保管理员是一个托管帐户（仅云帐户或密码同步的帐户）。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>32010</p>
            </td>
            <td>
              <p>CryptoError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示在生成密码加密密钥时或者解密从云服务到达的密码时发生错误。此错误可能表示你的环境存在问题。请查看你的事件日志的详细信息来了解详细信息并解决此问题。你还可以尝试禁用再重新启用密码写回服务来解决此问题。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>32011</p>
            </td>
            <td>
              <p>OnBoardingServiceError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示本地服务无法正确地与密码重置 web 服务进行通信来启动登记过程。这可能是由于防火墙规则导致的，也可能是因为获取你的租户的授权令牌时出现问题。若要解决此问题，请确保你没有阻止基于 TCP 443 和 TCP 9350-9354  的出站连接或者到 <a href="https://ssprsbprodncu-sb.accesscontrol.chinacloudapi.cn/">https://ssprsbprodncu-sb.accesscontrol.chinacloudapi.cn/</a> 的出站连接，并确保你用来登记的 AAD 管理员帐户不是联合的。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>32012</p>
            </td>
            <td>
              <p>OnBoardingServiceDisableError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>已废弃 - 此事件在 Azure AD Connect 中不存在，只存在于支持写回的非常早的 DirSync 内部版本中。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>32013</p>
            </td>
            <td>
              <p>OffBoardingError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示本地服务无法正确地与密码重置 web 服务进行通信来启动卸载过程。这可能是由于防火墙规则导致的，也可能是因为获取你的租户的授权令牌时出现问题。若要解决此问题，请确保你没有阻止基于 443 的出站连接或者到 <a href="https://ssprsbprodncu-sb.accesscontrol.chinacloudapi.cn/">https://ssprsbprodncu-sb.accesscontrol.chinacloudapi.cn/</a> 的出站连接，并确保你用来卸载的 AAD 管理员帐户不是联合的。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>32014</p>
            </td>
            <td>
              <p>ServiceBusWarning </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示我们必须重新尝试连接到你的租户的服务总线实例。正常情况下，这应当无需顾虑，但如果你很多次看到此事件，请考虑检查到服务总线的网络连接，特别是当使用高延迟或低带宽连接时。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>32015</p>
            </td>
            <td>
              <p>ReportServiceHealthError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>为了监视你的密码写回服务的运行状况，我们每 5 分钟向我们的密码重置 web 服务发送一次检测信号数据。此事件表示将此运行状况信息发送回云 web 服务时发生错误。此运行状况信息不包含 OII 或 PII 数据，是纯粹的检测信号和基本的服务统计信息，以便我们可以在云中提供服务状态信息。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>33001</p>
            </td>
            <td>
              <p>ADUnKnownError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示 AD 返回了未知的错误，有关此错误的详细信息，请检查 Azure AD Connect 服务器事件日志来自 ADSync 源的事件。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>33002</p>
            </td>
            <td>
              <p>ADUserNotFoundError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示在本地目录中未找到尝试重置或更改密码的用户。如果已在本地删除了该用户但在云中未删除，或者出现了同步问题时，可能会发生此情况。有关详细信息，请检查你的同步日志以及最近运行的几次同步的详细信息。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>33003</p>
            </td>
            <td>
              <p>ADMutliMatchError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>当密码重置或更改请求来自云时，我们使用在 Azure AD Connect 的设置过程中指定的云定位点来确定如何将该请求链接回你的本地环境中的用户。此事件表示我们在你的本地目录中找到了具有相同的云定位点属性的两个用户。有关详细信息，请检查你的同步日志以及最近运行的几次同步的详细信息。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>33004</p>
            </td>
            <td>
              <p>ADPermissionsError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示管理代理服务帐户在相关帐户上没有合适的权限来设置新密码。请确保用户的林中的 MA 帐户对林中的所有对象都具有重置和更改密码的权限。有关如何执行此操作的详细信息，请参阅<a href="/documentation/articles/active-directory-passwords-getting-started/#step-4-set-up-the-appropriate-active-directory-permissions">步骤 4：设置适当的 Active Directory 权限</a>。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>33005</p>
            </td>
            <td>
              <p>ADUserAccountDisabled </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示我们试图重置或更改在本地已被禁用的帐户的密码。请启用该帐户，然后重试操作。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>33006</p>
            </td>
            <td>
              <p>ADUserAccountLockedOut </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示我们试图重置或更改在本地已被锁定的帐户的密码。如果用户在短时间内尝试了太多次更改或重置密码操作，则会发生锁定。请解锁该帐户并重试操作。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>33007</p>
            </td>
            <td>
              <p>ADUserIncorrectPassword </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示用户在执行密码更改操作时指定了错误的当前密码。请指定正确的当前密码，然后重试。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>33008</p>
            </td>
            <td>
              <p>ADPasswordPolicyError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>当密码写回服务尝试在你的本地目录中设置的密码不符合域在密码期限、历史记录、复杂度或筛选方面的要求时，将发生此事件。</p>
              <ul>
                <li class="unordered">
										如果你有最短密码期限，并且最近在此时间窗口内已更改过密码，你将不能再次更改密码，直到它达到你的域中指定的期限。对于测试目的，最短期限应设置为 0。<br\><br\></li>
              </ul>
              <ul>
                <li class="unordered">
										如果你启用了密码历史记录要求，则必须选择在最近 N 次未使用过的密码，其中 N 是密码历史记录设置。如果你选择了在最近 N 次中使用过的密码，则在此情况下将会失败。对于测试目的，历史记录应设置为 0。<br\><br\></li>
              </ul>
              <ul>
                <li class="unordered">
										如果你有密码复杂性要求，则当用户尝试更改或重置密码时会强制实施所有这些要求。<br\><br\></li>
              </ul>
              <ul>
                <li class="unordered">
										如果你启用了密码筛选器，并且用户选择了不满足筛选条件的密码，则重置或更改操作将失败。<br\><br\></li>
              </ul>
            </td>
          </tr>
          <tr>
            <td>
              <p>33009</p>
            </td>
            <td>
              <p>ADConfigurationError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>此事件表示将密码写回到你的本地目录时由于 Active Directory 存在配置问题而出现问题。有关发生了什么错误的详细信息，请检查 Azure AD Connect 计算机的应用程序事件日志以查找来自 ADSync 服务的消息。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>34001</p>
            </td>
            <td>
              <p>ADPasswordPolicyOrPermissionError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>已废弃 - 此事件在 Azure AD Connect 中不存在，只存在于支持写回的非常早的 DirSync 内部版本中。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>34002</p>
            </td>
            <td>
              <p>ADNotReachableError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>已废弃 - 此事件在 Azure AD Connect 中不存在，只存在于支持写回的非常早的 DirSync 内部版本中。</p>
            </td>
          </tr>
          <tr>
            <td>
              <p>34003</p>
            </td>
            <td>
              <p>ADInvalidAnchorError </p>
            </td>
            <td>
              <p>PasswordResetService</p>
            </td>
            <td>
              <p>已废弃 - 此事件在 Azure AD Connect 中不存在，只存在于支持写回的非常早的 DirSync 内部版本中。</p>
            </td>
          </tr>
        </tbody></table>
## <a name="troubleshoot-password-writeback-connectivity"></a>排查密码写回连接问题

如果遇到 Azure AD Sync 密码写回组件的服务中断，可以使用以下快速步骤来解决此问题：

 - [重新启动 Azure AD Sync 服务](#restart-the-azure-AD-sync-service)
 - [禁用再重新启用密码写回功能](#disable-and-re-enable-the-password-writeback-feature)
 - [安装最新版本的 Azure AD Connect](#install-the-latest-azure-ad-connect-release)
 - [排查密码写回问题](#troubleshoot-password-writeback)

通常，我们建议按照上述顺序执行这些步骤，从而以最快的方式恢复你的服务。

### <a name="restart-the-azure-AD-sync-service"></a>重新启动 Azure AD Sync 服务
重新启动 Azure AD Sync 服务可以帮助解决连接问题或该服务出现的其他暂时性问题。

 1.	以管理员身份单击运行 **Azure AD Connect** 的服务器上的“开始”。
 2.	在搜索框中键入“services.msc”，然后按 **Enter**。
 3.	找到 **Microsoft Azure AD Connect** 条目。
 4.	右键单击该服务条目，单击“重新启动”，并等待操作完成。

    ![][002]

这些步骤将重新建立与云服务的连接，并且可解决可能遇到的任何中断。如果重新启动同步服务无法解决你遇到的问题，我们建议你接下来尝试禁用密码写回功能，然后重新启用该功能。

### <a name="disable-and-re-enable-the-password-writeback-feature"></a>禁用再重新启用密码写回功能
禁用再重新启用密码写回功能可以帮助解决连接问题。

 1.	以管理员身份打开 **Azure AD Connect 配置向导**。
 2.	在“连接到 Azure AD”对话框中，输入你的 **Azure AD 全局管理员凭据**。
 3.	在“连接到 AD DS”对话框中，输入你的 **AD 域服务管理员凭据**。
 4.	在“唯一标识你的用户”对话框中，单击“下一步”按钮。
 5.	在“可选功能”对话框中，取消选中“密码写回”复选框。



    ![][003]

 6.	单击“下一步”完成剩余的对话框页，而不更改任何内容，直到转到“准备好配置”页。
 7.	确保配置页上的“密码写回”选项显示为禁用，然后单击绿色的“配置”按钮提交所做的更改。
 8.	在“已完成”对话框中，取消选择“立即同步”选项，然后单击“完成”关闭向导。
 9.	重新打开 **Azure AD Connect 配置向导**。
 10.	**重复步骤 2-8**，但确保选中“可选功能”屏幕上的“密码写回”选项以重新启用该服务。

    ![][004]

这些步骤将重新建立与云服务的连接，并且可解决可能遇到的任何中断。

如果禁用再重新启用密码写回功能无法解决你遇到的问题，我们建议你接下来尝试重新安装 Azure AD Connect。

### <a name="install-the-latest-azure-ad-connect-release"></a>安装最新版本的 Azure AD Connect
重新安装 Azure AD Connect 包可以解决所有可能会影响你在本地 AD 环境中连接到我们的云服务或管理你的密码的配置问题。我们建议你仅在尝试了上述前两个步骤后再执行此步骤。

 1.	可从[此处](/documentation/articles/active-directory-aadconnect/#download-azure-ad-connect)下载最新版本的 Azure AD Connect。
 2.	由于你已安装 Azure AD Connect，只需执行就地升级将 Azure AD Connect 安装更新到最新版本。
 3.	执行下载的程序包，然后按照屏幕说明进行操作来更新 Azure AD Connect 计算机。无需执行其他手动步骤，除非你自定义了现成的规则，在这种情况下，你应该**先备份这些规则，然后再继续进行升级，并在完成后手动重新部署这些规则**。

这些步骤将重新建立与云服务的连接，并且可解决可能遇到的任何中断。

如果安装最新版本的 Azure AD Connect 服务器无法解决你遇到的问题，我们建议你在安装最新的同步 QFE 之后，最后再尝试禁用密码写回，然后重新启用该功能。

如果这样做仍无法解决你遇到的问题，我们建议你查看[排查密码写回问题](#troubleshoot-password-writeback)和 [Azure AD 密码管理常见问题](/documentation/articles/active-directory-passwords-faq/)，以了解其中是否讨论了你遇到的问题。


<br/> <br/> <br/>

## 密码重置文档的链接
以下是所有 Azure AD 密码重置文档页面的链接：

* [**重置自己的密码**](/documentation/articles/active-directory-passwords-update-your-own-password/) - 了解如何以系统用户的身份重置或更改自己的密码
* [**工作原理**](/documentation/articles/active-directory-passwords-how-it-works/) - 了解六个不同的服务组件及其功能
* [**入门**](/documentation/articles/active-directory-passwords-getting-started/) - 了解如何让用户重置及更改云密码或本地密码
* [**自定义**](/documentation/articles/active-directory-passwords-customize/) - 了解如何根据组织的需求自定义服务的外观和行为
* [**最佳实践**](/documentation/articles/active-directory-passwords-best-practices/) - 了解如何快速部署且有效管理组织的密码
* [**深入分析**](/documentation/articles/active-directory-passwords-get-insights/) - 了解集成式报告功能
* [**常见问题**](/documentation/articles/active-directory-passwords-faq/) - 获取常见问题的解答
* [**了解更多**](/documentation/articles/active-directory-passwords-learn-more/) - 深入探索服务工作原理的技术细节




[001]: ./media/active-directory-passwords-troubleshoot/001.jpg "Image_001.jpg"
[002]: ./media/active-directory-passwords-troubleshoot/002.jpg "Image_002.jpg"
[003]: ./media/active-directory-passwords-troubleshoot/003.jpg "Image_003.jpg"
[004]: ./media/active-directory-passwords-troubleshoot/004.jpg "Image_004.jpg"

 
<!---HONumber=Mooncake_0620_2016-->