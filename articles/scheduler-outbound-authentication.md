<properties 
 pageTitle="计划程序出站身份验证" 
 description="" 
 services="scheduler" 
 documentationCenter=".NET" 
 authors="krisragh" 
 manager="dwrede" 
 editor=""/>
<tags 
 ms.service="scheduler" 
 ms.date="08/04/2015" 
 wacn.date="09/16/2015"/>
 
# 计划程序出站身份验证

计划程序作业可能需要调用要求进行身份验证的服务。这样，被调用的服务可以确定计划程序作业是否可以访问其资源。其中的某些服务包括其他 Azure 服务、Salesforce.com、Facebook 和安全的自定义网站。

## 添加和删除身份验证

将身份验证添加到计划程序作业非常简单 – 只需在创建或更新作业时将一个 JSON 子元素 `authentication` 添加到 `request` 元素。在 PUT、PATCH 或 POST 请求中（作为 `authentication` 对象的一部分）传递给计划程序服务的机密永远不会在响应中返回。在响应中，机密信息将设置为 null，或者包含一个表示已经过身份验证的实体的公共令牌。

若要删除身份验证，请对作业显式执行 PUT 或 PATCH，并将 `authentication` 对象设置为 null。响应中不会传回任何身份验证属性。

目前，支持的身份验证类型仅包括 `ClientCertificate` 模型（表示使用 SSL/TLS 客户端证书）、 `Basic` 模型（表示基本身份验证）和 `ActiveDirectoryOAuth` 模型（表示 Active Directory OAuth 身份验证）。

## ClientCertificate 身份验证的请求正文

使用 `ClientCertificate` 模型添加身份验证时，请在请求正文中指定以下附加元素。  

|元素|说明|
|:---|:---|
|_authentication（父元素）_|用于使用 SSL 客户端证书的身份验证对象。|
|_type_|必需。身份验证的类型。对于 SSL 客户端证书，该值必须是 `ClientCertificate`。|
|_pfx_|必需。PFX 文件的 Base64 编码内容。|
|_password_|必需。用于访问 PFX 文件的密码。|


## ClientCertificate 身份验证的响应正文

发送包含身份验证信息的请求时，响应将包含以下与身份验证相关的元素。

|元素 |说明 |
|:--|:--|
|_authentication（父元素）_ |用于使用 SSL 客户端证书的身份验证对象。|
|_type_ |身份验证的类型。对于 SSL 客户端证书，该值为 `ClientCertificate`。|
|_certificateThumbprint_ |证书的指纹。|
|_certificateSubjectName_ |证书的使用者可分辨名称。|
|_certificateExpiration_ |证书的过期日期。|

## ClientCertificate 身份验证的示例请求和响应

以下示例请求将发出包含 `ClientCertificate` 身份验证的 PUT 请求。该请求如下所示：


	PUT https://management.core.chinacloudapi.cn/7e2dffb5-45b5-475a-91be-d3d9973c82d5/cloudservices/cs-brazilsouth-scheduler/resources/scheduler/~/JobCollections/testScheduler/jobs/testScheduler 
	x-ms-version: 2013-03-01
	User-Agent: Microsoft.WindowsAzure.Scheduler.SchedulerClient/3.0.0.0 AzurePowershell/v0.8.10
	Content-Type: application/json; charset=utf-8
	Host: management.core.chinacloudapi.cn
	Content-Length: 4013
	Expect: 100-continue

	{
	  "action": {
		"type": "http",
		"request": {
		  "uri": "https://management.core.chinacloudapi.cn/7e2dffb5-45b5-475a-91be-d3d9973c82d5/cloudservices/CS-NorthCentralUS-scheduler/resources/scheduler/~/JobCollections/testScheduler/jobs/test",
		  "method": "GET",
		  "headers": {
			"x-ms-version": "2013-03-01"
		  },
		  "authentication": {
			"type": "clientcertificate",
			"password": "test",
			"pfx": "long-pfx-key”
		  }
		}
	  },
	  "recurrence": {
		"frequency": "minute",
		"interval": 1
	  }
	}

发送此请求后，响应将如下所示：

	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Pragma: no-cache
	Content-Length: 721
	Content-Type: application/json; charset=utf-8
	Expires: -1
	Server: 1.0.6198.153 (rd_rdfe_stable.141027-2149) Microsoft-HTTPAPI/2.0
	x-ms-servedbyregion: ussouth2
	X-AspNet-Version: 4.0.30319
	X-Powered-By: ASP.NET
	 

	{
	  "id": "testScheduler",
	  "action": {
		"request": {
		  "uri": "https://management.core.chinacloudapi.cn\/7e2dffb5-45b5-475a-91be-d3d9973c82d5\/cloudservices\/CS-NorthCentralUS-scheduler\/resources\/scheduler\/~\/JobCollections\/testScheduler\/jobs\/test",
		  "method": "GET",
		  "headers": {
			"x-ms-version": "2013-03-01"
		  },
		  "authentication": {
			"type": "ClientCertificate",
			"certificateThumbprint": "C1645E2AF6317D9FCF9C78FE23F9DE0DAFAD2AB5",
			"certificateExpiration": "2021-01-01T08:00:00Z",
			"certificateSubjectName": "CN=Scheduler Management"
		  }
		},
		"type": "http"
	  },
	  "recurrence": {
		"frequency": "minute",
		"interval": 1
	  },
	  "state": "enabled",
	  "status": {
		"nextExecutionTime": "2014-10-29T21:52:35.2108904Z",
		"executionCount": 0,
		"failureCount": 0,
		"faultedCount": 0
	  }
	}
