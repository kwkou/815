<properties
    pageTitle="Azure Service Fabric 反向代理 | Azure"
    description="使用 Service Fabric 的反向代理从群集内部和外部与微服务通信"
    services="service-fabric"
    documentationcenter=".net"
    author="BharatNarasimman"
    manager="timlt"
    editor="vturecek" />
<tags
    ms.assetid="47f5c1c1-8fc8-4b80-a081-bc308f3655d3"
    ms.service="service-fabric"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="required"
    ms.date="04/07/2017"
    wacn.date="05/15/2017"
    ms.author="bharatn"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="7b719db402f65c74e6ff3d3f6c77a6f31a13498a"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="reverse-proxy-in-azure-service-fabric"></a>Azure Service Fabric 中的反向代理
Azure Service Fabric 中内置的反向代理可以访问 Service Fabric 群集中用于公开 HTTP 终结点的微服务。

## <a name="microservices-communication-model"></a>微服务通信模型
Service Fabric 中的微服务通常在群集的一部分虚拟机中运行，并且可出于各种原因从一个虚拟机移到另一个虚拟机。 因此，微服务的终结点可能会动态变化。 与微服务通信的经典模式是下面的解析循环。

1. 一开始通过命名服务解析服务位置。
2. 连接到服务。
3. 确定连接失败的原因，必要时再次解析服务位置。

此过程通常涉及将客户端通信库包装到重试循环中，以便执行服务解析和重试策略。
有关详细信息，请参阅[与服务连接和通信](/documentation/articles/service-fabric-connect-and-communicate-with-services/)。

### <a name="communicating-by-using-the-reverse-proxy"></a>使用反向代理通信
Service Fabric 中的反向代理在群集的所有节点上运行。 它会代表客户端执行整个服务解析流程，然后再转发客户端请求。 因此，在群集上运行的客户端可以通过同一节点本地运行的反向代理，使用任何客户端 HTTP 通信库来与目标服务通信。

![内部通信][1]

## <a name="reaching-microservices-from-outside-the-cluster"></a>从群集外部访问微服务
微服务的默认外部通信模型为“选择加入”模型，在该模型中，无法直接从外部客户端访问每个服务。 [Azure 负载均衡器](/documentation/articles/load-balancer-overview/)充当微服务和外部客户端之间的网络边界，可以进行网络地址转换并将外部请求转发到内部的 IP:端口终结点。 若要允许外部客户端直接访问微服务的终结点，必须先将负载均衡器配置为将流量转发到群集中服务使用的每个端口。 另外，大多数微服务（尤其是有状态微服务）并不驻留在群集的所有节点上。 这些微服务在故障转移时可在节点之间移动。 在这种情况下，负载均衡器无法有效确定要将流量转发到的副本的目标节点位置。

### <a name="reaching-microservices-via-the-reverse-proxy-from-outside-the-cluster"></a>从群集外部通过反向代理访问微服务
可以在负载均衡器中直接配置反向代理的端口，而无需配置单个服务的端口。 这种配置可让群集外部的客户端使用反向代理访问群集内部的服务，无需经过额外的配置。

![外部通信][0]

> [AZURE.WARNING]
> 在负载均衡器中配置反向代理的端口后，可从群集外部访问群集中公开 HTTP 终结点的所有微服务。
>
>


## <a name="uri-format-for-addressing-services-by-using-the-reverse-proxy"></a>使用反向代理访问服务时所用的 URI 格式
反向代理使用特定的统一资源标识符 (URI) 格式来识别传入请求应该转发到的服务分区：

    http(s)://<Cluster FQDN | internal IP>:Port/<ServiceInstanceName>/<Suffix path>?PartitionKey=<key>&PartitionKind=<partitionkind>&ListenerName=<listenerName>&TargetReplicaSelector=<targetReplicaSelector>&Timeout=<timeout_in_seconds>

* **http(s)：**可以将反向代理配置为接受 HTTP 或 HTTPS 流量。 对于 HTTPS 流量，反向代理中会发生安全套接字层 (SSL) 终止。 反向代理使用 HTTP 将请求转发到群集中服务。

    请注意，目前不支持 HTTPS 服务。
