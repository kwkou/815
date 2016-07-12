<properties
	pageTitle="Azure 密钥保管库入门 | Azure"
	description="本教程将会帮助你开始使用 Azure 密钥保管库在 Azure 中创建强化容器，以存储和管理 Azure 中的加密密钥和机密。"
	services="key-vault"
	documentationCenter=""
	authors="cabailey"
	manager="mbaldwin"
	tags="azure-resource-manager"/>

<tags
	ms.service="key-vault"
	ms.date="04/07/2016"
	wacn.date="05/13/2016"/>

# Azure 密钥保管库入门 #
在大多数区域中提供了 Azure 密钥保管库。有关详细信息，请参阅[密钥保管库定价页](/home/features/key-vault/pricing/)。

## 介绍  
本教程将会帮助你开始使用 Azure 密钥保管库在 Azure 中创建强化容器（保管库），以存储和管理 Azure 中的加密密钥和机密。本教程将指导你完成使用 Azure PowerShell 创建包含密钥或密码（稍后可用于 Azure 应用程序）的保管库程序。然后，将会说明应用程序如何使用该密钥或密码。

**估计完成时间：**20 分钟。

>[AZURE.NOTE]  本教程未说明如何编写其中一个步骤所包括的 Azure 应用程序，但说明了如何授权应用程序使用密钥保管库中的密钥或机密。
>
>目前，无法在 Azure 门户中配置 Azure 密钥保管库。请改用这些 Azure PowerShell 说明。或者，有关跨平台命令行接口说明，请参阅[此对应教程](/documentation/articles/key-vault-manage-with-cli/)。

有关 Azure 密钥保管库的概述信息，请参阅[什么是 Azure 密钥保管库？](/documentation/articles/key-vault-whatis/)

## 先决条件

若要完成本教程，你必须准备好以下各项：

