<properties
	pageTitle="关于 Windows 虚拟机的映像 | Azure"
	description="了解如何对 Azure 中的 Windows 虚拟机使用映像。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="07/21/2016"
	wacn.date="09/12/2016"
	ms.author="cynthn"/>

# 关于 Windows 虚拟机的映像

[AZURE.INCLUDE [了解部署模型](../../includes/learn-about-deployment-models-classic-include.md)]

[AZURE.INCLUDE [virtual-machines-common-classic-about-images](../../includes/virtual-machines-common-classic-about-images.md)]

更多关于在资源管理器模型查找镜像，请点击[这里](/documentation/articles/virtual-machines-windows-cli-ps-findimage/)。

## 使用映像

可以使用 Azure PowerShell 模块管理 Azure 订阅可用的映像。也可以使用 Azure 经典管理门户完成某些映像任务，但命令行提供更多选项。


-	**获取所有映像**：`Get-AzureVMImage`返回当前订阅中可用的所有映像的列表：映像以及 Azure 或合作伙伴提供的映像。这意味着你可能会收到一个大列表。下一个示例演示如何获取较短的列表。
-	**获取映像系列**：`Get-AzureVMImage | select ImageFamily` 通过显示 **ImageFamily** 字符串属性获取映像系列的列表。
-	**获取特定系列中的所有映像**：`Get-AzureVMImage | Where-Object {$_.ImageFamily -eq $family}`
-	**查找 VM 映像**：`Get-AzureVMImage | where {(gm -InputObject $_ -Name DataDiskConfigurations) -ne $null} | Select -Property Label, ImageName` 这通过筛选 DataDiskConfiguration 属性来完成，它仅适用于 VM 映像。此示例还仅根据标签和映像名称来筛选输出。
-	**保存通用映像**：`Save-AzureVMImage -ServiceName "myServiceName" -Name "MyVMtoCapture" -OSState "Generalized" -ImageName "MyVmImage" -ImageLabel "This is my generalized image"`
-	**保存专用映像**：`Save-AzureVMImage -ServiceName "mySvc2" -Name "MyVMToCapture2" -ImageName "myFirstVMImageSP" -OSState "Specialized" -Verbose`

	>[Azure.Tip] 如果要创建包含数据磁盘和操作系统磁盘的 VM 映像，OSState 参数是必需的。如果未使用此参数，该 cmdlet 将创建 OS 映像。该参数的值根据操作系统磁盘是否已准备好重用来指示映像是通用的还是专用的。

-	**删除映像**：`Remove-AzureVMImage -ImageName "MyOldVmImage"`


## 后续步骤

还可以[使用经典管理门户创建 Windows 计算机](/documentation/articles/virtual-machines-windows-classic-tutorial/)

<!---HONumber=Mooncake_0905_2016-->