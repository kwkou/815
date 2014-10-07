<properties linkid="Install-Config-Windows-Azure-PowerShell" urlDisplayName="Windows Azure PowerShell" pageTitle="如何安装和配置 Windows Azure PowerShell" description="了解如何安装和配置 Windows Azure PowerShell。" umbracoNaviHide="0" disqusComments="1" writer="kathydav" editor="tysonn" manager="jeffreyg" documentationCenter="" services="" solutions="" authors="" title="如何安装和配置 Windows Azure PowerShell" />

# 如何安装和配置 Windows Azure PowerShell#

<div class="dev-center-tutorial-selector sublanding"><a href="/zh-cn/manage/install-and-configure-windows-powershell/" title="PowerShell" class="current">PowerShell</a><a href="/zh-cn/manage/install-and-configure-cli/" title="跨平台 CLI">跨平台 CLI</a></div>

您可以使用 Windows PowerShell 在 Windows Azure 中执行各种任务，要么在命令提示符下以交互方式执行，要么通过脚本自动执行。Windows Azure PowerShell 是一个通过 Windows PowerShell 提供用于管理 Windows Azure 的 cmdlet 的模块。您可以使用 cmdlet 创建、测试、部署和管理通过 Windows Azure 平台提供的解决方案和服务。在大多数情况下，您可以使用 cmdlet 执行与通过 Windows Azure 管理门户执行的相同任务。例如，您可以创建和配置云服务、虚拟机、虚拟网络和网站。

