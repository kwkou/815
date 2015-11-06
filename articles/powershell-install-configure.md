<properties pageTitle="如何安装和配置 Azure PowerShell" description="了解如何安装和配置 Azure PowerShell。" editor="tysonn" manager="stevenka" documentationCenter="" services="" authors="coreyp-at-msft"/>

<tags ms.service="multiple" ms.date="02/20/2015" wacn.date="10/3/2015"/>

# 如何安装和配置 Azure PowerShell#

<div class="dev-center-tutorial-selector sublanding"><a href="/documentation/articles/powershell-install-configure/" title="PowerShell" class="current">PowerShell</a><a href="/documentation/articles/xplat-cli/" title="跨平台 CLI">跨平台 CLI</a></div>

你可以使用 Windows PowerShell 在 Azure 中执行各种任务，不管是在命令提示符下以交互方式执行，还是通过脚本自动执行。Azure PowerShell 是一个模块，提供用于通过 Windows PowerShell 管理 Azure 的 cmdlet。你可以使用 cmdlet 来创建、测试、部署和管理通过 Azure 平台传送的解决方案和服务。在大多数情况下，这些 cmdlet 可让你执行在 Azure 管理门户中可以执行的任务。例如，你可以创建和配置云服务、虚拟机、虚拟网络和 Web 应用程序。

