
## 启动 PowerShell 会话

首先你需要安装并运行最新的 [Azure PowerShell](http://msdn.microsoft.com/zh-cn/library/mt619274.aspx)。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)。


>[AZURE.NOTE] 由于本主题中的示例使用的是 [Azure Resource Manager 部署模型](/documentation/articles/resource-group-overview/)，因此示例将使用 [Azure Resource Manager cmdlet](http://msdn.microsoft.com/zh-cn/library/azure/mt125356.aspx)。

运行 [**Add-AzureRmAccount**](http://msdn.microsoft.com/zh-cn/library/mt619267.aspx) cmdlet，然后就会出现一个要求你输入凭据的登录屏幕。使用登录 Azure 门户预览时所用的同一凭据。

	Add-AzureRmAccount -EnvironmentationName AzureChinaCloud

如果你有多个订阅，请使用 [**Set-AzureRmContext**](http://msdn.microsoft.com/zh-cn/library/mt619263.aspx) cmdlet 选择你的 PowerShell 会话应使用的订阅。若要查看当前 PowerShell 会话正在使用哪个订阅，请运行 [**Get AzureRmContext**](http://msdn.microsoft.com/zh-cn/library/mt619265.aspx)。若要查看所有订阅，请运行 [**Get AzureRmSubscription**](http://msdn.microsoft.com/zh-cn/library/mt619284.aspx)。

	Set-AzureRmContext -SubscriptionId '4cac86b0-1e56-bbbb-aaaa-000000000000'

<!---HONumber=Mooncake_1024_2016-->