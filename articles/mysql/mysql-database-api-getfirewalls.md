<properties linkid="" urlDisplayName="" pageTitle="使用PowerShell管理MySQL Database on Azure - Azure 微软云" metaKeywords="Azure 云,技术文档,文档与资源,MySQL,数据库,入门指南,Azure MySQL, MySQL PaaS,Azure MySQL PaaS, API, Azure MySQL Service, Azure RDS" description="本文介绍如何通过API实现更多MySQL Database on Azure的查询、创建、修改、删除等操作。" metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="sofia" solutions="" manager="" editor="" />  

<tags ms.service="mysql" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="cn" />

> [AZURE.LANGUAGE]
- [中文](/documentation/articles/mysql-database-api-getfirewalls/)
- [English](/documentation/articles/mysql-database-enus-api-getfirewalls/)

#查看服务器防火墙规则

##请求
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>方法</strong>
    </td>
  <th align="left"><strong>请求 URI</strong>
    </td>
  
  <tr>
    <td>GET   </td>
    <td>https://management.chinacloudapi.cn/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.MySql/servers/{server-name}/firewalls/{firewall-rule}?api-version=2015-09-01
	</td>
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
无

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
  "id": "/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.MySql/servers/{server-name}/firewalls/{firewall-rule}",
  "name": "{server-name}/{firewall-rule}",
  "type": "Microsoft.MySql/servers/firewalls",
  "properties": {
    "startIpAddress": "0.0.0.0",
    "endIpAddress": "255.255.255.255"
  }
}
```
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>参数</strong>
    </td>
  <th align="left"><strong>描述</strong>
    </td>
  
  <tr>
    <td>startIpAddress</td>
    <td>IP白名单起始IP</td>
  </tr>
  <tr>
    <td>endIpAddress</td>
    <td>IP白名单终止IP</td>
  </tr>
</table>