
<properties
   pageTitle="Service Fabric 群集安全性：客户端角色 | Azure"
   description="本文介绍两个客户端角色以及提供给这些角色的权限。"
   services="service-fabric"
   documentationCenter=".net"
   authors="mani-ramaswamy"
   manager="coreysa"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="05/18/2016"
   wacn.date="07/04/2016"/>



# 适用于 Service Fabric 客户端的基于角色的访问控制

Azure Service Fabric 针对连接到 Service Fabric 群集的客户端支持两种不同的访问控制类型：管理员和用户。访问控制可让群集管理员针对不同的用户组限制特定群集操作的访问权限，使群集更加安全。

管理员对管理功能（包括读取/写入功能）拥有完全访问权限。默认情况下，用户只有管理功能的读取访问权限（例如查询功能），以及解析应用程序和服务的能力。

可在创建群集时为每个角色提供不同的证书，以指定两个客户端角色（管理员和客户端）。有关设置安全 Service Fabric 群集的详细信息，请参阅 [Service Fabric 群集安全性](/documentation/articles/service-fabric-cluster-security/)。


## 默认访问控制设置


管理员访问控制类型对所有 FabricClient API 拥有完全访问权限。它可以对 Service Fabric 群集执行任何读取和写入操作，包括：

### 应用程序和服务操作
* **CreateService**：创建服务 							
* **CreateServiceFromTemplate**：从模板创建服务 							
* **UpdateService**：更新服务 							
* **DeleteService**：删除服务 							
* **ProvisionApplicationType**：预配应用程序类型 							
* **CreateApplication**：创建应用程序   							
* **DeleteApplication**：删除应用程序 							
* **UpgradeApplication**启动或中断应用程序升级 							
* **UnprovisionApplicationType**：取消预配应用程序类型 							
* **MoveNextUpgradeDomain**：使用显式更新域恢复应用程序升级 							
* **ReportUpgradeHealth**：恢复应用程序升级并提供当前升级进度 							
* **ReportHealth**：报告运行状况 							
* **PredeployPackageToNode**：预先部署 API							
* **CodePackageControl**：重新启动代码包 							
* **RecoverPartition**：恢复一个分区 							
* **RecoverPartitions**：恢复多个分区 							
* **RecoverServicePartitions**：恢复服务分区 							
* **RecoverSystemPartitions**：恢复系统服务分区 							


### 群集操作
* **ProvisionFabric**：预配 MSI 和/或群集清单 							
* **UpgradeFabric**：启动群集升级 							
* **UnprovisionFabric**：取消预配 MSI 和/或群集清单 						
* **MoveNextFabricUpgradeDomain**：使用显式更新域恢复群集升级 							
* **ReportFabricUpgradeHealth**：恢复群集升级并提供当前升级进度 							
* **StartInfrastructureTask**：启动基础结构任务 							
* **FinishInfrastructureTask**：完成基础结构任务 							
* **InvokeInfrastructureCommand**：基础结构任务管理命令  							
* **ActivateNode**：激活一个节点 							
* **DeactivateNode**：停用一个节点 							
* **DeactivateNodesBatch**：停用多个节点 							
* **RemoveNodeDeactivations**：在多个节点上还原停用操作 							
* **GetNodeDeactivationStatus**：检查停用状态 							
* **NodeStateRemoved**：报告已删除的节点状态 							
* **ReportFault**：报告错误 							
* **FileContent**：映像存储客户端文件传输（群集外部） 							
* **FileDownload**：启动映像存储客户端文件下载（群集外部） 							
* **InternalList**：映像存储客户端文件列表操作（内部） 							
* **Delete**：映像存储客户端删除操作  							
* **Upload**：映像存储客户端上载操作 							
* **NodeControl**：启动、停止和重新启动节点 							
* **MoveReplicaControl**：将副本从一个节点移到另一个节点 							

### 其他操作
* **Ping**：客户端 ping 							
* **Query**：允许所有查询
* **NameExists**：检查命名 URI 是否存在 							



用户访问控制类型默认限制为以下操作。（管理员访问控制也有权访问这些操作）。

* **EnumerateSubnames**：枚举命名 URI 							
* **EnumerateProperties**：枚举命名属性 							
* **PropertyReadBatch**：命名属性读取操作 							
* **GetServiceDescription**：长时间轮询服务通知和读取服务描述 							
* **ResolveService**：根据投诉解决服务问题 							
* **ResolveNameOwner**：解析命名 URI 所有者 							
* **ResolvePartition**：解析系统服务 							
* **ServiceNotifications**：基于事件的服务通知 							
* **GetUpgradeStatus**：轮询应用程序升级状态 							
* **GetFabricUpgradeStatus**：轮询群集升级状态 							
* **InvokeInfrastructureQuery**：查询基础结构任务 							
* **List**：映像存储客户端文件列表操作 							
* **ResetPartitionLoad**：重置故障转移单元的负载 							
* **ToggleVerboseServicePlacementHealthReporting**：切换详细服务放置运行状况报告 							

## 更改客户端角色的默认设置

在群集清单文件中，你可以根据需要向客户端提供管理功能。你可以更改默认设置，方法是在创建群集过程中转到“结构设置”选项，然后在“名称”、“管理员”、“用户”和“值”字段中提供上述设置。

## 后续步骤

[Service Fabric 群集安全性](/documentation/articles/service-fabric-cluster-security/)

[创建 Service Fabric 群集](/documentation/articles/service-fabric-cluster-creation-via-portal/)

<!---HONumber=Mooncake_0627_2016-->