##<a name="Public-IP-address"></a> 公共 IP 地址

公共 IP 地址提供保留的或面向 Internet 的动态 IP 地址资源。尽管可以创建作为独立对象的公共 IP 地址，但你需要将其关联到另一个对象才能实际使用该地址。你可以将公共 IP 地址关联到负载平衡器、应用程序网关或 NIC 以提供对这些资源的 Internet 访问。

|属性|说明|示例值|
|---|---|---|
|**publicIPAllocationMethod**|定义 IP 地址是*静态*的还是*动态*的。|static、dynamic|
|**idleTimeoutInMinutes**|定义空闲超时，默认值为 4 分钟。如果在此时段内未收到给定会话的其他数据包，将会终止该会话。|介于 4 和 30 之间的任何值|
|**ipAddress**|分配给对象的 IP 地址。这是只读属性。|104\.42.233.77|

###<a name="DNS-settings"></a> DNS 设置

公共 IP 地址具有一个名为 **dnsSettings** 的子对象，该对象包含以下属性：

|属性|说明|示例值|
|---|---|---|
|**domainNameLabel**|命名的主机，用于名称解析。|www、ftp、vm1|
|**fqdn**|公共 IP 的完全限定名称。|www.chinanorth.chinacloudapp.cn|
|**reverseFqdn**|完全限定的域名，可解析为 IP 地址并在 DNS 中注册为 PTR 记录。|www.contoso.com。|

采用 JSON 格式的示例公共 IP 地址：

	{
	   "name": "PIP01",
	   "location": "North US",
	   "tags": { "key": "value" },
	   "properties": {
	      "publicIPAllocationMethod": "Static",
	      "idleTimeoutInMinutes": 4,
		  "ipAddress": "104.42.233.77",
	      "dnsSettings": {
	         "domainNameLabel": "mylabel",
			 "fqdn": "mylabel.chinanorth.chinacloudapp.cn",
	         "reverseFqdn": "contoso.com."
	      }
	   }
	} 

### 其他资源

- 获取有关[公共 IP 地址](/documentation/articles/virtual-networks-reserved-public-ip/)的详细信息。
- 了解[实例层级公共 IP 地址](/documentation/articles/virtual-networks-instance-level-public-ip/)。
- 阅读公共 IP 地址的 [REST API 参考文档](https://msdn.microsoft.com/zh-cn/library/azure/mt163638.aspx)。

<!---HONumber=82-->