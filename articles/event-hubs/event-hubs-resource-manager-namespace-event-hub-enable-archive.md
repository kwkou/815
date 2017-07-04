<properties
    pageTitle="使用模板创建 Azure 事件中心命名空间并启用存档 | Azure"
    description="使用 Azure Resource Manager 模板创建包含事件中心和事件中心命名空间以及启用存档"
    services="event-hubs"
    documentationcenter=".net"
    author="ShubhaVijayasarathy"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="8bdda6a2-5ff1-45e3-b696-c553768f1090"
    ms.service="event-hubs"
    ms.devlang="tbd"
    ms.topic="article"
    ms.tgt_pltfrm="dotnet"
    ms.workload="na"
    ms.date="03/07/2017"
    wacn.date="04/17/2017"
    ms.author="shvija;sethm"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="89ced1f70d294586eec2d6d315c21360554636e8"
    ms.lasthandoff="04/07/2017" />

# <a name="create-an-event-hubs-namespace-with-event-hub-and-enable-archive-using-an-azure-resource-manager-template"></a>使用 Azure Resource Manager 模板创建包含事件中心的事件中心命名空间并启用存档
本文介绍如何使用 Azure Resource Manager 模板创建包含一个事件中心的事件中心类型命名空间，并在事件中心上启用存档功能。 本文介绍如何定义要部署的资源以及如何定义执行部署时指定的参数。 可将此模板用于自己的部署，或自定义此模板以满足要求

有关创建模板的详细信息，请参阅[创作 Azure Resource Manager 模板][Authoring Azure Resource Manager templates]。
<!-- Not available in Mooncake for the Azure Resources Naming Conventions -->

有关完整的模板，请参阅 GitHub 上的 [事件中心和启用存档模板][Event Hub and enable Archive template] 。

> [AZURE.NOTE]
> 若要检查最新模板，请访问 [Azure 快速启动模板][Azure Quickstart Templates] 库并搜索事件中心。
> 
> 

## <a name="what-will-you-deploy"></a>你将部署什么内容？
使用此模板，可部署包含事件中心的事件中心命名空间以及启用事件中心存档。

[事件中心](/documentation/articles/event-hubs-what-is-event-hubs/)是一种事件处理服务，用于向 Azure 提供大规模的事件与遥测数据入口，并且具有较低的延迟和较高的可靠性。 使用事件中心存档，可以按照所选指定时间或大小间隔将事件中心中的流数据自动传送到所选的 Azure Blob 存储。

若要自动运行部署，请单击以下按钮：

[![部署到 Azure](./media/event-hubs-resource-manager-namespace-event-hub/deploybutton.png)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-eventhubs-create-namespace-and-enable-archive%2Fazuredeploy.json)

## <a name="parameters"></a>Parameters

使用 Azure 资源管理器，可以定义在部署模板时想要指定的值的参数。 该模板具有一个名为 `Parameters` 的部分，其中包含所有参数值。 你应该为随着要部署的项目或要部署到的环境而变化的值定义参数。 不要为永远保持不变的值定义参数。 每个参数值可在模板中用来定义所部署的资源。

模板定义以下参数。

### <a name="eventhubnamespacename"></a>eventHubNamespaceName

要创建的事件中心命名空间的名称。

    "eventHubNamespaceName":{  
         "type":"string",
         "metadata":{  
             "description":"Name of the EventHub namespace"
          }
    }

### <a name="eventhubname"></a>eventHubName

在事件中心命名空间中创建的事件中心的名称。

    "eventHubName":{  
        "type":"string",
        "metadata":{  
            "description":"Name of the Event Hub"
        }
    }

### <a name="messageretentionindays"></a>messageRetentionInDays

要在事件中心中保留消息的天数。 

    "messageRetentionInDays":{
        "type":"int",
        "defaultValue": 1,
        "minValue":"1",
        "maxValue":"7",
        "metadata":{
           "description":"How long to retain the data in Event Hub"
         }
     }

### <a name="partitioncount"></a>partitionCount

要在事件中心中创建的分区数。

    "partitionCount":{
        "type":"int",
        "defaultValue":2,
        "minValue":2,
        "maxValue":32,
        "metadata":{
            "description":"Number of partitions chosen"
        }
     }

### <a name="archiveenabled"></a>archiveEnabled

在事件中心上启用存档。

    "archiveEnabled":{
        "type":"string",
        "defaultValue":"true",
        "allowedValues": [
        "false",
        "true"],
        "metadata":{
            "description":"Enable or disable the Archive for your Event Hub"
        }
     }

### <a name="archiveencodingformat"></a>archiveEncodingFormat

指定用于序列化事件数据的编码格式。

    "archiveEncodingFormat":{
        "type":"string",
        "defaultValue":"Avro",
        "allowedValues":[
        "Avro"],
        "metadata":{
            "description":"The encoding format Archive serializes the EventData"
        }
    }

### <a name="archivetime"></a>archiveTime

