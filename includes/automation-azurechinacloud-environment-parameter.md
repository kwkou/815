> [AZURE.NOTE]
若要使用中国云环境，以下 Azure PowerShell 命令需要添加**“-Environment AzureChinaCloud”**参数。<br />
>	`Add-AzureAccount`<br />
>	`Get-AzurePublishSettingsFile`<br />
>	`Import-AzurePublishSettingsFile`<br />
>	`New-AzureProfile`<br />
>	`New-AzureStorageContext`<br />
>	`Set-AzureSubscription`<br />
>	`Show-AzurePortal`<br />
>例如，`Add-AzureAccount -Credential $Cred -ErrorAction Stop` 应变为 `Add-AzureAccount -Credential $Cred -ErrorAction Stop -Environment AzureChinaCloud`
> 

<!---HONumber=79-->