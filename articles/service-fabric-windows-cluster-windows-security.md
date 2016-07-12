<properties
   pageTitle="使用 Windows 安全性保护 Windows 上运行的群集 | Azure"
   description="了解如何使用 Windows 安全性在 Windows 上运行的独立群集中配置节点到节点安全性和客户端到节点安全性。"
   services="service-fabric"
   documentationCenter=".net"
   authors="rwike77"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.date="06/14/2016"
   wacn.date="07/04/2016"/>


# 使用 Windows 安全性保护 Windows 上的独立群集

为了防止未经授权访问 Service Fabric 群集，必须保护资源，尤其是其中有正在运行的生产工作负荷时。本指南介绍如何在 *ClusterConfig.JSON* 文件中使用 Windows 安全性配置节点到节点和客户端到节点安全性，内容对应于 [Create a standalone cluster running on Windows](/documentation/articles/service-fabric-cluster-creation-for-windows-server/)（创建 Windows 上运行的独立群集）中的安全性配置步骤。有关 Service Fabric 如何使用 Windows 安全性的详细信息，请参阅 [Cluster security scenarios](/documentation/articles/service-fabric-cluster-security/)（群集安全方案）。

>[AZURE.NOTE]
由于升级群集后无法更改安全性选项，因此，请慎重地为节点到节点安全性选择适当的选项。更改安全性选项需要重建整个群集。

## 配置 Windows 安全性
随 [Microsoft.Azure.ServiceFabric.WindowsServer.<version>.zip](http://go.microsoft.com/fwlink/?LinkId=730690) 独立群集包一起下载的示例 *ClusterConfig.Windows.JSON* 配置文件包含用于配置 Windows 安全性的模板。在 **Properties** 节中配置 Windows 安全性：

```
"security": {
            "ClusterCredentialType": "Windows",
            "ServerCredentialType": "Windows",
            "WindowsIdentities": {
		"ClusterIdentity" : "[domain\machinegroup]",
                "ClientIdentities": [{
                    "Identity": "[domain\username]",
                    "IsAdmin": true
                }]
            }
        }
```

|**配置设置**|**说明**|
|-----------------------|--------------------------|
|ClusterCredentialType|通过将 **ClusterCredentialType** 参数设置为 *Windows* 来启用 Windows 安全性。|
|ServerCredentialType|通过将 **ServerCredentialType** 参数设置为 *Windows* 来为客户端启用 Windows 安全性。这表示群集的客户端和群集本身在 Active Directory 域中运行。|
|WindowsIdentities|包含群集和客户端标识。|
|ClusterIdentity|配置节点到节点安全性。组托管服务帐户或计算机名称的逗号分隔列表。|
|ClientIdentities|配置客户端到节点安全性。客户端用户帐户的数组。|
|标识|客户端标识或域用户。|
|IsAdmin|如果为 true，则指定域用户具有管理员客户端访问权限；如果为 false，则指定域用户具有用户客户端访问权限。|

使用 **ClusterIdentity** 配置[节点到节点安全性](/documentation/articles/service-fabric-cluster-security/#node-to-node-security)。若要在节点之间建立信任关系，这些节点必须能够相互识别。这可以通过两种方法来实现：指定包含群集中所有节点的组托管服务帐户，或者指定群集中所有节点的域节点标识。强烈建议使用[组托管服务帐户 (gMSA)](https://technet.microsoft.com/zh-cn/library/hh831782.aspx) 方法，尤其是对于较大的群集（包含 10 个以上的节点）或可能会扩大或收缩的群集。使用此方法可以在 gMSA 中添加或删除节点，而无需更改群集清单。此方法不需要创建群集管理员对其有访问权限、可在其中添加和删除成员的域组。有关详细信息，请参阅 [Getting Started with Group Managed Service Accounts](http://technet.microsoft.com/zh-cn/library/jj128431.aspx)（组托管服务帐户入门）。

使用 **ClientIdentities** 配置[客户端到节点安全性](/documentation/articles/service-fabric-cluster-security/#client-to-node-security)。若要在客户端与群集之间建立信任关系，必须对群集进行配置，使其知道可以信任哪些客户端标识。这可以通过两种不同的方法来实现：指定可以连接的域组用户，或者指定可以连接的域节点用户。Service Fabric 针对连接到 Service Fabric 群集的客户端支持两种不同的访问控制类型：管理员和用户。访问控制可让群集管理员针对不同的用户组限制某些类型的特定群集操作的访问权限，使群集更加安全。管理员对管理功能（包括读取/写入功能）拥有完全访问权限。默认情况下，用户只有管理功能的读取访问权限（例如查询功能），以及解析应用程序和服务的能力。

以下示例 **security** 节将配置 Windows 安全性，指定 *ServiceFabric/clusterA.contoso.com* 中的计算机是群集的一部分，并且 *CONTOSO\\usera* 具有管理员客户端访问权限：

```
"security": {
    "ClusterCredentialType": "Windows",
    "ServerCredentialType": "Windows",
    "WindowsIdentities": {
		"ClusterIdentity" : "ServiceFabric/clusterA.contoso.com",
        "ClientIdentities": [{
            "Identity": "CONTOSO\\usera",
        "IsAdmin": true
        }]
    }
},
```

## 后续步骤

在 *ClusterConfig.JSON* 文件中配置 Windows 安全性之后，请继续执行 [Create a standalone cluster running on Windows](/documentation/articles/service-fabric-cluster-creation-for-windows-server/)（创建 Windows 上运行的独立群集）中的群集创建过程。

有关节点到节点安全性、客户端到节点安全性和基于角色的访问控制的详细信息，请参阅 [Cluster security scenarios](/documentation/articles/service-fabric-cluster-security/)（群集安全方案）。

请参阅 [Connect to a secure cluster](/documentation/articles/service-fabric-connect-to-secure-cluster/)（连接到安全群集），以获取有关使用 PowerShell 或 FabricClient 进行连接的示例。

<!---HONumber=Mooncake_0627_2016-->