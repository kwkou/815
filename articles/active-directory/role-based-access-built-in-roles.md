<properties
    pageTitle="操作和不操作 - Azure RBAC 中的角色 | Azure"
    description="本主题介绍适用于基于角色的访问控制 (RBAC) 的内置角色。"
    services="active-directory"
    documentationcenter=""
    author="kgremban"
    manager="femila"
    editor="" />
<tags
    ms.assetid="b547c5a5-2da2-4372-9938-481cb962d2d6"
    ms.service="active-directory"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="identity"
    ms.date="02/21/2017"
    wacn.date="04/05/2017"
    ms.author="kgremban" />  


# Azure 基于角色的访问控制的内置角色
Azure 基于角色的访问控制 (RBAC) 附带以下可分配到用户、组和服务的内置角色。不能修改内置角色的定义。但是，可以创建 [Azure RBAC 中的自定义角色](/documentation/articles/role-based-access-control-custom-roles/)，以满足组织的特定需要。

## Azure 中的角色
下表提供内置角色的简短说明。单击角色名称查看该角色**操作**和**不操作**的详细列表。**操作**属性指定对 Azure 资源允许的操作。操作字符串可以使用通配符。**不操作**属性指定从允许的操作中排除的操作。

> [AZURE.NOTE]
Azure 角色定义不断演化。本文尽可能地保持处于最新状态，但你总是可在 Azure PowerShell 中找到最新的角色定义。若适用，请使用 cmdlet `(get-azurermroledefinition "<role name>").actions` 或 `(get-azurermroledefinition "<role name>").notactions`。
>
>