## 基本身份验证的请求正文

使用 `Basic` 模型添加身份验证时，请在请求正文中指定以下附加元素。

|元素|说明|
|:--|:--|
|_authentication（父元素）_ |用于使用基本身份验证的身份验证对象。|
|_type_ |必需。身份验证的类型。对于基本身份验证，该值必须是 `Basic`。|
|_username_ |必需。要进行身份验证的用户名。|
|_password_ |必需。要进行身份验证的密码。|

## 基本身份验证的响应正文

发送包含身份验证信息的请求时，响应将包含以下与身份验证相关的元素。

|元素|说明|
|:--|:--|
|_authentication（父元素）_ |用于使用基本身份验证的身份验证对象。|
|_type_ |身份验证的类型。对于基本身份验证，该值为 `Basic`。|
|_username_ |经过身份验证的用户名。|

## 基本身份验证的示例请求和响应

以下示例请求将发出包含 `Basic` 身份验证的 PUT 请求。该请求如下所示：

	PUT https://management.core.chinacloudapi.cn/7e2dffb5-45b5-475a-91be-d3d9973c82d5/cloudservices/cs-brazilsouth-scheduler/resources/scheduler/~/JobCollections/testScheduler/jobs/testScheduler 
	x-ms-version: 2013-03-01
	User-Agent: Microsoft.WindowsAzure.Scheduler.SchedulerClient/3.0.0.0 AzurePowershell/v0.8.10
	Content-Type: application/json; charset=utf-8
	Host: management.core.chinacloudapi.cn
	Expect: 100-continue

	{
	  "action": {
		"type": "http",
		"request": {
		  "uri": "https://management.core.chinacloudapi.cn/7e2dffb5-45b5-475a-91be-d3d9973c82d5/cloudservices/CS-NorthCentralUS-scheduler/resources/scheduler/~/JobCollections/testScheduler/jobs/test",
		  "method": "GET",
		  "headers": {
			"x-ms-version": "2013-03-01"
		  },
		"authentication":{  
		  "username":"user1",
		  "password":"password",
		  "type":"basic"
		  }           
		}
	  },
	  "recurrence": {
		"frequency": "minute",
		"interval": 1
	  }
	}

发送此请求后，响应将如下所示：

	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Pragma: no-cache
	Content-Length: 721
	Content-Type: application/json; charset=utf-8
	Expires: -1
	Server: 1.0.6198.153 (rd_rdfe_stable.141027-2149) Microsoft-HTTPAPI/2.0
	x-ms-servedbyregion: ussouth2
	X-AspNet-Version: 4.0.30319
	X-Powered-By: ASP.NET

	{
	  "id": "testScheduler",
	  "action": {
		"request": {
		  "uri": "https://management.core.chinacloudapi.cn/7e2dffb5-45b5-475a-91be-d3d9973c82d5/cloudservices/CS-NorthCentralUS-scheduler/resources/scheduler/~/JobCollections/testScheduler/jobs/test",
		  "method": "GET",
		  "headers": {
			"x-ms-version": "2013-03-01"
		  },
		  "authentication":{  
			"username":"user1",
			"type":"Basic"
		  }
		},
		"type": "http"
	  },
	  "recurrence": {
		"frequency": "minute",
		"interval": 1
	  },
	  "state": "enabled",
	  "status": {
		"nextExecutionTime": "2014-10-29T21:52:35.2108904Z",
		"executionCount": 0,
		"failureCount": 0,
		"faultedCount": 0
	  }
	}

## ActiveDirectoryOAuth 身份验证的请求正文

使用 `ActiveDirectoryOAuth` 模型添加身份验证时，请在请求正文中指定以下附加元素。

|元素 |说明 |
|:--|:--|
|_authentication（父元素）_ |用于使用 ActiveDirectoryOAuth 身份验证的身份验证对象。|
|_type_ |必需。身份验证的类型。对于 ActiveDirectoryOAuth 身份验证，该值必须是 `ActiveDirectoryOAuth`。|
|_tenant_ |必需。租户标识符是用于标识 AD 租户的 ID。|
|_audience_ |必需。此元素设置为 https://management.core.chinacloudapi.cn/。|
|_clientId_ |必需。为 Azure AD 应用程序提供客户端标识符。|
|_secret_ |必需。正在请求令牌的客户端的机密。|

