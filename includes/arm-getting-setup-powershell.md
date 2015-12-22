## 为资源管理器模板设置 PowerShell

你必须拥有正确的 Windows PowerShell 和 Azure PowerShell 版本，才能将 Azure PowerShell 与资源管理器配合使用。

### 验证 PowerShell 版本

确保你有 Windows PowerShell 3.0 或 4.0 版。若要查找 Windows PowerShell 版本，请在 Windows PowerShell 命令提示符下键入以下命令。

	$PSVersionTable

你将收到以下类型的信息：

	Name                           Value
	----                           -----
	PSVersion                      3.0
	WSManStackVersion              3.0
	SerializationVersion           1.1.0.1
	CLRVersion                     4.0.30319.18444
	BuildVersion                   6.2.9200.16481
	PSCompatibleVersions           {1.0, 2.0, 3.0}
	PSRemotingProtocolVersion      2.2


确保 **PSVersion** 的值为 3.0 或 4.0。如果不是，请参阅 [Windows Management Framework 3.0](http://www.microsoft.com/download/details.aspx?id=34595) 或 [Windows Management Framework 4.0](http://www.microsoft.com/download/details.aspx?id=40855)。

### 设置你的 Azure 帐户和订阅


打开 Azure PowerShell 命令提示符，然后使用此命令登录到 Azure。

	Login-AzureRmAccount -Environment $(Get-AzureRmEnvironemnt -Name AzureChinaCloud)

如果有多个 Azure 订阅，你可以使用此命令列出 Azure 订阅。

	Get-AzureRmSubscription

你将收到以下类型的信息：

	SubscriptionId            : fd22919d-eaca-4f2b-841a-e4ac6770g92e
	SubscriptionName          : Visual Studio Ultimate with MSDN
	Environment               : AzureCloud
	SupportedModes            : AzureServiceManagement,AzureResourceManager
	DefaultAccount            : johndoe@contoso.com
	Accounts                  : {johndoe@contoso.com}
	IsDefault                 : True
	IsCurrent                 : True
	CurrentStorageAccountName :
	TenantId                  : 32fa88b4-86f1-419f-93ab-2d7ce016dba7

通过在 Azure PowerShell 命令提示符下运行以下命令设置当前的 Azure 订阅。将引号内的所有内容（包括 < and > 字符）替换为相应的名称。

	$subscr="<SubscriptionName from the display of Get-AzureRmSubscription>"
	Select-AzureRmSubscription -SubscriptionName $subscr -Current

有关 Azure 订阅和帐户的详细信息，请参阅[如何：连接到你的订阅](/documentation/articles/powershell-install-configure/#Connect)。

<!---HONumber=Mooncake_1207_2015-->