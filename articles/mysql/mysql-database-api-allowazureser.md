<properties linkid="" urlDisplayName="" pageTitle="使用PowerShell管理MySQL Database on Azure - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,入门指南,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, API, Azure MySQL Service, Azure RDS" description="本文介绍如何通过API实现更多MySQL Database on Azure的查询、创建、修改、删除等操作。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="sofia" solutions="" manager="" editor="" />  

<tags ms.service="mysql" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="cn" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-api-allowazureser/)
- [English](/documentation/articles/mysql-database-enus-api-allowazureser/)

#更新服务器-允许Azure服务访问

##请求
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>方法</strong>
    </td>
  <th align="left"><strong>请求 URI</strong>
    </td>
  
  <tr>
    <td>PATCH    </td>
    <td>https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.MySql/servers/{server-name}?api-version=2015-09-01</td>
  </tr>
</table>

###URI参数
无

###请求 Headers
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>请求包头</strong>
    </td>
  <th align="left"><strong>描述 URI</strong>
    </td>
  
  <tr>
    <td>x-ms-client-request-id</td>
    <td>可选。由客户端产生的不超过1KB字符的opaque值。强烈推荐设置此值，服务器端可通过此值获取客户端的活动信息。</td>
  </tr>
</table>

###请求 Body
创建或更新MySQL on Azure服务器，需写明一下参数，Json示例文件如下：
```
{
  "properties": {
    "allowAzureServices": "true"
  }
}
```
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>参数名称</strong>
    </td>
  <th align="left"><strong>描述</strong>
    </td>
  <tr>
    <td>allowAzureServices </td>
    <td>允许Azure其他服务访问设置</td>
  </tr>
</table>

##响应

HTTPs响应由状态码，响应 headers以及响应 body组成。
### 状态码
200 OK - 表明服务器成功返回。

### 响应 Headers

<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>请求包头</strong>
    </td>
  <th align="left"><strong>描述 URI</strong>
    </td>
  
  <tr>
    <td>x-ms-client-request-id</td>
    <td>可唯一确定数据库请求的值。请求ID用于追踪请求信息。</td>
  </tr>
</table>

### 响应 Body
Json示例文件如下：
```
{
      "id": "/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}p/providers/Microsoft.MySql/servers/{server-name}",
      "name": "{server-name}",
      "type": "Microsoft.MySql/servers",
      "location": "chinaeast",
      "sku": { "name": "MS2" },
      "properties": {
        "version": "5.5",
        "allowAzureServices": true,
        "dailyBackupTimeInHour": 10,
        "provisioningState": "Succeeded"
      }
    }
  ]
}
```
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>参数名称</strong>
    </td>
  <th align="left"><strong>描述</strong>
    </td>
  
  <tr>
    <td>name</td>
    <td>服务器名称</td>
  </tr>
  <tr>
    <td>provisioningState</td>
    <td>服务器更新状态信息</td>
  </tr>
</table>