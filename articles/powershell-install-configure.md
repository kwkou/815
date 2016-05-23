<properties
	pageTitle="如何安装和配置 Azure PowerShell"
	description="了解如何安装和配置 Azure PowerShell。"
	editor="tysonn"
	manager="stevenka"
	documentationCenter=""
	services=""
	authors="coreyp-at-msft"/>

<tags
	ms.service="multiple"
	ms.date="01/06/2016"
	wacn.date="04/11/2016"/>

# 如何安装和配置 Azure PowerShell#

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/powershell-install-configure)
- [Azure CLI](/documentation/articles/xplat-cli-install)

##什么是 Azure PowerShell？#
Azure PowerShell 是一个模块，提供用于通过 Windows PowerShell 管理 Azure 的 cmdlet。你可以使用 cmdlet 来创建、测试、部署和管理通过 Azure 平台传送的解决方案和服务。在大多数情况下，这些 cmdlet 可让你执行在 Azure 管理门户中可以执行的任务，例如，创建和配置云服务、虚拟机、虚拟网络和 Web 应用。

<a id="Install"></a>
## 步骤 1：安装

以下是安装 Azure PowerShell 的两种方法。你可以通过 WebPI 或 PowerShell 库进行安装：

> [AZURE.NOTE] 在安装后，你可能需要重新启动才能看到 Windows PowerShell 集成脚本环境 (ISE) 中的所有命令。

###从 WebPI 安装 Azure PowerShell

