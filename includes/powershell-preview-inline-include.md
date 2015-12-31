本主题包括 Azure PowerShell 1.0 预览版中的 PowerShell cmdlet。如果不希望立即安装此预览版本，你可以继续使用 Azure PowerShell 0.9.8。在大多数情况下，两个版本之间唯一的区别是 1.0 预览版 cmdlet 名称遵循模式 {谓词}-AzureRm {名词}；而 0.9.8 名称不包括 **Rm**（例如，用 New-AzureRmResourceGroup 取代 New-AzureResourceGroup）。当版本之间存在较大差异时，本主题会介绍两个版本的示例。

在使用 Azure PowerShell 0.9.8 时，首先必须通过运行 **Switch-AzureMode AzureResourceManager** 命令启用资源管理器模式。此命令在 1.0 预览版中并不需要。

有关 1.0 预览版本的信息，包括有关建议用法的注释，请参阅 [Azure PowerShell 1.0 预览版](https://azure.microsoft.com/blog/azps-1-0-pre/)。有关安装 Azure PowerShell 1.0 预览版的信息，请参阅 [Azure 资源管理器 Cmdlet](https://msdn.microsoft.com/library/mt125356.aspx)。有关资源管理器命令中的重大更改的信息，请参阅 [Azure 资源管理器管理 PowerShell cmdlet 的更改](/documentation/articles/powershell-preview-resource-manager-changes)。

<!---HONumber=79-->