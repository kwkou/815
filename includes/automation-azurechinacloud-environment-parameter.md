> [AZURE.NOTE]
若要使用中国云环境，以下 Azure PowerShell 命令需要添加**“-Environment AzureChinaCloud”**参数。
> 
	Add-AzureAccount
	Get-AzurePublishSettingsFile
	Import-AzurePublishSettingsFile
	New-AzureProfile
	New-AzureStorageContext
	Set-AzureSubscription
	Show-AzurePortal
例如，`Add-AzureAccount -Credential $Cred -ErrorAction Stop` 应变为 `Add-AzureAccount -Credential $Cred -ErrorAction Stop -Environment AzureChinaCloud`
> 

<!---HONumber=79-->