从 WebPI 安装 Azure PowerShell 1.0 和更高版本的方法与安装 0.9.x 版本是一样的。下载 [Azure PowerShell](http://aka.ms/webpi-azps) 并开始安装。如果你已安装 Azure PowerShell 0.9.x，系统将提示你卸载 0.9.x。如果你从 PowerShell 库安装了 Azure PowerShell 模块，安装程序将要求你在安装之前删除该模块，以确保 Azure PowerShell 环境一致。

> [AZURE.NOTE] 如果已安装 PowerShell 库 Azure 模块，则安装程序将自动删除这些模块。这是为了防止混淆已安装的模块与其所在的位置。PowerShell 库模块通常安装在 **%ProgramFiles%\\WindowsPowerShell\\Modules** 中。WebPI 安装程序将在 **%ProgramFiles%\\Microsoft SDKs\\Azure\\PowerShell** 中安装 Azure 模块。**PowerShellGet** 将卸载模块，如果在卸载时加载了模块依赖项，则会留下锁定的 .dll 及其文件夹。如果在安装过程中发生错误，请删除 **%ProgramFiles%\\WindowsPowerShell\\Modules** 文件夹中的 Azure* 文件夹。

如果已通过 PowerShell 库安装了 Azure PowerShell，但想要使用 WebPI 安装，则 WebPI 安装将自动删除从该库中安装的 cmdlet。

> [AZURE.NOTE] 从 WebPI 安装时，会发生一个有关 PowerShell **$env:PSModulePath** 的已知问题。如果你的计算机由于系统更新或其他安装而需要重新启动，有可能会导致 **$env:PSModulePath** 不包含 Azure PowerShell 的安装路径。可以通过重新启动计算机来更正此问题。

###从库安装 Azure PowerShell

使用以下命令从库安装 Azure PowerShell 1.0 或更高版本：

    # Install the Azure Resource Manager modules from the PowerShell Gallery
    Install-Module AzureRM
    Install-AzureRM

    # Install the Azure Service Management module from the PowerShell Gallery
    Install-Module Azure

    # Import AzureRM modules for the given version manifest in the AzureRM module
    Import-AzureRM

    # Import Azure Service Management module
    Import-Module Azure

####有关这些命令的详细信息

- **Install-Module AzureRM** 为 AzureRM 模块安装引导模块。此模块包含的 cmdlet 可帮助你以安全一致的方式更新、卸载和导入 AzureRM 模块。AzureRM 模块包含所需的模块和版本范围（最小和最大）列表，以确保不会对 AzureRM 的主要版本引入重大更改。有关语义版本的更多信息，请参阅 [semver.org](http://semver.org)。这意味着你可以使用 AzureRM 特定版本来创作你自己的 cmdlet，并了解所有通过引导程序安装的模块不会引入重大更改。
- **Install-AzureRM** 安装引导模块中声明的所有模块。
- **Install-Module Azure** 安装 Azure 模块。此模块是 Azure PowerShell 0.9.x 中的服务管理模块。这应该没有任何重要更改，而且可与前一版本的 Azure 模块互换。
- **Import-AzureRM** 导入 AzureRM 模块的模块与版本列表中的所有模块。这可确保加载的 Azure PowerShell 模块在 AzureRM 模块所需的版本范围内。
- **Import-Module Azure** 导入 Azure 服务管理模块。请注意，Azure 模块和 AzureRM 模块将载入你的 PowerShell 会话，并可一起使用。


## 步骤 2：启动
你可以通过标准的 Windows PowerShell 控制台或 PowerShell 集成脚本环境 (ISE) 运行 cmdlet。
您用于打开控制台的方法取决于您正在运行的 Windows 的版本：

- 在至少运行 Windows 8 或 Windows Server 2012 的计算机上，您可以使用内置搜索。从“开始”屏幕开始键入 power。此时将返回范围内的应用列表，包括 Windows PowerShell。若要打开控制台，请单击任一应用程序。（要将应用程序固定在“开始”屏幕，请右键单击此图标。）

- 在运行早于 Windows 8 或 Windows Server 2012 的版本的计算机上，请使用“开始”菜单。在“开始”菜单上，单击“所有程序”，单击“附件”，单击“Windows PowerShell”文件夹，，然后单击“Windows PowerShell”。

也可以运行 **Windows PowerShell ISE**，使用菜单项和键盘快捷方式来执行可在 Windows PowerShell 控制台中执行的许多相同任务。若要使用 ISE，请在 Windows PowerShell 控制台、Cmd.exe 或“运行”框中，键入 **powershell\_ise.exe**。

###帮助入门的命令

    # To make sure the Azure PowerShell module is available after you install
    Get-Module –ListAvailable 
	
	# If the Azure PowerShell module is not listed when you run Get-Module, you may need to import it
    Import-Module Azure 
	
    # To login to Azure Resource Manager
    Login-AzureRmAccount -EnvironmentName AzureChinaCloud

    # You can also use a specific Tenant if you would like a faster login experience
    # Login-AzureRmAccount -EnvironmentName AzureChinaCloud -TenantId xxxx

    # To view all subscriptions for your account
    Get-AzureRmSubscription

    # To select a default subscription for your current session
    Get-AzureRmSubscription –SubscriptionName “your sub” | Select-AzureRmSubscription

    # View your current Azure PowerShell session context
    # This session state is only applicable to the current session and will not affect other sessions
    Get-AzureRmContext

    # To select the default storage context for your current session
    Set-AzureRmCurrentStorageAccount –ResourceGroupName “your resource group” –StorageAccountName “your storage account name”

    # View your current Azure PowerShell session context
    # Note: the CurrentStorageAccount is now set in your session context
    Get-AzureRmContext

    # To import the Azure.Storage data plane module (blob, queue, table)
    Import-Module Azure.Storage

    # To list all of the blobs in all of your containers in all of your accounts
    Get-AzureRmStorageAccount | Get-AzureStorageContainer | Get-AzureStorageBlob


## 步骤 3：连接
cmdlet 需要使用你的订阅来管理你的服务。如果你没有 Azure 订阅，可以购买一个。有关说明，请参阅[如何购买 Azure](http://go.microsoft.com/fwlink/p/?LinkId=320795)。

1. 键入 **Login-AzureRmAccount -EnvironmentName AzureChinaCloud**

2. 键入与你的帐户关联的电子邮件地址和密码。Azure 将对凭据信息进行身份验证和保存，然后关闭该窗口。

--或者--

登录到你的工作帐户或学校帐户：

    $cred = Get-Credential
    Login-AzureRmAccount -EnvironmentName AzureChinaCloud -Credential $cred
> [AZURE.NOTE] 如果你的组织帐户有多个关联的租户，请指定 TenantId 参数：

    $loadersubscription = Get-AzureRmSubscription -SubscriptionName $YourSubscriptionName -TenantId $YourAssociatedSubscriptionTenantId


> [AZURE.NOTE] 这种非交互式登录方法仅适用于工作或学校帐户。工作或学校帐户是由你的公司或学校所管理的用户，并在你公司或学校的 Azure Active Directory 实例中定义。如果你当前没有工作或学校帐户，且已使用 Microsoft 帐户登录到 Azure 订阅，则你可以按照以下步骤轻松地创建一个工作或学校帐户。

> 1. 登录到“Azure 管理门户”，然后单击“Active Directory”[](https://manage.windowsazure.cn)。

> 2. 如果目录不存在，请选择“创建目录”，并提供所请求的信息。

> 3. 选择目录，并添加新用户。这个新用户可以使用工作或学校帐户登录。创建用户时，系统将为你提供用户电子邮件地址和临时密码。保存此信息，因为下面的步骤 5 将要用到。

> 4. 从门户中，选择“设置”，然后选择“管理员”。选择“添加”，并将新用户添加为共同管理员。这样工作或学校帐户即可管理 Azure 订阅。

> 5. 最后，从 Azure 门户注销，然后使用工作或学校帐户重新登录。如果这是使用此帐户首次登录，系统将提示更改密码。

> 有关使用工作或学校帐户注册 Azure 的详细信息，请参阅[以组织身份注册 Azure](/documentation/articles/sign-up-organization)。

> 有关 Azure 中的身份验证和订阅管理的详细信息，请参阅[管理帐户、订阅和管理角色](http://go.microsoft.com/fwlink/?LinkId=324796)。

### 查看帐户和订阅详细信息

你可以具有多个帐户和订阅以供 Azure PowerShell 使用。可以通过多次运行 **Add-AzureRmAccount -EnvironmentName AzureChinaCloud** 来添加多个帐户。

若要显示可用的 Azure 帐户，请键入 **Get-AzureAccount**。

若要显示 Azure 订阅，请键入 **Get-AzureRmSubscription**。

##<a id="Help"></a>获取帮助##

这些资源提供特定 cmdlet 的帮助信息：


-   从控制台内，可以使用内置的帮助系统。**Get-Help** cmdlet 提供对此系统的访问。 

- 要获得社区中的帮助信息，请尝试以下常见论坛：

	- [MSDN 上的 Azure 论坛](https://social.msdn.microsoft.com/Forums/azure/zh-CN/home?forum=windowsazurezhchs)
	- [CSDN](http://azure.csdn.net/)

##了解详细信息


若要详细了解如何使用 cmdlet，请参阅以下资源：

有关使用 Windows PowerShell 的基本说明，请参阅[使用 Windows PowerShell](http://go.microsoft.com/fwlink/p/?LinkId=321939)。

有关cmdlet的参考信息, 请参阅 [Azure 命令行参考](https://msdn.microsoft.com/zh-cn/library/azure/jj554330.aspx).

有关可帮助你了解如何使用脚本来管理 Azure 的示例脚本和说明，请参阅[脚本中心](http://go.microsoft.com/fwlink/p/?LinkId=321940)。


<!---HONumber=Mooncake_0405_2016-->