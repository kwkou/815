<properties
	pageTitle="自动缩放 Linux 虚拟机规模集 | Azure"
	description="使用 Azure CLI 为 Linux 虚拟机规模集设置自动缩放"
	services="virtual-machine-scale-sets"
	documentationCenter=""
	authors="davidmu1"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machine-scale-sets"
	ms.date="06/10/2016"
	wacn.date="08/29/2016"/>  


# 自动缩放虚拟机规模集中的 Linux 计算机

使用虚拟机缩放集可以轻松地将相同的虚拟机作为集来进行部署和管理。缩放集为超大规模应用程序提供高度可缩放且可自定义的计算层，并且它们支持 Windows 平台映像、Linux 平台映像、自定义映像和扩展。有关详细信息，请参阅[虚拟机规模集概述](/documentation/articles/virtual-machine-scale-sets-overview/)。

本教程演示如何使用 Ubuntu Linux 的最新版本创建 Linux 虚拟机的虚拟机规模集，并自动缩放集中的虚拟机。为此，将使用 Azure CLI 创建 Azure Resource Manager 模板并部署它。有关模板的详细信息，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates/)。若要了解有关规模集自动缩放的详细信息，请参阅[自动缩放和虚拟机规模集](/documentation/articles/virtual-machine-scale-sets-autoscale-overview/)。

在本教程中，你将部署以下资源和扩展：

- Microsoft.Storage/storageAccounts
- Microsoft.Network/virtualNetworks
- Microsoft.Network/publicIPAddresses
- Microsoft.Network/loadBalancers
- Microsoft.Network/networkInterfaces
- Microsoft.Compute/virtualMachines
- Microsoft.Compute/virtualMachineScaleSets
- Microsoft.Insights.VMDiagnosticsSettings
- Microsoft.Insights/autoscaleSettings

有关 Resource Manager 资源的详细信息，请参阅 [Azure Resource Manager 下的 Azure 计算、网络和存储提供程序](/documentation/articles/virtual-machines-linux-compare-deployment-models/)。

在开始执行本教程中的步骤之前，请[安装 Azure CLI](/documentation/articles/xplat-cli-install/)。

## 步骤 1：创建资源组和存储帐户

