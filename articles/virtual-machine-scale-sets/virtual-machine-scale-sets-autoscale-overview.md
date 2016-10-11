<properties
	pageTitle="自动缩放和虚拟机缩放集 | Azure"
	description="了解如何使用诊断和自动缩放资源自动缩放缩放集中的虚拟机。"
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

# 自动缩放和虚拟机缩放集

缩放集内虚拟机的自动缩放是指按照应用程序的需求在缩放集内创建或删除虚拟机，以符合性能需求，并满足服务级别协议 (SLA)。当工作量增大时，应用程序可能需要额外的资源才能有效执行任务。

自动缩放是一个自动化过程，可帮助减少管理开销。只要能够减少开销，就不再需要持续监视系统性能，也不再需要决定是添加还是删除资源。缩放是一个弹性过程；当系统上的负载增加时，可以预配较多的资源，但是当需求减少时，也可以解除所分配的资源，以便将成本降至最低，同时仍维持适当的性能，并符合 SLA。

使用 Azure Resource Manager 模板、Azure PowerShell 或 Azure CLI 在缩放集上设置自动缩放。

## 使用 Resource Manager 模板设置缩放

您无需单独部署和管理应用程序的每个资源，而是使用模板，以单个协调操作来部署和预配所有资源。在模板中，会定义应用程序资源，并针对不同的环境指定部署参数。模板中包含可用于构造部署值的 JSON 和表达式。若要了解详细信息，请参阅[创作 Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates/)。

在模板中，可以指定容量元素：

    "sku": {
      "name": "Standard_A0",
      "tier": "Standard",
      "capacity": 3
    },

容量会识别缩放集中的虚拟机数目。您可以部署具有不同值的模板，以便手动更改容量。如果您部署模板只是为了更改容量，则可以仅包含具有更新容量的 SKU 元素。

自动更改缩放集的容量，方法是将 Azure Resource Manager 提供的 autoscaleSettings 资源与安装在缩放集中的虚拟机上的 Azure 诊断扩展结合使用。

### 配置 Azure 诊断扩展

仅当缩放集中每个虚拟机上的指标收集成功时，才能完成自动缩放。Azure 诊断扩展提供监视和诊断功能，符合自动缩放资源的指标收集需求。您可以安装扩展作为 Resource Manager 模板的一部分。

此示例显示在模板中用来配置诊断扩展的变量：

	"diagnosticsStorageAccountName": "[concat(parameters('resourcePrefix'), 'saa')]",
	"accountid": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/', resourceGroup().name,'/providers/', 'Microsoft.Storage/storageAccounts/', variables('diagnosticsStorageAccountName'))]",
	"wadlogs": "<WadCfg> <DiagnosticMonitorConfiguration overallQuotaInMB=\"4096\" xmlns=\"http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration\"> <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter=\"Error\"/> <WindowsEventLog scheduledTransferPeriod=\"PT1M\" > <DataSource name=\"Application!*[System[(Level = 1 or Level = 2)]]\" /> <DataSource name=\"Security!*[System[(Level = 1 or Level = 2)]]\" /> <DataSource name=\"System!*[System[(Level = 1 or Level = 2)]]\" /></WindowsEventLog>",
	"wadperfcounter": "<PerformanceCounters scheduledTransferPeriod=\"PT1M\"><PerformanceCounterConfiguration counterSpecifier=\"\\Processor(_Total)\\Thread Count\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"Thread Count\" locale=\"en-us\"/></PerformanceCounterConfiguration></PerformanceCounters>",
	"wadcfgxstart": "[concat(variables('wadlogs'),variables('wadperfcounter'),'<Metrics resourceId=\"')]",
	"wadmetricsresourceid": "[concat('/subscriptions/',subscription().subscriptionId,'/resourceGroups/',resourceGroup().name ,'/providers/','Microsoft.Compute/virtualMachineScaleSets/',parameters('vmssName'))]",
	"wadcfgxend": "[concat('\"><MetricAggregation scheduledTransferPeriod=\"PT1H\"/><MetricAggregation scheduledTransferPeriod=\"PT1M\"/></Metrics></DiagnosticMonitorConfiguration></WadCfg>')]"

针对诸如存储帐户（在其中存储数据）名称和缩放集（从其中收集数据）名称之类的值部署模板时，会提供参数。另外，在此 Windows Server 示例中，只会收集 Thread Count 性能计数器，但是，Windows 或 Linux 中所有可用的性能计数器都可以用来收集诊断信息，并且可以包含在扩展配置中。

