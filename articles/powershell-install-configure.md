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
	wacn.date="01/29/2016"/>

# 如何安装和配置 Azure PowerShell#

<div class="dev-center-tutorial-selector sublanding"><a href="#" title="PowerShell" class="current">PowerShell</a><a href="/documentation/articles/xplat-cli-install/" title="Azure CLI">Azure CLI</a></div>

## 什么是 Azure PowerShell#
Azure PowerShell 是一个模块，提供用于通过 Windows PowerShell 管理 Azure 的 cmdlet。你可以使用 cmdlet 来创建、测试、部署和管理通过 Azure 平台传送的解决方案和服务。在大多数情况下，这些 cmdlet 可让你执行在 Azure 管理门户中可以执行的任务。例如，你可以创建和配置云服务、虚拟机、虚拟网络和 Web 应用。

<a id="Install"></a>
## 第1步：安装

以下是安装Azure PowerShell的两种方法，即使用WebPI或从PowerShell库中安装：

> [AZURE.NOTE] 您可能需要在安装后重新启动才能看到所有Windows PowerShell的命令。

###从WebPI安装Azure的PowerShell的

从WebPI中安装Azure PowerShell 1.0 或更高版本的方法和安装0.9.x版本是一样的，通过下载[Azure Powershell](http://aka.ms/webpi-azps)开始安装。如果您已经安装Azure PowerShell的0.9.x版本，系统会提示您首先卸载0.9.x的版本。如果从PowerShell库中安装Azure PowerShell模块，安装程序会要求你在安装前移除之前的模块，以确保一致的Azure PowerShell环境。

> [AZURE.NOTE] 如果您已经安装了PowerShell库中的Azure的模块，您会被要求将其卸载。这是为了防止在安装时那些模块和它们所在的安装位置发生冲突。PowerShell库中的模块将安装在**%ProgramFiles%\WindowsPowerShell\Modules**。相比之下WebPI安装程序将安装在Azure模块的**%ProgramFiles%\Microsoft SDKs\Azure\PowerShell**。如果**PowerShellGet**在卸载模块时发现一个模块被作为依赖加载，则保留被锁定.DLL文件和文​​件夹。如果你已经卸载了PowerShell的库模块但仍收到上述安装错误，请从你的**%ProgramFiles%\WindowsPowerShell\Modules**文件夹中删除Azure Powershell文件夹。

如果之前是从PowerShell库中安装Azure PowerShell 而现在想使用WebPI安装，则首先需要运行以下命令。

    # Uninstall the AzureRM component modules
    Uninstall-AzureRM

    # Uninstall AzureRM module
    Uninstall-Module AzureRM

    # Uninstall the Azure module
    Uninstall-Module Azure

    # Or, you can remove all Azure modules
    # Uninstall-Module Azure* -Force

> [AZURE.NOTE] 有一个已知的关于PowerShell的 **$env:PSModulePath**路径的问题会发生在WebPI的安装中。如果您的计算机因系统更新或其他安装而重新启动，可能会导致 **$env:PSModulePath** 没有被包括在已安装的Azure PowerShell的路径中。这可以通过重新启动机器或者将Azure PowerShell的路径添加到**$env:PSModulePath** 来进行修正。


###从库中安装Azure Powershell

使用下面的命令安装从库中安装Azure PowerShell 1.0 或更高版本：

    # Install the Azure Resource Manager modules from the PowerShell Gallery
    Install-Module AzureRM
    Install-AzureRM

    # Install the Azure Service Management module from the PowerShell Gallery
    Install-Module Azure

    # Import AzureRM modules for the given version manifest in the AzureRM module
    Import-AzureRM

    # Import Azure Service Management module
    Import-Module Azure

#### 关于这些命令

- **Install-Module AzureRM** 为AzureRM模​​块安装一个引导模块。该模块包含的cmdlet可以帮助更新、卸载，并以安全和一致的方式导入AzureRM模块。该AzureRM模块包含一个模块列表和版本范围（最小值和最大值）以确保没有对AzureRM的主要版本引入重大更改。有关语义版本的更多信息，请参阅[semver.org](http://semver.org)。这意味着您可以使用AzureRM特定版本来创作您自己的cmdlet并保证所有这些操作都会通过引导程序安装的模块而不会引入重大更改。
- **Install-AzureRM** 安装所有引导模块中声明的模块。
- **Install-Module Azure** 安装Azure的模块。该模块是Azure PowerShell 0.9.x.中的服务管理模块，和现在相比应该没有重大变化，因此可以和以前版本中的Azure的模块互换。
- **Import-AzureRM** 导入所有AzureRM模​​块的列表中的模块和版本。这确保已加载的Azure PowerShell模块是在AzureRM模​​块所需的版本范围内。
- **Import-Module Azure** 导入Azure服务管理模块。注意 Azure 模块和 AzureRM 模块可被同时装载到PowerShell会话，并且可以一起使用。


## Step 2: 启动
该模块为 Azure PowerShell安装了定制后的控制台。您可以从标准的Windows PowerShell控制台运行cmdlet，或在Azure PowerShell控制台运行。您打开控制台的方法取决于你的Windows的版本：

- 在运行比Windows 8或Windows Server 2012更新版本的计算机中，你可以使用内置的搜索。从开始屏幕上**开始**，这将显示所有应用名单，其中包括Windows PowerShell和Azure PowerShell的。要打开控制台，单击应用程序。（右键单击该图标可将其固定到开始屏幕上）
- 在运行比Windows 8或Windows Server 2012的更早版本的计算机中，请使用**开始菜单**。从**开始菜单** 中，单击**所有程序**，然后单击**Azure**中的 **Azure PowerShell**。

您也可以运行**Windows PowerShell ISE**并使用它的菜单项和键盘快捷方式来执行许多和在Windows PowerShell控制台执行相同的任务。要使用ISE，在Windows PowerShell控制台Cmd.exe中或者在 **运行** 框中键入**powershell_ise.exe**。

###帮助你开始的命令

    # To make sure the Azure PowerShell module is available after you install
    Get-Module –ListAvailable 
	
	# If the Azure PowerShell module is not listed when you run Get-Module, you may need to import it
    Import-Module Azure 
	
    # To login to Azure Resource Manager
    Login-AzureRmAccount

    # You can also use a specific Tenant if you would like a faster login experience
    # Login-AzureRmAccount -TenantId xxxx

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


## 第3步：连接
cmdlet 需要使用你的订阅来管理你的服务。如果您还没有Azure订阅请参阅[Azure 入门](/pricing/overview)。

1. 键入 **Login-AzureRmAccount -Environment AzureChinaCloud**

2. 在窗口中，键入与你的帐户相关联的电子邮件地址和密码，Azure 将对凭据信息进行身份验证和保存，然后关闭该窗口。

或者使用工作或学校帐户登录，则可以键入以下命令来绕过弹出窗口。

    $cred = Get-Credential
    Login-AzureAccount -Credential $cred

> [AZURE.NOTE] 如果您有您的单位帐户相关联的多个租户，可以指定TenantId参数：

    $loadersubscription = Get-AzureRmSubscription -SubscriptionName $YourSubscriptionName -TenantId $YourAssociatedSubscriptionTenantId


> [AZURE.NOTE]这种非交互式登录方法仅适用于工作或学校帐户。工作或学校帐户是由你的公司或学校所管理的用户，并在你公司或学校的 Azure Active Directory 实例中定义。如果你当前没有工作或学校帐户，且已使用 Microsoft 帐户登录到 Azure 订阅，则你可以按照以下步骤轻松地创建一个工作或学校帐户。
>
> 1. 登录到[Azure 管理门户](https://manage.windowsazure.cn)，然后单击“Active Directory”。
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
>有关使用工作或学校帐户注册 Windows Azure 的详细信息，请参阅[以组织身份注册 Windows Azure](/documentation/articles/sign-up-organization)。

### 查看帐户和订阅的详细信息

你可以具有多个帐户和订阅以供 Azure PowerShell 使用。您可以通过运行多次 Add-AzureAccount 来添加多个帐户。

若要获取可用的 Azure 帐户，请键入：

	Get-AzureAccount

若要获取 Azure 订阅，请键入：

	Get-AzureSubscription

##<a id="Help"></a>获得帮助##

这些资源提供特定 cmdlet 的帮助信息：


- 从控制台内，可以使用内置的帮助系统。**Get-Help** cmdlet 提供对此系统的访问。
- 要获得社区中的帮助信息，请尝试以下常见论坛：
	- [MSDN 上的 Azure 论坛](https://social.msdn.microsoft.com/Forums/azure/zh-CN/home?forum=windowsazurezhchs)
	- [CSDN](http://azure.csdn.net/)

##了解更多


请参阅以下资源，了解更多有关使用cmdlet的方法：

有关使用Windows PowerShell的基本说明，请参阅 [使用Windows PowerShell](https://technet.microsoft.com/zh-cn/library/dn425048.aspx).

有关cmdlet的参考信息, 请参阅 [Azure 命令行参考](https://msdn.microsoft.com/zh-cn/library/azure/jj554330.aspx).