1. **登录到 Azure** - 在命令行接口（Bash、终端、命令提示符）中，通过键入 `azure config mode arm` 确保处于资源管理器模式，然后通过键入 `azure login` 来[使用工作或学校 ID 登录](/documentation/articles/xplat-cli-connect/#use-the-log-in-method)，并按照提示完成 Azure 帐户的交互式登录体验。

	> [AZURE.NOTE] 如果有工作或学校 ID，而且知道尚未启用双因素身份验证，则可以使用 `azure login -u` 以及工作或学校 ID，在没有交互式会话的情况下进行登录。如果没有工作或学校 ID，则可以[从 Microsoft 个人帐户创建工作或学校 ID](/documentation/articles/resource-group-create-work-id-from-personal/)。

2. **创建资源组** - 所有资源都必须部署到资源组。对于本教程，将资源组命名为 **vmsstest1**。

        azure group create vmsstestrg1 chinaeast

3. **将存储帐户部署到新的资源组** - 本教程使用多个存储帐户来实现虚拟机缩放集。创建名为 **vmsstestsa** 的存储帐户。让命令界面窗口保持打开状态以便完成本教程中的后续步骤。

        azure storage account create -g vmsstestrg1 -l chinaeast --kind Storage --sku-name LRS vmsstestsa

## 步骤 2：创建模板
借助 Azure Resource Manager 模板，你可以使用资源和关联部署参数的 JSON 描述来统一部署和管理 Azure 资源。

1. 在常用的编辑器中，创建文件 C:\\VMSSTemplate.json 并添加初始 JSON 结构，以支持模板。

        {
          "$schema":"http://schema.management.azure.com/schemas/2014-04-01-preview/VM.json",
          "contentVersion": "1.0.0.0",
          "parameters": {
          }
          "variables": {
          }
          "resources": [
          ]
        }

2. 参数并不总是必需的，但它们有助于简化模板管理。它们提供了为模板指定值的方法，描述了值的类型、所需的默认值，有时还描述参数的允许值。将这些参数添加到已添加到模板中的参数父元素下。

        "vmName": {
          "type": "string"
        },
        "vmSSName": {
          "type": "string"
        },
        "instanceCount": {
          "type": "string"
        },
        "adminUsername": {
          "type": "string"
        },
        "adminPassword": {
          "type": "securestring"
        },
        "resourcePrefix": {
          "type": "string"
        }
            
	- 用于访问规模集中虚拟机的单独虚拟机的名称。
	- 存储模板的存储帐户名称。
	- 最初在缩放集中创建的虚拟机实例数。
	- 虚拟机上的管理员帐户的名称和密码。
	- 在资源组中创建的资源的前缀。

3. 可以在模板中使用变量指定可能经常发生变化的值或需要通过组合参数值创建的值。将这些变量添加到已添加到模板中的变量父元素下。

        "dnsName1": "[concat(parameters('resourcePrefix'),'dn1')]",
        "dnsName2": "[concat(parameters('resourcePrefix'),'dn2')]",
        "vmSize": "Standard_A0",
        "imagePublisher": "Canonical",
        "imageOffer": "UbuntuServer",
        "imageVersion": "15.10",
        "addressPrefix": "10.0.0.0/16",
        "subnetName": "Subnet",
        "subnetPrefix": "10.0.0.0/24",
        "publicIP1": "[concat(parameters('resourcePrefix'),'ip1')]",
        "publicIP2": "[concat(parameters('resourcePrefix'),'ip2')]",
        "loadBalancerName": "[concat(parameters('resourcePrefix'),'lb1')]",
        "virtualNetworkName": "[concat(parameters('resourcePrefix'),'vn1')]",
        "nicName1": "[concat(parameters('resourcePrefix'),'nc1')]",
        "nicName2": "[concat(parameters('resourcePrefix'),'nc2')]",
        "vnetID": "[resourceId('Microsoft.Network/virtualNetworks',variables('virtualNetworkName'))]",
        "publicIPAddressID1": "[resourceId('Microsoft.Network/publicIPAddresses',variables('publicIP1'))]",
        "publicIPAddressID2": "[resourceId('Microsoft.Network/publicIPAddresses',variables('publicIP2'))]",
        "lbID": "[resourceId('Microsoft.Network/loadBalancers',variables('loadBalancerName'))]",
        "nicId": "[resourceId('Microsoft.Network/networkInterfaces',variables('nicName2'))]",
        "frontEndIPConfigID": "[concat(variables('lbID'),'/frontendIPConfigurations/loadBalancerFrontEnd')]",
        "storageAccountType": "Standard_LRS",
        "storageAccountSuffix": [ "a", "g", "m", "s", "y" ],
        "diagnosticsStorageAccountName": "[concat(parameters('resourcePrefix'), 'a')]",
        "accountid": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/', resourceGroup().name,'/providers/','Microsoft.Storage/storageAccounts/', variables('diagnosticsStorageAccountName'))]",
        "wadlogs": "<WadCfg><DiagnosticMonitorConfiguration>",
        "wadperfcounter": "<PerformanceCounters scheduledTransferPeriod=\"PT1M\"><PerformanceCounterConfiguration counterSpecifier=\"\\Processor\\PercentProcessorTime\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"CPU percentage guest OS\" locale=\"en-us\"/></PerformanceCounterConfiguration></PerformanceCounters>",
        "wadcfgxstart": "[concat(variables('wadlogs'),variables('wadperfcounter'),'<Metrics resourceId=\"')]",
        "wadmetricsresourceid": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/',resourceGroup().name ,'/providers/','Microsoft.Compute/virtualMachineScaleSets/',parameters('vmssName'))]",
        "wadcfgxend": "[concat('\"><MetricAggregation scheduledTransferPeriod=\"PT1H\"/><MetricAggregation scheduledTransferPeriod=\"PT1M\"/></Metrics></DiagnosticMonitorConfiguration></WadCfg>')]"
        
	- 网络接口所用的 DNS 名称。
	- 在缩放集中使用的虚拟机大小。有关虚拟机大小的详细信息，请参阅[虚拟机大小](/documentation/articles/virtual-machines-size-specs/)。
	- 用于定义将在缩放集中虚拟机上运行的操作系统的平台映像信息。有关选择映像的详细信息，请参阅[使用 Windows PowerShell 和 Azure CLI 来浏览和选择 Azure 虚拟机映像](/documentation/articles/resource-groups-vm-searching/)。
	- 虚拟网络和子网的 IP 地址名称和前缀。
	- 虚拟网络、负载均衡器和网络接口的名称和标识符。
	- 与缩放集中虚拟机关联的帐户的存储帐户名称。
	- 已安装在虚拟机上的诊断扩展的设置。有关诊断扩展的详细信息，请参阅[使用 Azure Resource Manager 模板创建具有监视和诊断功能的 Windows 虚拟机](/documentation/articles/virtual-machines-windows-extensions-diagnostics-template/)。

4. 将存储帐户资源添加到已添加到模板中的资源父元素下。此模板使用一个循环来创建建议的 5 个存储帐户，其中将存储操作系统磁盘和诊断数据。这组帐户可在一个缩放集中最多支持 100 个虚拟机，这是当前的最大值。每个存储帐户通过将变量中定义的字母指示符与模板的参数中提供的后缀组合来命名。

        {
          "type": "Microsoft.Storage/storageAccounts",
          "name": "[concat(parameters('resourcePrefix'), variables('storageAccountSuffix')[copyIndex()])]",
          "apiVersion": "2015-06-15",
          "copy": {
            "name": "storageLoop",
            "count": 5
          },
          "location": "[resourceGroup().location]",
          "properties": {
            "accountType": "[variables('storageAccountType')]"
          }
        },

5. 添加虚拟网络资源。有关详细信息，请参阅[网络资源提供程序](/documentation/articles/resource-groups-networking/)。

        {
          "apiVersion": "2015-06-15",
          "type": "Microsoft.Network/virtualNetworks",
          "name": "[variables('virtualNetworkName')]",
          "location": "[resourceGroup().location]",
          "properties": {
            "addressSpace": {
              "addressPrefixes": [
                "[variables('addressPrefix')]"
              ]
            },
            "subnets": [
              {
                "name": "[variables('subnetName')]",
                "properties": {
                  "addressPrefix": "[variables('subnetPrefix')]"
                }
              }
            ]
          }
        },

6. 添加负载均衡器和网络接口所使用的公共 IP 地址资源。

        {
          "apiVersion": "2016-03-30",
          "type": "Microsoft.Network/publicIPAddresses",
          "name": "[variables('publicIP1')]",
          "location": "[resourceGroup().location]",
          "properties": {
            "publicIPAllocationMethod": "Dynamic",
            "dnsSettings": {
              "domainNameLabel": "[variables('dnsName1')]"
            }
          }
        },
        {
          "apiVersion": "2016-03-30",
          "type": "Microsoft.Network/publicIPAddresses",
          "name": "[variables('publicIP2')]",
          "location": "[resourceGroup().location]",
          "properties": {
            "publicIPAllocationMethod": "Dynamic",
            "dnsSettings": {
              "domainNameLabel": "[variables('dnsName2')]"
            }
          }
        },

7. 添加缩放集使用的负载均衡器资源。有关详细信息，请参阅 [Azure Resource Manager 对负载均衡器的支持](/documentation/articles/load-balancer-arm/)。
        
        {
          "apiVersion": "2015-06-15",
          "name": "[variables('loadBalancerName')]",
          "type": "Microsoft.Network/loadBalancers",
          "location": "[resourceGroup().location]",
          "dependsOn": [
            "[concat('Microsoft.Network/publicIPAddresses/', variables('publicIP1'))]"
          ],
          "properties": {
            "frontendIPConfigurations": [
              {
                "name": "loadBalancerFrontEnd",
                "properties": {
                  "publicIPAddress": {
                    "id": "[variables('publicIPAddressID1')]"
                  }
                }
              }
            ],
            "backendAddressPools": [
              {
                "name": "bepool1"
              }
            ],
            "inboundNatPools": [
              {
                "name": "natpool1",
                "properties": {
                  "frontendIPConfiguration": {
                    "id": "[variables('frontEndIPConfigID')]"
                  },
                  "protocol": "tcp",
                  "frontendPortRangeStart": 50000,
                  "frontendPortRangeEnd": 50500,
                  "backendPort": 22
                }
              }
            ]
          }
        },

8. 添加单独虚拟机使用的网络接口资源。由于虚拟机规模集中的虚拟机不可使用公共 IP 地址直接访问，因此将在规模集所在的同一虚拟网络中创建单独虚拟机，并使用它来远程访问集中的虚拟机。

        {
          "apiVersion": "2016-03-30",
          "type": "Microsoft.Network/networkInterfaces",
          "name": "[variables('nicName1')]",
          "location": "[resourceGroup().location]",
          "dependsOn": [
            "[concat('Microsoft.Network/publicIPAddresses/', variables('publicIP2'))]",
            "[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]"
          ],
          "properties": {
            "ipConfigurations": [
              {
                "name": "ipconfig1",
                "properties": {
                  "privateIPAllocationMethod": "Dynamic",
                  "publicIPAddress": {
                    "id": "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIP2'))]"
                  },
                  "subnet": {
                    "id": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/',resourceGroup().name,'/providers/Microsoft.Network/virtualNetworks/',variables('virtualNetworkName'),'/subnets/',variables('subnetName'))]"
                  }
                }
              }
            ]
          }
        },

9. 在规模集所在的同一网络中添加单独的虚拟机。

        {
          "apiVersion": "2016-03-30",
          "type": "Microsoft.Compute/virtualMachines",
          "name": "[parameters('vmName')]",
          "location": "[resourceGroup().location]",
          "dependsOn": [
            "storageLoop",
            "[concat('Microsoft.Network/networkInterfaces/', variables('nicName1'))]"
          ],
          "properties": {
            "hardwareProfile": {
              "vmSize": "[variables('vmSize')]"
            },
            "osProfile": {
              "computername": "[parameters('vmName')]",
              "adminUsername": "[parameters('adminUsername')]",
              "adminPassword": "[parameters('adminPassword')]"
            },
            "storageProfile": {
              "imageReference": {
                "publisher": "[variables('imagePublisher')]",
                "offer": "[variables('imageOffer')]",
                "sku": "[variables('imageVersion')]",
                "version": "latest"
              },
              "osDisk": {
                "name": "osdisk1",
                "vhd": {
                  "uri":  "[concat('https://',parameters('resourcePrefix'),'sa.blob.core.chinacloudapi.cn/vhds/',parameters('resourcePrefix'),'osdisk1.vhd')]"
                },
                "caching": "ReadWrite",
                "createOption": "FromImage"
              }
            },
            "networkProfile": {
              "networkInterfaces": [
                {
                  "id": "[resourceId('Microsoft.Network/networkInterfaces',variables('nicName1'))]"
                }
              ]
            }
          }
        },

10.	添加虚拟机缩放集资源，并指定将在缩放集中的所有虚拟机上安装的诊断扩展。此资源的许多设置都与虚拟机资源相似。主要区别在于此资源中添加了指定应在规模集中初始化虚拟机的数量的容量元素和指定规模集中虚拟机更新方式的 upgradePolicy。在所有存储帐户都根据 dependsOn 元素的指定创建之前，不会创建规模集。

            {
              "type": "Microsoft.Compute/virtualMachineScaleSets",
              "apiVersion": "2016-03-30",
              "name": "[parameters('vmSSName')]",
              "location": "[resourceGroup().location]",
              "dependsOn": [
                "storageLoop",
                "[concat('Microsoft.Network/loadBalancers/', variables('loadBalancerName'))]",
                "[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]"
              ],
              "sku": {
                "name": "[variables('vmSize')]",
                "tier": "Standard",
                "capacity": "[parameters('instanceCount')]"
              }
              "properties": {
                "upgradePolicy": {
                  "mode": "Manual"
                },
                "virtualMachineProfile": {
                  "storageProfile": {
                    "osDisk": {
                      "vhdContainers": [
                        "[concat('https://', parameters('resourcePrefix'), 'a.blob.core.chinacloudapi.cn/vmss')]",
                        "[concat('https://', parameters('resourcePrefix'), 'g.blob.core.chinacloudapi.cn/vmss')]",
                        "[concat('https://', parameters('resourcePrefix'), 'm.blob.core.chinacloudapi.cn/vmss')]",
                        "[concat('https://', parameters('resourcePrefix'), 's.blob.core.chinacloudapi.cn/vmss')]",
                        "[concat('https://', parameters('resourcePrefix'), 'y.blob.core.chinacloudapi.cn/vmss')]"
                      ],
                      "name": "vmssosdisk",
                      "caching": "ReadOnly",
                      "createOption": "FromImage"
                    },
                    "imageReference": {
                      "publisher": "[variables('imagePublisher')]",
                      "offer": "[variables('imageOffer')]",
                      "sku": "[variables('imageVersion')]",
                      "version": "latest"
                    }
                  },
                  "osProfile": {
                    "computerNamePrefix": "[parameters('vmSSName')]",
                    "adminUsername": "[parameters('adminUsername')]",
                    "adminPassword": "[parameters('adminPassword')]"
                  },
                  "networkProfile": {
                    "networkInterfaceConfigurations": [
                      {
                        "name": "[variables('nicName2')]",
                        "properties": {
                          "primary": "true",
                          "ipConfigurations": [
                            {
                              "name": "ip1",
                              "properties": {
                                "subnet": {
                                  "id": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/',resourceGroup().name,'/providers/Microsoft.Network/virtualNetworks/',variables('virtualNetworkName'),'/subnets/',variables('subnetName'))]"
                                },
                                "loadBalancerBackendAddressPools": [
                                  {
                                    "id": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/',resourceGroup().name,'/providers/Microsoft.Network/loadBalancers/',variables('loadBalancerName'),'/backendAddressPools/bepool1')]"
                                  }
                                ],
                                "loadBalancerInboundNatPools": [
                                  {
                                    "id": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/',resourceGroup().name,'/providers/Microsoft.Network/loadBalancers/',variables('loadBalancerName'),'/inboundNatPools/natpool1')]"
                                  }
                                ]
                              }
                            }
                          ]
                        }
                      }
                    ]
                  },
                  "extensionProfile": {
                    "extensions": [
                      {
                        "name":"LinuxDiagnostic",
                        "properties": {
                          "publisher":"Microsoft.OSTCExtensions",
                          "type":"LinuxDiagnostic",
                          "typeHandlerVersion":"2.1",
                          "autoUpgradeMinorVersion":false,
                          "settings": {
                            "xmlCfg":"[base64(concat(variables('wadcfgxstart'),variables('wadmetricsresourceid'),variables('wadcfgxend')))]",
                            "storageAccount":"[variables('diagnosticsStorageAccountName')]"
                          },
                          "protectedSettings": {
                            "storageAccountName":"[variables('diagnosticsStorageAccountName')]",
                            "storageAccountKey":"[listkeys(variables('accountid'), '2015-06-15').key1]",
                            "storageAccountEndPoint":"https://core.chinacloudapi.cn"
                          }
                        }
                      }
                    ]
                  }
                }
              }
            },

11.	添加 autoscaleSettings 资源以定义缩放集如何根据集中虚拟机上的处理器使用率进行调整。
        
            {
              "type": "Microsoft.Insights/autoscaleSettings",
              "apiVersion": "2015-04-01",
              "name": "[concat(parameters('resourcePrefix'),'as1')]",
              "location": "[resourceGroup().location]",
              "dependsOn": [
                "[concat('Microsoft.Compute/virtualMachineScaleSets/',parameters('vmSSName'))]"
              ],
              "properties": {
                "enabled": true,
                "name": "[concat(parameters('resourcePrefix'),'as1')]",
                "profiles": [
                  {
                    "name": "Profile1",
                    "capacity": {
                      "minimum": "1",
                      "maximum": "10",
                      "default": "1"
                    },
                    "rules": [
                      {
                        "metricTrigger": {
                          "metricName": "\\Processor\\PercentProcessorTime",
                          "metricNamespace": "",
                          "metricResourceUri": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/',resourceGroup().name,'/providers/Microsoft.Compute/virtualMachineScaleSets/',parameters('vmSSName'))]",
                          "timeGrain": "PT1M",
                          "statistic": "Average",
                          "timeWindow": "PT5M",
                          "timeAggregation": "Average",
                          "operator": "GreaterThan",
                          "threshold": 50.0
                        },
                        "scaleAction": {
                          "direction": "Increase",
                          "type": "ChangeCount",
                          "value": "1",
                          "cooldown": "PT5M"
                        }
                      }
                    ]
                  }
                ],
                "targetResourceUri": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/', resourceGroup().name,'/providers/Microsoft.Compute/virtualMachineScaleSets/',parameters('vmSSName'))]"
              }
            }
    
    对于本教程，以下是重要值：
    
    - **metricName** - 这与我们在 wadperfcounter 变量中定义的性能计数器相同。使用该变量，诊断扩展可收集 **Processor\\PercentProcessorTime** 计数器。
    - **metricResourceUri** - 虚拟机规模集的资源标识符。
    - **timeGrain** - 这是收集的指标的粒度。在此模板中，它设置为 1 分钟。
    - **statistic** - 这可确定如何组合指标以调节自动缩放操作。可能的值包括：Average、Min、Max。在此模板中，我们要查找缩放集中虚拟机的平均总 CPU 使用率。
    - **timeWindow** - 这是收集实例数据的时间范围。它必须介于 5 分钟和 12 个小时之间。
    - **timeAggregation** - 这可确定应如何随着时间的推移组合收集的数据。默认值为 Average。可能的值包括：Average、Minimum、Maximum、Last、Total、Count。
    - **operator** - 这是用于比较指标数据和阈值的运算符。可能的值包括：Equals、NotEquals、GreaterThan、GreaterThanOrEqual、LessThan、LessThanOrEqual。
    - **threshold** - 这是将触发缩放操作的值。在此模板中，当缩放集中的虚拟机的平均 CPU 使用率超过 50% 时，会将虚拟机添加到缩放集。
    - **direction** - 这可确定在达到阈值时执行的操作。可能的值为 Increase 或 Decrease。在此模板中，如果在定义的时间窗口中阈值超过 50%，则将增加缩放集中的虚拟机数。
    - **type** - 这是应发生的操作类型，此属性必须设置为 ChangeCount。
    - **value** - 这是已在规模集中添加或删除的虚拟机数。此值必须大于或等于 1。默认值为 1。在此模板中，当达到该阈值时，缩放集中的虚拟机数将递增 1。
    - **cooldown** - 这是在上次缩放操作后，在下次操作发生之前要等待的时间。此值必须介于 1 分钟和 1 周之间。

12.	保存模板文件。

## 步骤 3：将模板上载到存储空间

只要你知道在步骤 1 中创建的存储帐户的帐户名称和主密钥，就可以从命令行界面上载模板。

1. 在命令行界面（Bash、终端、命令提示符）中，运行以下命令以设置访问存储帐户所需的环境变量：

		export AZURE_STORAGE_ACCOUNT={account_name}
		export AZURE_STORAGE_ACCESS_KEY={key}

	可以通过在 Azure 门户预览中查看存储帐户资源时单击密钥图标来获取密钥。使用 Windows 命令提示符时，请键入 **set** 而不是 export。

2. 创建用于存储模板的容器：

		azure storage container create -p Blob templates

3. 将模板文件上载到新容器。

		azure storage blob upload VMSSTemplate.json templates VMSSTemplate.json

## 步骤 4：部署模板

创建模板后，便可以开始部署资源了。使用以下命令启动相关进程：

	azure group deployment create --template-uri https://vmsstestsa.blob.core.chinacloudapi.cn/templates/VMSSTemplate.json vmsstestrg1 vmsstestdp1

当你按 Enter 时，系统将提示你为所指定的变量提供值。提供以下值：

	vmName: vmsstestvm1
	vmSSName: vmsstest1
	instanceCount: 5
	adminUserName: vmadmin1
	adminPassword: VMpass1
	resourcePrefix: vmsstest

成功部署所有资源应大约需要 15 分钟。

>[AZURE.NOTE]你还可以利用门户的功能来部署资源。为此，请使用以下链接：
https://portal.azure.cn/#create/Microsoft.Template/uri/<link to VM Scale Set JSON template>

## 步骤 5：监视资源

你可以使用以下方法获取有关虚拟机缩放集的一些信息：

 - Azure 门户预览 - 当前使用门户可以获取有限数量的信息。
 - Azure CLI - 使用此命令可获取一些信息：

		azure resource show -n vmsstest1 -r Microsoft.Compute/virtualMachineScaleSets -o 2015-06-15 -g vmsstestrg1

 - 就像连接任何其他虚拟机一样连接到 jumpbox 虚拟机，然后可以远程访问缩放集中的虚拟机，以监视单个进程。

>[AZURE.NOTE]用于获取有关规模集的信息的完整 REST API 可在[虚拟机规模集](https://msdn.microsoft.com/zh-cn/library/mt589023.aspx)中找到。

## 步骤 6：删除资源

由于你需要为 Azure 中使用的资源付费，因此，删除不再需要的资源总是一种良好的做法。你不需要从资源组中分别删除每个资源，而是可以删除资源组，这样就会自动删除它包含的所有资源。

		azure group delete vmsstestrg1

## 后续步骤

- 在 [Azure Insights 跨平台 CLI 快速入门示例](/documentation/articles/insights-cli-samples/)中查找 Azure Insights 监视功能的示例
- 在[使用自动缩放操作在 Azure Insights 中发送电子邮件和 Webhook 警报通知](/documentation/articles/insights-autoscale-to-webhook-email/)和[使用审核日志在 Azure Insights 中发送电子邮件和 Webhook 警报通知](/documentation/articles/insights-auditlog-to-webhook-email/)中了解有关通知功能的信息
- 请查看[自动缩放运行 Ubuntu/Apache/PHP 应用的 VM 规模集](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-lapstack-autoscale)模板，该模板设置 LAMP 堆栈以执行虚拟机规模集的自动缩放功能。

<!---HONumber=Mooncake_0822_2016-->