<!-- ARM: tested -->

<properties
   pageTitle="Linux VM 扩展的示例配置 | Azure"
   description="使用扩展为 Linux VM 创作模板的示例配置"
   services="virtual-machines-linux"
   documentationCenter=""
   authors="kundanap"
   manager="timlt"
   editor=""
   tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="03/29/2016"
	wacn.date="06/06/2016"/>

# Linux VM 扩展配置示例

> [AZURE.SELECTOR]
- [PowerShell - 模板](/documentation/articles/virtual-machines-windows-extensions-configuration-samples/)
- [CLI - 模板](/documentation/articles/virtual-machines-linux-extensions-configuration-samples/)

<br>

>[AZURE.NOTE] 你从 GitHub 仓库 "azure-quickstart-templates" 中下载的模板，需要做一些修改才能适用于 Azure 中国云环境。例如，替换一些终结点 -- "blob.core.windows.net" 替换成 "blob.core.chinacloudapi.cn"，"cloudapp.azure.com" 替换成 "chinacloudapp.cn"；改掉一些不支持的 VM 映像，还有，改掉一些不支持的 VM 大小。

本文提供为 Linux VM 配置 Azure VM 扩展的示例配置。

> [AZURE.NOTE] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是经典部署模型。


若要了解有关这些扩展的详细信息，请单击此处：[Azure VM 扩展概述。](/documentation/articles/virtual-machines-windows-extensions-features/)

若要了解有关创作扩展模板的详细信息，请单击此处：[创作扩展模板。](/documentation/articles/virtual-machines-windows-extensions-authoring-templates/)

本文列出了一些 Linux 扩展的预期配置值。

## VM 扩展的示例模板代码段。
部署扩展的模板代码段如下所示：

      {
	      "type": "Microsoft.Compute/virtualMachines/extensions",
	      "name": "MyExtension",
	      "apiVersion": "2015-05-01-preview",
	      "location": "[parameters('location')]",
	      "dependsOn": ["[concat('Microsoft.Compute/virtualMachines/',parameters('vmName'))]"],
	      "properties":
	      {
		      "publisher": "Publisher Namespace",
		      "type": "extension Name",
		      "typeHandlerVersion": "extension version",
              "autoUpgradeMinorVersion":true,
		      "settings": {
		      	// Extension specific configuration goes in here.
		      }
	      }
      }

### CloudLink SecureVM 代理
          {
            "publisher": "CloudLinkEMC.SecureVM",
            "type": "CloudLinkSecureVMLinuxAgent",
            "typeHandlerVersion": "4.0",
            "settings": {
              "CloudLinkCenter" : "specify valid IP/FQDN to CloudLinkCenter"
            }
          }

### 适用于 Linux 的 CustomScript 扩展。
    {
        "publisher": " Microsoft.OSTCExtensions",
        "type": "CustomScriptForLinux",
        "typeHandlerVersion": "1.3",
        "settings": {
            "fileUris": [
                "http: //Yourstorageaccount.blob.core.chinacloudapi.cn/customscriptfiles/start.ps1"
            ],
            "commandToExecute": "powershell.exe-ExecutionPolicyUnrestricted-Filestart.ps1"
        }
    }


### Datadog 代理
        {
          "publisher": "Datadog.Agent",
          "type": "DatadogLinuxAgent",
          "typeHandlerVersion": "0.4",
          "settings": {
            "api_key" : "API Key from https://app.datadoghq.com/account/settings#api"
          }
        }

### Chef 代理
        {
          "publisher": "Chef.Bootstrap.WindowsAzure",
          "type": "CentosChefClient|LinuxChefClient",
          "typeHandlerVersion": "1210.12",
          "settings": {
            "validation_key" : " Validation key",
            "client_rb" : "client_rb file",
            "runlist" : "Optional runlist"
          }
        }

### VM 访问扩展（密码重置）
有关更新的架构，请参阅 [VMAccessForLinux 文档](https://github.com/Azure/azure-linux-extensions/tree/master/VMAccess)

        {
          "publisher": "Microsoft.OSTCExtensions",
          "type": "VMAccessForLinux",
          "typeHandlerVersion": "1.2",
          "protectedSettings": {
            "username": "(required, string) the name of the user",
            "password": "(optional, string) the password of the user",
            "reset_ssh": "(optional, boolean) whether or not reset the ssh",
            "ssh_key": "(optional, string) the public key of the user, base64 encoded pem",
            "remove_user": "(optional, string) the user name to remove"
          }
        }

### OS 修补
有关更新的架构，请参阅 [OSPatching 文档](https://github.com/Azure/azure-linux-extensions/tree/master/OSPatching)

        {
        "publisher": "Microsoft.OSTCExtensions",
        "type": "OSPatchingForLinux",
        "typeHandlerVersion": "2.9",
        "Settings": {
          "disabled": false,
          "stop": false,
          "rebootAfterPatch": "RebootIfNeed|Required|NotRequired|Auto",
          "category": "Important|ImportantAndRecommended",
          "installDuration": "<hr:min>",
          "oneoff": false,
          "intervalOfWeeks": "<number>",
          "dayOfWeek": "Sunday|Monday|Tuesday|Wednesday|Thursday|Friday|Saturday|Everyday",
          "startTime": "<hr:min>",
          "vmStatusTest": {
              "local": false,
              "idleTestScript": "<path_to_idletestscript>",
              "healthyTestScript": "<path_to_healthytestscript>"
          }
        }
        }

### Docker 扩展
有关更新的架构，请参阅 [Docker 扩展文档](https://github.com/Azure/azure-docker-extension/blob/master/README.md#1-configuration-schema)

        {
          "publisher": "Microsoft.Azure.Extensions ",
          "type": "DockerExtension ",
          "typeHandlerVersion": "1.0",
          "Settings": {
            "docker":{
                "port": "2376",
                "options": ["-D", "--dns=8.8.8.8"]
            },
            "compose": {
                "cache" : {
                    "image" : "memcached",
                    "ports" : ["11211:11211"]
                },
                "blog": {
                    "image": "ghost",
                    "ports": ["80:2368"]
                }
            }
            }
        }

        ### Linux Diagnostics Extension
        {
        "storageAccountName": "storage account to receive data",
        "storageAccountKey": "key of the account",
        "perfCfg": [
        {
            "query": "SELECT PercentAvailableMemory, AvailableMemory, UsedMemory ,PercentUsedSwap FROM SCX_MemoryStatisticalInformation",
            "table": "LinuxMemory"
        }
        ],
        "fileCfg": [
        {
            "file": "/var/log/mysql.err",
            "table": "mysqlerr"
        }
        ]
        }

在上述示例中，请将版本号替换为最新版本号。

以下是使用扩展创建 Linux VM 的完整 VM 模板：

[Linux VM 上的自定义脚本扩展](https://github.com/Azure/azure-quickstart-templates/blob/b1908e74259da56a92800cace97350af1f1fc32b/mongodb-on-ubuntu/azuredeploy.json/)

<!---HONumber=Mooncake_0503_2016-->