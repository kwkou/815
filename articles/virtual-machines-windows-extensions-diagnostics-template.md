<!-- ARM: tested -->

<properties
	pageTitle="使用 Azure 资源管理器模板创建具有监视和诊断功能的 Windows 虚拟机 | Microsoft Azure"
	description="使用 Azure 资源管理器模板新建具有 Azure 诊断扩展的 Windows 虚拟机。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="sbtron"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="12/15/2015"
	wacn.date="06/07/2016"/>

# 使用 Azure 资源管理器模板创建具有监视和诊断功能的 Windows 虚拟机

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代经典部署模型。

Azure 诊断扩展可在基于 Windows 的 Azure 虚拟机上提供监视和诊断功能。通过将该扩展纳入为 Azure 资源管理器模板的一部分，可以在虚拟机上启用这些功能。有关将任何扩展纳入为虚拟机模板一部分的详细信息，请参阅[使用 VM 扩展创作 Azure 资源管理器模板](/documentation/articles/virtual-machines-windows-extensions-authoring-templates/)。本文介绍如何将 Azure 诊断扩展添加到 Windows 虚拟机模板中。
  

## 将 Azure 诊断扩展添加到 VM 资源定义中 

若要在 Windows 虚拟机上启用诊断扩展，需要将该扩展添加为资源管理器模板中的 VM 资源。

对于简单的基于资源管理器的虚拟机，请将扩展配置添加到该虚拟机的 *resources* 数组：

	"resources": [
                {
                    "name": "Microsoft.Insights.VMDiagnosticsSettings",
                    "type": "extensions",
                    "location": "[resourceGroup().location]",
                    "apiVersion": "2015-06-15",
                    "dependsOn": [
                        "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
                    ],
                    "tags": {
                        "displayName": "AzureDiagnostics"
                    },
                    "properties": {
                        "publisher": "Microsoft.Azure.Diagnostics",
                        "type": "IaaSDiagnostics",
                        "typeHandlerVersion": "1.5",
                        "autoUpgradeMinorVersion": true,
                        "settings": {
                            "xmlCfg": "[base64(concat(variables('wadcfgxstart'), variables('wadmetricsresourceid'), variables('vmName'), variables('wadcfgxend')))]",
                            "storageAccount": "[parameters('existingdiagnosticsStorageAccountName')]"
                        },
                        "protectedSettings": {
                            "storageAccountName": "[parameters('existingdiagnosticsStorageAccountName')]",
                            "storageAccountKey": "[listkeys(variables('accountid'), '2015-05-01-preview').key1]",
                            "storageAccountEndPoint": "https://core.chinacloudapi.cn"
                        }
                    }
                }
            ]


另一个常见惯例是在模板的根资源节点处添加扩展配置，而不是在虚拟机的资源节点下进行定义。使用这个方法时，必须用 *name* 和 *type* 值显式指定扩展与虚拟机之间的分层关系。例如：
  
	"name": "[concat(variables('vmName'),'Microsoft.Insights.VMDiagnosticsSettings')]",
    "type": "Microsoft.Compute/virtualMachines/extensions",

扩展始终与虚拟机关联，你可以直接在虚拟机的资源节点下定义扩展，也可以在基础级别定义扩展并使用分层命名约定将其与虚拟机关联。

对于虚拟机缩放集，扩展配置在 *VirtualMachineProfile* 的 *extensionProfile* 属性中指定。
   
值为 **Microsoft.Azure.Diagnostics** 的 *publisher* 属性和值为 **IaaSDiagnostics** 的 *type* 属性可唯一标识 Azure 诊断扩展。

*name* 属性的值可用来参考资源组中的扩展。特别将其设为 **Microsoft.Insights.VMDiagnosticsSettings** 后，它可以轻易地被 Azure 经典管理门户识别，从而确保监视图表在 Azure 经典管理门户中正确显示。

*typeHandlerVersion* 指定你要使用的扩展的版本。将 *autoUpgradeMinorVersion* 次要版本设置为“true”可确保获得可用的最新扩展次要版本。强烈建议你始终将 *autoUpgradeMinorVersion* 设置为“true”，这样就可以随时获得并使用具有所有新功能和缺陷修复的最新的可用诊断扩展。