此示例显示模板中扩展的定义：

    "extensionProfile": {
      "extensions": [
        {
          "name": "Microsoft.Insights.VMDiagnosticsSettings",
          "properties": {
            "publisher": "Microsoft.Azure.Diagnostics",
            "type": "IaaSDiagnostics",
            "typeHandlerVersion": "1.5",
            "autoUpgradeMinorVersion": true,
            "settings": {
              "xmlCfg": "[base64(concat(variables('wadcfgxstart'),variables('wadmetricsresourceid'),variables('wadcfgxend')))]",
              "storageAccount": "[variables('diagnosticsStorageAccountName')]"
            },
            "protectedSettings": {
              "storageAccountName": "[variables('diagnosticsStorageAccountName')]",
              "storageAccountKey": "[listkeys(variables('accountid'), variables('apiVersion')).key1]",
              "storageAccountEndPoint": "https://core.chinacloudapi.cn"
            }
          }
        }
      ]
    }

当诊断扩展运行时，会将数据收集到一个表中，该表位于您指定的存储帐户中。在 WADPerformanceCounters 表中，可以找到收集的数据。例如，这是当第一次在缩放集中创建虚拟机时所收集的线程计数：

![](./media/virtual-machine-scale-sets-autoscale-overview/ThreadCountBefore2.png)

### 配置 autoScaleSettings 资源

autoscaleSettings 资源使用诊断扩展所收集的信息，决定如何在缩放集中增加或减少虚拟机数目。

此示例显示模板中资源的配置：

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
                  "metricName": "\\Process(_Total)\\Thread Count",
                  "metricNamespace": "",
                  "metricResourceUri": "[concat('/subscriptions/',subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Compute/virtualMachineScaleSets/', parameters('vmSSName'))]",
                  "timeGrain": "PT1M",
                  "statistic": "Average",
                  "timeWindow": "PT5M",
                  "timeAggregation": "Average",
                  "operator": "GreaterThan",
                  "threshold": 650
                },
                "scaleAction": {
                  "direction": "Increase",
                  "type": "ChangeCount",
                  "value": "1",
                  "cooldown": "PT5M"
                }
              },
              {
                "metricTrigger": {
                  "metricName": "\\Process(_Total)\\Thread Count",
                  "metricNamespace": "",
                  "metricResourceUri": "[concat('/subscriptions/',subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Compute/virtualMachineScaleSets/', parameters('vmSSName'))]",
                  "timeGrain": "PT1M",
                  "statistic": "Average",
                  "timeWindow": "PT5M",
                  "timeAggregation": "Average",
                  "operator": "LessThan",
                  "threshold": 550
                },
                "scaleAction": {
                  "direction": "Decrease",
                  "type": "ChangeCount",
                  "value": "1",
                  "cooldown": "PT5M"
                }
              }
            ]
          }
        ],
        "targetResourceUri": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.Compute/virtualMachineScaleSets/', parameters('vmSSName'))]"
      }
    }

在上述示例中，创建了两个规则来定义自动缩放操作。第一个规则定义扩大操作，第二个规则定义缩小操作。在规则中提供了这些值：

- **metricName** — 这与您在诊断扩展的 wadperfcounter 变量中定义的性能计数器相同。上述示例使用的是 Thread Count 计数器。
- **metricResourceUri** — 这是虚拟机规模集的资源标识符。此标识符包含资源组的名称、资源提供程序的名称和要缩放的缩放集的名称。
- **timeGrain** - 这是收集的指标的粒度。在上述示例中，数据以一分钟的间隔进行收集。此值与 timeWindow 结合使用。
- **statistic** - 这可确定如何组合指标以调节自动缩放操作。可能的值包括：Average、Min、Max。
- **timeWindow** — 这是收集实例数据的时间范围。它必须介于 5 分钟和 12 个小时之间。
- **timeAggregation** - 这可确定应如何随着时间的推移组合收集的数据。默认值为 Average。可能的值包括：Average、Minimum、Maximum、Last、Total、Count。
- **operator** - 这是用于比较指标数据和阈值的运算符。可能的值包括：Equals、NotEquals、GreaterThan、GreaterThanOrEqual、LessThan、LessThanOrEqual。
- **threshold** - 这是将触发缩放操作的值。必须确保在为扩大操作设置的阈值与为缩小操作设置的阈值之间提供足够的差异。如果设置的值相同，例如在上述示例中均设置为 600 个线程，系统就会预期缩放集大小持续增大和减小，并且不会实施缩放操作。
- **direction** - 这可确定在达到阈值时执行的操作。可能的值为 Increase 或 Decrease。
- **type** - 这是应发生的操作类型，此属性必须设置为 ChangeCount。
- **value** - 这是已在规模集中添加或删除的虚拟机数。此值必须大于或等于 1。
- **cooldown** - 这是在上次缩放操作后，在下次操作发生之前要等待的时间。此值必须介于一分钟到一周之间。

