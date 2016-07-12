可以通过下述不同方式来安装这些模块：PowerShell 库和 Web 平台安装程序。虽然选择不同的安装方式将决定默认情况下模块在计算机中的安装位置，但最终结果几乎是一样的。

从 PowerShell 库安装时，你的文件将默认位于 *%ProgramFiles%\\WindowsPowerShell\\Modules* 中。从 Web 平台安装程序安装时，你的文件将默认位于 *%ProgramFiles%\\Microsoft SDKs\\Azure\\PowerShell* 中。因此，你需要坚持使用其中一种方式，以免今后更新 cmdlet 时遇到错误。Web 平台安装程序每月都会收到更新的 cmdlet。库会在更新版本的 cmdlet 发布时即时收到它们。由于这个原因，有些人更愿意使用库。

有关安装 Azure PowerShell 的详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

**从 PowerShell 库安装模块**

1. 若要直接从库安装资源管理器模块，请以管理员身份打开 Windows PowerShell 并键入以下命令：

		Install-Module AzureRM
		Install-AzureRM

2. 安装这些模块后，你需要先将它们导入，然后才能使用它们：

		Import-AzureRM

**使用 Web 平台安装程序安装模块**

- 你可以使用 [Web 平台安装程序](http://aka.ms/webpi-azps)来安装模块。单击该链接时，将启动安装程序。

- 如果使用 Web 平台安装程序时出现错误，则可能是因为你已经通过库安装了旧版 cmdlet。请参阅此[博客文章](https://azure.microsoft.com/blog/azps-1-0)，以便获取删除旧版模块的帮助，让一切重回正轨。通常情况下，如果你在使用 Web 平台安装程序的时候切换到库，则会发生错误，反之亦然。删除此前安装的模块将会解决此问题，然后你就可以在新位置进行安装。





<!---HONumber=Mooncake_0425_2016-->
