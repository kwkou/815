<!-- ARM: tested -->

<properties
   pageTitle="对 Windows VM 扩展故障进行故障排除 | Microsoft Azure"
   description="了解如何对 Azure Windows VM 扩展故障进行故障排除"
   services="virtual-machines-windows"
   documentationCenter=""
   authors="kundanap"
   manager="timlt"
   editor=""
   tags="top-support-issue,azure-resource-manager"/>

<tags
   ms.service="virtual-machines-windows"
   ms.date="03/29/2016"
   wacn.date="06/29/2016"/>

# 对 Azure Windows VM 扩展故障进行故障排除。

[AZURE.INCLUDE [arm-api-version-powershell](../includes/arm-api-version-powershell.md)]

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代经典部署模型。

[AZURE.INCLUDE [virtual-machines-common-extensions-troubleshoot](../includes/virtual-machines-common-extensions-troubleshoot.md)]

## 查看扩展状态：
Azure 资源管理器模板可从 Azure PowerShell 或 Azure CLI 执行。一旦执行该模板，就可以从 Azure 资源管理器或命令行工具查看扩展状态。下面是一些示例：

Azure PowerShell：

      Get-AzureRmVM -ResourceGroupName $RGName -Name $vmName -Status

下面是示例输出：

      Extensions:  {
      "ExtensionType": "Microsoft.Compute.CustomScriptExtension",
      "Name": "myCustomScriptExtension",
      "SubStatuses": [
        {
          "Code": "ComponentStatus/StdOut/succeeded",
          "DisplayStatus": "Provisioning succeeded",
          "Level": "Info",
          "Message": "    Directory: C:\\temp\\n\\n\\nMode                LastWriteTime     Length Name
              \\n----                -------------     ------ ----                              \\n-a---          9/1/2015   2:03 AM         11
              test.txt                          \\n\\n",
                      "Time": null
          },
        {
          "Code": "ComponentStatus/StdErr/succeeded",
          "DisplayStatus": "Provisioning succeeded",
          "Level": "Info",
          "Message": "",
          "Time": null
        }
    }
  ]

## 对扩展故障进行故障排除：

### 在 VM 上重新运行扩展：

如果你使用自定义脚本扩展在 VM 上运行脚本，有时可能会遇到错误，即 VM 已成功创建但脚本却运行失败。在这些情况下，从此错误中恢复的建议方法是删除该扩展并再次重新运行该模板。注意：未来此功能将得到增强，不再需要卸载该扩展。

#### 从 Azure Powershell 删除该扩展

    Remove-AzureVMExtension -ResourceGroupName $RGName -VMName $vmName -Name "myCustomScriptExtension"

删除该扩展后，可以重新执行模板在 VM 上运行脚本。

<!---HONumber=Mooncake_1207_2015-->