*settings* 元素包含扩展的配置属性（有时称为公共配置），这些属性可以从扩展设置和读回。*xmlcfg* 属性包含诊断日志基于 xml 的配置、性能计数器等等，这些项目将由诊断代理收集。有关 xml 架构本身的详细信息，请参阅[诊断配置架构](https://msdn.microsoft.com/zh-cn/library/azure/dn782207.aspx)。常见的做法是将实际的 xml 配置存储为 Azure 资源管理器模板中的变量，然后再进行连接和 base64 编码，以设置 *xmlcfg* 的值。请参阅[诊断配置变量](#diagnostics-configuration-variables)一节，以深入了解如何在变量中存储 xml。*storageAccount* 属性指定要向其传输诊断数据的存储帐户名称。
 
*protectedSettings* 中的属性（有时称为专用配置）可进行设置，但在设置之后无法读回。*protectedSettings* 的只写性质使其非常适合存储类似存储帐户密钥（写入诊断数据的位置）这样的密码。

## 将诊断存储帐户指定为参数 

上述诊断扩展 json 代码段假定两个参数 *existingdiagnosticsStorageAccountName* 和 *existingdiagnosticsStorageAccountName*，以指定将存储诊断数据的诊断存储帐户。将诊断存储帐户指定为参数可让你轻松地跨不同环境更改诊断存储帐户，例如，你可能想要使用不同诊断存储帐户进行测试，并且使用另外一个进行生产部署。

        "existingdiagnosticsStorageAccountName": {
            "type": "string",
            "metadata": {
        "description": "The name of an existing storage account to which diagnostics data will be transfered."
			}        
		},
        "existingdiagnosticsStorageResourceGroup": {
            "type": "string",
            "metadata": {
        "description": "The resource group for the storage account specified in existingdiagnosticsStorageAccountName"
      		}
        }

最佳做法是在不同于虚拟机资源组的其他资源组中指定诊断存储帐户。资源组可以视为具有自己的生存期的部署单位，可以部署虚拟机以及在新配置更新时重新部署，但是你可能想要跨这些虚拟机部署继续在相同的存储帐户中存储诊断数据。在不同的资源中拥有存储帐户可让存储帐户接受来自各种虚拟机部署的数据，方便解决各种版本之间的问题。

>[AZURE.NOTE] 如果从 Visual Studio 创建 Windows 虚拟机模板，默认存储帐户可能会设置为使用将虚拟机 VHD 上载到的存储帐户。这是为了简化 VM 的初始设置。你应该重构模板以使用可以当作参数传入的不同存储帐户。

##<a name="diagnostics-configuration-variables"></a> 诊断配置变量
 
上述诊断扩展 json 代码段会定义 *accountid* 变量，以简化获取诊断存储的存储帐户密钥的过程：
	
	"accountid": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/',parameters('existingdiagnosticsStorageResourceGroup'), '/providers/','Microsoft.Storage/storageAccounts/', parameters('existingdiagnosticsStorageAccountName'))]"


诊断扩展的 *xmlcfg* 属性使用连接在一起的多个变量定义。这些变量值的格式为 xml，因此必须在设置 json 变量时正确转义。

下面介绍了诊断配置 xml，它会收集标准系统级别性能计数器以及一些 Windows 事件日志和诊断基础结构日志。该配置 xml 已正确转义和格式化，因此可以直接将配置粘贴到模板的 variables 节。有关该配置 xml 的更易理解的示例，请参阅[诊断配置架构](https://msdn.microsoft.com/zh-cn/library/azure/dn782207.aspx)。
    
        "wadlogs": "<WadCfg> <DiagnosticMonitorConfiguration overallQuotaInMB="4096" xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration"> <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter="Error"/> <WindowsEventLog scheduledTransferPeriod="PT1M" > <DataSource name="Application!*[System[(Level = 1 or Level = 2)]]" /> <DataSource name="Security!*[System[(Level = 1 or Level = 2)]]" /> <DataSource name="System!*[System[(Level = 1 or Level = 2)]]" /></WindowsEventLog>",
        "wadperfcounters1": "<PerformanceCounters scheduledTransferPeriod="PT1M"><PerformanceCounterConfiguration counterSpecifier="\\Processor(_Total)\\% Processor Time" sampleRate="PT15S" unit="Percent"><annotation displayName="CPU utilization" locale="en-us"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier="\\Processor(_Total)\\% Privileged Time" sampleRate="PT15S" unit="Percent"><annotation displayName="CPU privileged time" locale="en-us"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier="\\Processor(_Total)\\% User Time" sampleRate="PT15S" unit="Percent"><annotation displayName="CPU user time" locale="en-us"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier="\\Processor Information(_Total)\\Processor Frequency" sampleRate="PT15S" unit="Count"><annotation displayName="CPU frequency" locale="en-us"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier="\\System\\Processes" sampleRate="PT15S" unit="Count"><annotation displayName="Processes" locale="en-us"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier="\\Process(_Total)\\Thread Count" sampleRate="PT15S" unit="Count"><annotation displayName="Threads" locale="en-us"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier="\\Process(_Total)\\Handle Count" sampleRate="PT15S" unit="Count"><annotation displayName="Handles" locale="en-us"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier="\\Memory\\% Committed Bytes In Use" sampleRate="PT15S" unit="Percent"><annotation displayName="Memory usage" locale="en-us"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier="\\Memory\\Available Bytes" sampleRate="PT15S" unit="Bytes"><annotation displayName="Memory available" locale="en-us"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier="\\Memory\\Committed Bytes" sampleRate="PT15S" unit="Bytes"><annotation displayName="Memory committed" locale="en-us"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier="\\Memory\\Commit Limit" sampleRate="PT15S" unit="Bytes"><annotation displayName="Memory commit limit" locale="en-us"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier="\\PhysicalDisk(_Total)\\% Disk Time" sampleRate="PT15S" unit="Percent"><annotation displayName="Disk active time" locale="en-us"/></PerformanceCounterConfiguration>",
        "wadperfcounters2": "<PerformanceCounterConfiguration counterSpecifier="\\PhysicalDisk(_Total)\\% Disk Read Time" sampleRate="PT15S" unit="Percent"><annotation displayName="Disk active read time" locale="en-us"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier="\\PhysicalDisk(_Total)\\% Disk Write Time" sampleRate="PT15S" unit="Percent"><annotation displayName="Disk active write time" locale="en-us"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier="\\PhysicalDisk(_Total)\\Disk Transfers/sec" sampleRate="PT15S" unit="CountPerSecond"><annotation displayName="Disk operations" locale="en-us"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier="\\PhysicalDisk(_Total)\\Disk Reads/sec" sampleRate="PT15S" unit="CountPerSecond"><annotation displayName="Disk read operations" locale="en-us"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier="\\PhysicalDisk(_Total)\\Disk Writes/sec" sampleRate="PT15S" unit="CountPerSecond"><annotation displayName="Disk write operations" locale="en-us"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier="\\PhysicalDisk(_Total)\\Disk Bytes/sec" sampleRate="PT15S" unit="BytesPerSecond"><annotation displayName="Disk speed" locale="en-us"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier="\\PhysicalDisk(_Total)\\Disk Read Bytes/sec" sampleRate="PT15S" unit="BytesPerSecond"><annotation displayName="Disk read speed" locale="en-us"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier="\\PhysicalDisk(_Total)\\Disk Write Bytes/sec" sampleRate="PT15S" unit="BytesPerSecond"><annotation displayName="Disk write speed" locale="en-us"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier="\\LogicalDisk(_Total)\\% Free Space" sampleRate="PT15S" unit="Percent"><annotation displayName="Disk free space (percentage)" locale="en-us"/></PerformanceCounterConfiguration></PerformanceCounters>",
        "wadcfgxstart": "[concat(variables('wadlogs'), variables('wadperfcounters1'), variables('wadperfcounters2'), '<Metrics resourceId="')]",
        "wadmetricsresourceid": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name , '/providers/', 'Microsoft.Compute/virtualMachines/')]",
        "wadcfgxend": ""><MetricAggregation scheduledTransferPeriod="PT1H"/><MetricAggregation scheduledTransferPeriod="PT1M"/></Metrics></DiagnosticMonitorConfiguration></WadCfg>"