* **Cluster FQDN | internal IP：**对于外部客户端，可以配置反向代理，以便可以通过群集域（例如 mycluster.chinaeast.cloudapp.chinacloudapi.cn）访问反向代理。 默认情况下，反向代理在每个节点上运行。 对于内部流量，可在本地主机或任意内部节点 IP（例如 10.0.0.1）上访问反向代理。
* **Port：**为反向代理指定的端口，例如 19008。
* **ServiceInstanceName：**在不使用“fabric:/”方案的情况下尝试访问的已部署服务实例的完全限定名称。 例如，若要访问 *fabric:/myapp/myservice/* 服务，可以使用 *myapp/myservice*。
* **Suffix path:**要连接到的服务的实际 URL 路径，例如 *myapi/values/add/3*。
* **PartitionKey：**对于分区服务，这是针对要访问的分区计算出的分区键。 请注意，这*不*是分区 ID GUID。 对于使用单独分区方案的服务，此参数不是必需的。
* **PartitionKind：**服务分区方案。 该方案可以是“Int64Range”或“Named”。 对于使用单独分区方案的服务，此参数不是必需的。
* **ListenerName** 服务中的终结点采用以下形式：{"Endpoints":{"Listener1":"Endpoint1","Listener2":"Endpoint2" ...}}。 当服务公开了多个终结点时，此参数标识应将客户端请求转发到的终结点。 如果服务只有一个侦听器，则可以省略此项。
* **TargetReplicaSelector** 这指定应当如何选择目标副本或实例。
  * 当目标服务为有状态服务时，TargetReplicaSelector 可以是下列其中一项：“PrimaryReplica”、“RandomSecondaryReplica”或“RandomReplica”。 如果未指定此参数，默认值为“PrimaryReplica”。
  * 当目标服务为无状态服务时，反向代理将选择服务分区的一个随机实例来将实例转发到其中。
* **Timeout：**此参数指定反向代理针对服务创建的 HTTP 请求（代表客户端请求）的超时。 默认值为 60 秒。 这是一个可选参数。

### <a name="example-usage"></a>用法示例
以 *fabric:/MyApp/MyService* 服务为例，该服务可针对以下 URL 打开一个 HTTP 侦听器：

    http://10.0.0.5:10592/3f0d39ad-924b-4233-b4a7-02617c6308a6-130834621071472715/

下面是该服务的资源：

* `/index.html`
* `/api/users/<userId>`

如果服务使用单独分区方案，则 *PartitionKey* 和 *PartitionKind* 查询字符串参数不是必需的，可以使用网关访问服务，如下所示：

* 外部访问方式：`http://mycluster.chinaeast.cloudapp.chinacloudapi.cn:19008/MyApp/MyService`
* 内部访问方式：`http://localhost:19008/MyApp/MyService`

如果服务使用“统一 Int64”分区方案，则必须使用 *PartitionKey* 和 *PartitionKind* 查询字符串来访问服务的分区：

* 外部访问方式：`http://mycluster.chinaeast.cloudapp.chinacloudapi.cn:19008/MyApp/MyService?PartitionKey=3&PartitionKind=Int64Range`
* 内部访问方式：`http://localhost:19008/MyApp/MyService?PartitionKey=3&PartitionKind=Int64Range`

若要访问服务公开的资源，可直接在 URL 中将资源路径置于服务名称之后：

* 外部访问方式：`http://mycluster.chinaeast.cloudapp.chinacloudapi.cn:19008/MyApp/MyService/index.html?PartitionKey=3&PartitionKind=Int64Range`
* 内部访问方式： `http://localhost:19008/MyApp/MyService/api/users/6?PartitionKey=3&PartitionKind=Int64Range`

然后，网关会将这些请求转发到服务的 URL：

* `http://10.0.0.5:10592/3f0d39ad-924b-4233-b4a7-02617c6308a6-130834621071472715/index.html`
* `http://10.0.0.5:10592/3f0d39ad-924b-4233-b4a7-02617c6308a6-130834621071472715/api/users/6`

## <a name="special-handling-for-port-sharing-services"></a>针对端口共享服务进行特殊处理
无法访问某个服务时，Azure 应用程序网关会尝试重新解析服务地址以及重试请求。 这是应用程序网关的主要优势之一，因为客户端代码不需要实现自身的服务解析和解析循环。

无法访问某个服务时，通常表示服务实例或副本已在其常规生命周期中转移到其他节点。 发生这种情况时，应用程序网关可能会收到网络连接错误，指出在原来解析过的地址上的某个终结点不再处于开放状态。

不过，副本或服务实例可能会共享主机进程，在通过基于 http.sys 的 Web 服务器进行托管的情况下还可能会共享端口，这些 Web 服务器包括：

* [System.Net.HttpListener](https://msdn.microsoft.com/zh-cn/library/system.net.httplistener%28v=vs.110%29.aspx)
* [ASP.NET Core WebListener](https://docs.asp.net/latest/fundamentals/servers.html#weblistener)
* [Katana](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.OwinSelfHost/)

在这样的情形下，可能会出现 Web 服务器出现在主机进程中并且能够响应请求，而被解析的服务实例或副本却再也不能在主机上使用的情况。 这种情况下，网关会从 Web 服务器收到 HTTP 404 响应。 因此，HTTP 404 具有两种不同的含义：

- 情况 #1：服务地址正确，但用户请求的资源不存在。
- 情况 #2：服务地址不正确，且用户请求的资源可能在其他节点上。

第一种情况是正常的 HTTP 404，属于用户错误。 在第二种情况中，用户请求的资源确实存在。 但是，应用程序网关找不到该资源，因为服务本身已移动。 应用程序网关需要重新解析地址，然后重试请求。

因此，应用程序网关需要通过某种方式来区分这两种情况。 为了进行这种区分，需要从服务器获得提示。

* 默认情况下，应用程序网关假设第二种情况属于事实，因此会尝试重新解析和重新发出请求。
* 若要向应用程序网关指示第一种情况属于事实，服务应返回以下 HTTP 响应标头：

  `X-ServiceFabric : ResourceNotFound`

此 HTTP 响应标头指示的是正常的 HTTP 404 情形，即所请求的资源不存在，因此应用程序网关不会尝试重新解析服务地址。

## <a name="setup-and-configuration"></a>安装和配置
可以使用 [Azure Resource Manager 模板](/documentation/articles/service-fabric-cluster-creation-via-arm/)在 Service Fabric 中为群集启用反向代理。

请参阅[在安全群集中配置 HTTPS 反向代理](https://github.com/ChackDan/Service-Fabric/tree/master/ARM Templates/ReverseProxySecureSample#configure-https-reverse-proxy-in-a-secure-cluster)中的 Azure Resource Manager 模板示例，使用证书配置安全反向代理并处理证书滚动更新。

首先，获取要部署的群集的模板。 可以使用示例模板，或者创建自定义 Resource Manager 模板。 然后，可以使用以下步骤启用反向代理：

1. 在模板的[“参数”部分](/documentation/articles/resource-group-authoring-templates/)定义反向代理的端口。


    	"SFReverseProxyPort": {
        	"type": "int",
        	"defaultValue": 19008,
        	"metadata": {
            	"description": "Endpoint for Service Fabric Reverse proxy"
        	}
    	},

2. 为 **Cluster** [资源类型节](/documentation/articles/resource-group-authoring-templates/)中的每个 nodetype 对象指定端口。

    端口由参数名称 reverseProxyEndpointPort 标识。

	    {
	        "apiVersion": "2016-09-01",
	        "type": "Microsoft.ServiceFabric/clusters",
	        "name": "[parameters('clusterName')]",
	        "location": "[parameters('clusterLocation')]",
	        ...
	       "nodeTypes": [
	          {
	           ...
	           "reverseProxyEndpointPort": "[parameters('SFReverseProxyPort')]",
	           ...
	          },
	        ...
	        ],
	        ...
	    }

3. 若要从 Azure 群集外部与反向代理通信，请为步骤 1 中指定的端口设置 Azure 负载均衡器规则。

    	{
        	"apiVersion": "[variables('lbApiVersion')]",
        	"type": "Microsoft.Network/loadBalancers",
        	...
        	...
        	"loadBalancingRules": [
            	...
            	{
                	"name": "LBSFReverseProxyRule",
                	"properties": {
                    	"backendAddressPool": {
                        	"id": "[variables('lbPoolID0')]"
                    	},
                    	"backendPort": "[parameters('SFReverseProxyPort')]",
                    	"enableFloatingIP": "false",
                    	"frontendIPConfiguration": {
                        	"id": "[variables('lbIPConfig0')]"
                    	},
                    	"frontendPort": "[parameters('SFReverseProxyPort')]",
                    	"idleTimeoutInMinutes": "5",
                    	"probe": {
                        	"id": "[concat(variables('lbID0'),'/probes/SFReverseProxyProbe')]"
                    	},
                    	"protocol": "tcp"
                	}
            	}
        	],
        	"probes": [
            	...
            	{
                	"name": "SFReverseProxyProbe",
                	"properties": {
                    		"intervalInSeconds": 5,
                    		"numberOfProbes": 2,
                    		"port":     "[parameters('SFReverseProxyPort')]",
                    		"protocol": "tcp"
                	}
            	}  
        	]
    	}

4. 若要在反向代理的端口上配置 SSL 证书，请将证书添加到 **Cluster** [资源类型节](/documentation/articles/resource-group-authoring-templates/)中的 ***reverseProxyCertificate*** 属性。

	    {
	        "apiVersion": "2016-09-01",
	        "type": "Microsoft.ServiceFabric/clusters",
	        "name": "[parameters('clusterName')]",
	        "location": "[parameters('clusterLocation')]",
	        "dependsOn": [
	            "[concat('Microsoft.Storage/storageAccounts/', parameters('supportLogStorageAccountName'))]"
	        ],
	        "properties": {
	            ...
	            "reverseProxyCertificate": {
	                "thumbprint": "[parameters('sfReverseProxyCertificateThumbprint')]",
	                "x509StoreName": "[parameters('sfReverseProxyCertificateStoreName')]"
	            },
	            ...
	            "clusterState": "Default",
	        }
	    }

### <a name="supporting-a-reverse-proxy-certificate-thats-different-from-the-cluster-certificate"></a>支持不同于群集证书的反向代理证书
 如果反向代理证书不同于用于保护群集的证书，应将前面指定的证书安装在虚拟机上，并将其添加到访问控制列表 (ACL)，使 Service Fabric 能够访问它。 可在 **virtualMachineScaleSets** [资源类型节](/documentation/articles/resource-group-authoring-templates/)中执行此操作。 若要安装，请将该证书添加到 osProfile。 模板的扩展节可以更新 ACL 中的证书。

      {
        "apiVersion": "[variables('vmssApiVersion')]",
        "type": "Microsoft.Compute/virtualMachineScaleSets",
        ....
          "osProfile": {
              "adminPassword": "[parameters('adminPassword')]",
              "adminUsername": "[parameters('adminUsername')]",
              "computernamePrefix": "[parameters('vmNodeType0Name')]",
              "secrets": [
                {
                  "sourceVault": {
                    "id": "[parameters('sfReverseProxySourceVaultValue')]"
                  },
                  "vaultCertificates": [
                    {
                      "certificateStore": "[parameters('sfReverseProxyCertificateStoreValue')]",
                      "certificateUrl": "[parameters('sfReverseProxyCertificateUrlValue')]"
                    }
                  ]
                }
              ]
            }
       ....
       "extensions": [
              {
                  "name": "[concat(parameters('vmNodeType0Name'),'_ServiceFabricNode')]",
                  "properties": {
                          "type": "ServiceFabricNode",
                          "autoUpgradeMinorVersion": false,
                          ...
                          "publisher": "Microsoft.Azure.ServiceFabric",
                          "settings": {
                            "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
                            "nodeTypeRef": "[parameters('vmNodeType0Name')]",
                            "dataPath": "D:\\\\SvcFab",
                            "durabilityLevel": "Bronze",
                            "testExtension": true,
                            "reverseProxyCertificate": {
                              "thumbprint": "[parameters('sfReverseProxyCertificateThumbprint')]",
                              "x509StoreName": "[parameters('sfReverseProxyCertificateStoreValue')]"
                            },
                      },
                      "typeHandlerVersion": "1.0"
                  }
              },
          ]
        }

> [AZURE.NOTE]
> 在现有群集上使用不同于群集证书的证书来启用反向代理时，请在启用反向代理之前在群集上安装反向代理证书并更新 ACL。 在执行步骤 1-4 开始部署以启用反向代理之前，请使用上述设置完成 [Azure Resource Manager 模板](/documentation/articles/service-fabric-cluster-creation-via-arm/)部署。

## <a name="next-steps"></a>后续步骤
* 参阅 [GitHub 上的示例项目](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)中服务之间的 HTTP 通信示例。
* [使用 Reliable Services 远程控制执行远程过程调用](/documentation/articles/service-fabric-reliable-services-communication-remoting/)
* [Reliable Services 中使用 OWIN 的 Web API](/documentation/articles/service-fabric-reliable-services-communication-webapi/)
* [使用 Reliable Services 的 WCF 通信](/documentation/articles/service-fabric-reliable-services-communication-wcf/)

[0]: ./media/service-fabric-reverseproxy/external-communication.png
[1]: ./media/service-fabric-reverseproxy/internal-communication.png
