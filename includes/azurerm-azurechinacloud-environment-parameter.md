> [AZURE.NOTE]
若要使用中国云环境，以下 AzureRm PowerShell 命令需要添加**“-Environment”**参数。
> 
	Add-AzureRmAccount
	Login-AzureRmAccount

>例如，`Login-AzureRmAccount` 应变为 `$china = Get-AzureRmEnvironment -Name AzureChinaCloud; Login-AzureRmAccount -Environment $china`
> 
