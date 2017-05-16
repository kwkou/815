<properties
    pageTitle="下载 Azure VM 模板 | Azure"
    description="下载 VM 模板，帮助在 Resource Manager 部署模型中自动部署"
    services="virtual-machines-windows"
    documentationcenter=""
    author="cynthn"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="51ef4f51-0942-4249-afea-4a3f87ce1ff8"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/22/2017"
    wacn.date="05/15/2017"
    ms.author="cynthn"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="56459fcef69795ca008420c3b18e56f7c071c6d9"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="download-the-template-for-a-vm"></a>下载 VM 模板
使用门户或 PowerShell 在 Azure 中创建 VM 时，系统会自动创建一个 Resource Manager 模板。 可以使用此模板快速复制部署。 该模板包含有关资源组中所有资源的信息。 对于虚拟机而言，这意味着该模板包含为在该资源组中支持该 VM 而创建的所有资源，包括网络资源。

## <a name="download-the-template-using-the-portal"></a>使用门户下载模板
1. 登录到 [Azure 门户预览](https://portal.azure.cn/)。
2. 在中心菜单中，选择“虚拟机”。
3. 从列表中选择虚拟机。
4. 选择“自动化脚本”。
5. 选择“下载”，将 .zip 文件保存到本地计算机。
6. 打开 .zip 文件，将文件解压缩到某个文件夹。 该 .zip 文件包含：

    * deploy.ps1
    * deploy.sh 
    * deployer.rb
    * DeploymentHelper.cs
    * parameters.json
    * template.json

template.json 文件是模板。

## <a name="download-the-template-using-powershell"></a>使用 PowerShell 下载模板
也可以使用 [Export-AzureRMResourceGroup](https://msdn.microsoft.com/zh-cn/library/mt715427.aspx) cmdlet 下载 .json 模板文件。 可以使用 `-path` 参数提供 .json 文件的文件名和路径。 本示例演示如何将名为 **myResourceGroup** 的资源组的模板下载到本地计算机上的 **C:\users\public\downloads** 文件夹。

        Export-AzureRmResourceGroup -ResourceGroupName "myResourceGroup" -Path "C:\users\public\downloads"

## <a name="next-steps"></a>后续步骤
若要详细了解如何使用模板部署资源，请参阅 [Resource Manager 模板演练](/documentation/articles/resource-manager-create-first-template/)。

<!--Update_Description: wording update-->