| 角色名称 | 说明 |
| --- | --- |
| [API 管理服务参与者](#api-management-service-contributor) |可管理 API 管理服务 |
| [Application Insights 组件参与者](#application-insights-component-contributor) |可管理 Application Insights 组件 |
| [自动化运算符](#automation-operator) |能够启动、停止、暂停和继续执行作业 |
| [备份参与者](#backup-contributor) | 可以管理恢复服务保管库中的备份 |
| [备份操作员](#backup-operator) | 可以管理恢复服务保管库中的备份，删除备份除外 |
| [备份读者](#backup-reader) | 可以查看所有备份管理服务 |
| [BizTalk 参与者](#biztalk-contributor) |可管理 BizTalk 服务 |
| [ClearDB MySQL DB 参与者](#cleardb-mysql-db-contributor) |可管理 ClearDB MySQL 数据库 |
| [参与者](#contributor) |可管理除访问权限以外的一切内容。 |
| [数据工厂参与者](#data-factory-contributor) |可创建和管理数据工厂，以及它们包含的子资源。 |
| [DevTest 实验室用户](#devtest-labs-user) |可查看一切内容，并可连接、启动、重启和关闭虚拟机 |
| [DNS 区域参与者](#dns-zone-contributor) |可以管理 DNS 区域和记录 |
| [DocumentDB 帐户参与者](#documentdb-account-contributor) |可管理 DocumentDB 帐户 |
| [Intelligent Systems 帐户参与者](#intelligent-systems-account-contributor) |可管理 Intelligent Systems 帐户 |
| [网络参与者](#network-contributor) |可管理所有网络资源 |
| [New elic APM 帐户参与者](#new-relic-apm-account-contributor) |可管理 New Relic 应用程序性能管理帐户和应用程序 |
| [所有者](#owner) |可管理一切内容（包括访问权限） |
| [读者](#reader) |可查看一切内容，但不可作出更改 |
| [Redis Cache 参与者](#redis-cache-contributor) |可管理 Redis 缓存 |
| [计划程序作业集合参与者](#scheduler-job-collections-contributor) |可管理计划程序作业集合 |
| [搜索服务参与者](#search-service-contributor) |可管理搜索服务 |
| [安全管理器](#security-manager) |可管理安全组件、安全策略和虚拟机 |
| [SQL DB 参与者](#sql-db-contributor) |可管理 SQL 数据库，但不包括其安全性相关的策略 |
| [SQL 安全管理器](#sql-security-manager) |可管理 SQL 服务器和数据库与安全性相关的策略 |
| [SQL Server 参与者](#sql-server-contributor) |可管理 SQL 服务器和数据库，但不包括其安全性相关的策略 |
| [经典存储帐户参与者](#classic-storage-account-contributor) |可管理经典存储帐户 |
| [存储帐户参与者](#storage-account-contributor) |可管理存储帐户 |
| [用户访问管理员](#user-access-administrator) |可管理用户对 Azure 资源的访问权限 |
| [经典虚拟机参与者](#classic-virtual-machine-contributor) |可管理经典虚拟机，但不包括与其连接的虚拟网络或存储帐户 |
| [虚拟机参与者](#virtual-machine-contributor) |可管理虚拟机，但不包括与其连接的虚拟网络或存储帐户 |
| [经典网络参与者](#classic-network-contributor) |可管理经典虚拟网络和保留 IP |
| [Web 计划参与者](#web-plan-contributor) |可管理 Web 计划 |
| [网站参与者](#website-contributor) |可管理网站，但不包括与其连接的 Web 计划 |

## 角色权限
下表描述授予每个角色的特定权限。这可能包括授予权限的 **Actions** 和限制权限的 **NotActions**。

### API 管理服务参与者 <a name="api-management-service-contributor"></a>
可管理 API 管理服务

| **操作** | |
| --- | --- |
| Microsoft.ApiManagement/Service/* |创建和管理 API 管理服务 |
| Microsoft.Authorization/*/read |读取授权 |
| Microsoft.Insights/alertRules/* |创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取角色和角色分配 |
| Microsoft.Support/* |创建和管理支持票证 |

### Application Insights 组件参与者 <a name="application-insights-component-contributor"></a>
可管理 Application Insights 组件

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取角色和角色分配 |
| Microsoft.Insights/alertRules/* |创建和管理警报规则 |
| Microsoft.Insights/components/* |创建和管理 Insights 组件 |
| Microsoft.Insights/webtests/* |创建和管理 Web 测试 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Support/* |创建和管理支持票证 |

### 自动化操作员 <a name="automation-operator"></a>
能够启动、停止、暂停和继续执行作业

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取角色和角色分配 |
| Microsoft.Automation/automationAccounts/jobs/read |读取自动化帐户作业 |
| Microsoft.Automation/automationAccounts/jobs/resume/action |继续自动化帐户作业 |
| Microsoft.Automation/automationAccounts/jobs/stop/action |停止自动化帐户作业 |
| Microsoft.Automation/automationAccounts/jobs/streams/read |读取自动化帐户作业流 |
| Microsoft.Automation/automationAccounts/jobs/suspend/action |暂停自动化帐户作业 |
| Microsoft.Automation/automationAccounts/jobs/write |写入自动化帐户作业 |
| Microsoft.Automation/automationAccounts/jobSchedules/read |读取自动化帐户作业计划 |
| Microsoft.Automation/automationAccounts/jobSchedules/write |读取自动化帐户作业计划 |
| Microsoft.Automation/automationAccounts/read |读取自动化帐户 |
| Microsoft.Automation/automationAccounts/runbooks/read |读取自动化 Runbook |
| Microsoft.Automation/automationAccounts/schedules/read |读取自动化帐户计划 |
| Microsoft.Automation/automationAccounts/schedules/write |写入自动化帐户计划 |
| Microsoft.Insights/components/* |创建和管理 Insights 组件 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Support/* |创建和管理支持票证 |

### 备份参与者 <a name="backup-contributor"></a>
可以管理所有备份管理操作，但创建恢复服务保管库和为他人提供访问权限除外

| **操作** | |
| --- | --- |
| Microsoft.Network/virtualNetworks/read | 读取虚拟网络 |
| Microsoft.RecoveryServices/Vaults/backupFabrics/operationResults/* | 管理备份管理操作的结果 |
| Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/* | 在恢复服务保管库的备份结构内创建和管理备份容器 |
| Microsoft.RecoveryServices/Vaults/backupJobs/* | 创建和管理备份作业 |
| Microsoft.RecoveryServices/Vaults/backupJobsExport/action | 将备份作业导出到 Excel |
| Microsoft.RecoveryServices/Vaults/backupManagementMetaData/* | 创建和管理与备份管理相关的元数据 |
| Microsoft.RecoveryServices/Vaults/backupOperationResults/* | 创建和管理备份管理操作的结果 |
| Microsoft.RecoveryServices/Vaults/backupPolicies/* | 创建和管理备份策略 |
| Microsoft.RecoveryServices/Vaults/backupProtectableItems/* | 创建和管理可以备份的项 |
| Microsoft.RecoveryServices/Vaults/backupProtectedItems/* | 创建和管理备份项 |
| Microsoft.RecoveryServices/Vaults/backupProtectionContainers/* | 创建和管理保存备份项的容器 |
| Microsoft.RecoveryServices/Vaults/certificates/* | 创建和管理与恢复服务保管库中的备份相关的证书 |
| Microsoft.RecoveryServices/Vaults/extendedInformation/* | 创建和管理与保管库相关的扩展信息 | 
| Microsoft.RecoveryServices/Vaults/read | 读取恢复服务保管库 |
| Microsoft.RecoveryServices/Vaults/refreshContainers/* | 管理用于获取新创建的容器的发现操作 |
| Microsoft.RecoveryServices/Vaults/registeredIdentities/* | 创建和管理已注册标识 |
| Microsoft.RecoveryServices/Vaults/usages/* | 创建和管理恢复服务保管库的使用情况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 |
| Microsoft.Storage/storageAccounts/read | 读取存储帐户 |
| Microsoft.Support/* |创建和管理支持票证 |

### 备份操作员 <a name="backup-operator"></a>
可以管理所有备份管理操作，但创建保管库、删除备份和为他人提供访问权限除外

| **操作** | |
| --- | --- |
| Microsoft.Network/virtualNetworks/read | 读取虚拟网络 |
| Microsoft.RecoveryServices/Vaults/backupFabrics/operationResults/read | 读取备份管理操作的结果 |
| Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/operationResults/read | 在保护容器上读取操作结果 |
| Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/backup/action | 对备份项执行按需备份操作 |
| Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/operationResults/read | 读取对备份项执行的操作的结果 |
| Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/operationStatus/read | 读取对备份项执行的操作的状态 |
| Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/read | 读取备份项 |
| Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/recoveryPoints/read | 读取备份项的恢复点 |
| Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/recoveryPoints/restore/action | 使用备份项的恢复点执行还原操作 |
| Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/write | 创建备份项 |
| Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/read | 读取保存备份项的容器 |
| Microsoft.RecoveryServices/Vaults/backupJobs/* | 创建和管理备份作业 |
| Microsoft.RecoveryServices/Vaults/backupJobsExport/action | 将备份作业导出到 Excel |
| Microsoft.RecoveryServices/Vaults/backupManagementMetaData/read | 读取与备份管理相关的元数据 |
| Microsoft.RecoveryServices/Vaults/backupOperationResults/* | 创建和管理备份管理操作的结果 |
| Microsoft.RecoveryServices/Vaults/backupPolicies/operationResults/read | 读取对备份策略执行的操作的结果 |
| Microsoft.RecoveryServices/Vaults/backupPolicies/read | 读取备份策略 |
| Microsoft.RecoveryServices/Vaults/backupProtectableItems/* | 创建和管理可以备份的项 |
| Microsoft.RecoveryServices/Vaults/backupProtectedItems/read | 读取备份项 |
| Microsoft.RecoveryServices/Vaults/backupProtectionContainers/read | 读取保存备份项的备份容器 |
| Microsoft.RecoveryServices/Vaults/extendedInformation/read | 读取与保管库相关的扩展信息 | 
| Microsoft.RecoveryServices/Vaults/extendedInformation/write | 写入与保管库相关的扩展信息 | 
| Microsoft.RecoveryServices/Vaults/read | 读取恢复服务保管库 |
| Microsoft.RecoveryServices/Vaults/refreshContainers/* | 管理用于获取新创建的容器的发现操作 |
| Microsoft.RecoveryServices/Vaults/registeredIdentities/operationResults/read | 读取对保管库的注册项执行的操作的结果 |
| Microsoft.RecoveryServices/Vaults/registeredIdentities/read | 读取保管库的注册项 |
| Microsoft.RecoveryServices/Vaults/registeredIdentities/write | 将注册项写入保管库 |
| Microsoft.RecoveryServices/Vaults/usages/read | 读取恢复服务保管库的使用情况 |
| Microsoft.Resources/deployments/* | 创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read | 读取资源组 |
| Microsoft.Storage/storageAccounts/read | 读取存储帐户 |
| Microsoft.Support/* | 创建和管理支持票证 |

### 备份读者 <a name="backup-reader"></a>
可以监视恢复服务保管库中的备份管理

| **操作** | |
| --- | --- |
| Microsoft.RecoveryServices/Vaults/backupFabrics/operationResults/read | 读取备份管理操作的结果 |
| Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/operationResults/read | 在保护容器上读取操作结果 |
| Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/operationResults/read | 读取对备份项执行的操作的结果 |
| Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/operationStatus/read | 读取对备份项执行的操作的状态 |
| Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/protectedItems/read | 读取备份项 |
| Microsoft.RecoveryServices/Vaults/backupFabrics/protectionContainers/read | 读取保存备份项的容器 |
| Microsoft.RecoveryServices/Vaults/backupJobs/operationResults/read | 读取备份作业的结果 |
| Microsoft.RecoveryServices/Vaults/backupJobs/read | 读取备份作业 |
| Microsoft.RecoveryServices/Vaults/backupJobsExport/action | 将备份作业导出到 Excel |
| Microsoft.RecoveryServices/Vaults/backupManagementMetaData/read | 读取与备份管理相关的元数据 |
| Microsoft.RecoveryServices/Vaults/backupOperationResults/read | 读取备份管理操作结果 |
| Microsoft.RecoveryServices/Vaults/backupPolicies/operationResults/read | 读取对备份策略执行的操作的结果 |
| Microsoft.RecoveryServices/Vaults/backupPolicies/read | 读取备份策略 |
| Microsoft.RecoveryServices/Vaults/backupProtectedItems/read | 读取备份项 |
| Microsoft.RecoveryServices/Vaults/backupProtectionContainers/read | 读取保存备份项的备份容器 |
| Microsoft.RecoveryServices/Vaults/extendedInformation/read | 读取与保管库相关的扩展信息 |
| Microsoft.RecoveryServices/Vaults/read | 读取恢复服务保管库 |
| Microsoft.RecoveryServices/Vaults/refreshContainers/read | 读取用于获取新创建的容器的发现操作的结果 |
| Microsoft.RecoveryServices/Vaults/registeredIdentities/operationResults/read | 读取对保管库的注册项执行的操作的结果 |
| Microsoft.RecoveryServices/Vaults/registeredIdentities/read | 读取保管库的注册项 |
| Microsoft.RecoveryServices/Vaults/usages/read | 读取恢复服务保管库的使用情况 |

### BizTalk 参与者 <a name="biztalk-contributor"></a>
可管理 BizTalk 服务

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取角色和角色分配 |
| Microsoft.BizTalkServices/BizTalk/* |创建和管理 BizTalk 服务 |
| Microsoft.Insights/alertRules/* |创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Support/* |创建和管理支持票证 |

### ClearDB MySQL DB 参与者 <a name="cleardb-mysql-db-contributor"></a>
可管理 ClearDB MySQL 数据库

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取角色和角色分配 |
| Microsoft.Insights/alertRules/* |创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Support/* |创建和管理支持票证 |
| successbricks.cleardb/databases/* |创建和管理 ClearDB MySQL 数据库 |

### 参与者 <a name="contributor"></a>
可管理除访问权限以外的一切内容

| **操作** | |
| --- | --- |
| * |创建和管理所有类型的资源 |

| **不操作** | |
| --- | --- |
| Microsoft.Authorization/*/Delete |无法删除角色和角色分配 |
| Microsoft.Authorization/*/Write |无法创建角色和角色分配 |

### 数据工厂参与者 <a name="data-factory-contributor"></a>
创建和管理数据工厂，以及它们包含的子资源。

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取角色和角色分配 |
| Microsoft.DataFactory/dataFactories/* |创建和管理数据工厂，以及它们包含的子资源。 |
| Microsoft.Insights/alertRules/* |创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Support/* |创建和管理支持票证 |

### 开发测试实验室用户 <a name="devtest-labs-user"></a>
可查看一切内容，并可连接、启动、重启和关闭虚拟机

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取角色和角色分配 |
| Microsoft.Compute/availabilitySets/read |读取可用性集属性 |
| Microsoft.Compute/virtualMachines/*/read |读取虚拟机属性（VM 大小、运行时状态、VM 扩展等） |
| Microsoft.Compute/virtualMachines/deallocate/action |解除分配虚拟机 |
| Microsoft.Compute/virtualMachines/read |读取虚拟机属性 |
| Microsoft.Compute/virtualMachines/restart/action |重启虚拟机 |
| Microsoft.Compute/virtualMachines/start/action |启动虚拟机 |
| Microsoft.DevTestLab/*/read |读取实验室属性 |
| Microsoft.DevTestLab/labs/createEnvironment/action |创建实验室环境 |
| Microsoft.DevTestLab/labs/formulas/delete |删除公式 |
| Microsoft.DevTestLab/labs/formulas/read |读取公式 |
| Microsoft.DevTestLab/labs/formulas/write |添加或修改公式 |
| Microsoft.DevTestLab/labs/policySets/evaluatePolicies/action |评估实验室策略 |
| Microsoft.Network/loadBalancers/backendAddressPools/join/action |加入负载均衡器后端地址池 |
| Microsoft.Network/loadBalancers/inboundNatRules/join/action |加入负载均衡器入站 NAT 规则 |
| Microsoft.Network/networkInterfaces/*/read |读取网络接口（例如，此网络接口所属的所有负载均衡器）的属性 |
| Microsoft.Network/networkInterfaces/join/action |将虚拟机连接到网络接口 |
| Microsoft.Network/networkInterfaces/read |读取网络接口 |
| Microsoft.Network/networkInterfaces/write |写入网络接口 |
| Microsoft.Network/publicIPAddresses/*/read |读取公共 IP 地址的属性 |
| Microsoft.Network/publicIPAddresses/join/action |加入公共 IP 地址 |
| Microsoft.Network/publicIPAddresses/read |读取网络公共 IP 地址 |
| Microsoft.Network/virtualNetworks/subnets/join/action |加入虚拟网络 |
| Microsoft.Resources/deployments/operations/read |读取部署操作 |
| Microsoft.Resources/deployments/read |读取部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Storage/storageAccounts/listKeys/action |列出存储帐户密钥 |

### DNS 区域参与者 <a name="dns-zone-contributor"></a>
可以管理 DNS 区域和记录。

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取角色和角色分配 |
| Microsoft.Insights/alertRules/* |创建和管理警报规则 |
| Microsoft.Network/dnsZones/* |创建和管理 DNS 区域和记录 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Support/* |创建和管理支持票证 |

### DocumentDB 帐户参与者 <a name="documentdb-account-contributor"></a>
可管理 DocumentDB 帐户

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取角色和角色分配 |
| Microsoft.DocumentDb/databaseAccounts/* |创建和管理 DocumentDB 帐户 |
| Microsoft.Insights/alertRules/* |创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Support/* |创建和管理支持票证 |

### Intelligent Systems 帐户参与者 <a name="intelligent-systems-account-contributor"></a>
可管理 Intelligent Systems 帐户

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取角色和角色分配 |
| Microsoft.Insights/alertRules/* |创建和管理警报规则 |
| Microsoft.IntelligentSystems/accounts/* |创建和管理智能系统帐户 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Support/* |创建和管理支持票证 |

### 网络参与者 <a name="network-contributor"></a>
可管理所有网络资源

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取角色和角色分配 |
| Microsoft.Insights/alertRules/* |创建和管理警报规则 |
| Microsoft.Network/* |创建并管理网络 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Support/* |创建和管理支持票证 |

### New Relic APM 帐户参与者 <a name="new-relic-apm-account-contributor"></a>
可管理 New Relic 应用程序性能管理帐户和应用程序

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取角色和角色分配 |
| Microsoft.Insights/alertRules/* |创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Support/* |创建和管理支持票证 |
| NewRelic.APM/accounts/* |创建并管理 New Relic 应用程序性能管理帐户 |

### 所有者 <a name="owner"></a>
可管理一切内容（包括访问权限）

| **操作** | |
| --- | --- |
| * |创建和管理所有类型的资源 |

### 读者 <a name="reader"></a>
可查看一切内容，但不可作出更改

| **操作** | |
| --- | --- |
| */read |读取除密码外的所有类型的资源。 |

### Redis Cache 参与者 <a name="redis-cache-contributor"></a>
可管理 Redis 缓存

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取角色和角色分配 |
| Microsoft.Cache/redis/* |创建和管理 Redis 缓存 |
| Microsoft.Insights/alertRules/* |创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Support/* |创建和管理支持票证 |

### 计划程序作业集合参与者 <a name="scheduler-job-collections-contributor"></a>
可管理计划程序作业集合

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取角色和角色分配 |
| Microsoft.Insights/alertRules/* |创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Scheduler/jobcollections/* |创建和管理作业集合 |
| Microsoft.Support/* |创建和管理支持票证 |

### 搜索服务参与者 <a name="search-service-contributor"></a>
可管理搜索服务

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取角色和角色分配 |
| Microsoft.Insights/alertRules/* |创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Search/searchServices/* |创建和管理搜索服务 |
| Microsoft.Support/* |创建和管理支持票证 |

### 安全管理器 <a name="security-manager"></a>
可管理安全组件、安全策略和虚拟机

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取角色和角色分配 |
| Microsoft.ClassicCompute/*/read |读取经典计算虚拟机的配置信息 |
| Microsoft.ClassicCompute/virtualMachines/*/write |为虚拟机写入配置 |
| Microsoft.ClassicNetwork/*/read |读取有关经典网络的配置信息 |
| Microsoft.Insights/alertRules/* |创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Security/* |创建和管理安全组件和策略 |
| Microsoft.Support/* |创建和管理支持票证 |

### SQL DB 参与者 <a name="sql-db-contributor"></a>
可管理 SQL 数据库，但不包括其安全性相关的策略

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取角色和角色分配 |
| Microsoft.Insights/alertRules/* |创建和管理警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Sql/servers/databases/* |创建和管理 SQL 数据库 |
| Microsoft.Sql/servers/read |读取 SQL Server |
| Microsoft.Support/* |创建和管理支持票证 |

| **不操作** | |
| --- | --- |
| Microsoft.Sql/servers/databases/auditingPolicies/* |无法编辑审核策略 |
| Microsoft.Sql/servers/databases/auditingSettings/* |无法编辑审核设置 |
| Microsoft.Sql/servers/databases/auditRecords/read |无法读取审核记录 |
| Microsoft.Sql/servers/databases/connectionPolicies/* |无法编辑连接策略 |
| Microsoft.Sql/servers/databases/dataMaskingPolicies/* |无法编辑数据屏蔽策略 |
| Microsoft.Sql/servers/databases/securityAlertPolicies/* |无法编辑安全警报策略 |
| Microsoft.Sql/servers/databases/securityMetrics/* |无法编辑安全度量值 |

### SQL 安全管理器 <a name="sql-security-manager"></a>
可管理 SQL 服务器和数据库与安全性相关的策略

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取 Microsoft 授权 |
| Microsoft.Insights/alertRules/* |创建和管理 Insights 警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Sql/servers/auditingPolicies/* |创建和管理 SQL 服务器审核策略 |
| Microsoft.Sql/servers/auditingSettings/* |创建和管理 SQL 服务器审核设置 |
| Microsoft.Sql/servers/databases/auditingPolicies/* |创建和管理 SQL 服务器数据库审核策略 |
| Microsoft.Sql/servers/databases/auditingSettings/* |创建和管理 SQL 服务器数据库审核设置 |
| Microsoft.Sql/servers/databases/auditRecords/read |读取审核记录 |
| Microsoft.Sql/servers/databases/connectionPolicies/* |创建和管理 SQL 服务器数据库连接策略 |
| Microsoft.Sql/servers/databases/dataMaskingPolicies/* |创建和管理 SQL 服务器数据库数据屏蔽策略 |
| Microsoft.Sql/servers/databases/read |读取 SQL 数据库 |
| Microsoft.Sql/servers/databases/schemas/read |读取 SQL 服务器数据库架构 |
| Microsoft.Sql/servers/databases/schemas/tables/columns/read |读取 SQL 服务器数据库表列 |
| Microsoft.Sql/servers/databases/schemas/tables/read |读取 SQL 服务器数据库表 |
| Microsoft.Sql/servers/databases/securityAlertPolicies/* |创建和管理 SQL 服务器数据库安全警报策略 |
| Microsoft.Sql/servers/databases/securityMetrics/* |创建和管理 SQL 服务器数据库安全度量值 |
| Microsoft.Sql/servers/read |读取 SQL Server |
| Microsoft.Sql/servers/securityAlertPolicies/* |创建和管理 SQL 服务器安全警报策略 |
| Microsoft.Support/* |创建和管理支持票证 |

### SQL Server 参与者 <a name="sql-server-contributor"></a>
可管理 SQL 服务器和数据库，但不包括其安全性相关的策略

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取授权 |
| Microsoft.Insights/alertRules/* |创建和管理 Insights 警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Sql/servers/* |创建和管理 SQL 服务器 |
| Microsoft.Support/* |创建和管理支持票证 |

| **不操作** | |
| --- | --- |
| Microsoft.Sql/servers/auditingPolicies/* |无法编辑 SQL 服务器审核策略 |
| Microsoft.Sql/servers/auditingSettings/* |无法编辑 SQL 服务器审核设置 |
| Microsoft.Sql/servers/databases/auditingPolicies/* |无法编辑 SQL 服务器数据库审核策略 |
| Microsoft.Sql/servers/databases/auditingSettings/* |无法编辑 SQL 服务器数据库审核设置 |
| Microsoft.Sql/servers/databases/auditRecords/read |无法读取审核记录 |
| Microsoft.Sql/servers/databases/connectionPolicies/* |无法编辑 SQL 服务器数据库连接策略 |
| Microsoft.Sql/servers/databases/dataMaskingPolicies/* |无法编辑 SQL 服务器数据库数据屏蔽策略 |
| Microsoft.Sql/servers/databases/securityAlertPolicies/* |无法编辑 SQL 服务器数据库安全警报策略 |
| Microsoft.Sql/servers/databases/securityMetrics/* |无法编辑 SQL 服务器数据库安全度量值 |
| Microsoft.Sql/servers/securityAlertPolicies/* |无法编辑 SQL 服务器安全警报策略 |

### 经典存储帐户参与者 <a name="classic-storage-account-contributor"></a>
可管理经典存储帐户

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取授权 |
| Microsoft.ClassicStorage/storageAccounts/* |创建和管理存储帐户 |
| Microsoft.Insights/alertRules/* |创建和管理 Insights 警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Support/* |创建和管理支持票证 |

### 存储帐户参与者 <a name="storage-account-contributor"></a>
可管理存储帐户，但不能访问。

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取所有授权 |
| Microsoft.Insights/alertRules/* |创建和管理 Insights 警报规则 |
| Microsoft.Insights/diagnosticSettings/* |管理诊断设置 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Storage/storageAccounts/* |创建和管理存储帐户 |
| Microsoft.Support/* |创建和管理支持票证 |

### 用户访问管理员 <a name="user-access-administrator"></a>
可管理用户对 Azure 资源的访问权限

| **操作** | |
| --- | --- |
| */read |读取除密码外的所有类型的资源。 |
| Microsoft.Authorization/* |管理授权 |
| Microsoft.Support/* |创建和管理支持票证 |

### 经典虚拟机参与者 <a name="classic-virtual-machine-contributor"></a>
可管理经典虚拟机，但不包括与其连接的虚拟网络或存储帐户

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取授权 |
| Microsoft.ClassicCompute/domainNames/* |创建和管理经典计算域名 |
| Microsoft.ClassicCompute/virtualMachines/* |创建和管理虚拟机 |
| Microsoft.ClassicNetwork/networkSecurityGroups/join/action |加入网络安全组 |
| Microsoft.ClassicNetwork/reservedIps/link/action |链接保留 IP |
| Microsoft.ClassicNetwork/reservedIps/read |读取保留 IP 地址 |
| Microsoft.ClassicNetwork/virtualNetworks/join/action |加入虚拟网络 |
| Microsoft.ClassicNetwork/virtualNetworks/read |读取虚拟网络 |
| Microsoft.ClassicStorage/storageAccounts/disks/read |读取存储帐户磁盘 |
| Microsoft.ClassicStorage/storageAccounts/images/read |读取存储帐户图像 |
| Microsoft.ClassicStorage/storageAccounts/listKeys/action |列出存储帐户密钥 |
| Microsoft.ClassicStorage/storageAccounts/read |读取经典存储帐户 |
| Microsoft.Insights/alertRules/* |创建和管理 Insights 警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Support/* |创建和管理支持票证 |

### 虚拟机参与者 <a name="virtual-machine-contributor"></a>
可管理虚拟机，但不包括与其连接的虚拟网络或存储帐户

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取授权 |
| Microsoft.Compute/availabilitySets/* |创建和管理计算可用性集 |
| Microsoft.Compute/locations/* |创建和管理计算位置 |
| Microsoft.Compute/virtualMachines/* |创建和管理虚拟机 |
| Microsoft.Compute/virtualMachineScaleSets/* |创建和管理虚拟机规模集 |
| Microsoft.Insights/alertRules/* |创建和管理 Insights 警报规则 |
| Microsoft.Network/applicationGateways/backendAddressPools/join/action |加入网络应用程序网关后端地址池 |
| Microsoft.Network/loadBalancers/backendAddressPools/join/action |加入负载均衡器后端地址池 |
| Microsoft.Network/loadBalancers/inboundNatPools/join/action |加入负载均衡器入站 NAT 池 |
| Microsoft.Network/loadBalancers/inboundNatRules/join/action |加入负载均衡器入站 NAT 规则 |
| Microsoft.Network/loadBalancers/read |读取负载均衡器 |
| Microsoft.Network/locations/* |创建和管理网络位置 |
| Microsoft.Network/networkInterfaces/* |创建和管理网络接口 |
| Microsoft.Network/networkSecurityGroups/join/action |加入网络安全组 |
| Microsoft.Network/networkSecurityGroups/read |读取网络安全组 |
| Microsoft.Network/publicIPAddresses/join/action |加入网络公共 IP 地址 |
| Microsoft.Network/publicIPAddresses/read |读取网络公共 IP 地址 |
| Microsoft.Network/virtualNetworks/read |读取虚拟网络 |
| Microsoft.Network/virtualNetworks/subnets/join/action |加入虚拟网络子网 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Storage/storageAccounts/listKeys/action |列出存储帐户密钥 |
| Microsoft.Storage/storageAccounts/read |读取存储帐户 |
| Microsoft.Support/* |创建和管理支持票证 |

### 经典网络参与者 <a name="classic-network-contributor"></a>
可管理经典虚拟网络和保留 IP

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取授权 |
| Microsoft.ClassicNetwork/* |创建和管理经典网络 |
| Microsoft.Insights/alertRules/* |创建和管理 Insights 警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Support/* |创建和管理支持票证 |

### Web 计划参与者 <a name="web-plan-contributor"></a>
可管理 Web 计划

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取授权 |
| Microsoft.Insights/alertRules/* |创建和管理 Insights 警报规则 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Support/* |创建和管理支持票证 |
| Microsoft.Web/serverFarms/* |创建和管理服务器场 |

### 网站参与者 <a name="website-contributor"></a>
可管理网站，但不包括与其连接的 Web 计划

| **操作** | |
| --- | --- |
| Microsoft.Authorization/*/read |读取授权 |
| Microsoft.Insights/alertRules/* |创建和管理 Insights 警报规则 |
| Microsoft.Insights/components/* |创建和管理 Insights 组件 |
| Microsoft.ResourceHealth/availabilityStatuses/read |读取资源的运行状况 |
| Microsoft.Resources/deployments/* |创建和管理资源组部署 |
| Microsoft.Resources/subscriptions/resourceGroups/read |读取资源组 |
| Microsoft.Support/* |创建和管理支持票证 |
| Microsoft.Web/certificates/* |创建和管理网站证书 |
| Microsoft.Web/listSitesAssignedToHostName/read |读取分配到主机名的站点 |
| Microsoft.Web/serverFarms/join/action |加入服务器场 |
| Microsoft.Web/serverFarms/read |读取服务器场 |
| Microsoft.Web/sites/* |创建和管理网站（站点创建还需要对关联的应用服务计划有写入权限） |

## 另请参阅
- [基于角色的访问控制](/documentation/articles/role-based-access-control-configure/)：RBAC 入门。
- [Azure RBAC 中的自定义角色](/documentation/articles/role-based-access-control-custom-roles/)：了解如何创建自定义角色，以满足访问需要。
- [创建访问权限更改历史记录报告](/documentation/articles/role-based-access-control-access-change-history-report/)：记录 RBAC 中的角色分配更改。
- [基于角色的访问控制故障排除](/documentation/articles/role-based-access-control-troubleshooting/)：获取解决常见问题的建议。

<!---HONumber=Mooncake_0327_2017-->
<!---Update_Description: wording update -->