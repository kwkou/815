<properties
    pageTitle="使用 Windows 安全性保护 Windows 上运行的群集 | Azure"
    description="了解如何使用 Windows 安全性在 Windows 上运行的独立群集中配置节点到节点安全性和客户端到节点安全性。"
    services="service-fabric"
    documentationcenter=".net"
    author="rwike77"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="ce3bf686-ffc4-452f-b15a-3c812aa9e672"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="NA"
    ms.date="01/17/2017"
    wacn.date="03/03/2017"
    ms.author="ryanwi" />  


# 使用 Windows 安全性保护 Windows 上的独立群集

为了防止未经授权访问 Service Fabric 群集，必须保护资源，尤其是其中有正在运行的生产工作负荷时。本文介绍如何在 *ClusterConfig.JSON* 文件中使用 Windows 安全性配置节点到节点和客户端到节点安全性，其内容与[创建 Windows 上运行的独立群集](/documentation/articles/service-fabric-cluster-creation-for-windows-server/)中的配置安全性步骤一致。有关 Service Fabric 如何使用 Windows 安全性的详细信息，请参阅[群集安全方案](/documentation/articles/service-fabric-cluster-security/)。

>[AZURE.NOTE]
由于升级群集后无法更改安全性选项，因此，请慎重地为节点到节点安全性选择适当的选项。更改安全性选项需要重建整个群集。

## 使用 gMSA 配置 Windows 安全性
随 [Microsoft.Azure.ServiceFabric.WindowsServer.<version>.zip](http://go.microsoft.com/fwlink/?LinkId=730690) 独立群集包一起下载的示例 *ClusterConfig.gMSA.Windows.MultiMachine.JSON* 配置文件包含用于使用 [组托管服务帐户 (gMSA)](https://technet.microsoft.com/zh-cn/library/hh831782.aspx) 配置 Windows 安全性的模板：


	"security": {
	            "ServerCredentialType": "Windows",
	            "WindowsIdentities": {
	                "ClustergMSAIdentity": "accountname@fqdn"
	                "ClusterSPN": "fqdn"
	                "ClientIdentities": [
	                    {
	                        "Identity": "domain\\username",
	                        "IsAdmin": true
	                    }
	                ]
	            }
	        }


| **配置设置** | **说明** |
| --- | --- |
| WindowsIdentities |包含群集和客户端标识。 |
| ClustergMSAIdentity |配置节点到节点安全性。组托管服务帐户。 |
| ClusterSPN |gMSA 帐户的完全限定域 SPN|
| ClientIdentities |配置客户端到节点安全性。客户端用户帐户的数组。 |
| 标识 |客户端标识，域用户。 |
| IsAdmin |如果为 true，则指定域用户具有管理员客户端访问权限；如果为 false，则指定域用户具有用户客户端访问权限。 |

Service Fabric 需要在 gMSA 下运行时，通过设置 **ClustergMSAIdentity** 来配置[节点到节点安全性](/documentation/articles/service-fabric-cluster-security/#node-to-node-security)。若要在节点之间建立信任关系，这些节点必须能够相互识别。这可以通过两种不同的方法实现：指定包含群集中所有节点的组托管服务帐户，或者指定包含群集中所有节点的域计算机组。强烈建议使用[组托管服务帐户 (gMSA)](https://technet.microsoft.com/zh-cn/library/hh831782.aspx) 方法，尤其是对于较大的群集（包含 10 个以上的节点）或可能会扩大或收缩的群集。此方法不需要创建群集管理员对其有访问权限、可在其中添加和删除成员的域组。这些帐户还可用于自动密码管理。有关详细信息，请参阅[组托管服务帐户入门](http://technet.microsoft.com/zh-cn/library/jj128431.aspx)。

使用 **ClientIdentities** 配置[客户端到节点安全性](/documentation/articles/service-fabric-cluster-security/#client-to-node-security)。若要在客户端与群集之间建立信任关系，必须对群集进行配置，使其知道可以信任哪些客户端标识。这可以通过两种不同的方法实现：指定可以连接的域组用户，或者指定可以连接的域节点用户。Service Fabric 针对连接到 Service Fabric 群集的客户端支持两种不同的访问控制类型：管理员和用户。访问控制可让群集管理员针对不同的用户组限制某些类型的特定群集操作的访问权限，使群集更加安全。管理员对管理功能（包括读取/写入功能）拥有完全访问权限。默认情况下，用户只有管理功能的读取访问权限（例如查询功能），以及解析应用程序和服务的能力。有关访问控制的详细信息，请参阅[适用于 Service Fabric 客户端的基于角色的访问控制](/documentation/articles/service-fabric-cluster-security-roles/)。

以下示例中的**安全性**部分使用 gMSA 配置了 Windows 安全性，并指定 *ServiceFabric.clusterA.contoso.com* gMSA 中的计算机是群集的一部分，并且 *CONTOSO\\usera* 具有管理员客户端访问权限：

	"security": {
	    "WindowsIdentities": {
	        "ClustergMSAIdentity" : "ServiceFabric.clusterA.contoso.com",
	        "ClusterSPN" : "clusterA.contoso.com",
	        "ClientIdentities": [{
	            "Identity": "CONTOSO\\usera",
	            "IsAdmin": true
	        }]
	    }
	}


## 使用计算机组配置 Windows 安全性
随 [Microsoft.Azure.ServiceFabric.WindowsServer.<version>.zip](http://go.microsoft.com/fwlink/?LinkId=730690) 独立群集包一起下载的示例 *ClusterConfig.Windows.MultiMachine.JSON* 配置文件包含用于配置 Windows 安全性的模板。在“属性”部分中配置 Windows 安全性：


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


| **配置设置** | **说明** |
| --- | --- |
| ClusterCredentialType |如果 ClusterIdentity 指定 Active Directory 计算机组名，则 **ClusterCredentialType** 设置为 *Windows*。 |
| ServerCredentialType |通过将 **ServerCredentialType** 参数设置为 *Windows* 为客户端启用 Windows 安全性。这表示群集的客户端和群集本身在 Active Directory 域中运行。 |
| ClusterIdentity |配置节点到节点安全性。计算机组名。 |

如果想要使用 Active Directory 域中的计算机组，则通过设置使用 **ClusterIdentity** 来配置[节点到节点安全性](/documentation/articles/service-fabric-cluster-security/#node-to-node-security)。有关详细信息，请参阅[在 Active Directory 中创建计算机组](https://msdn.microsoft.com/zh-cn/library/aa545347(v=cs.70).aspx)。

以下示例中的**安全性**部分配置了 Windows 安全性，并指定 *ServiceFabric/clusterA.contoso.com* 中的计算机是群集的一部分，并且 *CONTOSO\\usera* 具有管理员客户端访问权限：


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


> [AZURE.NOTE]
不应在域控制器上部署 Service Fabric。使用计算机组或 gMSA 时，请确保 ClusterConfig.json 不包含域控制器的 IP。
> 
> 

## 后续步骤

在 *ClusterConfig.JSON* 文件中配置 Windows 安全性之后，请继续执行[创建 Windows 上运行的独立群集](/documentation/articles/service-fabric-cluster-creation-for-windows-server/)中的群集创建过程。

有关节点到节点安全性、客户端到节点安全性和基于角色的访问控制的详细信息，请参阅[群集安全方案](/documentation/articles/service-fabric-cluster-security/)。

有关使用 PowerShell 或 FabricClient 进行连接的示例，请参阅[连接到安全群集](/documentation/articles/service-fabric-connect-to-secure-cluster/)。

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description: add "使用 gMSA 配置 Windows 安全性" section-->