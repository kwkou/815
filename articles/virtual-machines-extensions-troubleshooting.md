<properties
   pageTitle="对 Azure VM 扩展故障进行故障排除 | Windows Azure"
   description="了解如何对 Azure VM 扩展故障进行故障排除"
   services="virtual-machines"
   documentationCenter=""
   authors="kundanap"
   manager="timlt"
   editor=""
   tags="top-support-issue,azure-resource-manager"/>

<tags
   ms.service="virtual-machines"
   ms.date="09/01/2015"
   wacn.date="11/12/2015"/>

# 对 Azure VM 扩展故障进行故障排除。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-rm-include.md)]经典部署模型。


## Azure 资源管理器模板概述。

Azure 资源管理器模板可让你通过定义资源之间的依赖关系，使用 JSON 语言以声明方式指定 Azure IaaS 基础结构。


若要了解有关使用扩展创作模板的详细信息，请单击文章[创作扩展模板](/documentation/articles/virtual-machines-extensions-authoring-templates)。

在本文中，我们将了解如何对一些常见的 VM 扩展故障进行故障排除。

## 查看扩展状态：
Azure 资源管理器模板可从 Azure Powershell 或 Azure CLI 执行。一旦执行该模板，就可以从 Azure 资源管理器或命令行工具查看扩展状态。下面是一些示例：

Azure CLI：

      azure vm get-instance-view

Azure Powershell：

      Get-AzureVM -ResourceGroupName $RGName -Name $vmName -Status

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

#### 从 Azure CLI 删除该扩展：

      azure vm extension set --resource-group "KPRG1" --vm-name "kundanapdemo" --publisher-name "Microsoft.Compute.CustomScriptExtension" --name "myCustomScriptExtension" --version 1.4 --uninstall

其中“publsher-name”对应“azure vm get-instance-view”输出中的扩展类型，名称是模板中扩展资源的名称

#### 从 Azure Powershell 删除该扩展：

    Remove-AzureVMExtension -ResourceGroupName $RGName -VMName $vmName -Name "myCustomScriptExtension"

删除该扩展后，可以重新执行模板在 VM 上运行脚本。

<!---HONumber=Mooncake_1207_2015-->