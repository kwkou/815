<properties linkid="dev-net-common-tasks-cdn" urlDisplayName="CDN" pageTitle="Azure China CDN API doc-get all endpoints under subscription" metaKeywords="Azure CDN, Azure CDN, Azure blobs, Azure caching, Azure add-on, Live Streaming, 流媒体加速, CDN加速,CDN服务,主流CDN, 流媒体直播加速, 媒体服务, Azure Media Service, 缓存规则, HLS, CDN技术文档, CDN帮助文档, 视频直播加速, 直播加速" description="Learn How to create Live Streaming acceleration type CDN on Azure Management Portal and default caching rules for Live Streaming CDN" metaCanonical="" services="" documentationCenter=".NET" title="" authors="" solutions="" manager="" editor="" />
<tags ms.service="cdn"
    ms.date="6/6/2017"
    wacn.date="6/6/2017"
    wacn.lang="cn"
    />


> [AZURE.LANGUAGE]
- [中文](/documentation/articles/cdn-api-list-endpoints/)
- [English](/documentation/articles/cdn-enus-api-list-endpoints/) 

# 节点管理-获取订阅下所有节点信息

## 请求
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>方法</strong>
    </td>
  <th align="left"><strong>请求 URI</strong>
    </td>  
  <tr>
    <td>GET</td>
    <td>https://restapi.cdn.azure.cn/subscriptions/{subscriptionId}/endpoints?apiVersion=1.0</td>
  </tr>
</table>

### URI参数
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>参数名</strong>
    </td>
  <th align="left"><strong>描述</strong>
    </td>  
  <tr>
    <td>subscriptionId</td>
    <td>订阅唯一标识</td>
  </tr
</table>

### 请求 Headers

| 请求包头 | 描述 |
|:-----------|:-----------|
| x-azurecdn-request-date | 必填。符合yyyy-MM-dd hh:mm:ss格式的UTC当前请求时间 |
| Authorization | 必填。授权头请参考[CDN API签名机制](https://www.azure.cn/documentation/articles/cdn-api-signature/) |

### 请求 Body
无

## 响应

响应由状态码，响应 headers以及响应 body组成。
### 状态码
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>状态码</strong>
    </td>
  <th align="left"><strong>描述</strong>
    </td>
  <tr>
    <td>200</td>
    <td>表明服务器成功返回</td>
  </tr>
  <tr>
    <td>其他</td>
    <td>表示出错的通用回复</td>
  </tr>
</table>

### 响应 Headers

<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>响应包头</strong>
    </th>
  <th align="left"><strong>描述</strong>
    </th>

  <tr>
    <td>X-Correlation-Id</td>
    <td>该请求唯一标识，用于追踪请求信息。</td>
  </tr>
</table>

### 响应 Body
**请求成功的JSON示例**:
```
[
  {
    "EndpointID": "779bff4d-ef38-4fce-82d8-6b50cc4c183b",
    "Setting": {
      "CustomDomain": "www.example1.com",
      "Host": "www.example1.com",
      "ICP": "ICP123456",
      "Origin": {
        "Addresses": [
          "www.origin1.com"
        ]
      },
      "ServiceType": "Web"
    },
    "Status": {
      "Enabled": "true",
      "IcpVerifyStatus": "IcpVerified",
      "LifetimeStatus": "Normal",
      "CNameConfigured": "true",
      "FreeTrialExpired": "false",
      "TimeLastUpdated": "2017-04-28T07:34:54.849Z"
    }
  },
  {
    "EndpointID": "b4869250-3373-4650-bb7a-f6e809932c0e",
    "Setting": {
      "CustomDomain": "www.example2.com",
      "Host": "www.example2.com",
      "ICP": "ICP123456",
      "Origin": {
        "Addresses": [
          "www.origin2.com"
        ]
      },
      "ServiceType": "Web"
    },
    "Status": {
      "Enabled": "true",
      "IcpVerifyStatus": "IcpVerified",
      "LifetimeStatus": "Normal",
      "CNameConfigured": "true",
      "FreeTrialExpired": "false",
      "TimeLastUpdated": "2017-04-28T07:34:54.849Z"
    }
  }
]
```
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>参数名称</strong>
    </td>
  <th align="left"><strong>描述</strong>
    </td>

  <tr>
    <td>EndpointID</td>
    <td>节点唯一标识</td>
  </tr>
  <tr>
    <td>Enabled</td>
    <td>任务状态。
        <ul>
          <li>NotSet: 状态未知</li>
          <li>Processing: 正在处理</li>
          <li>Succeeded: 成功</li>
          <li>Failed: 失败</li>
        </ul>
  </tr>
  <tr>
    <td>IcpVerifyStatus</td>
    <td>ICP备案验证信息。
        <ul>
         <li>IcpVerifying：正在验证</li>
         <li>IcpVerifyFailed：验证失败</li>
         <li>IcpVerified：验证成功</li>
        </ul>
    </td>
  </tr>
  <tr>
    <td>LifetimeStatus</td>
    <td>节点状况。
        <ul>
         <li>Normal：正常</li>
         <li>Creating：正在创建</li>
         <li>CreationFailed：创建失败</li>
         <li>Deleting：正在删除</li>
         <li>Deleted：已删除</li>
         <li>Updating：正在更新</li>
         <li>Enabling：正在激活</li>
         <li>Disabling：正在禁用</li>
    </td>
  </tr>
  <tr>
    <td>CNameConfigured</td>
    <td>加速域名CNAME记录是否已配置</td>
  </tr>
  <tr>
    <td>FreeTrialExpired</td>
    <td>是否已过试用期</td>
  </tr>
  <tr>
    <td>TimeLastUpdated</td>
    <td>最新更新时间</td>
  </tr>
</table>

**请求失败的JSON示例**:
```
{
  "Succeeded": false,
  "ErrorInfo": {
    "Type": "MissingAuthorizationHeader",
    "Message": "Missing authorization header."
  }
}
```
<table width="100%" border="1" cellspacing="0" cellpadding="0">
  <th align="left"><strong>参数名称</strong>
    </td>
  <th align="left"><strong>描述</strong>
    </td>

  <tr>
    <td>Type</td>
    <td>错误类型
         <ul>
            <li>CredentialInvalid:凭据不合法</li>
            <li>ParameterMissing:缺少参数</li>
            <li>ParameterInvalid：参数不合法</li>
            <li>MissingAuthorizationHeader:缺少Authorization请求头</li>
            <li>InvalidRequestDateHeader:请求时间不合法</li>
            <li>MissingRequestDateHeader：缺少请求时间头</li>
            <li>AuthorizationHeaderExpired:Authorization请求头已失效</li>
            <li>InvalidAuthorizationHeader:Authorization请求头不合法</li>
            <li>ApiKeyNotFound：API密钥不存在</li>
            <li>InvalidApiKey:API密钥不合法</li>
            <li>WrongSignature:签名不对</li>
            <li>SubscriptionNotFound：订阅不存在</li>
            <li>EndpointDoesNotBelongToSubscription：节点不属于订阅</li>
            <li>EndpointNotInActiveState：节点不处于活跃状态</li>
            <li>EndpointNotFound：节点不存在</li>
            <li>MaliciousItemPathDetected：检查到恶意路径</li>
            <li>PermissionDenied：权限不够</li>
            <li>RequestThrottled：请求被限流</li>
         </ul>    
    </td>
  </tr>
  <tr>
    <td>Message</td>
    <td>错误信息</td>
  </tr>
</table>