上述配置中的指标定义 xml 节点是一个重要的配置元素，因为它定义如何聚合和存储之前在 *PerformanceCounter* 节点中的 xml 定义的性能计数器。

> [AZURE.IMPORTANT] 这些度量值是促使 Azure 门户预览中生成监视图表和警报的因素。如果你需要在 Azure 门户预览中查看 VM 监视数据，则必须在诊断配置中包括 **Metrics** 节点以及 *resourceID* 和 **MetricAggregation**。

以下是指标定义 xml 的示例：

		<Metrics resourceId="/subscriptions/subscription().subscriptionId/resourceGroups/resourceGroup().name/providers/Microsoft.Compute/virtualMachines/vmName">
			<MetricAggregation scheduledTransferPeriod="PT1H"/>
			<MetricAggregation scheduledTransferPeriod="PT1M"/>
		</Metrics>

*resourceID* 属性唯一标识你的订阅中的虚拟机。请确保使用 subscription() 和 resourceGroup() 函数，这样，模板就会根据你要部署到的订阅和资源组自动更新这些值。

如果要在一个循环中创建多个虚拟机，则必须用 copyIndex() 函数填充 *resourceID* 值，以便正确区分每个 VM。*xmlCfg* 值可以更新以支持此功能，如下所示：

	"xmlCfg": "[base64(concat(variables('wadcfgxstart'), variables('wadmetricsresourceid'), concat(parameters('vmNamePrefix'), copyindex()), variables('wadcfgxend')))]", 

