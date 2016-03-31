Azure PowerShell 目前提供两个版本 - 1.0 和 0.9.8。如果你暂时不想要更改现有的脚本，可以继续使用 0.9.8 版本。使用 1.0 版本时，应该先在预生产环境中仔细测试你的脚本，然后在生产环境中使用，以避免意外的影响。

1\.0 版 cmdlet 遵循命名模式 {谓词}-AzureRm{名词}；而 0.9.8 名称不包括 **Rm**（例如，使用 New-AzureResourceGroup 而不是 New-AzureRmResourceGroup）。在使用 Azure PowerShell 0.9.8 时，首先必须通过运行 **Switch-AzureMode AzureResourceManager** 命令启用资源管理器模式。此命令在 1.0 版中并不需要。

新功能只会添加到 1.0 版本。有关 1.0 版的信息，包括如何安装和卸载该版本，请参阅 [Azure PowerShell 1.0](https://azure.microsoft.com/blog/azps-1-0/)。

<!---HONumber=Mooncake_0104_2016-->