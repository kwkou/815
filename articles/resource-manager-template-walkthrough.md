<properties
   pageTitle="Resource Manager 模板演练 | Azure"
   description="用于预配基本 Azure IaaS 体系结构的 Resource Manager 模板的分步演练。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="navalev"
   manager=""
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="05/04/2016"
   wacn.date="06/27/2016"/>
   
# Resource Manager 模板演练

创建模板时的首要问题之一是“如何开始？”。用户可以从空白模板开始，按照[创作模板](/documentation/articles/resource-group-authoring-templates/#template-format)一文中所述的基本结构操作，并添加资源和相应的参数和变量。一个不错的开始替代方法是，浏览[快速入门库](https://github.com/Azure/azure-quickstart-templates)并寻找与你要创建的模板类似的方案。可以合并多个模板或编辑现有模板以适合你自己的特定方案。

让我们来看一下通用基础结构：

* 使用同一存储帐户的两个虚拟机具有相同的可用性集，并且处于虚拟网络的同一子网中。
* 每个虚拟机有单独的 NIC 和虚拟机 IP 地址。
* 端口 80 上有一个带有负载平衡规则的负载平衡器

![体系结构](./media/resource-group-overview/arm_arch.png)

本主题将指导你完成为该基础结构创建 Resource Manager 模板的步骤。你创建的最后一个模板基于名为 [2 VMs in a Load Balancer and load balancing rules（负载平衡器中的 2 个 VM 和负载平衡规则）](https://github.com/Azure/azure-quickstart-templates/tree/master/201-2-vms-loadbalancer-lbrules)的快速入门模板。

但是，要立即构建所有这些还有很多工作要做，因此，让我们首先创建一个存储帐户并将其部署。你掌握如何创建存储帐户后，可添加其他资源并重新部署模板以完成基础结构。

>[AZURE.NOTE] 创建模板时，你可以使用任何类型的编辑器。Visual Studio 提供了可简化模板开发的工具，但你不需要 Visual Studio 即可完成本教程。<!-- 有关使用 Visual Studio 创建 Web 应用和 SQL 数据库部署的教程，请参阅 [Creating and deploying Azure resource groups through Visual Studio（通过 Visual Studio 创建和部署 Azure 资源组）](/documentation/articles/vs-azure-tools-resource-groups-deployment-projects-create-deploy/)。 -->

## 创建 Resource Manager 模板

该模板是一个 JSON 文件，用于定义将部署的所有资源。此外，它还允许你定义在部署期间指定的参数、从其他值和表达式构造的变量，以及部署的输出。

让我们从最简单的模板开始：

    {
      "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {  },
      "variables": {  },
      "resources": [  ],
      "outputs": {  }
    }


将此文件另存为 **azuredeploy.json**（请注意，模板可以具有任何所需的名称，只是它必须是 json 文件）。

## 创建存储帐户
在 **resources** 节内，添加用于定义存储帐户的对象，如下所示。


	"resources": [
	  {
	    "type": "Microsoft.Storage/storageAccounts",
	    "name": "[parameters('storageAccountName')]",
	    "apiVersion": "2015-06-15",
	    "location": "[resourceGroup().location]",
	    "properties": {
	      "accountType": "Standard_LRS"
	    }
	  }
	]

你可能想知道这些属性和值来自何处。**type**、**name**、**apiVersion** 和 **location** 属性都是可供所有资源类型使用的标准元素。你可以在 [Resources](/documentation/articles/resource-group-authoring-templates/#resources) 中了解常见元素。**name** 设置为在部署过程中传递的参数值，**location** 作为资源组使用的位置。我们将在下面部分探讨如何确定 **type** 和 **apiVersion**。

**properties** 节包含特定资源类型特有的所有属性。在此节中指定的值完全匹配 REST API 中可供创建该资源类型的 PUT 操作。创建存储帐户时，必须提供 **accountType**。请注意，在[用于创建存储帐户的 REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt163564.aspx) 中，REST 操作的 properties 节也包含 **accountType** 属性，并阐述了允许的值。在本示例中，帐户类型设置为 **Standard\_LRS**，但你可以指定其他值或允许用户以参数形式传入帐户类型。

现在，让我们跳回到 **parameters** 节，并了解如何定义存储帐户的名称。你可以在 [Parameters](/documentation/articles/resource-group-authoring-templates/#parameters) 中了解有关如何使用参数的更多信息。


	"parameters" : {
		"storageAccountName": {
	      "type": "string",
	      "metadata": {
	        "description": "Storage Account Name"
	      }
	    }
	}

此处你定义了一个字符串类型的参数，用于保存存储帐户的名称。在部署模板过程中将提供此参数的值。

## 部署模板
我们已有用于创建新存储帐户的完整模板。你应该还记得，该模板保存在 **azuredeploy.json** 文件中：


	{
	  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
	  "contentVersion": "1.0.0.0",
	  "parameters" : {
		"storageAccountName": {
	      "type": "string",
	      "metadata": {
	        "description": "Storage Account Name"
	      }
	    }
	  },  
	  "resources": [
	    {
	      "type": "Microsoft.Storage/storageAccounts",
	      "name": "[parameters('storageAccountName')]",
	      "apiVersion": "2015-06-15",
	      "location": "[resourceGroup().location]",
	      "properties": {
	        "accountType": "Standard_LRS"
	      }
	    }
	  ]
	}


有很多方法可以部署模板，如[资源部署](/documentation/articles/resource-group-template-deploy/)一文所示。若要使用 Azure PowerShell 部署模板，请使用：


	# create a new resource group
	New-AzureRmResourceGroup -Name ExampleResourceGroup -Location "China East"

	# deploy the template to the resource group
	New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup -TemplateFile azuredeploy.json


或者，若要使用 Azure CLI 部署模板，请使用：


	azure group create -n ExampleResourceGroup -l "China East"

	azure group deployment create -f azuredeploy.json -g ExampleResourceGroup -n ExampleDeployment


现在，你是存储帐户的骄傲拥有者！

接下来的步骤将是添加部署本教程开始所述体系结构所需的所有资源。你将在你已处理的同一模板中添加这些资源。

## 可用性集
在存储帐户的定义后面，添加虚拟机的可用性集。在本例中，无需其他属性，因此其定义相当简单。如果想要定义更新域计数和容错域计数值，请参阅 [REST API for creating an Availability Set](https://msdn.microsoft.com/zh-cn/library/azure/mt163607.aspx)（用于创建可用性集的 REST API）中的完整 properties 节。


    {
       "type": "Microsoft.Compute/availabilitySets",
       "name": "[variables('availabilitySetName')]",
       "apiVersion": "2015-06-15",
       "location": "[resourceGroup().location]",
       "properties": {}
    }


请注意，**name** 已设置为某个变量的值。在此模板中，在几个不同的位置需要可用性集的名称。可以通过定义该值一次并在多个位置使用它，来更轻松地维护模板。

为 **type** 指定的值同时包含资源提供程序和资源类型。在可用性集中，资源提供程序为 **Microsoft.Compute**，资源类型为 **availabilitySets**。可通过运行以下 PowerShell 命令获取可用的资源提供程序列表：

    Get-AzureRmResourceProvider -ListAvailable

或者，如果你使用 Azure CLI，可以运行以下命令：

    azure provider list

本主题假设你要创建包含存储帐户、虚拟机和虚拟网络的模板，因此可以使用：

- Microsoft.Storage
- Microsoft.Compute
- Microsoft.Network

若要查看特定提供程序的资源类型，请运行以下 PowerShell 命令：

    (Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Compute).ResourceTypes

或者，在 Azure CLI 中，以下命令将以 JSON 格式返回可用的类型，并将它保存到文件中。

    azure provider show Microsoft.Compute --json > c:\temp.json

你应会看到 **availabilitySets** 成为 **Microsoft.Compute** 中的类型之一。类型的完整名称为 **Microsoft.Compute/availabilitySets**。你可以在模板中确定任何资源的资源类型名称。

## 公共 IP
定义公共 IP 地址。再次查看[公共 IP 地址的 REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt163590.aspx) 中要设置的属性。


    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/publicIPAddresses",
      "name": "[parameters('publicIPAddressName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "publicIPAllocationMethod": "Dynamic",
        "dnsSettings": {
          "domainNameLabel": "[parameters('dnsNameforLBIP')]"
        }
      }
    }


分配方法设置为 **Dynamic**，但你可以将它设置为所需的值或将它设置为接受参数值。你已让模板的用户能够传入域名标签的值。

现在，让我们看看如何确定 **apiVersion**。指定的值完全匹配创建资源时所要使用的 REST API 版本。因此，你可以查看该资源类型的 REST API 文档。或者，可以对特定类型运行以下 PowerShell 命令。

    ((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Network).ResourceTypes | Where-Object ResourceTypeName -eq publicIPAddresses).ApiVersions

这将返回以下值：

    2015-06-15
    2015-05-01-preview
    2014-12-01-preview

若要查看使用 Azure CLI 的 API 版本，请运行前面所示的同一 **azure provider show** 命令。

创建新模板时，请选择最新的 API 版本。

## 虚拟网络和子网
创建具有一个子网的虚拟网络。有关要设置的所有属性，请查看 [REST API for virtual networks（虚拟网络的 REST API）](https://msdn.microsoft.com/zh-cn/library/azure/mt163661.aspx)。


    {
       "apiVersion": "2015-06-15",
       "type": "Microsoft.Network/virtualNetworks",
       "name": "[parameters('vnetName')]",
       "location": "[resourceGroup().location]",
       "properties": {
         "addressSpace": {
           "addressPrefixes": [
             "10.0.0.0/16"
           ]
         },
         "subnets": [
           {
             "name": "[variables('subnetName')]",
             "properties": {
               "addressPrefix": "10.0.0.0/24"
             }
           }
         ]
       }
    }


## 负载平衡器
现在创建一个面向外部的负载平衡器。由于此负载平衡器使用公共 IP 地址，因此你必须在 **dependsOn** 节中声明公共 IP 地址的依赖性。这意味着在公共 IP 地址完成部署后，才部署负载平衡器。如果不定义此依赖性，你将收到错误，因为 Resource Manager 尝试以并行方式部署资源，并尝试将负载平衡器设置为尚不存在的公共 IP 地址。

你还要在此资源定义中创建后端地址池、几个用于通过 RDP 连接到 VM 的入站 NAT 规则，以及端口 80 上包含 TCP 探测的负载平衡规则。有关所有属性，请查看 [REST API for load balancer（负载平衡器的 REST API）](https://msdn.microsoft.com/zh-cn/library/azure/mt163574.aspx)。

    {
       "apiVersion": "2015-06-15",
       "name": "[parameters('lbName')]",
       "type": "Microsoft.Network/loadBalancers",
       "location": "[resourceGroup().location]",
       "dependsOn": [
         "[concat('Microsoft.Network/publicIPAddresses/', parameters('publicIPAddressName'))]"
       ],
       "properties": {
         "frontendIPConfigurations": [
           {
             "name": "LoadBalancerFrontEnd",
             "properties": {
               "publicIPAddress": {
                 "id": "[variables('publicIPAddressID')]"
               }
             }
           }
         ],
         "backendAddressPools": [
           {
             "name": "BackendPool1"
           }
         ],
         "inboundNatRules": [
           {
             "name": "RDP-VM0",
             "properties": {
               "frontendIPConfiguration": {
                 "id": "[variables('frontEndIPConfigID')]"
               },
               "protocol": "tcp",
               "frontendPort": 50001,
               "backendPort": 3389,
               "enableFloatingIP": false
             }
           },
           {
             "name": "RDP-VM1",
             "properties": {
               "frontendIPConfiguration": {
                 "id": "[variables('frontEndIPConfigID')]"
               },
               "protocol": "tcp",
               "frontendPort": 50002,
               "backendPort": 3389,
               "enableFloatingIP": false
             }
           }
         ],
         "loadBalancingRules": [
           {
             "name": "LBRule",
             "properties": {
               "frontendIPConfiguration": {
                 "id": "[variables('frontEndIPConfigID')]"
               },
               "backendAddressPool": {
                 "id": "[variables('lbPoolID')]"
               },
               "protocol": "tcp",
               "frontendPort": 80,
               "backendPort": 80,
               "enableFloatingIP": false,
               "idleTimeoutInMinutes": 5,
               "probe": {
                 "id": "[variables('lbProbeID')]"
               }
             }
           }
         ],
         "probes": [
           {
             "name": "tcpProbe",
             "properties": {
               "protocol": "tcp",
               "port": 80,
               "intervalInSeconds": 5,
               "numberOfProbes": 2
             }
           }
         ]
       }
    }

## 网络接口
你将创建 2 个网络接口，每个 VM 各用一个。可以使用 [copyIndex() 函数](/documentation/articles/resource-group-create-multiple/) 来迭代复制循环（称为 nicLoop），并创建 `numberOfInstances` 变量中定义的网络接口个数，而不必包含重复的网络接口项。网络接口取决于虚拟网络和负载平衡器的创建。它使用创建虚拟网络时所定义的子网，以及负载平衡器 ID 来配置负载平衡器地址池和入站 NAT 规则。有关所有属性，请查看 [REST API for network interfaces（网络接口的 REST API）](https://msdn.microsoft.com/zh-cn/library/azure/mt163668.aspx)。

    {
       "apiVersion": "2015-06-15",
       "type": "Microsoft.Network/networkInterfaces",
       "name": "[concat(parameters('nicNamePrefix'), copyindex())]",
       "location": "[resourceGroup().location]",
       "copy": {
         "name": "nicLoop",
         "count": "[variables('numberOfInstances')]"
       },
       "dependsOn": [
         "[concat('Microsoft.Network/virtualNetworks/', parameters('vnetName'))]",
         "[concat('Microsoft.Network/loadBalancers/', parameters('lbName'))]"
       ],
       "properties": {
         "ipConfigurations": [
           {
             "name": "ipconfig1",
             "properties": {
               "privateIPAllocationMethod": "Dynamic",
               "subnet": {
                 "id": "[variables('subnetRef')]"
               },
               "loadBalancerBackendAddressPools": [
                 {
                   "id": "[concat(variables('lbID'), '/backendAddressPools/BackendPool1')]"
                 }
               ],
               "loadBalancerInboundNatRules": [
                 {
                   "id": "[concat(variables('lbID'),'/inboundNatRules/RDP-VM', copyindex())]"
                 }
               ]
             }
           }
         ]
       }
    }

## 虚拟机
如同在创建[网络接口](#network-interface)时所做的一样，你将使用 copyIndex() 函数创建 2 个虚拟机。VM 的创建取决于存储帐户、网络接口和可用性集。如 `storageProfile` 属性中的定义，将从应用商店映像创建此 VM - `imageReference` 用于定义映像发布者、产品、SKU 和版本。最后，配置诊断配置文件以启用 VM 的诊断。

若要查找应用商店映像的相关属性，请遵循 [select Linux virtual machine images（选择 Linux 虚拟机映像）](/documentation/articles/virtual-machines-linux-cli-ps-findimage/)或 [select Windows virtual machine images（选择 Windows 虚拟机映像）](/documentation/articles/virtual-machines-windows-cli-ps-findimage/)文章。

    {
       "apiVersion": "2015-06-15",
       "type": "Microsoft.Compute/virtualMachines",
       "name": "[concat(parameters('vmNamePrefix'), copyindex())]",
       "copy": {
         "name": "virtualMachineLoop",
         "count": "[variables('numberOfInstances')]"
       },
       "location": "[resourceGroup().location]",
       "dependsOn": [
         "[concat('Microsoft.Storage/storageAccounts/', parameters('storageAccountName'))]",
         "[concat('Microsoft.Network/networkInterfaces/', parameters('nicNamePrefix'), copyindex())]",
         "[concat('Microsoft.Compute/availabilitySets/', variables('availabilitySetName'))]"
       ],
       "properties": {
         "availabilitySet": {
           "id": "[resourceId('Microsoft.Compute/availabilitySets',variables('availabilitySetName'))]"
         },
         "hardwareProfile": {
           "vmSize": "[parameters('vmSize')]"
         },
         "osProfile": {
           "computerName": "[concat(parameters('vmNamePrefix'), copyIndex())]",
           "adminUsername": "[parameters('adminUsername')]",
           "adminPassword": "[parameters('adminPassword')]"
         },
         "storageProfile": {
           "imageReference": {
             "publisher": "[parameters('imagePublisher')]",
             "offer": "[parameters('imageOffer')]",
             "sku": "[parameters('imageSKU')]",
             "version": "latest"
           },
           "osDisk": {
             "name": "osdisk",
             "vhd": {
               "uri": "[concat('http://',parameters('storageAccountName'),'.blob.core.windows.net/vhds/','osdisk', copyindex(), '.vhd')]"
             },
             "caching": "ReadWrite",
             "createOption": "FromImage"
           }
         },
         "networkProfile": {
           "networkInterfaces": [
             {
               "id": "[resourceId('Microsoft.Network/networkInterfaces',concat(parameters('nicNamePrefix'),copyindex()))]"
             }
           ]
         },
         "diagnosticsProfile": {
           "bootDiagnostics": {
              "enabled": "true",
              "storageUri": "[concat('http://',parameters('storageAccountName'),'.blob.core.windows.net')]"
           }
         }
    }

>[AZURE.NOTE] 对于**第三方供应商**发布的映像，必须指定名为 `plan` 的另一个属性。从快速入门库的[此模板](https://github.com/Azure/azure-quickstart-templates/tree/master/checkpoint-single-nic)中可找到示例。

你已定义模板的资源。

## Parameters

在 parameters 节中，定义可在部署模板时指定的值。只针对你认为应在部署期间更改的值定义参数。如果部署期间不提供默认值，你可以为所用的参数提供默认值。如 **imageSKU** 参数所示，你也可以定义允许的值。

    "parameters": {
        "storageAccountName": {
          "type": "string",
          "metadata": {
            "description": "Name of storage account"
          }
        },
        "adminUsername": {
          "type": "string",
          "metadata": {
            "description": "Admin username"
          }
        },
        "adminPassword": {
          "type": "securestring",
          "metadata": {
            "description": "Admin password"
          }
        },
        "dnsNameforLBIP": {
          "type": "string",
          "metadata": {
            "description": "DNS for Load Balancer IP"
          }
        },
        "vmNamePrefix": {
          "type": "string",
          "defaultValue": "myVM",
          "metadata": {
            "description": "Prefix to use for VM names"
          }
        },
        "imagePublisher": {
          "type": "string",
          "defaultValue": "MicrosoftWindowsServer",
          "metadata": {
            "description": "Image Publisher"
          }
        },
        "imageOffer": {
          "type": "string",
          "defaultValue": "WindowsServer",
          "metadata": {
            "description": "Image Offer"
          }
        },
        "imageSKU": {
          "type": "string",
          "defaultValue": "2012-R2-Datacenter",
          "allowedValues": [
            "2008-R2-SP1",
            "2012-Datacenter",
            "2012-R2-Datacenter"
          ],
          "metadata": {
            "description": "Image SKU"
          }
        },
        "lbName": {
          "type": "string",
          "defaultValue": "myLB",
          "metadata": {
            "description": "Load Balancer name"
          }
        },
        "nicNamePrefix": {
          "type": "string",
          "defaultValue": "nic",
          "metadata": {
            "description": "Network Interface name prefix"
          }
        },
        "publicIPAddressName": {
          "type": "string",
          "defaultValue": "myPublicIP",
          "metadata": {
            "description": "Public IP Name"
          }
        },
        "vnetName": {
          "type": "string",
          "defaultValue": "myVNET",
          "metadata": {
            "description": "VNET name"
          }
        },
        "vmSize": {
          "type": "string",
          "defaultValue": "Standard_D1",
          "metadata": {
            "description": "Size of the VM"
          }
        }
      }

## 变量

在 variables 节中，可以定义在模板中多处使用的值，或从其他表达式或变量构造的值。我们经常使用变量来简化模板的语法。

    "variables": {
        "availabilitySetName": "myAvSet",
        "subnetName": "Subnet-1",
        "vnetID": "[resourceId('Microsoft.Network/virtualNetworks',parameters('vnetName'))]",
        "subnetRef": "[concat(variables('vnetID'),'/subnets/',variables ('subnetName'))]",
        "publicIPAddressID": "[resourceId('Microsoft.Network/publicIPAddresses',parameters('publicIPAddressName'))]",
        "numberOfInstances": 2,
        "lbID": "[resourceId('Microsoft.Network/loadBalancers',parameters('lbName'))]",
        "frontEndIPConfigID": "[concat(variables('lbID'),'/frontendIPConfigurations/LoadBalancerFrontEnd')]",
        "lbPoolID": "[concat(variables('lbID'),'/backendAddressPools/BackendPool1')]",
        "lbProbeID": "[concat(variables('lbID'),'/probes/tcpProbe')]"
      }

你已经完成此模板！ 你可以将你的模板与[快速入门库](https://github.com/Azure/azure-quickstart-templates/tree/master/201-2-vms-loadbalancer-lbrules)中[包含负载平衡器和负载平衡规则模板的 2 个 VM](https://github.com/Azure/azure-quickstart-templates) 下的完整模板相比较。由于使用不同的版本号，你的模板可能会略有不同。

你可以使用部署存储帐户时使用的相同命令，重新部署该模板。在重新部署前无需删除存储帐户，因为 Resource Manager 将跳过重新创建已存在且未发生更改的资源。

## 后续步骤

- [Azure Resource Manager 模板可视化工具 (ARMViz)](http://armviz.io/#/) 是个强大的工具，可直观地显示 ARM 模板，因为这些模板只从 json 文件读取可能会变得太大而无法了解。
- 若要详细了解模板的结构，请参阅 [Authoring Azure Resource Manager templates（创作 Azure Resource Manager 模板）](/documentation/articles/resource-group-authoring-templates/)。
- 若要了解如何部署模板，请参阅 [Deploy a Resource Group with Azure Resource Manager template（使用 Azure Resource Manager 模板部署资源组）](/documentation/articles/resource-group-template-deploy/)

<!---HONumber=Mooncake_0620_2016-->