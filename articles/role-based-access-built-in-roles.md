<properties
	pageTitle="RBAC：内置角色 | Azure"
	description="本主题介绍适用于基于角色的访问控制 (RBAC) 的内置角色。"
	services="active-directory"
	documentationCenter=""
	authors="kgremban"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.date="05/20/2016"
	wacn.date="07/05/2016"/>

#RBAC：内置角色

Azure 基于角色的访问控制 (RBAC) 附带以下可分配到用户、组和服务的内置角色。不能修改内置角色的定义。但是，可以创建 [Azure RBAC 中的自定义角色](/documentation/articles/role-based-access-control-custom-roles/)，以满足你组织的特定需要。

## Azure 中的角色

下表提供内置角色的简短说明。单击角色名称查看该角色**操作**和**不操作**的详细列表。**操作**属性指定对 Azure 资源允许的操作。操作字符串可以使用通配符。**不操作**属性指定从允许的操作中排除的操作。

>[AZURE.NOTE] Azure 角色定义不断演化。本文尽可能地保持处于最新状态，但你总是可在 Azure PowerShell 中找到最新的角色定义。若适用，请使用 cmdlet `(get-azurermroledefinition "<role name>").actions` 或 `(get-azurermroledefinition "<role name>").notactions`。

| 角色名称 | 说明 |
| --------- | ----------- |
| [API 管理服务参与者](#api-management-service-contributor) | 可管理 API 管理服务 |
| [Application Insights 组件参与者](#application-insights-component-contributor) | 可管理 Application Insights 组件 |
| [自动化运算符](#automation-operator) | 能够启动、停止、暂停和继续执行作业 |
| [BizTalk 参与者](#biztalk-contributor) | 可管理 BizTalk 服务 |
| [ClearDB MySQL DB 参与者](#cleardb-mysql-db-contributor) | 可管理 ClearDB MySQL 数据库 |
| [参与者](#contributor) | 可管理除访问权限以外的一切内容。 |
| [数据工厂参与者](#data-factory-contributor) | 可管理数据工厂 |
| [DevTest 实验室用户](#devtest-labs-user) | 可查看一切内容，并可连接、启动、重启和关闭虚拟机 |
| [DocumentDB 帐户参与者](#documentdb-account-contributor) | 可管理 DocumentDB 帐户 |
| [Intelligent Systems 帐户参与者](#intelligent-systems-account-contributor) | 可管理 Intelligent Systems 帐户 |
| [网络参与者](#network-contributor) | 可管理所有网络资源 |
| [New elic APM 帐户参与者](#new-relic-apm-account-contributor) | 可管理 New Relic 应用程序性能管理帐户和应用程序 |
| [所有者](#owner) | 可管理一切内容（包括访问权限） |
| [读者](#reader) | 可查看一切内容，但不可作出更改 |
| [Redis Cache 参与者](#redis-cache-contributor]) | 可管理 Redis 缓存 |
| [计划程序作业集合参与者](#scheduler-job-collections-contributor) | 可管理计划程序作业集合 |
| [搜索服务参与者](#search-service-contributor) | 可管理搜索服务 |
| [安全管理器](#security-manager) | 可管理安全组件、安全策略和虚拟机 |
| [SQL DB 参与者](#sql-db-contributor) | 可管理 SQL 数据库，但不包括其安全性相关的策略 |
| [SQL 安全管理器](#sql-security-manager) | 可管理 SQL 服务器和数据库与安全性相关的策略 |
| [SQL Server 参与者](#sql-server-contributor) | 可管理 SQL 服务器和数据库，但不包括其安全性相关的策略 |
| [经典存储帐户参与者](#classic-storage-account-contributor) | 可管理经典存储帐户 |
| [存储帐户参与者](#storage-account-contributor) | 可管理存储帐户 |
| [用户访问管理员](#user-access-administrator) | 可管理用户对 Azure 资源的访问权限 |
| [经典虚拟机参与者](#classic-virtual-machine-contributor) | 可管理经典虚拟机，但不包括与其连接的虚拟网络或存储帐户 |
| [虚拟机参与者](#virtual-machine-contributor) | 可管理虚拟机，但不包括与其连接的虚拟网络或存储帐户 |
| [经典网络参与者](#classic-network-contributor) | 可管理经典虚拟网络和保留 IP |
| [Web 计划参与者](#web-plan-contributor) | 可管理 Web 计划 |
| [网站参与者](#website-contributor) | 可管理网站，但不包括与其连接的 Web 计划 |

## 角色权限
下表描述授予每个角色的特定权限。这可能包括授予权限的**操作**和限制权限的**不操作**。

### API 管理服务参与者
可管理 API 管理服务

| **操作** | |
| ------- | ------ |
| Microsoft.ApiManagement/Service/* | 创建和管理 API 管理服务 |
| Microsoft.Authorization/*/read | 读取授权 |
| Microsoft.Insights/alertRules/* | 创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取角色和角色分配 |
| Microsoft.Support/* | 创建和管理支持票证 |

### Application Insights 组件参与者
可管理 Application Insights 组件

| **操作** | |
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取角色和角色分配 |
| Microsoft.Insights/alertRules/* | 创建和管理警报规则 |
| Microsoft.Insights/components/* | 创建和管理 Insights 组件 |
| Microsoft.Insights/webtests/* | 创建和管理 Web 测试 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 |
| Microsoft.Support/* | 创建和管理支持票证 |

### 自动化运算符
能够启动、停止、暂停和继续执行作业

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取角色和角色分配 |
| Microsoft.Automation/automationAccounts/jobs/read | 读取自动化帐户作业 |
| Microsoft.Automation/automationAccounts/jobs/resume/action | 继续自动化帐户作业 |
| Microsoft.Automation/automationAccounts/jobs/stop/action | 停止自动化帐户作业 |
| Microsoft.Automation/automationAccounts/jobs/streams/read | 读取自动化帐户作业流 |
| Microsoft.Automation/automationAccounts/jobs/suspend/action | 暂停自动化帐户作业 |
| Microsoft.Automation/automationAccounts/jobs/write | 写入自动化帐户作业 |
| Microsoft.Automation/automationAccounts/jobSchedules/read | 读取自动化帐户作业计划 |
| Microsoft.Automation/automationAccounts/jobSchedules/write | 读取自动化帐户作业计划 |
| Microsoft.Automation/automationAccounts/read | 读取自动化帐户 |
| Microsoft.Automation/automationAccounts/runbooks/read | 读取自动化 Runbook |
| Microsoft.Automation/automationAccounts/schedules/read | 读取自动化帐户计划 |
| Microsoft.Automation/automationAccounts/schedules/write | 写入自动化帐户计划 |
| Microsoft.Insights/components/* | 创建和管理 Insights 组件 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 |
| Microsoft.Support/* | 创建和管理支持票证 |

### BizTalk 参与者
可管理 BizTalk 服务

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取角色和角色分配 |
| Microsoft.BizTalkServices/BizTalk/* | 创建和管理 BizTalk 服务 |
| Microsoft.Insights/alertRules/* | 创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 |
| Microsoft.Support/* | 创建和管理支持票证 |

### ClearDB MySQL DB 参与者
可管理 ClearDB MySQL 数据库

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取角色和角色分配 |
| Microsoft.Insights/alertRules/* | 创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 |
| Microsoft.Support/* | 创建和管理支持票证 |
| successbricks.cleardb/databases/* | 创建和管理 ClearDB MySQL 数据库 |

### 参与者
可管理除访问权限以外的一切内容

| **操作** ||
| ------- | ------ |
| * | 创建和管理所有类型的资源 |

| **不操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/Write | 无法创建角色和角色分配 |
| Microsoft.Authorization/*/Delete | 无法删除角色和角色分配 |

### 数据工厂参与者
可管理数据工厂

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取角色和角色分配 |
| Microsoft.DataFactory/dataFactories/* | 管理数据工厂 |
| Microsoft.Insights/alertRules/* | 创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 |
| Microsoft.Support/* | 创建和管理支持票证 |

### DevTest 实验室用户
可查看一切内容，并可连接、启动、重启和关闭虚拟机

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取角色和角色分配 |
| Microsoft.Compute/availabilitySets/read | 读取可用性集属性 |
| Microsoft.Compute/virtualMachines/*/read | 读取虚拟机属性（VM 大小、运行时状态、VM 扩展等） |
| Microsoft.Compute/virtualMachines/deallocate/action | 解除分配虚拟机 |
| Microsoft.Compute/virtualMachines/read | 读取虚拟机属性 |
| Microsoft.Compute/virtualMachines/restart/action | 重启虚拟机 |
| Microsoft.Compute/virtualMachines/start/action | 启动虚拟机 |
| Microsoft.DevTestLab/*/read | 读取实验室属性 |
| Microsoft.DevTestLab/labs/createEnvironment/action | 创建实验室环境 |
| Microsoft.DevTestLab/labs/formulas/delete | 删除公式 |
| Microsoft.DevTestLab/labs/formulas/read | 读取公式 |
| Microsoft.DevTestLab/labs/formulas/write | 添加或修改公式 |
| Microsoft.DevTestLab/labs/policySets/evaluatePolicies/action | 评估实验室策略 |
| Microsoft.Network/loadBalancers/backendAddressPools/join/action | 加入负载平衡器后端地址池 |
| Microsoft.Network/loadBalancers/inboundNatRules/join/action | 加入负载平衡器入站 NAT 规则 |
| Microsoft.Network/networkInterfaces/*/read | 读取网络接口（例如此网络接口所属的所有负载平衡器）的属性 |
| Microsoft.Network/networkInterfaces/join/action | 将虚拟机连接到网络接口 |
| Microsoft.Network/networkInterfaces/read | 读取网络接口 |
| Microsoft.Network/networkInterfaces/write | 写入网络接口 |
| Microsoft.Network/publicIPAddresses/*/read | 读取公共 IP 地址的属性 |
| Microsoft.Network/publicIPAddresses/join/action | 加入公共 IP 地址 |
| Microsoft.Network/publicIPAddresses/read | 读取网络公共 IP 地址 |
| Microsoft.Network/virtualNetworks/subnets/join/action | 加入虚拟网络 |
| Microsoft.Resources/deployments/operations/read | 读取部署操作 |
| Microsoft.Resources/deployments/read | 读取部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 |
| Microsoft.Storage/storageAccounts/listKeys/action | 列出存储帐户密钥 |

### DocumentDB 帐户参与者
可管理 DocumentDB 帐户

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取角色和角色分配 |
| Microsoft.DocumentDb/databaseAccounts/* | 创建和管理 DocumentDB 帐户 |
| Microsoft.Insights/alertRules/* | 创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 |
| Microsoft.Support/* | 创建和管理支持票证 |

### Intelligent Systems 帐户参与者
可管理 Intelligent Systems 帐户

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取角色和角色分配 |
| Microsoft.Insights/alertRules/* | 创建和管理警报规则 |
| Microsoft.IntelligentSystems/accounts/* | 创建和管理智能系统帐户 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 |
| Microsoft.Support/* | 创建和管理支持票证 |

### 网络参与者
可管理所有网络资源

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取角色和角色分配 |
| Microsoft.Insights/alertRules/* | 创建和管理警报规则 |
| Microsoft.Network/* | 创建并管理网络 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 |
| Microsoft.Support/* | 创建和管理支持票证 |

### New elic APM 帐户参与者
可管理 New Relic 应用程序性能管理帐户和应用程序

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取角色和角色分配 |
| Microsoft.Insights/alertRules/* | 创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 |
| Microsoft.Support/* | 创建和管理支持票证 |
| NewRelic.APM/accounts/* | 创建并管理 New Relic 应用程序性能管理帐户 |

### 所有者
可管理一切内容（包括访问权限）

| **操作** ||
| ------- | ------ |
| * | 创建和管理所有类型的资源 |

### 读取器
可查看一切内容，但不可作出更改

| **操作** ||
| ------- | ------ |
| **/读取 | 读取除机密以外所有类型的资源。|

### Redis Cache 参与者
可管理 Redis 缓存

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取角色和角色分配 |
| Microsoft.Cache/redis/* | 创建和管理 Redis 缓存 |
| Microsoft.Insights/alertRules/* | 创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 |
| Microsoft.Support/* | 创建和管理支持票证 |

### 计划程序作业集合参与者
可管理计划程序作业集合

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取角色和角色分配 |
| Microsoft.Insights/alertRules/* | 创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 | Microsoft.Scheduler/jobcollections/* | 创建和管理作业集合 |
| Microsoft.Support/* | 创建和管理支持票证 |

### 搜索服务参与者
可管理搜索服务

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取角色和角色分配 |
| Microsoft.Insights/alertRules/* | 创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 | Microsoft.Search/searchServices/* | 创建和管理搜索服务 |
| Microsoft.Support/* | 创建和管理支持票证 |

### 安全管理器
可管理安全组件、安全策略和虚拟机

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取角色和角色分配 |
| Microsoft.ClassicCompute/*/read | 读取经典计算虚拟机的配置信息 |
| Microsoft.ClassicCompute/virtualMachines/*/write | 为虚拟机写入配置 |
| Microsoft.ClassicNetwork/*/read | 读取有关经典网络的配置信息 |
| Microsoft.Insights/alertRules/* | 创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 | Microsoft.Security/* | 创建和管理安全组件和策略 |
| Microsoft.Support/* | 创建和管理支持票证 |

### SQL DB 参与者
可管理 SQL 数据库，但不包括其安全性相关的策略

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取角色和角色分配 |
| Microsoft.Insights/alertRules/* | 创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 | Microsoft.Sql/servers/databases/* | 创建和管理 SQL 数据库 |
| Microsoft.Sql/servers/read | 读取 SQL Server |
| Microsoft.Support/* | 创建和管理支持票证 |

| **不操作** ||
| ------- | ------ |
| Microsoft.Sql/servers/databases/auditingPolicies/* | 无法编辑审核策略 |
| Microsoft.Sql/servers/databases/auditingSettings/* | 无法编辑审核设置 |
| Microsoft.Sql/servers/databases/connectionPolicies/* | 无法编辑连接策略 |
| Microsoft.Sql/servers/databases/dataMaskingPolicies/* | 无法编辑数据屏蔽策略 |
| Microsoft.Sql/servers/databases/securityAlertPolicies/* | 无法编辑安全警报策略 |
| Microsoft.Sql/servers/databases/securityMetrics/* | 无法编辑安全度量值 |

### SQL 安全管理器
可管理 SQL 服务器和数据库与安全性相关的策略

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取 Microsoft 授权 |
| Microsoft.Insights/alertRules/* | 创建和管理 Insights 警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 | Microsoft.Sql/servers/auditingPolicies/* | 创建和管理 SQL 服务器审核策略 |
| Microsoft.Sql/servers/auditingSettings/* | 创建和管理 SQL 服务器审核设置 |
| Microsoft.Sql/servers/databases/auditingPolicies/* | 创建和管理 SQL 服务器数据库审核策略 |
| Microsoft.Sql/servers/databases/auditingSettings/* | 创建和管理 SQL 服务器数据库审核设置 |
| Microsoft.Sql/servers/databases/connectionPolicies/* | 创建和管理 SQL 服务器数据库连接策略 |
| Microsoft.Sql/servers/databases/dataMaskingPolicies/* | 创建和管理 SQL 服务器数据库数据屏蔽策略 |
| Microsoft.Sql/servers/databases/read | 读取 SQL 数据库 |
| Microsoft.Sql/servers/databases/schemas/read | 读取 SQL 服务器数据库架构 |
| Microsoft.Sql/servers/databases/schemas/tables/columns/read | 读取 SQL 服务器数据库表列 |
| Microsoft.Sql/servers/databases/schemas/tables/read | 读取 SQL 服务器数据库表 |
| Microsoft.Sql/servers/databases/securityAlertPolicies/* | 创建和管理 SQL 服务器数据库安全警报策略 |
| Microsoft.Sql/servers/databases/securityMetrics/* | 创建和管理 SQL 服务器数据库安全度量值 |
| Microsoft.Sql/servers/read | 读取 SQL Server |
| Microsoft.Sql/servers/securityAlertPolicies/* | 创建和管理 SQL 服务器安全警报策略 |
| Microsoft.Support/* | 创建和管理支持票证 |

### SQL Server 参与者
可管理 SQL 服务器和数据库，但不包括其安全性相关的策略

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取授权|
| Microsoft.Insights/alertRules/* | 创建和管理 Insights 警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 | Microsoft.Sql/servers/* | 创建和管理 SQL 服务器 |
| Microsoft.Support/* | 创建和管理支持票证 |

| **不操作** ||
| ------- | ------ |
| Microsoft.Sql/servers/auditingPolicies/* | 无法编辑 SQL 服务器审核策略 |
| Microsoft.Sql/servers/auditingSettings/* | 无法编辑 SQL 服务器审核设置 |
| Microsoft.Sql/servers/databases/auditingPolicies/* | 无法编辑 SQL 服务器数据库审核策略 |
| Microsoft.Sql/servers/databases/auditingSettings/* | 无法编辑 SQL 服务器数据库审核设置 |
| Microsoft.Sql/servers/databases/connectionPolicies/* | 无法编辑 SQL 服务器数据库连接策略 |
| Microsoft.Sql/servers/databases/dataMaskingPolicies/* | 无法编辑 SQL 服务器数据库数据屏蔽策略 |
| Microsoft.Sql/servers/databases/securityAlertPolicies/* | 无法编辑 SQL 服务器数据库安全警报策略 |
| Microsoft.Sql/servers/databases/securityMetrics/* | 无法编辑 SQL 服务器数据库安全度量值 |
| Microsoft.Sql/servers/securityAlertPolicies/* | 无法编辑 SQL 服务器安全警报策略 |

### 经典存储帐户参与者
可管理经典存储帐户

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取授权 |
| Microsoft.ClassicStorage/storageAccounts/* | 创建和管理存储帐户 |
| Microsoft.Insights/alertRules/* | 创建和管理 Insights 警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 | Microsoft.Support/* | 创建和管理支持票证 |

### 存储帐户参与者
可管理但不能访问存储帐户。

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取所有授权 |
| Microsoft.Insights/alertRules/* | 创建和管理 Insights 警报规则 |
| Microsoft.Insights/diagnosticSettings/* | 管理诊断设置 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 | Microsoft.Storage/storageAccounts/* | 创建和管理存储帐户 |
| Microsoft.Support/* | 创建和管理支持票证 |

### 用户访问管理员
可管理用户对 Azure 资源的访问权限

| **操作** ||
| ------- | ------ |
| */read | 读取除机密以外所有类型的资源 |
| Microsoft.Authorization/* | 管理授权 |
| Microsoft.Support/* | 创建和管理支持票证 |

### 经典虚拟机参与者
可管理经典虚拟机，但不包括与其连接的虚拟网络或存储帐户

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取授权 |
| Microsoft.ClassicCompute/domainNames/* | 创建和管理经典计算域名 |
| Microsoft.ClassicCompute/virtualMachines/* | 创建和管理虚拟机 |
| Microsoft.ClassicNetwork/networkSecurityGroups/join/action | 加入网络安全组 |
| Microsoft.ClassicNetwork/reservedIps/link/action | 链接保留 IP |
| Microsoft.ClassicNetwork/reservedIps/read | 读取保留 IP 地址 |
| Microsoft.ClassicNetwork/virtualNetworks/join/action | 加入虚拟网络 |
| Microsoft.ClassicNetwork/virtualNetworks/read | 读取虚拟网络 |
| Microsoft.ClassicStorage/storageAccounts/disks/read | 读取存储帐户磁盘 |
| Microsoft.ClassicStorage/storageAccounts/images/read | 读取存储帐户图像 |
| Microsoft.ClassicStorage/storageAccounts/listKeys/action | 列出存储帐户密钥 |
| Microsoft.ClassicStorage/storageAccounts/read | 读取经典存储帐户 |
| Microsoft.Insights/alertRules/* | 创建和管理 Insights 警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 | Microsoft.Support/* | 创建和管理支持票证 |

### 虚拟机参与者
可管理虚拟机，但不包括与其连接的虚拟网络或存储帐户

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取授权 |
| Microsoft.Compute/availabilitySets/* | 创建和管理计算可用性集 |
| Microsoft.Compute/locations/* | 创建和管理计算位置 |
| Microsoft.Compute/virtualMachines/* | 创建和管理虚拟机 |
| Microsoft.Compute/virtualMachineScaleSets/* | 创建和管理虚拟机规模集 |
| Microsoft.Insights/alertRules/* | 创建和管理 Insights 警报规则 |
| Microsoft.Network/applicationGateways/backendAddressPools/join/action | 加入网络应用程序网关后端地址池 |
| Microsoft.Network/loadBalancers/backendAddressPools/join/action | 加入负载平衡器后端地址池 |
| Microsoft.Network/loadBalancers/inboundNatPools/join/action | 加入负载平衡器入站 NAT 池 |
| Microsoft.Network/loadBalancers/inboundNatRules/join/action | 加入负载平衡器入站 NAT 规则 |
| Microsoft.Network/loadBalancers/read | 读取负载平衡器 |
| Microsoft.Network/locations/* | 创建和管理网络位置 |
| Microsoft.Network/networkInterfaces/* | 创建和管理网络接口 |
| Microsoft.Network/networkSecurityGroups/join/action | 加入网络安全组 |
| Microsoft.Network/networkSecurityGroups/read | 读取网络安全组 |
| Microsoft.Network/publicIPAddresses/join/action | 加入网络公共 IP 地址 |
| Microsoft.Network/publicIPAddresses/read | 读取网络公共 IP 地址 |
| Microsoft.Network/virtualNetworks/read | 读取虚拟网络 |
| Microsoft.Network/virtualNetworks/subnets/join/action | 加入虚拟网络子网 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 | Microsoft.Storage/storageAccounts/listKeys/action | 列出存储帐户密钥 |
| Microsoft.Storage/storageAccounts/read | 读取存储帐户 |
| Microsoft.Support/* | 创建和管理支持票证 |

### 经典网络参与者
可管理经典虚拟网络和保留 IP

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取授权 |
| Microsoft.ClassicNetwork/* | 创建和管理经典网络 |
| Microsoft.Insights/alertRules/* | 创建和管理 Insights 警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 | Microsoft.Support/* | 创建和管理支持票证 |

### Web 计划参与者
可管理 Web 计划

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取授权 |
| Microsoft.Insights/alertRules/* | 创建和管理 Insights 警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 | Microsoft.Support/* | 创建和管理支持票证 |
| Microsoft.Web/serverFarms/* | 创建和管理服务器场 |

### 网站参与者
可管理网站，但不包括与其连接的 Web 计划

| **操作** ||
| ------- | ------ |
| Microsoft.Authorization/*/read | 读取授权 |
| Microsoft.Insights/alertRules/* | 创建和管理 Insights 警报规则 |
| Microsoft.Insights/components/* | 创建和管理 Insights 组件 |
| Microsoft.ResourceHealth/availabilityStatuses/read | 读取资源的运行状况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 | Microsoft.Support/* | 创建和管理支持票证 |
| Microsoft.Web/certificates/* | 创建和管理网站证书 |
| Microsoft.Web/listSitesAssignedToHostName/read | 读取分配到主机名的站点 |
| Microsoft.Web/serverFarms/join/action | 加入服务器场 |
| Microsoft.Web/serverFarms/read | 读取服务器场 |
| Microsoft.Web/sites/* | 创建和管理网站 |

## 另请参阅
- [基于角色的访问控制](/documentation/articles/role-based-access-control-configure/)：在 Azure 门户中开始使用 RBAC。
- [Azure RBAC 中的自定义角色](/documentation/articles/role-based-access-control-custom-roles/)：了解如何创建自定义角色，以满足访问需要。
- [创建访问权限更改历史记录报告](/documentation/articles/role-based-access-control-access-change-history-report/)：记录 RBAC 中的角色分配更改。
- [基于角色的访问控制故障排除](/documentation/articles/role-based-access-control-troubleshooting/)：获取解决常见问题的建议。

<!---HONumber=Mooncake_0627_2016-->