该模块作为可下载文件分发，并且通过公开的存储库管理源代码。在本文后面的安装说明中将提供指向可下载文件的链接。有关源代码的信息，请参见 [Windows Azure PowerShell 代码存储库](https://github.com/WindowsAzure/azure-sdk-tools)。

本指南提供有关安装和设置 Windows Azure PowerShell 以便管理 Windows Azure 平台的基本信息。

## 目录
  
 * [使用 Windows Azure PowerShell 的先决条件](#Prereq)
 * [如何安装 Windows Azure PowerShell](#Install)
 * [如何连接到订阅](#Connect)
 * [如何使用 cmdlet：示例](#Ex)
 * [获取帮助](#Help)
 * [其他资源](#Resources)


### <a id="Prereq"></a>使用 Windows Azure PowerShell 的先决条件

Windows Azure 是基于订阅的平台。这意味着需要订阅才能使用平台。在大多数情况下，这还意味着 cmdlet 需要订阅信息以便执行与您的订阅有关的任务。（可以在没有此信息的情况下使用与存储相关的 cmdlet。）您通过配置计算机以便连接到您的订阅来提供此信息。在本文中的“如何连接到订阅”下提供了说明。

> [WACOM.NOTE] 提供多种订阅选项。有关信息，请参见 [Windows Azure 入门](http://go.microsoft.com/fwlink/p/?LinkId=320795)。

当您安装此模块时，安装程序将检查系统是否具备必需的软件并安装所有依赖项，如 Windows PowerShell 和 .NET Framework 的正确版本。

<h2> <a id="Install"></a>如何安装 Windows Azure PowerShell</h2>

您可以通过运行 [Microsoft Web 平台安装程序](http://go.microsoft.com/fwlink/p/?LinkId=320376)下载并安装 Windows Azure PowerShell 模块。系统提示后，单击“运行”。Microsoft Web 平台安装程序将加载，同时还加载可供安装的 Windows Azure PowerShell 模块。Web 平台安装程序将安装 Windows Azure PowerShell cmdlet 的所有依赖项。按照提示完成安装。

有关可用于 Windows Azure 的命令行工具的更多信息，请参见[命令行工具]( http://go.microsoft.com/fwlink/?LinkId=320796)。

安装此模块还会安装 Windows Azure PowerShell 的自定义控制台。您可以通过标准的 Windows PowerShell 控制台或 Windows Azure PowerShell 控制台运行 cmdlet。

您用于打开控制台的方法取决于您正在运行的 Windows 的版本：

- 在至少运行 Windows 8 或 Windows Server 2012 的计算机上，您可以使用内置搜索。从“开始”屏幕，开始键入 power。此时将生成应用程序的范围列表，包括 Windows PowerShell 和 Windows Azure PowerShell。单击任一应用程序以打开控制台窗口。（要将应用程序固定在“开始”屏幕，请右键单击此图标。）

- 在运行早于 Windows 8 或 Windows Server 2012 的版本的计算机上，可以使用“开始”菜单。在“开始”菜单上，依次单击“所有程序”、“Windows Azure”和“Windows Azure PowerShell”。


<h2><a id="Connect"></a>如何连接到订阅</h2>

使用 Windows Azure 要求订阅。如果您不具有订阅，请参见 [Windows Azure 入门](http://go.microsoft.com/fwlink/p/?LinkId=320795)。

cmdlet 要求您的订阅信息，以便可以使用它来管理您的服务。截至该模块的 .0.7 版本，有两种提供此信息的方法。您可以下载和使用包含该信息的管理证书，或者可以使用您的 Microsoft 帐户或组织 ID 登录到 Windows Azure。在您登录时，Windows Azure Active Directory (Windows Azure AD) 将对凭据进行身份验证。

为了帮助您选择适合您的需要的身份验证方法，请考虑以下方面：

- Windows Azure AD 方法可便于管理对订阅的访问，但可能会中断自动化。这些凭据可供 Windows Azure PowerShell 使用 12 小时。在它们到期后，您将需要再次登录。
- 在您使用证书方法时，只要订阅和证书有效，订阅信息就可用。该方法可便于将自动化用于长时间运行的任务。在您下载并导入信息后，无需再次提供它。但是，此方法使得管理对共享订阅的访问更加困难，例如在授权多人可以访问帐户时。

有关 Windows Azure 中的身份验证和订阅管理的更多信息，请参见 [管理帐户、订阅和管理角色](http://go.microsoft.com/fwlink/?LinkId=324796)。

<h3>使用 Windows Azure AD 方法</h3>

1. 按[如何安装 Windows Azure PowerShell](#Install) 中所述，打开 Windows Azure PowerShell 控制台。

2. 输入以下命令：

    `Add-AzureAccount`

3. 在窗口中，键入与您的帐户相关联的电子邮件地址和密码。

4. Windows Azure 将对凭据信息进行身份验证和保存，然后关闭该窗口。

<h3>使用证书方法</h3>

Windows Azure PowerShell 模块包括可帮助您下载和导入证书的 cmdlet。

- Get-AzurePublishSettingsFile 会在 Windows Azure 管理门户
中打开一个网页，您可以从中
下载订阅信息。信息包含在 .publishsettings 文件中。

- Import-AzurePublishSettingsFile 导入 .publishsettings 文件以供模块使用。此文件包含一个管理证书，其中具有安全凭据。

<div class="dev-callout"> 
<b>重要说明</b>
<p>我们建议在您导入发布设置后，
删除使用 <b>Get-AzurePublishSettingsFile</b> 下载的
发布配置文件。因为管理证书包含安全凭据，
所以不应由未授权用户访问。如果您需要
有关您的订阅的信息，可以从 <a href="http://manage.windowsazure.com/">Windows Azure 管理门户</a>或 <a href="http://go.microsoft.com/fwlink/p/?LinkId=324875">Microsoft Online Services 客户门户</a>获得。</p>
</div>

1. 使用您的 Windows Azure 帐户的凭据登录 [Windows Azure 管理门户](http://manage.windowsazure.com)。

2. 按[如何安装 Windows Azure PowerShell](#Install) 中所述，打开 Windows Azure PowerShell 控制台。

3. 输入以下命令：

    `Get-AzurePublishSettingsFile`

4. 当系统提示时，下载并保存发布配置文件，并记下 .publishsettings 文件的路径和名称。当您运行 Import-AzurePublishSettingsFile cmdlet 导入设置时，
必须提供此信息。默认的
位置和文件名格式为：

	C:\\Users\&lt;UserProfile&gt;\\Download\\[*MySubscription*-...]-*downloadDate*-credentials.publishsettings

5. 键入如下命令，用您的 Windows 帐户名称和路径以及文件名替换占位符：

    Import-AzurePublishSettingsFile C:\Users\&lt;UserProfile&gt;\Downloads\&lt;SubscriptionName&gt;-credentials.publishsettings

> [WACOM.NOTE] 如果在您导入发布设置后，将您添加到其他订阅中作为共同管理员，
则您将需要重复此过程来下载新的 .publishsettings 文件，然后导入这些设置。有关添加共同管理员来帮助管理订阅服务的信息，
请参见[添加和删除 Windows Azure 订阅的共同管理员](http://msdn.microsoft.com/zh-cn/library/windowsazure/gg456328.aspx)。

<h3> 查看帐户和订阅详细信息</h3>
您可以具有多个帐户和订阅以供 Windows Azure PowerShell 使用。您可以通过运行多次 Add-AzureAccount 来添加多个帐户。

若要查看可用帐户，请键入：

	`Get-AzureAccount`

若要查看订阅信息，请键入：

	`Get-AzureSubscription`

## <a id="Ex"></a>如何使用 cmdlet：示例##

在您安装了该模块并且配置了计算机以便连接到您的订阅后，可以创建一个网站作为示例，演示如果使用 cmdlet。

1. 打开 Windows Azure PowerShell shell，除非它已打开。

2. 确定要提供给网站的名称。您将需要使用符合 DNS 命名约定的名称。有效名称只能包含字母“a”到“z”以及数字“0”到“9”，并且还可以包含连字符“-”。该名称在 Windows Azure 中还必须是唯一的。

	在选择某一名称后，键入如下命令。例如，若要使用您的帐户和编号创建一个网站，以便可以多次使用此约定，请键入以下内容，并且以您的帐户名称替代：

	`New-AzureWebsite my-site-name-1`

	该 cmdlet 将创建该网站，然后列出与其有关的信息。

3. 若要查看该网站的状态，请键入：

	`Get-AzureWebsite`

	该 cmdlet 将列出名称和状态，以及新网站的宿主名称。

	> [WACOM.NOTE] 按如下所示运行，该命令将列出与当前订阅相关联的所有网站有关的信息。

4. 网站将在创建后启动。若要停止网站运行，请键入：

	`Stop-AzureWebsite -Name account-name-1`

5. 再次运行 Get-AzureWebsite 命令以便确认该网站的状态为“已停止”。
  
5. 若要完成此测试，您可以删除该网站。类型：

	`Remove-AzureWebsite -Name account-name-1`

	确认删除以便完成该任务。

##<a id="Help"></a>获取帮助##

这些资源提供特定 cmdlet 的帮助信息：


-   从控制台内，可以使用内置的帮助系统。Get-Help cmdlet 提供对此系统的访问。下表提供了您可用来获取帮助的一些命令示例。您可以通过键入 help 从控制台内获取更多信息。

    <table border="1" cellspacing="4" cellpadding="4">
    <tbody>
    <tr align="left" valign="top">
		<td><b>命令</b></td>
		<td><b>结果</b></td>
    </tr>
    <tr align="left" valign="top">
		<td>help</td>
		<td>介绍如何使用“帮助”系统。<p><b>注意</b>：说明内容包括有关不适用于 Windows Azure 模块的“帮助”文件的一些信息。尤其是，安装此模块时安装的“帮助”文件。这些文件不能单独下载。</p>
</td>
    </tr>
    <tr align="left" valign="top">
		<td>help azure</td>
		<td>列出 Windows Azure PowerShell 模块中的所有 cmdlet。</td>
    </tr>
	<tr align="left" valign="top">
		<td>help &lt;<b>语言</b>&gt;-dev</td>
		<td>列出用特定语言开发和管理 PHP 应用程序的 cmdlet。例如，help node-dev、help php-dev 或 help python-dev。</td>
    </tr>
	    <tr align="left" valign="top">
		<td>help &lt;<b>cmdlet</b>&gt;</td>
		<td>显示有关 Windows PowerShell cmdlet 的帮助。</td>
    </tr>
    <tr align="left" valign="top">
		<td>help &lt;<b>cmdlet</b>&gt; -parameter *</td>
		<td>显示 cmdlet 的参数定义。例如 help get-azuresubscription -parameter *</td>
    </tr>
    <tr align="left" valign="top">
		<td>help &lt;<b>cmdlet</b>&gt; -examples</td>
		<td> 显示 cmdlet 的示例命令的语法和说明。</td>
    </tr>
    <tr align="left" valign="top">
		<td>help &lt;<b>cmdlet</b>&gt; -full</td>
		<td>显示 cmdlet 的技术要求。</td>
    </tr>
    </tbody>
    </table>



- Windows Azure 库中也提供了有关 Windows Azure PowerShell 模块中的 cmdlet 的参考信息。有关信息，请参见 [Windows Azure Cmdlet 参考](http://msdn.microsoft.com/zh-cn/library/windowsazure/jj554330.aspx)。

要获得社区中的帮助信息，请尝试以下常见论坛：

- [MSDN 上的 Windows Azure]( http://go.microsoft.com/fwlink/p/?LinkId=320212)
- [堆栈溢出](http://go.microsoft.com/fwlink/?LinkId=320213)


## <a id="Resources"></a>其他资源##

这些是可用来了解如何使用 Windows Azure 和 Windows PowerShell 的资源。

- 要提供有关这些 cmdlet 的反馈、报告问题或访问源代码，请参见 [Windows Azure PowerShell 代码存储库](https://github.com/WindowsAzure/azure-sdk-tools)。

- 要了解有关 Windows PowerShell 命令行和脚本环境的信息，请参见 [TechNet 脚本中心](http://go.microsoft.com/fwlink/p/?LinkId=320211)。

- 有关安装、学习、使用和自定义 Windows PowerShell 的信息，请参见[使用 Windows PowerShell 编写脚本](http://go.microsoft.com/fwlink/p/?LinkId=320210)。

- 有关什么是脚本以及如何在 Windows PowerShell 中运行脚本的信息，请参见[运行脚本](http://go.microsoft.com/fwlink/p/?LinkId=320627)。本文包含有关如何创建脚本和将计算机配置为运行脚本的基本信息。

- 有关针对 Windows Azure AD 的 cmdlet 的信息，请参见[使用 Windows PowerShell 管理 Windows Azure AD](http://go.microsoft.com/fwlink/p/?LinkId=320628)。



  
  
  [Microsoft Online Services 客户门户]: https://mocp.microsoftonline.com/site/default.aspx
 
  