事件中心存档开始将数据存档在 Azure Blob 存储中的时间间隔。

    "archiveTime":{
        "type":"int",
        "defaultValue":300,
        "minValue":60,
        "maxValue":900,
        "metadata":{
             "description":"the time window in seconds for the archival"
        }
    }

### <a name="archivesize"></a>archiveSize

存档开始将数据存档在 Azure Blob 存储中的大小间隔。

    "archiveSize":{
        "type":"int",
        "defaultValue":314572800,
        "minValue":10485760,
        "maxValue":524288000,
        "metadata":{
            "description":"the size window in bytes for archival"
        }
    }

### <a name="destinationstorageaccountresourceid"></a>destinationStorageAccountResourceId

存档需要 Azure 存储帐户资源 ID，以启用到所需存储帐户的存档。

     "destinationStorageAccountResourceId":{
        "type":"string",
        "metadata":{
            "description":"Your existing storage Account resource id where you want the blobs be archived"
        }
     }

### <a name="blobcontainername"></a>blobContainerName

用于存档事件数据的 blob 容器。

     "blobContainerName":{
        "type":"string",
        "metadata":{
            "description":"Your existing storage container that you want the blobs archived in"
        }
    }

### <a name="apiversion"></a>apiVersion

模板的 API 版本。

     "apiVersion":{  
        "type":"string",
        "defaultValue":"2015-08-01",
        "metadata":{  
            "description":"ApiVersion used by the template"
        }
     }

## <a name="resources-to-deploy"></a>要部署的资源

创建包含一个事件中心的 **EventHubs** 类型的命名空间并启用存档。

    "resources":[  
          {  
             "apiVersion":"[variables('ehVersion')]",
             "name":"[parameters('eventHubNamespaceName')]",
             "type":"Microsoft.EventHub/Namespaces",
             "location":"[variables('location')]",
             "sku":{  
                "name":"Standard",
                "tier":"Standard"
             },
             "resources":[  
                {  
                   "apiVersion":"[variables('ehVersion')]",
                   "name":"[parameters('eventHubName')]",
                   "type":"EventHubs",
                   "dependsOn":[  
                      "[concat('Microsoft.EventHub/namespaces/', parameters('eventHubNamespaceName'))]"
                   ],
                   "properties":{  
                      "path":"[parameters('eventHubName')]",
                      "MessageRetentionInDays":"[parameters('messageRetentionInDays')]",
                      "PartitionCount":"[parameters('partitionCount')]",
                      "ArchiveDescription":{
                            "enabled":"[parameters('archiveEnabled')]",
                            "encoding":"[parameters('archiveEncodingFormat')]",
                            "intervalInSeconds":"[parameters('archiveTime')]",
                            "sizeLimitInBytes":"[parameters('archiveSize')]",
                            "destination":{
                                "name":"EventHubArchive.AzureBlockBlob",
                                "properties":{
                                    "StorageAccountResourceId":"[parameters('destinationStorageAccountResourceId')]",
                                    "BlobContainer":"[parameters('blobContainerName')]"
                                }
                            } 
                      }

                   }

                }
             ]
          }
       ]

## <a name="commands-to-run-deployment"></a>运行部署的命令

[AZURE.INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

## <a name="powershell"></a>PowerShell

    New-AzureRmResourceGroupDeployment -ResourceGroupName \<resource-group-name\> -TemplateFile https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-eventhubs-create-namespace-and-enable-archive/azuredeploy.json

## <a name="azure-cli"></a>Azure CLI

    azure config mode arm

    azure group deployment create \<my-resource-group\> \<my-deployment-name\> --template-uri [https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-eventhubs-create-namespace-and-enable-archive/azuredeploy.json][]

## <a name="next-steps"></a>后续步骤
访问以下链接可以了解有关事件中心的详细信息：

* [事件中心概述](/documentation/articles/event-hubs-what-is-event-hubs/)
* [创建事件中心](/documentation/articles/event-hubs-create/)
* [事件中心常见问题](/documentation/articles/event-hubs-faq/)

[Authoring Azure Resource Manager templates]: /documentation/articles/resource-group-authoring-templates/
[Azure Quickstart Templates]: https://github.com/Azure/azure-quickstart-templates/?term=event+hubs
[Using Azure PowerShell with Azure Resource Manager]: /documentation/articles/powershell-azure-resource-manager/
[Using the Azure CLI for Mac, Linux, and Windows with Azure Resource Management]: /documentation/articles/xplat-cli-azure-resource-manager/
[Event Hub and consumer group template]: https://github.com/Azure/azure-quickstart-templates/blob/master/201-eventhubs-create-namespace-and-enable-archive/
[Azure Resources Naming Conventions]: /documentation/articles/guidance-naming-conventions/
[Event Hub and enable Archive template]: https://github.com/Azure/azure-quickstart-templates/tree/master/201-eventhubs-create-namespace-and-enable-archive


<!-- Update_Description: update meta properties; wording update -->