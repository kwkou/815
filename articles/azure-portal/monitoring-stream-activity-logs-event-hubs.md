<properties
	pageTitle="将 Azure 活动日志流式传输到事件中心 | Azure"
	description="了解如何将 Azure 活动日志流式传输到事件中心。"
	authors="johnkemnetz"
	manager="rboucher"
	editor=""
	services="monitoring-and-diagnostics"
	documentationCenter="monitoring-and-diagnostics"/>

<tags
	ms.service="monitoring-and-diagnostics"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/17/2016"
	wacn.date="10/17/2016"
	ms.author="johnkem"/>  


# 将 Azure 活动日志流式传输到事件中心
可以将 [**Azure 活动日志**](/documentation/articles/monitoring-overview-activity-logs/)以近实时方式流式传输到任何应用程序，方法是使用门户预览中的内置“导出”选项，或者通过 Azure PowerShell Cmdlet 或 Azure CLI 在日志配置文件中启用服务总线规则 ID。

## 可以对活动日志和事件中心执行的操作
可以通过下述几种方式将流式传输功能用于活动日志：

- **流式传输到第三方日志记录和遥测系统** – 一段时间后，事件中心的流式传输功能就会成为一种机制，用于将活动日志通过管道传输到第三方 SIEM 和日志分析解决方案。
- **生成自定义遥测和日志记录平台** – 如果已经有一个自定义生成的遥测平台，或者正想生成一个，则可利用事件中心高度可缩放的发布-订阅功能，灵活地引入活动日志。

## 启用活动日志的流式传输
可以通过编程方式或门户预览启用活动日志的流式传输。不管使用哪种方式，都可以选取一个服务总线命名空间（如果该空间不存在，则创建一个）。选择的命名空间是创建事件中心的地方（如果这是用户第一次流式传输活动日志），或者是将活动日志流式传输到事件中心的地方（如果用户此前已将活动日志流式传输到该命名空间），而策略则定义流式传输机制所具有的权限。目前，流式传输到事件中心需要“管理”、“读取”和“发送”权限。在经典管理门户中针对服务总线命名空间的“配置”选项卡下，可以创建或修改服务总线命名空间的共享访问策略。若要更新活动日志的日志配置文件，使之包括流式传输，则必须在服务总线授权规则中让客户端拥有 ListKey 权限。

### 通过 Azure 门户预览 
1. 使用门户预览左侧的菜单导航到“活动日志”边栏选项卡。

    ![在门户预览中导航到“活动日志”](./media/monitoring-overview-activity-logs/activity-logs-portal-navigate.png)  

2. 单击边栏选项卡顶部的“导出”按钮。

    ![门户预览中的“导出”按钮](./media/monitoring-overview-activity-logs/activity-logs-portal-export.png)  

3. 在出现的边栏选项卡中，可以选择要流式传输事件的区域，还可以选择服务总线命名空间，在其中创建事件中心来流式传输这些事件。

    ![“导出活动日志”边栏选项卡](./media/monitoring-overview-activity-logs/activity-logs-portal-export-blade.png)  

4. 单击“保存”保存这些设置。这些设置会即时应用到订阅。


### 通过 PowerShell Cmdlet
如果日志配置文件已存在，则需先删除该配置文件。

1. 使用 `Get-AzureRmLogProfile` 确定日志配置文件是否存在。
2. 如果存在，使用 `Remove-AzureRmLogProfile` 将其删除。
3. 使用 `Set-AzureRmLogProfile` 创建配置文件：

```
Add-AzureRmLogProfile -Name my_log_profile -StorageAccountId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/my_storage -serviceBusRuleId /subscriptions/s1/resourceGroups/Default-ServiceBus-chinaeast/providers/Microsoft.ServiceBus/namespaces/mytestSB/authorizationrules/RootManageSharedAccessKey -Locations chinaeast,chinanorth -RetentionInDays 90 -Categories Write,Delete,Action
```

服务总线规则 ID 是特定格式的字符串，例如：{服务总线资源 ID}/authorizationrules/{密钥名称}

### 通过 Azure CLI
如果日志配置文件已存在，则需先删除该配置文件。

1. 使用 `azure insights logprofile list` 确定日志配置文件是否存在。
2. 如果存在，使用 `azure insights logprofile delete` 将其删除。
3. 使用 `azure insights logprofile add` 创建配置文件：

```
azure insights logprofile add --name my_log_profile --storageId /subscriptions/s1/resourceGroups/insights-integration/providers/Microsoft.Storage/storageAccounts/my_storage --serviceBusRuleId /subscriptions/s1/resourceGroups/Default-ServiceBus-chinaeast/providers/Microsoft.ServiceBus/namespaces/mytestSB/authorizationrules/RootManageSharedAccessKey --locations chinaeast,chinanorth --retentionInDays 90 –categories Write,Delete,Action
```

服务总线规则 ID 是以下格式的字符串：`{service bus resource ID}/authorizationrules/{key name}`。
 
## 如何使用事件中心的日志数据？
[此处提供活动日志的架构](/documentation/articles/monitoring-overview-activity-logs/)。每个事件都采用 JSON blob（称为“记录”）数组的形式。

## 后续步骤
- [将活动日志存档到存储帐户](/documentation/articles/monitoring-archive-activity-log/)
- [阅读 Azure 活动日志概述](/documentation/articles/monitoring-overview-activity-logs/)
- [根据活动日志事件设置警报](/documentation/articles/insights-auditlog-to-webhook-email/)

<!---HONumber=Mooncake_1010_2016-->