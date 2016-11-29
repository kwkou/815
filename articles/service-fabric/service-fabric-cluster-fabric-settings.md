
<properties
   pageTitle="自定义 Service Fabric 群集设置和结构升级策略 | Azure"
   description="本文介绍可以自定义的结构设置和结构升级策略。"
   services="service-fabric"
   documentationCenter=".net"
   authors="chackdan"
   manager="timlt"
   editor=""/>  


<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/20/2016"
   wacn.date="11/28/2016"
   ms.author="chackdan"/>  


# 自定义 Service Fabric 群集设置和结构升级策略

本文档说明如何为 Service Fabric 群集自定义各种结构设置和结构升级策略。可以使用门户或 Resource Manager 模板完成自定义。

## 可以自定义的结构设置


下面是可以自定义的结构设置：

### 节名称：Security

|**参数**|**允许的值**|**指导或简短说明**|
|-----------------------|--------------------------|--------------------------|
|ClusterProtectionLevel|None 或 EncryptAndSign| 不安全的群集为 None（默认值），安全的群集为 EncryptAndSign。 |

### 节名称：Hosting

|**参数**|**允许的值**|**指导或简短说明**|
|-----------------------|--------------------------|--------------------------|
|ServiceTypeRegistrationTimeout|以秒为单位的时间，默认值为 300| 允许在结构中注册 ServiceType 的最长时间|
|ServiceTypeDisableFailureThreshold|整数，默认值为 1| 这是失败计数的阈值，超过此值后，将通知 FailoverManager (FM) 禁用该节点上的服务类型，并尝试在另一个节点上放置对象。|
|ActivationRetryBackoffInterval|以秒为单位的时间，默认值为 5|每次激活失败的退让间隔；在每次连续激活失败后，系统将重试激活最多 MaxActivationFailureCount 次。每次尝试的重试间隔是连续激活失败与激活退让间隔的积。|
|ActivationMaxRetryInterval|以秒为单位的时间，默认值为 300| 在每次连续激活失败时，系统将重试激活最多 ActivationMaxFailureCount 次。ActivationMaxRetryInterval 指定每次激活失败之后、重试之前等待的时间间隔 |
|ActivationMaxFailureCount|整数，默认值为 10| 系统在放弃之前重试失败激活的次数 |

### 节名称：FailoverManager

|**参数**|**允许的值**|**指导或简短说明**|
|-----------------------|--------------------------|--------------------------|
|PeriodicLoadPersistInterval|以秒为单位的时间，默认值为 10| 确定 FM 检查新负载报告的频率|

### 节名称：Federation

|**参数**|**允许的值**|**指导或简短说明**|
|-----------------------|--------------------------|--------------------------|
|LeaseDuration|以秒为单位的时间，默认值为 30|节点与其邻居之间的租约持续时间。|
|LeaseDurationAcrossFaultDomain|以秒为单位的时间，默认值为 30|所有容错域中的节点与其邻居之间的租约持续时间。|

### 节名称：ClusterManager

|**参数**|**允许的值**|**指导或简短说明**|
|-----------------------|--------------------------|--------------------------|
|UpgradeStatusPollInterval|以秒为单位的时间，默认值为 60|轮询应用程序升级状态的频率。此值确定任何 GetApplicationUpgradeProgress 调用的更新速率|
|UpgradeHealthCheckInterval|以秒为单位的时间，默认值为 60|受监视应用程序升级期间的运行状况检查频率|
|FabricUpgradeStatusPollInterval|以秒为单位的时间，默认值为 60|轮询结构升级状态的频率。此值确定任何 GetFabricUpgradeProgress 调用的更新速率 |
|FabricUpgradeHealthCheckInterval|以秒为单位的时间，默认值为 60|受监视结构升级期间的运行状况检查频率|



## 后续步骤

有关群集管理的详细信息，请阅读以下文章：

[在 Azure 群集中添加、滚动更新和删除证书](/documentation/articles/service-fabric-cluster-security-update-certs-azure/)

<!---HONumber=Mooncake_1121_2016-->