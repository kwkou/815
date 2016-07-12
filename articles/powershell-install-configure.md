<!-- Remove script center -->
<properties
	pageTitle="如何安装和配置 Azure PowerShell"
	description="了解如何安装和配置 Azure PowerShell。"
	editor="tysonn"
	manager="dongill"
	documentationCenter=""
	services=""
	authors="coreyp-at-msft"/>

<tags
	ms.service="multiple"
	ms.date="04/22/2016"
	wacn.date="06/20/2016"/>

# 如何安装和配置 Azure PowerShell#

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/powershell-install-configure/)
- [Azure CLI](/documentation/articles/xplat-cli-install/)

##什么是 Azure PowerShell？
Azure PowerShell 是一组模块，提供用于通过 Windows PowerShell 管理 Azure 的 cmdlet。你可以使用 cmdlet 来创建、测试、部署和管理通过 Azure 平台传送的解决方案和服务。在大多数情况下，这些 cmdlet 可让你执行在 Azure 管理门户中可以执行的任务，例如，创建和配置云服务、虚拟机、虚拟网络和 Web 应用。

<a id="Install"></a>
## 步骤 1：安装

以下是安装 Azure PowerShell 的两种方法。你可以通过 WebPI 或 PowerShell 库进行安装：

###从 WebPI 安装 Azure PowerShell

