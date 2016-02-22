## 流量管理器配置文件

使用流量管理器及其子终结点资源可将 DNS 路由到 Azure 内部和 Azure 外部的终结点。此类流量分配由路由策略方法控制。使用流量管理器还能监视终结点的运行状况，并根据终结点的运行状况重定向流量。

| 属性 | 说明 |
|---|---|
|**trafficRoutingMethod**| 可能的值为 *Performance*、*Weighted* 和 *Priority* | 
| **dnsConfig** | 配置文件的 FQDN | 
| **协议** | 监视协议，可能的值为 *HTTP* 和 *HTTPS*|
| **端口** | 监视端口 |  
| **路径** | 监视路径 |
| **终结点** | 终结点资源的容器 | 

### 终结点 

终结点是流量管理器配置文件的子资源。它表示要根据流量管理器配置文件资源中配置的策略，将用户流量分配到的服务或 Web 终结点。

| 属性 | 说明 | 
|---|---| 
| **类型** | 终结点的类型，可能的值为“Azure 终结点”、“外部终结点”和“嵌套终结点”。 | 
| **targetResourceId** | 服务或 Web 终结点的公共 IP 地址。这可能是 Azure 或外部终结点。 | 
| **重量** | 流量管理中使用的终结点权重。 | 
| **Priority** | 用于定义故障转移操作的终结点优先级 |

采用 Json 格式的流量管理器示例：


        {
            "apiVersion": "[variables('tmApiVersion')]",
            "type": "Microsoft.Network/trafficManagerProfiles",
            "name": "VMEndpointExample",
            "location": "global",
            "dependsOn": [
                "[concat('Microsoft.Network/publicIPAddresses/', variables('publicIPAddressName'), '0')]",
                "[concat('Microsoft.Network/publicIPAddresses/', variables('publicIPAddressName'), '1')]",
                "[concat('Microsoft.Network/publicIPAddresses/', variables('publicIPAddressName'), '2')]",
            ],
            "properties": {
                "profileStatus": "Enabled",
                "trafficRoutingMethod": "Weighted",
                "dnsConfig": {
                    "relativeName": "[parameters('dnsname')]",
                    "ttl": 30
                },
                "monitorConfig": {
                    "protocol": "http",
                    "port": 80,
                    "path": "/"
                },
                "endpoints": [
                    {
                        "name": "endpoint0",
                        "type": "Microsoft.Network/trafficManagerProfiles/azureEndpoints",
                        "properties": {
                            "targetResourceId": "[resourceId('Microsoft.Network/publicIPAddresses',concat(variables('publicIPAddressName'), 0))]",
                            "endpointStatus": "Enabled",
                            "weight": 1
                        }
                    },
                    {
                        "name": "endpoint1",
                        "type": "Microsoft.Network/trafficManagerProfiles/azureEndpoints",
                        "properties": {
                            "targetResourceId": "[resourceId('Microsoft.Network/publicIPAddresses',concat(variables('publicIPAddressName'), 1))]",
                            "endpointStatus": "Enabled",
                            "weight": 1
                        }
                    },
                    {
                        "name": "endpoint2",
                        "type": "Microsoft.Network/trafficManagerProfiles/azureEndpoints",
                        "properties": {
                            "targetResourceId": "[resourceId('Microsoft.Network/publicIPAddresses',concat(variables('publicIPAddressName'), 2))]",
                            "endpointStatus": "Enabled",
                            "weight": 1
                        }
                    }
                ]
            }
        }

 
## 其他资源

有关详细信息，请阅读[流量管理器的 REST API 文档](https://msdn.microsoft.com/zh-cn/library/azure/mt163664.aspx)。

<!---HONumber=Mooncake_1221_2015-->