根据您使用的性能计数器，会以不同的方式使用模板配置中的某些元素。在所示示例中，使用的性能计数器是 Thread Count，且扩大操作的阈值设置为 650，缩小操作的阈值设置为 550。如果使用诸如 %Processor Time 之类的计数器，阈值会设置为 CPU 使用率百分比，以确定缩放操作。

当缩放集中的虚拟机上所创建的负载触发缩放操作时，缩放集中的虚拟机数目会根据模板中的值元素而增加。例如，在容量设置为 3 且缩放操作值设置为 1 的缩放集中。

当创建的负载导致平均线程计数超过 650 的阈值时：

![](./media/virtual-machine-scale-sets-autoscale-overview/ThreadCountAfter.png)  


会触发扩大操作，让缩放集的容量增加 1：

    "sku": {
      "name": "Standard_A0",
      "tier": "Standard",
      "capacity": 4
    },

且向缩放集添加一个虚拟机。

经过五分钟的冷却期间之后，如果虚拟机上的线程平均数目仍然超过 600，则会向缩放集再添加一个虚拟机。如果平均线程计数保持在 550 以下，则缩放集的容量会减少 1，并且从缩放集中删除一个虚拟机。

## 使用 Azure PowerShell 设置缩放

Azure PowerShell 可用于设置缩放集的自动缩放。使用 PowerShell 创建本文先前所述规则的示例如下：

    $rule1 = New-AutoscaleRule -MetricName "\Processor(_Total)\Thread Count" -MetricResourceId "/subscriptions/{subscription-id}/resourceGroups/myrg1/providers/Microsoft.Compute/virtualMachineScaleSets/myvmss1" -Operator GreaterThan -MetricStatistic Average -Threshold 650 -TimeGrain 00:01:00 -ScaleActionCooldown 00:05:00 -ScaleActionDirection Increase -ScaleActionScaleType ChangeCount -ScaleActionValue "1"  
 
    $rule2 = New-AutoscaleRule -MetricName "\Processor(_Total)\% Processor Time" -MetricResourceId "/subscriptions/df602c9c-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/rainvmss/providers/Microsoft.Compute/virtualMachineScaleSets/rainvmss1" -Operator LessThan -MetricStatistic Average -Threshold 10 -TimeGrain 00:01:00 -ScaleActionCooldown 00:10:00 -ScaleActionDirection Decrease -ScaleActionScaleType ChangeCount -ScaleActionValue "2" 
 
    $profile1 = New-AutoscaleProfile -DefaultCapacity "2" -MaximumCapacity "10" -MinimumCapacity "2" -Rules $rule1, $rule2 -Name "ScalePuppy" 
 
    Add-AutoscaleSetting -Location "China East" -Name 'MyScaleVMSSSetting' -ResourceGroup rainvmss -TargetResourceId "/subscriptions/df602c9c-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/rainvmss/providers/Microsoft.Compute/virtualMachineScaleSets/rainvmss1" -AutoscaleProfiles $profile1 

## 使用 Azure CLI 设置缩放

（尚未提供任何信息）

## 调查缩放操作

- [Azure 门户预览](https://portal.azure.cn) — 当前使用门户可以获取有限数量的信息。
- Azure PowerShell - 使用此命令可获取一些信息：

        Get-AzureRmResource -name vmsstest1 -ResourceGroupName vmsstestrg1 -ResourceType Microsoft.Compute/virtualMachineScaleSets -ApiVersion 2015-06-15
        Get-Autoscalesetting -ResourceGroup rainvmss -DetailedOutput
        
- 就像连接任何其他虚拟机一样连接到 jumpbox 虚拟机，然后可以远程访问缩放集中的虚拟机，以监视单个进程。

## 后续步骤

- 请参阅[自动缩放虚拟机缩放集中的虚拟机](/documentation/articles/virtual-machine-scale-sets-windows-autoscale/)，以查看有关如何创建配置了自动缩放的缩放集的示例。
- 在 [Azure Insights PowerShell 快速启动示例](/documentation/articles/insights-powershell-samples/)中查找 Azure Insights 监视功能的示例
- 在[使用自动缩放操作在 Azure Insights 中发送电子邮件和 Webhook 警报通知](/documentation/articles/insights-autoscale-to-webhook-email/)和[使用审核日志在 Azure Insights 中发送电子邮件和 Webhook 警报通知](/documentation/articles/insights-auditlog-to-webhook-email/)中了解有关通知功能的信息

<!---HONumber=Mooncake_0822_2016-->