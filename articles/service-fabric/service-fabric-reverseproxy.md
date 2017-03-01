<properties
    pageTitle="Service Fabric 反向代理 | Azure"
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
    ms.date="01/04/2017"
    wacn.date="02/20/2017"
    ms.author="bharatn" />  


# Service Fabric 反向代理

Service Fabric 反向代理是内建于 Service Fabric 中的反向代理，其作用是处理 Service Fabric 群集中用于公开 HTTP 终结点的微服务。

## 微服务通信模型

Service Fabric 中的微服务通常在群集的一部分 VM 中运行，并且可以出于各种原因从一个 VM 移动到另一个 VM。因此，微服务的终结点可能会动态变化。与微服务通信的经典模式是下面的解析循环。

1. 一开始通过命名服务解析服务位置。
2. 连接到服务。
3. 确定连接失败的原因，必要时重新解析服务位置。

此过程通常涉及将客户端通信库包装到重试循环中，以便执行服务解析和重试策略。有关本主题的详细信息，请参阅[与服务通信](/documentation/articles/service-fabric-connect-and-communicate-with-services/)。

### 通过 SF 反向代理进行通信
Service Fabric 反向代理在群集的所有节点上运行。它会代表客户端执行整个服务解析流程，然后再转发客户端请求。因此，在群集上运行的客户端可以通过在同一节点上以本地方式运行的 SF 反向代理，直接使用任何客户端 HTTP 通信库与目标服务通信。

![内部通信][1]

## 从群集外部访问微服务
微服务的默认外部通信模型为“选择加入”，即默认情况下，不能直接从外部客户端访问每个服务。[Azure 负载均衡器](/documentation/articles/load-balancer-overview/) 充当微服务和外部客户端之间的网络边界，可以进行网络地址转换并将外部请求转发到内部的 **IP:端口**终结点。若要允许外部客户端直接访问微服务的终结点，必须先将 Azure Load Balancer 配置为将流量转发到群集中服务使用的每个端口。另外，大多数微服务（尤其是有状态微服务）并不是位于群集的所有节点上，这些微服务在故障转移时可以在节点之间移动，因此在这样的情况下，Azure Load Balancer 无法有效地确定副本的目标节点的位置，无法向其转发流量。

### 从群集外部通过 SF 反向代理访问微服务
可以在 Azure 负载均衡器中直接配置 SF 反向代理端口，不需配置各个服务的端口。因此，群集外部的客户端可以通过反向代理访问群集内部的服务，不需额外进行配置。

![外部通信][0]

>[AZURE.WARNING] 在负载均衡器上配置反向代理的端口以后，即可从群集外部访问群集中公开了 HTTP 终结点的所有微服务。


## 通过反向代理进行服务寻址的 URI 格式

