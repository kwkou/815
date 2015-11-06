

## 为资源管理器模板设置 PowerShell

在将 Azure PowerShell 与资源管理器模板配合使用并通过资源组部署 Azure 资源和工作负载之前，请执行以下步骤。

### 步骤 1：验证 PowerShell 版本

在将 Windows PowerShell 与 ARM 配合使用之前，你必须安装了 Windows PowerShell 3.0 或 4.0 版。若要查找 Windows PowerShell 版本，请在 Windows PowerShell 命令提示符下键入以下命令。

	$PSVersionTable

你应看到类似如下的内容。

	Name                           Value
	----                           -----
	PSVersion                      3.0
	WSManStackVersion              3.0
	SerializationVersion           1.1.0.1
	CLRVersion                     4.0.30319.18444
	BuildVersion                   6.2.9200.16481
	PSCompatibleVersions           {1.0, 2.0, 3.0}
	PSRemotingProtocolVersion      2.2

确保 **PSVersion** 的值为 3.0 或 4.0。若要安装兼容版本，请参阅 [Windows Management Framework 3.0](http://www.microsoft.com/download/details.aspx?id=34595) 或 [Windows Management Framework 4.0](http://www.microsoft.com/zh-cn/download/details.aspx?id=40855)。

你还必须有 Azure PowerShell 0.8.0 或更高版本。可以使用此命令在 Azure PowerShell 命令提示符下查看已安装的 Azure PowerShell 版本。

	Get-Module azure | format-table version

你应看到类似如下的内容。

	Version
	-------
	0.8.16.1

有关说明以及指向最新版本的链接，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)。


### 步骤 2：设置你的 Azure 帐户和订阅

如果你还没有 Azure 订阅，可以注册获取[试用版](/pricing/1rmb-trial/)。

使用此命令列出你的 Azure 订阅。

	Get-AzureSubscription

对于要向其部署新资源的订阅，请注意"帐户"属性。使用"帐户"属性中列出的帐户，通过运行此命令登录到 Azure。

	Add-AzureAccount -Environment AzureChinaCloud

在 Windows Azure 登录对话框中指定帐户的电子邮件地址及其密码。

通过在 Azure PowerShell 命令提示符下运行以下命令设置你的 Azure 订阅。将引号内的所有内容（包括 < 和 > 字符）替换为相应的名称。

	$subscr="<subscription name>"
	Select-AzureSubscription -SubscriptionName $subscr -Current
	Set-AzureSubscription -Environment AzureChinaCloud -SubscriptionName $subscr

你可以从 **Get-AzureSubscription** 命令输出的 **SubscriptionName** 属性获取相应的订阅名称。

有关 Azure 订阅和帐户的详细信息，请参阅[如何：连接到订阅](/documentation/articles/powershell-install-configure#Connect)。

### 步骤 3：切换到 Azure 资源管理器模块

使用此命令切换到 Azure 资源管理器的 Azure PowerShell 命令集。

	Switch-AzureMode AzureResourceManager

> [AZURE.NOTE] 你可以使用 **Switch-AzureMode AzureServiceManagement** 命令切换回 Azure 模块。

<!---HONumber=56-->