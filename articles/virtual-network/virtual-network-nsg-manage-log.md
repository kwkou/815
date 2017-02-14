<properties
    pageTitle="监视 NSG 的操作、事件和计数器 | Azure"
    description="了解如何为 NSG 启用计数器、事件和操作日志记录"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager" />
<tags
    ms.assetid="2e699078-043f-48bd-8aa8-b011a32d98ca"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="01/03/2017"
    wacn.date="02/10/2017"
    ms.author="jdial" />  


# 网络安全组 (NSG) 的日志分析

可以为 NSG 启用以下诊断日志类别：

* **事件：**包含根据 MAC 地址向 VM 和实例角色应用 NSG 规则时所针对的条目。每隔 60 秒收集一次这些规则的状态。
* **规则计数器：**包含的条目适用于应用每个 NSG 规则以拒绝或允许流量的次数。

> [AZURE.NOTE]
诊断日志仅适用于通过 Azure Resource Manager 部署模型部署的 NSG。对于通过经典部署模型部署的 NSG，不能启用诊断日志记录。若要更好地了解两种模型，请参阅[了解 Azure 部署模型](/documentation/articles/resource-manager-deployment-model/)一文。

对于通过任一 Azure 部署模型创建的 NSG，默认情况下启用活动日志记录（以前称为审核或操作日志）。若要确定已针对活动日志中的 NSG 完成了哪些操作，请查找包含以下资源类型的条目：Microsoft.ClassicNetwork/networkSecurityGroups、Microsoft.ClassicNetwork/networkSecurityGroups/securityRules、Microsoft.Network/networkSecurityGroups、Microsoft.Network/networkSecurityGroups/securityRules。若要详细了解活动日志，请阅读 [Azure 活动日志概述](/documentation/articles/monitoring-overview-activity-logs/)一文。

## 启用诊断日志记录

对于*每个* 需要为其收集数据的 NSG，必须启用诊断日志记录。如果还没有 NSG，请根据[创建网络安全组](/documentation/articles/virtual-networks-create-nsg-arm-pportal/)一文中的步骤创建一个。可以使用以下任意方法启用 NSG 诊断日志记录：

### Azure 门户预览

若要使用门户来启用日志记录，请登录到[门户](https://portal.azure.cn)。单击“更多服务”，然后键入“网络安全组”。选择要为其启用日志记录的 NSG。选择 **NetworkSecurityGroupEvent**、**NetworkSecurityGroupRuleCounter**，或者两类日志都选择。

### PowerShell

评估以下信息，然后输入文中的一个命令：

- 若要确定适用于 `-ResourceId` 参数的值，可根据需要替换以下 [text]，然后输入命令 `Get-AzureRmNetworkSecurityGroup -Name [nsg-name] -ResourceGroupName [resource-group-name]`。命令中的 ID 输出看起来类似于 */subscriptions/[订阅 ID]/resourceGroups/[资源组]/providers/Microsoft.Network/networkSecurityGroups/[NSG 名称]*。
- 如果只需从日志类别收集数据，可将 `-Categories [category]` 添加到本文中命令的末尾，其中，类别为 *NetworkSecurityGroupEvent* 或 *NetworkSecurityGroupRuleCounter*。如果不使用 `-Categories` 参数，则可为两种日志类别启用数据收集。

### Azure 命令行界面 (CLI)

评估以下信息，然后输入文中的一个命令：

- 若要确定适用于 `-ResourceId` 参数的值，可根据需要替换以下 [text]，然后输入命令 `azure network nsg show [resource-group-name] [nsg-name]`。命令中的 ID 输出看起来类似于 */subscriptions/[订阅 ID]/resourceGroups/[资源组]/providers/Microsoft.Network/networkSecurityGroups/[NSG 名称]*。
- 如果只需从日志类别收集数据，可将 `-Categories [category]` 添加到本文中命令的末尾，其中，类别为 *NetworkSecurityGroupEvent* 或 *NetworkSecurityGroupRuleCounter*。如果不使用 `-Categories` 参数，则可为两种日志类别启用数据收集。

## 记录的数据

将为两种日志写入 JSON 格式的数据。针对每个日志类型写入的特定数据将列在以下各节：

### 事件日志
此日志包含的信息涉及哪些 NSG 规则根据 MAC 地址应用到 VM 和云服务角色实例。以下示例数据是针对每个事件记录的：

    {
        "time": "[DATE-TIME]",
        "systemId": "007d0441-5d6b-41f6-8bfd-930db640ec03",
        "category": "NetworkSecurityGroupEvent",
        "resourceId": "/SUBSCRIPTIONS/[SUBSCRIPTION-ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG-NAME]",
        "operationName": "NetworkSecurityGroupEvents",
        "properties": {
            "vnetResourceGuid":"{5E8AEC16-C728-441F-B0CA-B791E1DBC2F4}",
            "subnetPrefix":"192.168.1.0/24",
            "macAddress":"00-0D-3A-92-6A-7C",
            "primaryIPv4Address":"192.168.1.4",
            "ruleName":"UserRule_default-allow-rdp",
            "direction":"In",
            "priority":1000,
            "type":"allow",
            "conditions":{
                "protocols":"6",
                "destinationPortRange":"3389-3389",
                "sourcePortRange":"0-65535",
                "sourceIP":"0.0.0.0/0",
                "destinationIP":"0.0.0.0/0"
                }
            }
    }

### 规则计数器日志

此日志包含的信息涉及每个应用到资源的规则。每次应用规则时，都会记录以下示例数据：

    {
        "time": "[DATE-TIME]",
        "systemId": "007d0441-5d6b-41f6-8bfd-930db640ec03",
        "category": "NetworkSecurityGroupRuleCounter",
        "resourceId": "/SUBSCRIPTIONS/[SUBSCRIPTION ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME]TESTRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG-NAME]",
        "operationName": "NetworkSecurityGroupCounters",
        "properties": {
            "vnetResourceGuid":"{5E8AEC16-C728-441F-B0CA-791E1DBC2F4}",
            "subnetPrefix":"192.168.1.0/24",
            "macAddress":"00-0D-3A-92-6A-7C",
            "primaryIPv4Address":"192.168.1.4",
            "ruleName":"UserRule_default-allow-rdp",
            "direction":"In",
            "type":"allow",
            "matchedConnections":125
            }
    }

<!---HONumber=Mooncake_0206_2017-->
<!--Update_Description: delete contents about audit log and add PowerShell and CLI steps of diagnostic log-->