反向代理使用特定的 URI 格式来确定传入请求所应转发到的服务分区：


	http(s)://<Cluster FQDN | internal IP>:Port/<ServiceInstanceName>/<Suffix path>?PartitionKey=<key>&PartitionKind=<partitionkind>&Timeout=<timeout_in_seconds>


 - **http(s):** 可以将反向代理配置为接受 HTTP 或 HTTPS 流量。如果为 HTTPS 流量，则会在反向代理中出现 SSL 终止的情况。由反向代理转发到群集中服务的请求是通过 HTTP 进行的。
 - **群集 FQDN| internal IP:** For external clients, the reverse proxy can be configured so that it is reachable through the cluster domain (e.g., mycluster.chinaeast.cloudapp.chinacloudapi.cn). By default the reverse proxy runs on every node, so for internal traffic it can be reached on localhost or on any internal node IP (e.g., 10.0.0.1).
 - **Port:** 为反向代理指定的端口。例如：19008。
 - **ServiceInstanceName:** 这是要在不使用“fabric:/”方案的情况下访问的服务的完全限定式已部署服务实例名称。例如，若要访问服务 *fabric:/myapp/myservice/*，可使用 *myapp/myservice*。
 - **Suffix path:** 这是要连接到的服务的实际 URL 路径。例如，*myapi/values/add/3*
 - **PartitionKey:** 对于已分区服务，这是针对你要访问的分区进行计算所得的分区键。请注意，这*不* 是分区 ID GUID。对于使用单独分区方案的服务，此参数不是必需的。
 - **PartitionKind:** 服务分区方案。该方案可以是“Int64Range”或“Named”。对于使用单独分区方案的服务，此参数不是必需的。
 - **Timeout:** 此参数指定反向代理针对服务创建的 HTTP 请求（代表客户端请求）的超时。此参数的默认值为 60 秒。这是一个可选参数。

### 用法示例

以服务 **fabric:/MyApp/MyService** 为例，该服务可针对以下 URL 打开一个 HTTP 侦听器：


	http://10.0.05:10592/3f0d39ad-924b-4233-b4a7-02617c6308a6-130834621071472715/


使用以下资源：

 - `/index.html`
 - `/api/users/<userId>`

如果服务使用单独分区方案，则 *PartitionKey* 和 *PartitionKind* 查询字符串参数不是必需的，可以通过网关访问服务，如下所示：

 - 外部访问方式：`http://mycluster.chinaeast.cloudapp.chinacloudapi.cn:19008/MyApp/MyService`
 - 内部访问方式：`http://localhost:19008/MyApp/MyService`

如果服务使用“统一 Int64”分区方案，则必须使用 *PartitionKey* 和 *PartitionKind* 查询字符串来访问服务的分区：

 - 外部访问方式：`http://mycluster.chinaeast.cloudapp.chinacloudapi.cn:19008/MyApp/MyService?PartitionKey=3&PartitionKind=Int64Range`
 - 内部访问方式：`http://localhost:19008/MyApp/MyService?PartitionKey=3&PartitionKind=Int64Range`

若要访问服务所公开的资源，可直接在 URL 中将资源路径置于服务名称之后：

 - 外部访问方式：`http://mycluster.chinaeast.cloudapp.chinacloudapi.cn:19008/MyApp/MyService/index.html?PartitionKey=3&PartitionKind=Int64Range`
 - 内部访问方式：`http://localhost:19008/MyApp/MyService/api/users/6?PartitionKey=3&PartitionKind=Int64Range`

然后，网关会将这些请求转发到服务的 URL：

 - `http://10.0.05:10592/3f0d39ad-924b-4233-b4a7-02617c6308a6-130834621071472715/index.html`
 - `http://10.0.05:10592/3f0d39ad-924b-4233-b4a7-02617c6308a6-130834621071472715/api/users/6`

## 针对端口共享服务进行特殊处理

无法访问某个服务时，应用程序网关会尝试重新解析服务地址以及重试请求。这是网关的主要优势之一，因为客户端代码不需实施自己的服务解析操作和解析循环。

当某个服务无法访问时，通常意味着服务实例或副本已在其常规生命周期中转移到其他节点。发生这种情况时，网关可能会收到网络连接错误，指出在原来解析过的地址上的某个终结点不再处于开放状态。

不过，副本或服务实例可能会共享主机进程，在通过基于 http.sys 的 Web 服务器进行托管的情况下还可能会共享端口，这些 Web 服务器包括：

 - [System.Net.HttpListener](https://msdn.microsoft.com/zh-cn/library/system.net.httplistener%28v=vs.110%29.aspx)
 - [ASP.NET Core WebListener](https://docs.asp.net/latest/fundamentals/servers.html#weblistener)
 - [Katana](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.OwinSelfHost/)

在这样的情形下，可能会出现 Web 服务器出现在主机进程中并且能够响应请求，而被解析的服务实例或副本却再也不能在主机上使用的情况。这种情况下，网关会从 Web 服务器收到 HTTP 404 响应。因此，HTTP 404 具有两种不同的含义：

 1. 服务地址正确，但用户所请求的资源不存在。
 2. 服务地址不正确，且用户所请求的资源实际上可能存在于其他节点上。

第一种情况为正常 HTTP 404，属于用户错误。而第二种情况则是用户请求的资源确实存在，但网关却找不到该资源，因为服务本身已移动，这种情况下网关需重新解析地址，然后重试。

因此，网关需要通过某种方式来区分这两种情况。为了进行这种区分，需要从服务器获得提示。

 - 默认情况下，应用程序网关会假定第二种情况属于事实，因此会尝试重新解析和重新发出请求。
 - 若要向应用程序网关指示第一种情况属于事实，服务应返回以下 HTTP 响应标头：

`X-ServiceFabric : ResourceNotFound`

此 HTTP 响应标头指示的是正常的 HTTP 404 情形，即所请求的资源不存在，因此网关不会尝试重新解析服务地址。

## 安装和配置
可以通过 [Azure Resource Manager 模板](/documentation/articles/service-fabric-cluster-creation-via-arm/)为群集启用 Service Fabric 反向代理。

获得要部署的群集的模板以后（不管你是通过示例模板获得，还是通过创建自定义 Resource Manager 模板来获得），即可通过以下步骤在模板中启用反向代理。

1. 在模板的[“参数”部分](/documentation/articles/resource-group-authoring-templates/)定义反向代理的端口。


    	"SFReverseProxyPort": {
        	"type": "int",
        	"defaultValue": 19008,
        	"metadata": {
            	"description": "Endpoint for Service Fabric Reverse proxy"
        	}
    	},

2. 为**群集**的[“资源类型”部分](/documentation/articles/resource-group-authoring-templates/)中的每个 nodetype 对象指定端口

    对于“2016-09-01”以前的 apiVersion，端口由参数名称 ***httpApplicationGatewayEndpointPort*** 标识

    	{
        	"apiVersion": "2016-03-01",
        	"type": "Microsoft.ServiceFabric/clusters",
        	"name": "[parameters('clusterName')]",
        	"location": "[parameters('clusterLocation')]",
        	...
       		"nodeTypes": [
          	   {
	        	...
	        	"httpApplicationGatewayEndpointPort": "[parameters('SFReverseProxyPort')]",
	        	...
          	   },
        	...
        	],
        	...
    	}

    对于“2016-09-01”或以后的 apiVersion，端口由参数名称 ***reverseProxyEndpointPort*** 标识


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


3. 若要从 Azure 群集外部与反向代理通信，请为步骤 1 中指定的端口设置 **Azure 负载均衡器规则**。


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

4. 若要在反向代理的端口上配置 SSL 证书，请在**群集**的[“资源类型”部分](/documentation/articles/resource-group-authoring-templates/)将证书添加到 httpApplicationGatewayCertificate 属性中

    对于“2016-09-01”以前的 apiVersion，证书由参数名称 ***httpApplicationGatewayCertificate*** 标识

    	{
        	"apiVersion": "2016-03-01",
        	"type": "Microsoft.ServiceFabric/clusters",
        	"name": "[parameters('clusterName')]",
        	"location": "[parameters('clusterLocation')]",
        	"dependsOn": [
            		"[concat('Microsoft.Storage/storageAccounts/', parameters('supportLogStorageAccountName'))]"
        	],
        	"properties": {
            		...
            		"httpApplicationGatewayCertificate": {
                		"thumbprint": "[parameters('sfReverseProxyCertificateThumbprint')]",
                		"x509StoreName": "[parameters('sfReverseProxyCertificateStoreName')]"
            		},
            		...
            		"clusterState": "Default",
        	}
    	}

    对于“2016-09-01”或以后的 apiVersion，证书由参数名称 ***reverseProxyCertificate*** 标识
    

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


## 后续步骤
 - 请参阅 [GitHUb 上的示例项目](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started/tree/master/Services/WordCount)中服务之间的 HTTP 通信示例。

 - [使用 Reliable Services 远程控制执行远程过程调用](/documentation/articles/service-fabric-reliable-services-communication-remoting/)

 - [Reliable Services 中使用 OWIN 的 Web API](/documentation/articles/service-fabric-reliable-services-communication-webapi/)

 - [使用 Reliable Services 的 WCF 通信](/documentation/articles/service-fabric-reliable-services-communication-wcf/)


[0]: ./media/service-fabric-reverseproxy/external-communication.png
[1]: ./media/service-fabric-reverseproxy/internal-communication.png

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description: wording udpate-->