<properties
	pageTitle="基于 Azure VM 创建 VM 映像 | Azure"
	description="了解如何基于 Resource Manager 部署模型中创建的现有 Azure VM 创建通用化 VM 映像"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>  


<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="10/10/2016"
	wacn.date="01/05/2017"
	ms.author="cynthn"/>  



# 下载 VM 模板

使用门户或 PowerShell 在 Azure 中创建 VM 时，系统会自动创建一个 Resource Manager 模板。可以使用此模板快速复制部署。该模板包含有关资源组中所有资源的信息。对于虚拟机而言，这意味着该模板包含为于在该资源组中支持该 VM 而创建的所有资源，包括网络资源。

## 使用门户下载模板

1. 登录到 [Azure 门户预览](https://portal.azure.cn/)。
2. 在中心菜单中，选择“虚拟机”。
3. 从列表中选择虚拟机。
5. 选择“自动化脚本”。
6. 选择“下载”，将 .zip 文件保存到本地计算机。
7. 打开该 .zip 文件，将文件解压缩到某个文件夹。该 .zip 文件包含：
	
	- deploy.ps1
	- deploy.sh
	- deployer.rb
	- DeploymentHelper.cs
	- parameters.json
	- template.json

template.json 文件是模板。
	
## 使用 PowerShell 下载模板

也可以使用 [Export-AzureRMResourceGroup](https://msdn.microsoft.com/zh-cn/library/mt715427.aspx) cmdlet 下载 .json 模板文件。可以使用 `-path` 参数提供 .json 文件的文件名和路径。本示例演示如何将名为 **myResourceGroup** 的资源组的模板下载到本地计算机上的 **C:\\users\\public\\downloads** 文件夹。

	Export-AzureRmResourceGroup -ResourceGroupName "myResourceGroup" -Path "C:\users\public\downloads"

## 后续步骤

若要详细了解如何使用模板部署资源，请参阅 [Resource Manager template walkthrough](/documentation/articles/resource-manager-template-walkthrough/)（Resource Manager 模板演练）。

<!---HONumber=Mooncake_1114_2016-->