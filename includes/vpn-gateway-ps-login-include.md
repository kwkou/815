在开始此配置之前，必须登录到 Azure 帐户。 该 cmdlet 将提示您提供您的 Azure 帐户的登录凭据。 登录后它会下载你的帐户设置，以便这些信息可供 Azure PowerShell 使用。 有关详细信息，请参阅[将 Windows PowerShell 与 Resource Manager 配合使用](/documentation/articles/powershell-azure-resource-manager/)。

使用提升的权限打开 PowerShell 控制台，然后连接到帐户。 使用下面的示例来帮助连接：

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud

如果有多个 Azure 订阅，请查看该帐户的订阅。

    Get-AzureRmSubscription

指定要使用的订阅。

    Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"