## ActiveDirectoryOAuth 身份验证的响应正文

发送包含身份验证信息的请求时，响应将包含以下与身份验证相关的元素。

|元素 |说明 |
|:--|:--|
|_authentication（父元素）_ |用于使用 ActiveDirectoryOAuth 身份验证的身份验证对象。|
|_type_ |身份验证的类型。对于 ActiveDirectoryOAuth 身份验证，该值为 `ActiveDirectoryOAuth`。|
|_tenant_ |用于标识 AD 租户的租户标识符。|
|_audience_ |此元素设置为 https://management.core.chinacloudapi.cn/。|
|_clientId_ |Azure AD 应用程序的客户端标识符。|

## ActiveDirectoryOAuth 身份验证的示例请求和响应

以下示例请求将发出包含 `ActiveDirectoryOAuth` 身份验证的 PUT 请求。该请求如下所示：

	PUT https://management.core.chinacloudapi.cn/7e2dffb5-45b5-475a-91be-d3d9973c82d5/cloudservices/cs-brazilsouth-scheduler/resources/scheduler/~/JobCollections/testScheduler/jobs/testScheduler 
	x-ms-version: 2013-03-01
	User-Agent: Microsoft.WindowsAzure.Scheduler.SchedulerClient/3.0.0.0 AzurePowershell/v0.8.10
	Content-Type: application/json; charset=utf-8
	Host: management.core.chinacloudapi.cn
	Expect: 100-continue

	{
	  "action": {
		"type": "http",
		"request": {
		  "uri": "https://management.core.chinacloudapi.cn/7e2dffb5-45b5-475a-91be-d3d9973c82d5/cloudservices/CS-NorthCentralUS-scheduler/resources/scheduler/~/JobCollections/testScheduler/jobs/test",
		  "method": "GET",
		  "headers": {
			"x-ms-version": "2013-03-01"
		  },
		  "authentication":{  
			"tenant":"contoso.com",
			"audience":"https://management.core.chinacloudapi.cn/",
			"clientId":"8a14db88-4d1a-46c7-8429-20323727dfab",
			"secret": "&lt;secret-key&gt;",
			"type":"ActiveDirectoryOAuth"
		  }                      
		}
	  },
	  "recurrence": {
		"frequency": "minute",
		"interval": 1
	  }
	}

发送此请求后，响应将如下所示：

	HTTP/1.1 201 Created
	Cache-Control: no-cache
	Pragma: no-cache
	Content-Length: 721
	Content-Type: application/json; charset=utf-8
	Expires: -1
	Server: 1.0.6198.153 (rd_rdfe_stable.141027-2149) Microsoft-HTTPAPI/2.0
	x-ms-servedbyregion: ussouth2
	X-AspNet-Version: 4.0.30319
	X-Powered-By: ASP.NET


	{
	  "id": "testScheduler",
	  "action": {
		"request": {
		  "uri": "https://management.core.chinacloudapi.cn/7e2dffb5-45b5-475a-91be-d3d9973c82d5/cloudservices/CS-NorthCentralUS-scheduler/resources/scheduler/~/JobCollections/testScheduler/jobs/test",
		  "method": "GET",
		  "headers": {
			"x-ms-version": "2013-03-01"
		  },
		  "authentication":{  
			"tenant":"contoso.com",
			"audience":"https://management.core.chinacloudapi.cn/",
			"clientId":"8a14db88-4d1a-46c7-8429-20323727dfab",
			"type":"ActiveDirectoryOAuth"
		  }
		},
		"type": "http"
	  },
	  "recurrence": {
		"frequency": "minute",
		"interval": 1
	  },
	  "state": "enabled",
	  "status": {
		"nextExecutionTime": "2014-10-29T21:52:35.2108904Z",
		"executionCount": 0,
		"failureCount": 0,
		"faultedCount": 0
	  }
	}

## 另请参阅

 [计划程序是什么？](/documentation/articles/scheduler-intro) [计划程序的概念、术语和实体层次结构](/documentation/articles/scheduler-concepts-terms)
 
 [开始在管理门户中使用计划程序](/documentation/articles/scheduler-get-started-portal)
 
 [Azure 计划程序中的计划和计费](/documentation/articles/scheduler-plans-billing)
 
 [如何使用 Azure 计划程序生成复杂的计划和高级重复执行](/documentation/articles/scheduler-advanced-complexity)
 
 [计划程序 REST API 参考](https://msdn.microsoft.com/zh-CN/library/dn528946)   
 
 [计划程序的高可用性和可靠性](/documentation/articles/scheduler-high-availability-reliability)
 
 [计划程序的限制、默认值和错误代码](/documentation/articles/scheduler-limits-defaults-errors)
 
 [计划程序的限制、默认值和错误代码](/documentation/articles/scheduler-limits-defaults-errors)
 

<!---HONumber=69-->