- Azure 订阅。如果你没有订阅，可以注册[试用版](/pricing/1rmb-trial)。
- Azure PowerShell，**最低版本为 1.1.0**。若要安装 Azure PowerShell 并将其与 Azure 订阅相关联，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。如果你已安装了 Azure PowerShell，但不知道版本，请在 Azure PowerShell 控制台中键入 `(Get-Module azure -ListAvailable).Version`。如果已安装 Azure PowerShell 版本 0.9.1 到 0.9.8，仍可以使用本教程，但需要进行一些细微更改。例如，必须使用 `Switch-AzureMode AzureResourceManager` 命令，并且某些 Azure 密钥保管库命令已更改。有关版本 0.9.1 到 0.9.8 的密钥保管库 cmdlet 的列表，请参阅 [Azure 密钥保管库 Cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/dn868052(v=azure.98).aspx)。 
- 配置为使用你在本教程中所创建的密钥或密码的应用程序。你可以从 [Microsoft 下载中心](http://www.microsoft.com/en-us/download/details.aspx?id=45343)获取示例应用程序。有关说明，请参阅随附的自述文件。


本教程专为 Azure PowerShell 新手设计，但它假定你了解基本概念，如模块、cmdlet 和会话。有关详细信息，请参阅 [Windows PowerShell 入门](https://technet.microsoft.com/zh-cn/library/hh857337.aspx)。

若要获取你在本教程中看到的任何 cmdlet 的详细帮助，请使用 **Get-Help** cmdlet。

	Get-Help <cmdlet-name> -Detailed

例如，若要获取有关 **Login-AzureRmAccount** cmdlet 的帮助，请输入：

	Get-Help Login-AzureRmAccount -Detailed

还可阅读以下教程以熟悉如何在 Azure PowerShell 中使用 Azure 资源管理器：

- [如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)
- [将 Azure PowerShell 用于资源管理器](/documentation/articles/powershell-azure-resource-manager/)


## <a id="connect"></a>连接到订阅 ##

启动 Azure PowerShell 会话，然后使用以下命令登录你的 Azure 帐户：

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud

请注意，如果你使用特定的 Azure 实例（例如 Azure Government），请结合此命令使用 -Environment 参数。例如：`Login-AzureRmAccount –Environment (Get-AzureRmEnvironment –Name AzureUSGovernment)`

在弹出的浏览器窗口中，输入你的 Azure 帐户用户名和密码。Azure PowerShell 会获取与此帐户关联的所有订阅，并按默认使用第一个订阅。

如果你有多个订阅，并想要指定其中一个订阅供 Azure 密钥保管库使用，请键入以下内容以查看帐户的订阅：

    Get-AzureRmSubscription

然后，若要指定要使用的订阅，请键入：

    Set-AzureRmContext -SubscriptionId <subscription ID>

有关配置 Azure PowerShell 的详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。


## <a id="resource"></a>创建新的资源组 ##

使用 Azure 资源管理器时，会在资源组中创建所有相关资源。在本教程中，我们将创建名为 **ContosoResourceGroup** 的新资源组：

	New-AzureRmResourceGroup –Name 'ContosoResourceGroup' –Location 'China East'


## <a id="vault"></a>创建密钥保管库 ##

使用 [New-AzureRmKeyVault](https://msdn.microsoft.com/zh-cn/library/azure/mt603736.aspx) cmdlet 创建密钥保管库。此 cmdlet 包含三个必需参数：**资源组名称**、**密钥保管库名称**和**地理位置**。

例如，如果使用的保管库名称为 **ContosoKeyVault**，资源组名称为 **ContosoResourceGroup**，位置为**中国东部**，请键入：

    New-AzureRmKeyVault -VaultName 'ContosoKeyVault' -ResourceGroupName 'ContosoResourceGroup' -Location 'China East'

此 cmdlet 的输出会显示你刚刚创建的密钥保管库属性。两个最重要的属性是：

- **保管库名称**：在本示例中为 **ContosoKeyVault**。你将在其他密钥保管库 cmdlet 中使用此名称。
- **保管库 URI**：在本示例中为 https://contosokeyvault.vault.chinacloudapi.cn/。通过其 REST API 使用保管库的应用程序必须使用此 URI。

你的 Azure 帐户现已获取在此密钥保管库上执行任何作业的授权。而且没有其他人有此授权。

>[AZURE.NOTE]  如果在尝试创建新的密钥保管库时看到错误“未注册使用命名空间‘Microsoft.KeyVault’的订阅”，请运行 `Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.KeyVault"`，然后重新运行 New-AzureRmKeyVault 命令。有关详细信息，请参阅 [Register-AzureRmProvider](https://msdn.microsoft.com/zh-cn/library/mt679020.aspx)。
>

## <a id="add"></a>将密钥或机密添加到保管库 ##

如果你希望 Azure 密钥保管库为你创建一个受软件保护的密钥，请使用 [Add-AzureKeyVaultKey](https://msdn.microsoft.com/zh-cn/library/azure/dn868048.aspx) cmdlet，并键入以下内容：

    $key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVault' -Name 'ContosoFirstKey' -Destination 'Software'

但是，如果 C:\\ 驱动器中已存储了一个包含现有的受软件保护的密钥，并且要将文件名为 softkey.pfx 的 .PFX 文件上载到 Azure 密钥保管库，请键入以下内容来设置 **securepfxpwd** 变量，以将 .PFX 文件的密码设为 **123**：

    $securepfxpwd = ConvertTo-SecureString –String '123' –AsPlainText –Force

然后键入以下内容以从 .PFX 文件导入密钥，这样，便会使用密钥保管库服务中的软件来保护密钥：

    $key = Add-AzureKeyVaultKey -VaultName 'ContosoKeyVault' -Name 'ContosoFirstKey' -KeyFilePath 'c:\softkey.pfx' -KeyFilePassword $securepfxpwd


现在，你可以通过使用密钥的 URI，引用已创建或上载到 Azure 密钥保管库的密钥。使用 **https://ContosoKeyVault.vault.chinacloudapi.cn/keys/ContosoFirstKey** 可始终获取当前版本，而使用 **https://ContosoKeyVault.vault.chinacloudapi.cn/keys/ContosoFirstKey/cgacf4f763ar42ffb0a1gca546aygd87** 可获取此特定版本。

若要显示此密钥的 URI，请键入：

	$Key.key.kid

若要将名为 SQLPassword 且其 Azure 密钥保管库的值为 Pa$$w0rd 的机密添加到保管库，请先键入以下内容，将 Pa$$w0rd 的值转换成安全字符串：

	$secretvalue = ConvertTo-SecureString 'Pa$$w0rd' -AsPlainText -Force

然后键入以下内容：

	$secret = Set-AzureKeyVaultSecret -VaultName 'ContosoKeyVault' -Name 'SQLPassword' -SecretValue $secretvalue

现在，你可以通过使用密码的 URI，引用已添加到 Azure 密钥保管库的此密码。使用 **https://ContosoVault.vault.chinacloudapi.cn/secrets/SQLPassword** 可始终获取当前版本，而使用 **https://ContosoVault.vault.chinacloudapi.cn/secrets/SQLPassword/90018dbb96a84117a0d2847ef8e7189d** 可获取此特定版本。

若要显示此机密的 URI，请键入：

	$secret.Id

让我们查看一下刚刚创建的密钥或机密：

- 若要查看密钥，请键入：`Get-AzureKeyVaultKey –VaultName 'ContosoKeyVault'`
- 若要查看机密，请键入：`Get-AzureKeyVaultSecret –VaultName 'ContosoKeyVault'`

现在，你可以在应用程序中使用该密钥保管库以及密钥或机密。必须授权应用程序使用这些信息。

## <a id="register"></a>将应用程序注册到 Azure Active Directory ##

此步骤通常由开发人员在独立的计算机上完成。这并非 Azure 密钥保管库的特有状况，在此列出是为了让过程完整。


>[AZURE.IMPORTANT] 若要完成本教程，你的帐户、保管库以及将在本步骤中注册的应用程序全都必须位于相同的 Azure 目录中。

使用密钥保管库的应用程序必须使用 Azure Active Directory 的令牌进行身份验证。为此，应用程序的所有者首先必须在其 Azure Active Directory 中注册该应用程序。注册结束后，应用程序所有者将获得以下值：


- **应用程序 ID**（也称为客户端 ID）和**身份验证密钥**（也称为共享机密）。应用程序必须向 Azure Active Directory 提供这两个值才能获取令牌。如何将应用程序配置为执行此操作取决于应用程序。对于密钥保管库示例应用程序，应用程序所有者将在 app.config 文件中设置这些值。

要在 Azure Active Directory 中注册应用程序，请执行以下操作：

1. 登录到 Azure 门户。
2. 单击左侧的“Active Directory”，然后选择你要在其中注册应用程序的目录。<br> <br> **注意：**你必须选择包含用于创建密钥保管库的 Azure 订阅的相同目录。如果你不知道是哪个目录，请单击“设置”，找到用于创建密钥保管库的订阅，并记下最后一列中显示的目录名称。

3. 单击“应用程序”。如果你的目录中尚未添加任何应用，则此页只会显示“添加应用”链接。单击该链接，或者单击命令栏上的“添加”。
4.	在“添加应用程序”向导的“要执行什么操作?”页面上，单击“添加我的组织正在开发的应用程序”。
5.	在“向我们说明你的应用程序”页上，指定应用程序名称，然后选择“WEB 应用程序和/或 WEB API”（默认值）。单击“下一步”图标。
6.	在“应用程序属性”页上，为你的 Web 应用程序指定“登录 URL”和“应用程序ID URI”。如果你的应用程序没有这些值，可以在此步骤中虚构这些值（例如，可以在两个框中指定 http://test1.contoso.com ）。这些网站是否存在并不重要；重要的是目录中每个应用程序的应用程序 ID URI 都不相同。目录会使用此字符串来识别你的应用程序。
7.	单击“完成”图标以保存向导中的更改。
8.	在“快速启动”页上，单击“配置”。
9.	滚动到“密钥”部分，选择持续时间，然后单击“保存”。页面将会刷新，随后显示密钥值。你必须使用此密钥值和“客户端 ID”值来配置应用程序。（有关此配置的说明仅适用于特定的应用程序。）
10.	从此页中复制客户端 ID，后面的步骤将使用它来设置对保管库的权限。

## <a id="authorize"></a>授权应用程序使用密钥或机密 ##

若要授权应用程序访问保管库中的密钥或机密，请使用 [Set-AzureRmKeyVaultAccessPolicy](https://msdn.microsoft.com/zh-cn/library/azure/mt603625.aspx) cmdlet。

例如，如果保管库名称是 **ContosoKeyVault**，要授权的应用程序的客户端 ID 为 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed，而你希望授权应用程序使用保管库中的密钥来进行解密和签名，请运行以下命令：


	Set-AzureRmKeyVaultAccessPolicy -VaultName 'ContosoKeyVault' -ServicePrincipalName 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed -PermissionsToKeys decrypt,sign

如果要授权同一应用程序读取保管库中的机密，请运行以下命令：


	Set-AzureRmKeyVaultAccessPolicy -VaultName 'ContosoKeyVault' -ServicePrincipalName 8f8c4bbd-485b-45fd-98f7-ec6300b7b4ed -PermissionsToSecrets Get


## <a id="delete"></a>删除密钥保管库以及关联的密钥和机密 ##

如果你不再需要密钥保管库及其包含的密钥或机密，可以使用 [Remove-AzureRmKeyVault](https://msdn.microsoft.com/zh-cn/library/azure/mt619485.aspx) cmdlet 来删除密钥保管库：

	Remove-AzureRmKeyVault -VaultName 'ContosoKeyVault'

或者，你可以删除整个 Azure 资源组，其中包括密钥保管库和你加入该组的任何其他资源：

	Remove-AzureRmResourceGroup -ResourceGroupName 'ContosoResourceGroup'


## <a id="other"></a>其他 Azure PowerShell Cmdlet ##

可能会发现有助于管理 Azure 密钥保管库的其他命令：

- `$Keys = Get-AzureKeyVaultKey -VaultName 'ContosoKeyVault'`：此命令获取以表格形式显示的所有密钥和所选属性。
- `$Keys[0]`：此命令显示特定密钥的完整属性列表
- `Get-AzureKeyVaultSecret`：此命令列出以表格形式显示的所有机密名称和所选属性。
- `Remove-AzureKeyVaultKey -VaultName 'ContosoKeyVault' -Name 'ContosoFirstKey'`：示范如何删除特定密钥。
- `Remove-AzureKeyVaultSecret -VaultName 'ContosoKeyVault' -Name 'SQLPassword'`：示范如何删除特定机密。


## <a id="next"></a>后续步骤 ##

有关在 web 应用程序中使用 Azure 密钥保管库的后续教程，请参阅[从 Web 应用程序使用 Azure 密钥保管库](/documentation/articles/key-vault-use-from-web-application/)。

有关 Azure 密钥保管库的最新 Azure PowerShell cmdlet 列表，请参阅 [Azure 密钥保管库 Cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/dn868052.aspx)。
 

有关编程参考，请参阅 [Azure 密钥保管库开发人员指南](/documentation/articles/key-vault-developers-guide/)。

<!---HONumber=Mooncake_0411_2016-->