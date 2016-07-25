可以使用以下 PowerShell 命令启用非 SSL 终结点

	Set-AzureRmRedisCache -Name "<your cache name>" -ResourceGroupName "<your resource group name>" -EnableNonSslPort $true

<!---HONumber=Mooncake_0718_2016-->