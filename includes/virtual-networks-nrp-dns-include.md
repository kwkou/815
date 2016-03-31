## Azure DNS

Azure DNS 是 DNS 域的托管服务，它使用 Azure 基础结构提供名称解析。


| 属性 | 说明 | 示例值 |
|---|---|---|
| DNS 区域 | 托管特定域的 DNS 记录的域区域信息 | /subscriptions/{guid}/.../providers/Microsoft.Network/dnszones/contoso.com"，
providers/Microsoft.Network/dnszones/contoso.com/A/www |
| DNS 区域 | 托管特定域的 DNS 记录的域区域信息 | /subscriptions/{guid}/.../providers/Microsoft.Network/dnszones/contoso.com"


### DNS 记录集

DNS 区域具有一个名为记录集的子对象。记录集是按 DNS 区域的类型排列的主机记录集合。记录类型包括 A、AAAA、CNAME、MX、NS、SOA、SRV 和 TXT。

| 属性 | 说明 | 示例值 |
|---|---|---|
| A | IPv4 记录类型 | /subscriptions/{guid}/.../providers/Microsoft.Network/dnszones/contoso.com/A/www |
| AAAA | IPv6 记录类型| /subscriptions/{guid}/.../providers/Microsoft.Network/dnszones/contoso.com/AAAA/hostrecord |
| CNAME | Canonical 名称记录类型 <sup>1</sup> | /subscriptions/{guid}/.../providers/Microsoft.Network/dnszones/contoso.com/CNAME/www |
| MX | 邮件记录类型 | /subscriptions/{guid}/.../providers/Microsoft.Network/dnszones/contoso.com/MX/mail |
| NS | 名称服务器记录类型 | /subscriptions/{guid}/.../providers/Microsoft.Network/dnszones/contoso.com/NS/ |
| SOA | 颁发机构记录类型开头 <sup>2</sup> | /subscriptions/{guid}/.../providers/Microsoft.Network/dnszones/contoso.com/SOA |
| SRV | 选择记录类型 | /subscriptions/{guid}/.../providers/Microsoft.Network/dnszones/contoso.com/SRV |

<sup>1</sup> 每个记录集只允许有一个值。

<sup>2</sup> 每个 DNS 区域只允许有一种记录类型 SOA。

采用 Json 格式的 DNS 区域的示例：

	{
	  "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json",
	  "contentVersion": "1.0.0.0",
	  "parameters": {
	    "newZoneName": {
	      "type": "String",
	      "metadata": {
	          "description": "The name of the DNS zone to be created."
	      }
	    },
	    "newRecordName": {
	      "type": "String",
	      "defaultValue": "www",
	      "metadata": {
	          "description": "The name of the DNS record to be created.  The name is relative to the zone, not the FQDN."
	      }
	    }
	  },
	  "resources": 
	  [
	    {
	      "type": "microsoft.network/dnszones",
	      "name": "[parameters('newZoneName')]",
	      "apiVersion": "2015-05-04-preview",
	      "location": "global",
	      "properties": {
	      }
	    },
	    {
	      "type": "microsoft.network/dnszones/a",
		  "name": "[concat(parameters('newZoneName'), concat('/', parameters('newRecordName')))]",
      	"apiVersion": "2015-05-04-preview",
      	"location": "global",
	  	"properties": 
	  	{
        	"TTL": 3600,
			"ARecords": 
			[
			    {
				    "ipv4Address": "1.2.3.4"
				},
				{
				    "ipv4Address": "1.2.3.5"
				}
			]
	  	},
	  	"dependsOn": [
        	"[concat('Microsoft.Network/dnszones/', parameters('newZoneName'))]"
      	]
    	}
	  	]
	}

## 其他资源

有关详细信息，请阅读 [DNS 区域的 REST API 文档](https://msdn.microsoft.com/zh-cn/library/azure/mt130626.aspx)。

有关详细信息，请阅读 [DNS 记录集的 REST API 文档](https://msdn.microsoft.com/zh-cn/library/azure/mt130627.aspx)。

<!---HONumber=Mooncake_1221_2015-->