##<a name="Network-Security-Group"></a> 网络安全组

使用 NSG 资源可以通过实现允许和拒绝规则，为工作负载创建安全边界。可将此类规则应用到 VM、NIC 或子网。

|属性|说明|示例值|
|---|---|---|
|**子网**|NSG 应用到的子网 ID 的列表。|/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd|
|**securityRules**|构成 NSG 的安全规则的列表|请参阅下面的[安全规则](#Security-rule)|
|**defaultSecurityRules**|每个 NSG 中存在的默认安全规则的列表|请参阅下面的[默认安全规则](#Default-security-rules)|

- **安全规则** - 一个 NSG 可以有多个定义的安全规则。每个规则可以允许或拒绝不同类型的流量。

###<a name="Security-rule"></a> 安全规则

安全规则是 NSG 的子资源，它包含以下属性。

|属性|说明|示例值|
|---|---|---|
|**description**|规则的说明|允许子网 X 中所有 VM 的入站流量|
|**protocol**|要与规则匹配的协议|TCP、UDP 或 *|
|**sourcePortRange**|要与规则匹配的源端口范围|80、100-200、*| 
|**destinationPortRange**|要与规则匹配的目标端口范围|80、100-200、*|
|**sourceAddressPrefix**|要与规则匹配的源地址前缀|10\.10.10.1、10.10.10.0/24、VirtualNetwork|
|**destinationAddressPrefix**|要与规则匹配的目标地址前缀|10\.10.10.1、10.10.10.0/24、VirtualNetwork|
|**direction**|要与规则匹配的流量方向|inbound 或 outbound|
|**priority**|规则的优先级。按优先顺序检查规则，只要应用了一个规则，就不会测试其他规则来进行匹配。|10、100、65000|
|**access**|规则匹配时要应用的访问类型|allow 或 deny|

采用 JSON 格式的示例 NSG：

	{
	    "name": "NSG-BackEnd",
	    "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-BackEnd",
	    "etag": "W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"",
	    "type": "Microsoft.Network/networkSecurityGroups",
	    "location": "chinanorth",
	    "tags": {
	        "displayName": "NSG - Front End"
	    },
	    "properties": {
	        "provisioningState": "Succeeded",
	        "resourceGuid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
	        "securityRules": [
	            {
	                "name": "rdp-rule",
	                "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-BackEnd/securityRules/rdp-rule",
	                "etag": "W/"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"",
	                "properties": {
	                    "provisioningState": "Succeeded",
	                    "description": "Allow RDP",
	                    "protocol": "Tcp",
	                    "sourcePortRange": "*",
	                    "destinationPortRange": "3389",
	                    "sourceAddressPrefix": "Internet",
	                    "destinationAddressPrefix": "*",
	                    "access": "Allow",
	                    "priority": 100,
	                    "direction": "Inbound"
	                }
	            }
	        ],
	        "defaultSecurityRules": [
	            { [...],
	        "subnets": [
	            {
	                "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd"
	            }
	        ]
	    }
	}

###<a name="Default-security-rules"></a> 默认安全规则

默认安全规则的属性与安全规则中提供的属性相同。默认安全规则的用途是在应用了 NSG 的资源之间提供基本连接。请确保知道存在哪些[默认安全规则](/documentation/articles/virtual-networks-nsg/#Default-Rules)。

### 其他资源

- 获取有关 [NSG](/documentation/articles/virtual-networks-nsg/) 的详细信息。
- 阅读 NSG 的[ REST API 参考文档](https://msdn.microsoft.com/zh-cn/library/azure/mt163615.aspx)
- 阅读 NSG 的[ REST API 参考文档](https://msdn.microsoft.com/zh-cn/library/azure/mt163580.aspx)

<!---HONumber=82-->