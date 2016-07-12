在 Azure 中使用映像来提供包含操作系统的新虚拟机。一个映像也可能包含一个或多个数据磁盘。映像可从多个源获取：

-	Azure 在库中提供映像。你可以找到最新版本的 Windows Server 和 Linux 操作系统的分发。一些映像还包含应用程序，如 SQL Server。
-	开源社区通过 [VM 仓库](http://vmdepot.msopentech.com/List/Index)提供映像。
-	你还可以通过捕获现有的 Azure 虚拟机用作映像或上载映像来在 Azure 中存储和使用自己的映像。

## 关于 VM 映像和 OS 映像

可以在 Azure 中使用两种类型的映像：*VM 映像*和 *OS 映像*。VM 映像包括在创建该映像时附加到虚拟机的操作系统和所有磁盘。这是较新类型的映像。在引入 VM 映像之前，Azure 中的映像只能包含一个通用操作系统，而不能包含任何附加磁盘。只包含一个通用操作系统的 VM 映像基本上与原始类型的映像（即，OS 映像）相同。

你可以基于 Azure 中的虚拟机或复制和上载的在别处运行的虚拟机，创建你自己的映像。如果要使用某个映像来创建多个虚拟机，则需要通过对其进行通用化来准备将其用作映像。若要创建 Windows Server 映像，请在上载 .vhd 文件之前在服务器上运行 Sysprep 命令对其进行通用化。有关 Sysprep 的详细信息，请参阅[如何使用 Sysprep：简介](https://technet.microsoft.com/zh-cn/library/bb457073.aspx)。若要创建 Linux 映像，根据软件分发，你需要运行一组特定于该分发的命令，并运行 Azure Linux 代理。

## 使用映像

可以使用适用于 Mac、Linux 和 Windows 的 Azure 命令行界面 (CLI) 或 Azure PowerShell 模块管理可供你的 Azure 订阅使用的映像。也可以使用 Azure 经典管理门户完成某些映像任务，但命令行为你提供更多选项。

有关使用这些工具进行部署的信息，请参阅[使用 PowerShell 和 Azure CLI 来浏览和选择 Azure 虚拟机映像](/documentation/articles/virtual-machines-linux-cli-ps-findimage/)。

在经典部署中使用这些工具的示例：

- 有关 CLI，请参阅[将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure 服务管理配合使用](/documentation/articles/virtual-machines-command-line-tools/)中的“用于管理 Azure 虚拟机映像的命令”
- 有关 Azure PowerShell，请参阅下面的命令列表。有关查找用于创建 VM 的映像的示例，请参阅[使用 Azure PowerShell 创建和预配置基于 Windows 的虚拟机](/documentation/articles/virtual-machines-windows-classic-create-powershell/)中的“步骤 3：确定 ImageFamily”

-	**获取所有映像**：`Get-AzureVMImage`返回当前订阅中可用的所有映像的列表：你的映像以及 Azure 或合作伙伴提供的映像。这意味着你可能会收到一个大列表。下一个示例演示如何获取较短的列表。
-	**获取映像系列**：`Get-AzureVMImage | select ImageFamily` 通过显示 **ImageFamily** 字符串属性获取映像系列的列表。
-	**获取特定系列中的所有映像**：`Get-AzureVMImage | Where-Object {$_.ImageFamily -eq $family}`
-	**查找 VM 映像**：`Get-AzureVMImage | where {(gm –InputObject $_ -Name DataDiskConfigurations) -ne $null} | Select -Property Label, ImageName` 这通过筛选 DataDiskConfiguration 属性来完成，它仅适用于 VM 映像。此示例还仅根据标签和映像名称来筛选输出。
-	**保存通用映像**：`Save-AzureVMImage –ServiceName "myServiceName" –Name "MyVMtoCapture" –OSState "Generalized" –ImageName "MyVmImage" –ImageLabel "This is my generalized image"`
-	**保存专用映像**：`Save-AzureVMImage –ServiceName "mySvc2" –Name "MyVMToCapture2" –ImageName "myFirstVMImageSP" –OSState "Specialized" -Verbose`

	>[Azure.Tip]如果要创建包含数据磁盘和操作系统磁盘的 VM 映像，OSState 参数是必需的。如果未使用此参数，该 cmdlet 将创建 OS 映像。该参数的值根据操作系统磁盘是否已准备好重用来指示映像是通用的还是专用的。

-	**删除映像**：`Remove-AzureVMImage –ImageName "MyOldVmImage"`

## 其他资源

[创建 Linux 虚拟机的不同方式](/documentation/articles/virtual-machines-linux-creation-choices/)

[创建 Windows 虚拟机的不同方式](/documentation/articles/virtual-machines-windows-creation-choices/)