MetricAggregation 值 *PT1H* 和 *PT1M* 表示一分钟的聚合和一小时的聚合。

##<a id="wadmetrics-tables-in-storage"></a> 存储中的 WADMetrics 表

上述指标配置将在你的诊断存储帐户中生成具有以下命名约定的表：

- **WADMetrics**：所有 WADMetrics 表的标准前缀
- **PT1H** 或 **PT1M**：表示表中包含 1 小时或 1 分钟的聚合数据
- **P10D**：表示表中包含其开始收集数据起 10 天内的数据
- **V2S**：字符串常量
- **yyyymmdd**：表开始收集数据的日期

示例：*WADMetricsPT1HP10DV2S20151108* 将包含从 2015 年 11 月 11 日开始 10 天内长达一小时的聚合指标数据

每个 WADMetrics 表都包含以下列：

- **PartitionKey**：partitionkey 基于 *resourceID* 值构建，用于唯一标识 VM 资源，例如：002Fsubscriptions:<subscriptionID>:002FresourceGroups:002F<ResourceGroupName>:002Fproviders:002FMicrosoft:002ECompute:002FvirtualMachines:002F<vmName>  
- **RowKey**：采用 <Descending time tick>:<Performance Counter Name> 格式。递减时间刻度计算公式为最大时间刻度减去聚合期的开始时间。例如，如果取样期间从 2015 年 11 月 10 日 00:00Hrs UTC 开始，则计算公式为：DateTime.MaxValue.Ticks - (new DateTime(2015,11,10,0,0,0,DateTimeKind.Utc).Ticks)。对于内存可用字节性能计数器，行键如下所示：2519551871999999999\_\_:005CMemory:005CAvailable:0020Bytes
- **CounterName**：是性能计数器的名称。它与 xml 配置中定义的 *counterSpecifier* 相匹配。
- **Maximum**：聚合期间性能计数器的最大值。
- **Minimum**：聚合期间性能计数器的最小值。
- **Total**：聚合期间报告的性能计数器的所有值的总和。
- **Count**：针对性能计数器报告的值总数。
- **Average**：聚合期间性能计数器的平均（总计/计数）值。


## 后续步骤

- 有关具有诊断扩展的 Windows 虚拟机的完整示例模板，请参阅 [201-vm-monitoring-diagnostics-extension](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-monitoring-diagnostics-extension)   
- 使用 [Azure PowerShell](/documentation/articles/virtual-machines-windows-ps-manage/) 或 [Azure 命令行](/documentation/articles/virtual-machines-linux-cli-deploy-templates/)部署资源管理器模板
- 了解有关[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates/)的详细信息

<!---HONumber=Mooncake_0118_2016-->