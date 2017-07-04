
## 启动 PowerShell 会话
首先你需要安装并运行最新的 [Azure PowerShell](https://msdn.microsoft.com/zh-cn/library/mt619274(v=azure.300).aspx)。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。


>[AZURE.NOTE] SQL 数据库的许多新功能仅在使用 [Azure Resource Manager 部署模型](/documentation/articles/resource-group-overview/)时才可用，因此示例使用面向 Resource Manager 的 [Azure SQL 数据库 PowerShell cmdlets](https://msdn.microsoft.com/zh-cn/library/azure/mt574084(v=azure.300).aspx)。向后兼容支持服务管理（经典）部署模型 [Azure SQL 数据库服务管理 cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/dn546723(v=azure.300).aspx)，但建议使用 Resource Manager cmdlet。


运行 [**Add-AzureRmAccount**](https://msdn.microsoft.com/zh-cn/library/azure/mt619267(v=azure.300).aspx) cmdlet，然后就会出现一个要求输入凭据的登录屏幕。使用与登录 Azure 门户相同的凭据。

	Add-AzureRmAccount -EnvironmentName AzureChinaCloud

如果你有多个订阅，请使用 [**Set-AzureRmContext**](https://msdn.microsoft.com/zh-cn/library/azure/mt619263(v=azure.300).aspx) cmdlet 选择你的 PowerShell 会话应使用的订阅。若要查看当前 PowerShell 会话正在使用哪个订阅，请运行 [**Get AzureRmContext**](https://msdn.microsoft.com/zh-cn/library/azure/mt619265(v=azure.300).aspx)。若要查看所有订阅，请运行 [**Get AzureRmSubscription**](https://msdn.microsoft.com/zh-cn/library/azure/mt619284(v=azure.300).aspx)。

	Set-AzureRmContext -SubscriptionId '4cac86b0-1e56-bbbb-aaaa-000000000000'

<!---HONumber=Mooncake_0116_2017-->