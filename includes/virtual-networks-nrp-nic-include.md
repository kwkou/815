## NIC
 
网络接口卡 (NIC) 资源提供与虚拟网络资源中现有子网的网络连接。尽管可以将 NIC 作为独立对象来创建，但你需要将其关联到另一个对象才能实际提供连接。NIC 可以用于将 VM 连接到一个子网、公共 IP 地址或负载平衡器。

|属性|说明|示例值|
|---|---|---|
|**virtualMachine**|与 NIC 关联的 VM。|/subscriptions/{guid}/../Microsoft.Compute/virtualMachines/vm1|
|**macAddress**|NIC 的 MAC 地址|介于 4 和 30 之间的任何值|
|**networkSecurityGroup**|与 NIC 关联的 NSG|/subscriptions/{guid}/../Microsoft.Network/networkSecurityGroups/myNSG1|
|**dnsSettings**|NIC 的 DNS 设置|请参阅 [PIP](#Public-IP-address)|

网络接口卡 (NIC) 表示可关联到虚拟机 (VM) 的网络接口。一个 VM 可以有一个或多个 NIC。

![单个 VM 上的 NIC](./media/resource-groups-networking/Figure3.png)

### IP 配置
NIC 具有一个名为 **ipConfigurations** 的子对象，包含以下属性：

|属性|说明|示例值|
|---|---|---|
|**subnet**|与 NIC 连接的子网。|/subscriptions/{guid}/../Microsoft.Network/virtualNetworks/myvnet1/subnets/mysub1|
|**privateIPAddress**|子网中 NIC 的 IP 地址|10\.0.0.8|
|**privateIPAllocationMethod**|IP 分配方法|动态或静态|
|**enableIPForwarding**|NIC 是否可以用于路由|true 或 false|
|**primary**|NIC 是否是 VM 的主 NIC|true 或 false|
|**publicIPAddress**|与 NIC 关联的 PIP|请参阅 [DNS 设置](#DNS-settings)|
|**loadBalancerBackendAddressPools**|与 NIC 关联的后端地址池||
|**loadBalancerInboundNatRules**|与 NIC 关联的入站负载平衡器 NAT 规则||

采用 JSON 格式的示例公共 IP 地址：

	{
	    "name": "lb-nic1-be",
	    "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/nrprg/providers/Microsoft.Network/networkInterfaces/lb-nic1-be",
	    "etag": "W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"",
	    "type": "Microsoft.Network/networkInterfaces",
	    "location": "eastus",
	    "properties": {
	        "provisioningState": "Succeeded",
	        "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
	        "ipConfigurations": [
	            {
	                "name": "NIC-config",
	                "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/nrprg/providers/Microsoft.Network/networkInterfaces/lb-nic1-be/ipConfigurations/NIC-config",
	                "etag": "W/"0027f1a2-3ac8-49de-b5d5-fd46550500b1"",
	                "properties": {
	                    "provisioningState": "Succeeded",
	                    "privateIPAddress": "10.0.0.4",
	                    "privateIPAllocationMethod": "Dynamic",
	                    "subnet": {
	                        "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/NRPRG/providers/Microsoft.Network/virtualNetworks/NRPVnet/subnets/NRPVnetSubnet"
	                    },
	                    "loadBalancerBackendAddressPools": [
	                        {
	                            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/backendAddressPools/NRPbackendpool"
	                        }
	                    ],
	                    "loadBalancerInboundNatRules": [
	                        {
	                            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/nrprg/providers/Microsoft.Network/loadBalancers/nrplb/inboundNatRules/rdp1"
	                        }
	                    ]
	                }
	            }
	        ],
	        "dnsSettings": { ... },
	        "macAddress": "00-0D-3A-10-F1-29",
	        "enableIPForwarding": false,
	        "primary": true,
	        "virtualMachine": {
	            "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/nrprg/providers/Microsoft.Compute/virtualMachines/web1"
	        }
	    }
	}

### 其他资源

- 阅读 NIC 的[ REST API 参考文档](https://msdn.microsoft.com/zh-cn/library/azure/mt163579.aspx)

<!---HONumber=79-->