该模块作为可下载文件分发，并且通过公开的存储库管理源代码。在本文后面的安装说明中将提供指向可下载文件的链接。有关源代码的信息，请参阅 [Azure PowerShell 代码存储库](https://github.com/Azure/azure-powershell)。

本指南提供有关安装和设置 Azure PowerShell 以便管理 Azure 平台的基本信息。

### <a id="Prereq"></a>使用 Azure PowerShell 的先决条件

Azure 是基于订阅的平台。这意味着需要订阅才能使用平台。在大多数情况下，这还意味着 cmdlet 需要订阅信息以便执行与你的订阅有关的任务。（可以在没有此信息的情况下使用与存储相关的 cmdlet。） 你通过配置计算机以便连接到你的订阅来提供此信息。在本文中的“如何连接到订阅”下提供了说明。

> [AZURE.NOTE]从版本 0.8.5 开始，Azure PowerShell 模块需要 Microsoft .NET Framework 4.5。

当你安装此模块时，安装程序将检查系统是否具备必需的软件并安装所有依赖项，如 Windows PowerShell 和 .NET Framework 的正确版本。

<h2> <a id="Install"></a>如何安装 Azure PowerShell</h2>

你可以通过运行 [Microsoft Web 平台安装程序](http://go.microsoft.com/fwlink/p/?LinkId=320376)下载并安装 Azure PowerShell 模块。出现提示时，单击“运行”。Web 平台安装程序将安装 Azure PowerShell 模块和所有依赖项。按照提示完成安装。

> [AZURE.NOTE]如果你只想要下载 PowerShell 安装程序，请访问 https://github.com/Azure/azure-powershell/releases。也可以在此存储库中找到 PowerShell Cmdlet 的源代码

有关可用于 Azure 的命令行工具的详细信息，请参阅[命令行工具](/downloads/)。

安装此模块还会安装 Azure PowerShell 的自定义控制台。你可以通过标准的 Windows PowerShell 控制台或 Azure PowerShell 控制台运行 cmdlet。

你用于打开控制台的方法取决于你正在运行的 Windows 的版本：

- 在至少运行 Windows 8 或 Windows Server 2012 的计算机上，你可以使用内置搜索。从“开始”屏幕，开始键入 **power**。此时将返回范围内的应用程序的列表，包括 Windows PowerShell 和 Azure PowerShell。若要打开控制台，请单击任一应用程序。（要将应用程序固定在“开始”屏幕，请右键单击此图标。）

- 在运行早于 Windows 8 或 Windows Server 2012 的版本的计算机上，请使用“开始”菜单。在“开始”菜单上，依次单击“所有程序”、“Azure”和“Azure PowerShell”。

<h2><a id="Connect"></a>如何连接到订阅</h2>

使用 Azure 需要一个订阅。如果你没有订阅，请参阅 [Azure 入门](/pricing/overview)。

cmdlet 需要使用你的订阅来管理你的服务。可通过两种方法向 Windows PowerShell 提供订阅信息。可以使用包含这些信息的管理证书，或者使用你的 Microsoft 帐户或者工作或学校帐户登录 Azure。当你登录时，Azure Active Directory \(Azure AD\) 将对凭据进行身份验证，并返回访问令牌，使 Azure PowerShell 能够管理你的帐户。

为了帮助你选择适合你的需要的身份验证方法，请考虑以下方面：

- Azure AD 是建议的身份验证方法，因为使用此方法可以更轻松地管理对订阅的访问。安装 0.8.6 版的更新后，如果你使用工作或学校帐户，则还可以启用 Azure AD 身份验证的自动化方案。这种方法也适用于 Azure 资源管理器 API。
- 在你使用证书方法时，只要订阅和证书有效，订阅信息就可用。但是，此方法使得管理对共享订阅的访问更加困难，例如在授权多人可以访问帐户时。此外，Azure 资源管理器 API 不接受证书身份验证。

有关 Azure 中的身份验证和订阅管理的详细信息，请参阅[管理帐户、订阅和管理角色](https://msdn.microsoft.com/zh-cn/library/azure/hh531793.aspx)。

<h3>使用 Azure AD 方法</h3>

1. 根据[如何安装 Azure PowerShell](#Install) 中所述，打开 Azure PowerShell 控制台。

2. 输入以下命令：

		Add-AzureAccount -Environment AzureChinaCloud

3. 在窗口中，键入与你的帐户相关联的电子邮件地址和密码。

4. Azure 将对凭据信息进行身份验证和保存，然后关闭该窗口。

5. 从 0.8.6 开始，如果你使用工作或学校帐户登录，则可以键入以下命令来绕过弹出窗口。这会弹出标准的 Windows PowerShell 凭据窗口，供你输入工作或学校帐户的用户名和密码。

        $cred = Get-Credential
        Add-AzureAccount -Environment AzureChinaCloud -Credential $cred

	> [AZURE.NOTE]有关安全性和使用凭据的详细信息，请参阅[将密码和其他敏感数据部署到 ASP.NET 和 Azure 网站的最佳做法](http://www.asp.net/identity/overview/features-api/best-practices-for-deploying-passwords-and-other-sensitive-data-to-aspnet-and-azure)。

	> [AZURE.NOTE]这种非交互式登录方法仅适用于工作或学校帐户。工作或学校帐户是由你的公司或学校所管理的用户，并在你公司或学校的 Azure Active Directory 实例中定义。如果你当前没有工作或学校帐户，且已使用 Microsoft 帐户登录到 Azure 订阅，则你可以按照以下步骤轻松地创建一个工作或学校帐户。
	>
	> 1. 登录到“Azure 管理门户”，然后单击“Active Directory”[](https://manage.windowsazure.cn)。
	>
	> 2. 如果目录不存在，请选择“创建目录”，并提供所请求的信息。
	>
	> 3. 选择目录，并添加新用户。这个新用户可以使用工作或学校帐户登录。
	>
	>     创建用户时，系统将为你提供用户电子邮件地址和临时密码。保存此信息，因为此信息还要用于另一个步骤。
	>
	> 4. 从管理门户中，选择“设置”，然后选择“管理员”。选择“添加”，并将新用户添加为共同管理员。这样工作或学校帐户即可管理 Azure 订阅。
	>
	> 5. 最后，从 Azure 门户注销，然后使用工作或学校帐户重新登录。如果这是使用此帐户首次登录，系统将提示更改密码。
	>
	>有关使用工作或学校帐户注册 Microsoft Azure 的详细信息，请参阅[以组织身份注册 Microsoft Azure](/documentation/articles/sign-up-organization)。

<h3>使用证书方法</h3>

Azure 模块包含可帮助你下载和导入证书的 cmdlet。

> [AZURE.NOTE]AzureResourceManager 模块中的 cmdlet 需要 Azure AD 方法 \(Add-AzureAccount\)。这些 cmdlet 不支持发布设置文件。有关 AzureResourceManager 模块中的 cmdlet 的详细信息，请参阅 [Azure 资源管理器 Cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/dn708504.aspx)。


- **Get-AzurePublishSettingsFile** cmdlet 会在 Azure 管理门户中打开一个网页，你可以从中下载订阅信息。信息包含在 .publishsettings 文件中。

- **Import-AzurePublishSettingsFile** 导入 .publishsettings 文件以供模块使用。此文件包含一个管理证书，其中具有安全凭据。

> [AZURE.IMPORTANT]我们建议在你导入发布设置后，删除使用 <b>Get-AzurePublishSettingsFile</b> 下载的发布配置文件。因为管理证书包含安全凭据，所以不应由未授权用户访问。如需有关订阅的信息，可以从 [Azure 管理门户](http://manage.windowsazure.cn/)获得。

1. 使用你的 Azure 帐户的凭据登录 [Azure 管理门户](http://manage.windowsazure.cn)。

2. 根据[如何安装 Azure PowerShell](#Install) 中所述，打开 Azure PowerShell 控制台。

3. 输入以下命令：

		Get-AzurePublishSettingsFile -Environment AzureChinaCloud

4. 当系统提示时，下载并保存发布配置文件，并记下 .publishsettings 文件的路径和名称。当你运行 **Import-AzurePublishSettingsFile** cmdlet 导入设置时，必须提供此信息。默认的位置和文件名格式为：

		C:\\Users\<UserProfile>\\Download\\[*MySubscription*-...]-*downloadDate*-credentials.publishsettings

5. 键入类似于下面的命令，用你的 Windows 帐户名称和路径以及文件名替换占位符：

		Import-AzurePublishSettingsFile -Environment AzureChinaCloud C:\Users\<UserProfile>\Downloads\<SubscriptionName>-credentials.publishsettings

> [AZURE.NOTE]如果在你导入发布设置后，将你添加到其他订阅中作为共同管理员，则你将需要重复此过程来下载新的 .publishsettings 文件，然后导入这些设置。有关添加共同管理员来帮助管理订阅服务的信息，请参阅[为 Azure 订阅添加和删除协同管理员](http://msdn.microsoft.com/zh-cn/library/windowsazure/gg456328.aspx)。

<h3> 查看帐户和订阅详细信息</h3>
你可以具有多个帐户和订阅以供 Azure PowerShell 使用。你可以通过运行多次 Add-AzureAccount -Environment AzureChinaCloud 来添加多个帐户。

若要获取可用的 Azure 帐户，请键入：

	Get-AzureAccount

若要获取 Azure 订阅，请键入：

	Get-AzureSubscription

## <a id="Ex"></a>如何使用 cmdlet：示例 ##

在安装该模块并配置计算机以便连接到你的订阅后，可以创建一个 Azure Web 应用程序。此示例将帮助你开始使用 Azure cmdlet。

1. 启动 Azure PowerShell 控制台。

2. 选择 Web 应用程序的名称。请选择符合 DNS 命名约定的名称。有效名称只能包含字母“a”到“z”以及数字“0”到“9”，以及连字符（“-”）。

	Web 应用程序名称在 Azure 中必须唯一。我们将在此示例中使用“mySite”，但请务必选择其他名称，例如，你的帐户名称后接一个数字。

	在选择某一名称后，键入如下命令。将“mySite”替换为你的 Web 应用程序名称。

		New-AzureWebsite mySite

	该 cmdlet 将创建 Web 应用程序，然后返回代表新 Web 应用程序的对象。对象属性包含有关 Web 应用程序的有用信息。

3. 若要获取有关 Web 应用程序的信息，键入此命令。该命令将返回订阅中所有 Web 应用程序（包括你刚刚创建的应用程序）的某些信息。

		Get-AzureWebsite

4. 若要获取有关 Web 应用程序的详细信息，请在命令中包含 Web 应用程序名称。请务必将“mySite”替换为你的 Web 应用程序名称。

		Get-AzureWebsite -Name mySite

5. Web 应用程序在创建后将会启动。若要停止 Web 应用程序，请键入此命令，包括你的 Web 应用程序名称。

		Stop-AzureWebsite -Name mySite

6. 若要验证该站点的状态是否为“已停止”，请再次运行 Get-AzureWebsite 命令。

		Get-AzureWebsite

7. 若要完成此测试，请删除 Web 应用程序。键入：

		Remove-AzureWebsite -Name mySite

8. 若要完成该任务，请确认已删除 Web 应用程序。

		Get-AzureWebsite -Name mySite

##<a id="Help"></a>获取帮助##

这些资源提供特定 cmdlet 的帮助信息：


-   从控制台内，可以使用内置的帮助系统。**Get-Help** cmdlet 提供对此系统的访问。下表提供了你可用来获取帮助的一些命令示例。你可以通过键入 **help** 从控制台内获取更多信息。


<table border="1" cellspacing="4" cellpadding="4">
  <tr align="left" valign="top">
    <td><b>命令</b></td>
    <td><b>结果</b></td>
  </tr>
  <tr align="left" valign="top">
    <td>Get-Help</td>
    <td>介绍如何使用“帮助”系统。
      <p><b>注意</b>：说明中包含的某些关于“帮助”文件的信息不适用于 Azure 模块。尤其是，安装此模块时安装的“帮助”文件。这些文件不能单独下载。</p></td>
  </tr>
  <tr align="left" valign="top">
    <td>Get-Help Azure</td>
    <td>获取 Azure 模块中的所有 cmdlet。</td>
  </tr>
  <tr align="left" valign="top">
    <td>Get-Help &lt;<b>语言</b>>-dev</td>
    <td>获取用特定语言开发和管理 PHP 应用程序的 cmdlet。例如，help node-dev、help php-dev 或 help python-dev。</td>
  </tr>
  <tr align="left" valign="top">
    <td>Get-Help &lt;<b>cmdlet</b>></td>
    <td>获取有关 Windows PowerShell cmdlet 的帮助。请将 <cmdlet> 替换为 cmdlet 名称。</td>
  </tr>
  <tr align="left" valign="top">
    <td>Get-Help &lt;<b>cmdlet</b>> -Parameter *</td>
    <td>获取 cmdlet 参数的说明。星号 (*) 表示“所有”。</td>
  </tr>
  <tr align="left" valign="top">
    <td>Get-Help &lt;<b>cmdlet</b>> -Examples</td>
    <td>获取此 cmdlet 的语法和用法示例。</td>
  </tr>
  <tr align="left" valign="top">
    <td>Get-Help &lt;<b>cmdlet</b>> -Full</td>
    <td>获取 cmdlet 的帮助，包括技术详细信息。</td>
  </tr>
</table>




- Azure 库中也提供了有关 Azure PowerShell 模块中 cmdlet 的参考信息。有关信息，请参阅 [Azure Cmdlet 参考](http://msdn.microsoft.com/zh-cn/library/azure/jj554330.aspx)。

要获得社区中的帮助信息，请尝试以下常见论坛：

- [MSDN 上的 Azure 论坛](https://social.msdn.microsoft.com/Forums/azure/zh-CN/home?forum=windowsazurezhchs)
- [堆栈溢出](http://go.microsoft.com/fwlink/?LinkId=320213)


## <a id="Resources"></a>其他资源 ##

下面列出了可用来了解如何使用 Azure 和 Windows PowerShell 的资源。

- 若要了解如何访问 Azure 存储组件，请参阅[对 Azure 存储空间使用 Azure PowerShell](/documentation/articles/storage-powershell-guide-full)。

- 若要提供有关这些 cmdlet 的反馈、报告问题或访问源代码，请参阅 [Azure PowerShell 代码存储库](https://github.com/WindowsAzure/azure-sdk-tools)。

- 若要了解有关 Windows PowerShell 命令行和脚本环境的信息，请参阅 [TechNet 脚本中心](https://technet.microsoft.com/zh-cn/scriptcenter/powershell.aspx)。

- 有关安装、学习、使用和自定义 Windows PowerShell 的信息，请参阅[使用 Windows PowerShell 编写脚本](https://technet.microsoft.com/zh-cn/library/bb978526.aspx)。

- 有关什么是脚本以及如何在 Windows PowerShell 中运行脚本的信息，请参阅[运行脚本](https://technet.microsoft.com/zh-cn/library/bb613481.aspx)。本文包含有关如何创建脚本和将计算机配置为运行脚本的基本信息。

- 有关适用于 Azure AD 的 cmdlet 的信息，请参阅[使用 Windows PowerShell 管理 Azure AD](https://technet.microsoft.com/zh-cn/library/jj151815.aspx)。





