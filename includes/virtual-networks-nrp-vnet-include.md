## 虚拟网络
虚拟网络 (VNET) 和子网资源可帮助定义 Azure 中运行的工作负载的安全边界。VNet 的特征包括一个地址空间（定义为 CIDR 块）的集合。

>[AZURE.NOTE]网络管理员应熟悉 CIDR 表示法。如果你不熟悉 CIDR，请[了解详细信息](http://whatismyipaddress.com/cidr)。

![包含多个子网的 VNet](./media/resource-groups-networking/Figure4.png)

VNet 包含以下属性。

|属性|说明|示例值|
|---|---|---|
|**addressSpace**|在 CIDR 表示法中构成 VNet 的地址前缀集合|192\.168.0.0/16|
|**子网**|构成 VNet 的子网集合|请参阅下面的[子网](#Subnets)。|
|**ipAddress**|分配给对象的 IP 地址。这是只读属性。|104\.42.233.77|

### 子网
子网是 VNet 的子资源，可帮助定义使用 IP 地址前缀在 CIDR 块中定义地址空间的段。可以将 NIC 添加到子网，并连接到 VM，以便为各种工作负荷提供连接。

子网包含以下属性。

|属性|说明|示例值|
|---|---|---|
|**addressPrefix**|在 CIDR 表示法中构成子网的单个地址前缀|192\.168.1.0/24|
|**networkSecurityGroup**|应用到子网的 NSG|请参阅 [NSG](#Network-Security-Group)|
|**routeTable**|应用到子网的路由表|请参阅 [UDR](#Route-table)|
|**ipConfigurations**|NIC 用来连接子网的 IP 配置对象集合|请参阅 [UDR](#Route-table)|


采用 JSON 格式的示例 VNet：

	{
	    "name": "TestVNet",
	    "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet",
	    "etag": "W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"",
	    "type": "Microsoft.Network/virtualNetworks",
	    "location": "chinanorth",
	    "tags": {
	        "displayName": "VNet"
	    },
	    "properties": {
	        "provisioningState": "Succeeded",
	        "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
	        "addressSpace": {
	            "addressPrefixes": [
	                "192.168.0.0/16"
	            ]
	        },
	        "subnets": [
	            {
	                "name": "FrontEnd",
	                "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
	                "etag": "W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"",
	                "properties": {
	                    "provisioningState": "Succeeded",
	                    "addressPrefix": "192.168.1.0/24",
	                    "networkSecurityGroup": {
	                        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-BackEnd"
	                    },
	                    "routeTable": {
	                        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/routeTables/UDR-FrontEnd"
	                    },
	                    "ipConfigurations": [
	                        {
	                            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICWEB1/ipConfigurations/ipconfig1"
	                        },
	                        ...]
	                }
	            },
	            ...]
	    }
	}

### 其他资源

- 获取有关 [VNet](/documentation/articles/virtual-networks-overview/) 的详细信息。
- 阅读 VNet 的 [REST API 参考文档](https://msdn.microsoft.com/zh-cn/library/azure/mt163650.aspx)。
- 阅读子网的 [REST API 参考文档](https://msdn.microsoft.com/zh-cn/library/azure/mt163618.aspx)。

<!---HONumber=82-->