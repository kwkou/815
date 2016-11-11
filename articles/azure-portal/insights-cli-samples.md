<properties
	pageTitle="Azure Insights：Azure Insights CLI 快速入门示例 | Azure"
	description="示例 CLI 命令可以帮助访问 Azure Insights 监视功能。Azure Insights 是一种 Azure 服务，允许你基于配置的遥测数据值自动缩放云服务、虚拟机和 Web Apps、发送警报通知或调用 Web URL。"
	authors="kamathashwin"
	manager=""
	editor=""
	services="monitoring-and-diagnostics"
	documentationCenter="monitoring-and-diagnostics"/>  

<tags
	ms.service="monitoring-and-diagnostics"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/08/2016"
	ms.author="ashwink"
	wacn.date="10/17/2016"/>  

# Azure Insights 跨平台 CLI 快速启动示例

本文演示可帮助访问 Azure Insights 监视功能的示例命令行接口 (CLI) 命令。Azure Insights 允许你基于配置的遥测数据值自动缩放云服务、虚拟机和 Web Apps 以及发送警报通知或调用 Web URL。


## 先决条件

如果尚未安装 Azure CLI，请参阅[安装 Azure CLI](/documentation/articles/xplat-cli-install/)。如果不熟悉 Azure CLI，可在[将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure Resource Manager 配合使用](/documentation/articles/xplat-cli-azure-resource-manager/)中了解相关详细信息。


在 Windows 中，从 [Node.js 网站](https://nodejs.org/)安装 npm。完成安装之后，通过“以管理员身份运行”权限使用 CMD.exe，从安装 npm 的文件夹执行以下命令：

```
npm install azure-cli --global
```

接下来，导航到所需的任何文件夹/位置并在命令行处输入：

```
azure help
```

## 登录 Azure

第一步是登录 Azure 帐户。

```
azure login -e AzureChinaCloud 
```

运行此命令后，必须按照屏幕上的说明进行登录。之后，将看到你的帐户、TenantId 和默认订阅 ID。所有命令都在默认订阅的上下文中工作。

若要列出当前订阅的详细信息，请使用以下命令。

```
azure account show
```

若要将工作上下文更改为其他订阅，请使用以下命令。

```
azure account set "subscription ID or subscription name"
```

若要使用 Azure Resource Manager 和 Azure Insights 命令，需要处于 Azure Resource Manager 模式下。

```
azure config mode arm
```

若要查看所有支持的 Azure Insights 命令的列表，请执行以下操作。

```
azure insights
```

## 查看订阅的审核日志

若要查看审核日志的列表，请执行以下操作。

```
azure insights logs list [options]
```

尝试以下操作以查看所有可用选项。

```
azure insights logs list -help
```

下面是一个按 resourceGroup 列出日志的示例

```
azure insights logs list --resourceGroup "myrg1"
```

按调用方列出日志的示例

```
azure insights logs list --caller "myname@company.com"
```

在开始和结束日期内，按调用方对资源类型列出日志的示例

```
azure insights logs list --resourceProvider "Microsoft.Web" --caller "myname@company.com" --startTime 2016-03-08T00:00:00Z --endTime 2016-03-16T00:00:00Z
```

## 使用警报
可以按照该部分中的信息使用警报。

### 获取资源组中的警报规则

```
azure insights alerts rule list abhingrgtest123
azure insights alerts rule list abhingrgtest123 --ruleName andy0323
```

### 创建指标警报规则

```
azure insights alerts actions email create --customEmails foo@microsoft.com
azure insights alerts actions webhook create https://someuri.com
azure insights alerts rule metric set andy0323 eastus abhingrgtest123 PT5M GreaterThan 2 /subscriptions/df602c9c-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/Default-Web-EastUS/providers/Microsoft.Web/serverfarms/Default1 BytesReceived Total
```

### 创建日志警报规则

```
azure insights alerts rule log set ruleName eastus resourceGroupName someOperationName
```

### 创建 webtest 警报规则

```
azure insights alerts rule webtest set leowebtestr1-webtestr1 eastus Default-Web-chinaeast PT5M 1 GSMT_AvRaw /subscriptions/b67f7fec-69fc-4974-9099-a26bd6ffeda3/resourcegroups/Default-Web-chinaeast/providers/microsoft.insights/webtests/leowebtestr1-webtestr1
```

### 删除警报规则

```
azure insights alerts rule delete abhingrgtest123 andy0323
```

## 日志配置文件
可以按照此部分中的信息使用日志配置文件。

### 获取日志配置文件

```
azure insights logprofile list
azure insights logprofile get -n default
```


### 添加没有保留期的日志配置文件

```
azure insights logprofile add --name default --storageId /subscriptions/1a66ce04-b633-4a0b-b2bc-a912ec8986a6/resourceGroups/insights-integration/providers/Microsoft.Storage/storageAccounts/insightsintegration7777 --locations chinanorth,chinaeast
```

### 删除日志配置文件

```
azure insights logprofile delete --name default
```

### 添加具有保留期的日志配置文件

```
azure insights logprofile add --name default --storageId /subscriptions/1a66ce04-b633-4a0b-b2bc-a912ec8986a6/resourceGroups/insights-integration/providers/Microsoft.Storage/storageAccounts/insightsintegration7777 --locations chinanorth,chinaeast --retentionInDays 90
```

### 添加具有保留期和 EventHub 的日志配置文件

```
azure insights logprofile add --name default --storageId /subscriptions/1a66ce04-b633-4a0b-b2bc-a912ec8986a6/resourceGroups/insights-integration/providers/Microsoft.Storage/storageAccounts/insightsintegration7777 --serviceBusRuleId /subscriptions/1a66ce04-b633-4a0b-b2bc-a912ec8986a6/resourceGroups/Default-ServiceBus-EastUS/providers/Microsoft.ServiceBus/namespaces/testshoeboxeastus/authorizationrules/RootManageSharedAccessKey --locations chinanorth,chinaeast --retentionInDays 90
```


## 诊断
可以按照此部分中的信息使用诊断设置。

### 获取诊断设置

```
azure insights diagnostic get --resourceId /subscriptions/df602c9c-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/andyrg/providers/Microsoft.Logic/workflows/andy0315logicapp
```

### 禁用诊断设置

```
azure insights diagnostic set --resourceId /subscriptions/df602c9c-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/andyrg/providers/Microsoft.Logic/workflows/andy0315logicapp --storageId /subscriptions/df602c9c-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/Default-Storage-chinaeast/providers/Microsoft.Storage/storageAccounts/shibanitesting --enabled false
```

### 启用没有保留期的诊断设置

```
azure insights diagnostic set --resourceId /subscriptions/df602c9c-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/andyrg/providers/Microsoft.Logic/workflows/andy0315logicapp --storageId /subscriptions/df602c9c-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/Default-Storage-chinaeast/providers/Microsoft.Storage/storageAccounts/shibanitesting --enabled true
```


## 自动缩放
按照此部分中的信息使用自动缩放设置。需要修改这些示例。

### 获取资源组的自动缩放设置

```
azure insights autoscale setting list montest2
```

### 按名称获取资源组的自动缩放设置

```
azure insights autoscale setting list montest2 -n setting2
```


### 设置自动缩放设置

```
azure insights autoscale setting set montest2 -n setting2 --settingSpec
```

<!---HONumber=Mooncake_0503_2016-->