从 WebPI 安装 Azure PowerShell 1.0 和更高版本的方法与安装 0.9.x 版本是一样的。下载 [Azure PowerShell](http://aka.ms/webpi-azps) 并开始安装。如果安装了 Azure PowerShell 0.9.x，将在升级期间卸载版本 0.9.x。如果从 PowerShell 库安装了 Azure PowerShell 模块，安装程序将在安装之前自动删除这些模块，以确保 Azure PowerShell 环境一致。

> [AZURE.NOTE] 如果之前从 PowerShell 库安装了 Azure 模块，安装程序将自动删除这些模块。这是为了防止混淆已安装的模块版本及其所在的位置。PowerShell 库模块通常安装在 **%ProgramFiles%\\WindowsPowerShell\\Modules** 中。相反，WebPI 安装程序将在 **%ProgramFiles(x86)%\\Microsoft SDKs\\Azure\\PowerShell** 中安装 Azure 模块。如果在安装过程中发生错误，可以手动删除 **%ProgramFiles%\\WindowsPowerShell\\Modules** 文件夹中的 Azure* 文件夹，然后重试安装。

一旦完成安装，你的 ```$env:PSModulePath``` 设置中应会有包含 Azure PowerShell cmdlet 的目录。

> [AZURE.NOTE] 从 WebPI 安装时，会发生一个有关 PowerShell **$env:PSModulePath** 的已知问题。如果你的计算机由于系统更新或其他安装而需要重新启动，有可能会导致更新 **$env:PSModulePath** 不包含 Azure PowerShell 的安装路径。如果发生这种情况，当你在安装或升级之后尝试使用 Azure PowerShell cmdlet 时，可能会看到“cmdlet 无法识别”消息。如果发生这种情况，重启计算机应该可以解决该问题。

如果尝试加载或执行 cmdlet，会收到如下消息：

```
    PS C:\> Get-AzureRmResource
    Get-AzureRmResource : The term 'Get-AzureRmResource' is not recognized as the name of a cmdlet, function,
    script file, or operable program. Check the spelling of the name, or if a path was included, verify that the path is
    correct and try again.
    At line:1 char:1
    + Get-AzureRmResource
    + ~~~~~~~~~~~~~~~~~~~~~~~
        + CategoryInfo          : ObjectNotFound: (get-azurermresourcefork:String) [], CommandNotFoundException
        + FullyQualifiedErrorId : CommandNotFoundException
```

这可以通过以下方法更正：重新启动计算机，或者按照以下方式从 C:\Program Files\WindowsPowerShell\Modules\Azure\XXXX\ 导入 cmdlet（其中 XXXX 是所安装 PowerShell 的版本）：
```
import-module "C:\\Program Files\\WindowsPowerShell\\Modules\\Azure\\XXXX\\azure.psd1" 
import-module "C:\\Program Files\\WindowsPowerShell\\Modules\\Azure\\XXXX\\expressroute\\expressroute.psd1" 
```

###从 PowerShell 库安装 Azure PowerShell

在提升的 Windows PowerShell 或 PowerShell 集成脚本环境 (ISE) 提示符下，使用以下命令从 PowerShell 库安装 Azure PowerShell 1.3.0 或更高版本：

    # Install the Azure Resource Manager modules from the PowerShell Gallery
    Install-Module AzureRM

    # Install the Azure Service Management module from the PowerShell Gallery
    Install-Module Azure

####有关这些命令的详细信息

- **Install-Module AzureRM** 为 Azure Resource Manager cmdlet 安装汇总模块。AzureRM 模块取决于每个 Azure Resource Manager 模块的特定版本范围。包含的版本范围可以确保在安装同一主要版本的 AzureRM 模块时，不包含重大的模块更改。安装 AzureRM 模块时，会从 PowerShell 库下载并安装先前未安装的所有 Azure Resource Manager 模块。有关 Azure PowerShell 模块所用的语义版本控制的详细信息，请参阅 [semver.org](http://semver.org)。 
- **Install-Module Azure** 安装 Azure 模块。此模块是 Azure PowerShell 0.9.x 中的服务管理模块。这应该没有任何重要更改，而且可与前一版本的 Azure 模块互换。

## 步骤 2：启动
你可以通过标准的 Windows PowerShell 控制台或 PowerShell 集成脚本环境 (ISE) 运行 cmdlet。您用于打开控制台的方法取决于您正在运行的 Windows 的版本：

- 在至少运行 Windows 8 或 Windows Server 2012 的计算机上，您可以使用内置搜索。从“开始”屏幕开始键入 power。此时将返回范围内的应用列表，包括 Windows PowerShell。若要打开控制台，请单击任一应用程序。（若要将应用固定在“开始”屏幕，请右键单击此图标。）

- 在运行早于 Windows 8 或 Windows Server 2012 的版本的计算机上，请使用“开始”菜单。从“开始”菜单中，依次单击“所有程序”、“附件”、“Windows PowerShell”文件夹和“Windows PowerShell”。

也可以运行 **Windows PowerShell ISE**，使用菜单项和键盘快捷方式来执行可在 Windows PowerShell 控制台中执行的许多相同任务。若要使用 ISE，请在 Windows PowerShell 控制台、Cmd.exe 或“运行”框中，键入 **powershell\_ise.exe**。

###帮助入门的命令

    # To make sure the Azure PowerShell module is available after you install
    Get-Module –ListAvailable 
	
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

    # To list all of the blobs in all of your containers in all of your accounts
    Get-AzureRmStorageAccount | Get-AzureStorageContainer | Get-AzureStorageBlob


## 步骤 3：连接
cmdlet 需要使用你的订阅来管理你的服务。如果你没有 Azure 订阅，可以购买一个。有关说明，请参阅[如何购买 Azure](https://www.azure.cn/pricing/overview/)。

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

> 有关使用工作或学校帐户注册 Azure 的详细信息，请参阅[以组织身份注册 Azure](/documentation/articles/sign-up-organization/)。

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

有关使用 Windows PowerShell 的基本说明，请参阅[使用 Windows PowerShell](https://msdn.microsoft.com/powershell/scripting/getting-started/fundamental/using-windows-powershell)。

有关cmdlet的参考信息, 请参阅 [Azure 命令行参考](https://msdn.microsoft.com/zh-cn/library/azure/jj554330.aspx).

<!-- 有关可帮助你了解如何使用脚本来管理 Azure 的示例脚本和说明，请参阅[脚本中心](http://go.microsoft.com/fwlink/p/?LinkId=321940)。 -->


<!---HONumber=Mooncake_0613_2016-->