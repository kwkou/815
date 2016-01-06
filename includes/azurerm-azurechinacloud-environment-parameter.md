> [AZURE.NOTE]
若要使用中国云环境，以下 AzureRm PowerShell 命令需要添加**“-Environment”**参数。<br />
>	`Add-AzureRmAccount`<br />
>	`Login-AzureRmAccount`<br />
>例如，`Login-AzureRmAccount` 应变为 `$china = Get-AzureRmEnvironment -Name AzureChinaCloud; Login-AzureRmAccount -Environment $china` (如果你使用的是 Azure PowerShell 1.0.0 或者 1.0.1)，或者 `Login-AzureRmAccount -EnvironmentName AzureChinaCloud` (如果你使用的是 Azure PowerShell 1.0.2 或者 更高)
> 