##<a name="Route-table"></a> 路由表

路由表资源包含用于定义流量如何在 Azure 基础结构中流动的路由。你可以使用用户定义的路由 (UDR) 将所有流量从给定子网发送到虚拟设备，例如防火墙或入侵检测系统 (IDS)。可以将路由表关联到子网。

路由表包含以下属性。

|属性|说明|示例值|
|---|---|---|
|**routes**|路由表中用户定义的路由集合|请参阅[用户定义的路由](#User-defined-routes)|
|**子网**|路由表应用到的子网集合|请参阅[子网](#Subnets)|


###<a name="User-defined-routes"></a> 用户定义的路由

你可以根据流量的目标地址创建 UDR，以指定流量应发送到何处。可以将路由视为根据网络数据包目标地址定义的默认网关。

UDR 包含以下属性。

|属性|说明|示例值|
|---|---|---|
|**addressPrefix**|目标的地址前缀或完整 IP 地址|192\.168.1.0/24、192.168.1.101|
|**nextHopType**|流量要发送到的设备的类型。|VirtualAppliance、VPN 网关、Internet|
|**nextHopIpAddress**|下一跃点的 IP 地址|192\.168.1.4|


采用 JSON 格式的示例路由表：

	{
	    "name": "UDR-BackEnd",
	    "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/routeTables/UDR-BackEnd",
	    "etag": "W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"",
	    "type": "Microsoft.Network/routeTables",
	    "location": "chinanorth",
	    "properties": {
	        "provisioningState": "Succeeded",
	        "routes": [
	            {
	                "name": "RouteToFrontEnd",
	                "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/routeTables/UDR-BackEnd/routes/RouteToFrontEnd",
	                "etag": "W/"v"",
	                "properties": {
	                    "provisioningState": "Succeeded",
	                    "addressPrefix": "192.168.1.0/24",
	                    "nextHopType": "VirtualAppliance",
	                    "nextHopIpAddress": "192.168.0.4"
	                }
	            }
	        ],
	        "subnets": [
	            {
	                "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/BackEnd"
	            }
	        ]
	    }
	}

### 其他资源

- 获取有关 [UDR](/documentation/articles/virtual-networks-udr-overview/) 的详细信息。
- 阅读路由表的 [REST API 参考文档](https://msdn.microsoft.com/zh-cn/library/azure/mt502549.aspx)
- 阅读用户定义的路由 (UDR) 的 [REST API 参考文档](https://msdn.microsoft.com/zh-cn/library/azure/mt502539.aspx)。

<!---HONumber=82-->